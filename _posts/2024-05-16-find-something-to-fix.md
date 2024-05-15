---
layout: post
title: Find Something to Fix
---

When I'm idling, I like to just... look at things. I might scroll HN, or trace random transactions on a blockchain explorer, or poke about in trending git repositories surfaced by [Github's Explore](https://github.com/explore).

I'm not looking for anything in particular. I'm hardly going to come back to anything I see today. But over time, just simply looking at things has helped me form a decent feel of what's out there.

This is handy in odd, unpredictable ways - Simply being aware of what else is out there is a problem-solving super power.

Sometimes, this extends to fixing something. Typically, this takes the form of trying to make [nixpkgs](https://github.com/NixOS/nixpkgs) just ever so slightly greener. For its various faults, nixpkgs does have good toolling. PRs are well labelled, there's a small army of people contributing, and OfBorg[^1] and Hydra[^2] do an amazing job of CI/CD.

I do maintain some packages for nixpkgs, and generally help out with things when I can, but fixing a random package is a window into another world. Where the window leads is decided by whichever failing Hydra job[^3] or GitHub issue I chance upon, and it can be a journey.

Earlier this week, this led me to the [bicpl](https://github.com/BIC-MNI/bicpl) package - it was failing to build for darwin platforms[^4], and was a dependency for 4 other packages, causing them to fail as well.

Stepping through this window revealed that bicpl is a library built by the [McConnel Brain Imaging Centre](https://www.mcgill.ca/bic/) at the Montreal Neurological Institute. It's a C library used to process brain imaging data, and is used by a number of other programs. Neuroimaging wasn't a field I expected to be looking at today, but nevertheless that's where I found myself.

Looking at their [code](https://github.com/BIC-MNI/bicpl), it appeared that while the master branch gave the impression the project was abandoned, occasional fixed did crop up on other branches. In particular, the [develop-apple](https://github.com/BIC-MNI/bicpl/tree/develop-apple) branch did contain a fix for the darwin build issues, so bringing that into nixpkgs was simply a matter of bumping the version we were building.

Another issue surfaced after the version bump - the builder insisted that time-related functions weren't properly declared:

{% highlight shell %}
/tmp/nix-build-bicpl-unstable-2023-01-19.drv-0/source/Prog_utils/random.c:80:16: error: call to undeclared function 'time'; ISO C99 and later do not support implicit function declarations [-Wimplicit-function-declaration]
        seed = time(NULL);
{% endhighlight %}

Now, I haven't touched C in a long, long time, but fortunately this does give us enough information to go on. A quick search for `-Wimplicit-function-declaration` later, and we find that [clang16 has](https://releases.llvm.org/16.0.0/tools/clang/docs/ReleaseNotes.html#potentially-breaking-changes) made this warning into an error. Nixpkgs, in an effort to keep things clean, safe, and stable, started defaulting to clang16 for darwin on [2023-06-11](https://github.com/NixOS/nixpkgs/commit/bcbdb800cf7659d6ff36ac114121a056fe8c9656)[^5], which is also when the bicpl package started to fail.

{% highlight diff %}
diff --git a/Prog_utils/random.c b/Prog_utils/random.c
index f61dbb6..f5dcb9c 100644
--- a/Prog_utils/random.c
+++ b/Prog_utils/random.c
@@ -23,6 +23,7 @@
 #endif

 #include <stdlib.h>
+#include <time.h>

 static  VIO_BOOL  initialized = FALSE;
{% endhighlight %}

Armed with this, the fix was simple - we just needed to include the `time.h` header file. Now, nixpkgs does have convenient ways to patch source files - I could add the patch to the repository and include it, but I elected to send [a PR](https://github.com/BIC-MNI/bicpl/pull/9) to the upstream repository instead. This way, the fix would be available to everyone, and not just nixpkgs users. It's an issue that woudl inevitably bite any users outside of nixpkgs too, as clang16 grows in use. At the same time, I opened [a nixpkgs PR](https://github.com/NixOS/nixpkgs/pull/311264), pulling in the patch during the build for now. At the very least, this would fix any issues within nixpkgs and anyone downstream of it.

{% highlight diff %}
diff --git a/pkgs/development/libraries/science/biology/bicpl/default.nix b/pkgs/development/libraries/science/biology/bicpl/default.nix
index 5ad45b1f8c0431..45d1b159d07471 100644
--- a/pkgs/development/libraries/science/biology/bicpl/default.nix
+++ b/pkgs/development/libraries/science/biology/bicpl/default.nix
@@ -2,6 +2,7 @@
   lib,
   stdenv,
   fetchFromGitHub,
+  fetchpatch,
   cmake,
   libminc,
   netpbm,
@@ -9,16 +10,24 @@

 stdenv.mkDerivation rec {
   pname = "bicpl";
-  version = "unstable-2020-10-15";
+  version = "unstable-2023-01-19";

-  # current master is significantly ahead of most recent release, so use Git version:
+  # master is not actively maintained, using develop and develop-apple branches
   src = fetchFromGitHub {
     owner = "BIC-MNI";
     repo = pname;
-    rev = "a58af912a71a4c62014975b89ef37a8e72de3c9d";
-    sha256 = "0iw0pmr8xrifbx5l8a0xidfqbm1v8hwzqrw0lcmimxlzdihyri0g";
+    rev = "884b3ac8db945a17df51a325d29f49b825a61c3e";
+    hash = "sha256-zAA+hPwjMawQ1rJuv8W30EqKO+AI0aq9ybquBnKlzC0=";
   };

+  patches = [
+    # fixes build by including missing time.h header
+    (fetchpatch {
+      url = "https://github.com/RaghavSood/bicpl/commit/3def4acd6bae61ff7a930ef8422ad920690382a6.patch";
+      hash = "sha256-VdAKuLWTZY7JriK1rexIiuj8y5ToaSEJ5Y+BbnfdYnI=";
+    })
+  ];
+
   nativeBuildInputs = [ cmake ];
   buildInputs = [
     libminc
{% endhighlight %}

With that done and merged, [bicpl and its dependents](https://hydra.nixos.org/eval/1806287#tabs-now-succeed) were building successfully again.

As a pleasant surprise, despite bicpl's repository being inactive, the upstream PR was merged within <24 hours, allowing me to [remove the patch](https://github.com/NixOS/nixpkgs/pull/311553) from the nixpkgs build.

## But why?

By almost any definition, this was a pointless exercise - I don't use bicpl, or any of its derivates. Heck, I don't even use C enough to have to care about breaking changes in clang16. But random-walk discoveries like this are how I've learned about countless things over the years.

Now I know bicpl exists - I know there's a small sliver of open source centered around dealing with brain imaging data and how to manipulate it. A sliver that's been at it since [at least 2001](https://github.com/BIC-MNI/mni-acmacros/commit/9d54abc95839c16fe8d3b63927e44f63a4dfcd76)[^6]. I don't know if anyone's going to use this package from nixpkgs, probably not. But having it functional means that if someone does, it's one less barrier for them to cross.

Just based on the brief look through the window of open-source brain imaging, I can hazard a guess that their data files contain some kind of volumetric data, expressed as geomtric shapes and coordinates. Maybe there's some type or tag associated with these values to indicate density or mass. Maybe it's computed on the fly. I know that they're active enough to notice uncalled for PRs on inactive repositories, and that they understand enough about software engineering to evaluate it quickly[^7].

That's good enough for me.

And who knows, maybe one day I will need to know that clang16 broke implicit function declarations. I've used more arbitrary knowledge in the past[^8]. Might even scan a brain someday.

---
# Notes/References

[^1]: [OfBorg](https://github.com/NixOS/ofborg) is nixpkgs' bot to help automatically build, label, and check PRs.
[^2]: [Hydra](https://nixos.org/hydra/) is NixOS' CI/CD system. Like OfBorg, it automatically builds and reports the state of the Nix ecosystem.
[^3]: Hydra manages the release channels, and builds the entire nixpkgs package set. It helpfully reports [failing](https://hydra.nixos.org/eval/1806308#tabs-still-fail) and [newly failing](https://hydra.nixos.org/eval/1806308#tabs-now-fail) jobs for each evaluation.
[^4]: OfBorg and Hydra build packages for x86 and ARM on linux and darwin.
[^5]: If you look at the associated [pull request](https://github.com/NixOS/nixpkgs/pull/241692), you'll notice it's actually from July, and was merged in October. However, it was merged to `staging` first, since changing the default compiler for darwin is a big deal. Due to the way nixpkgs is structured, this changes `stdenv`, and essentially triggers a "rebuild-the-world" event, requiring nearly every darwin package to be rebuilt. By merging to staging first, Hydra can build and cache the new builds. Once this is done, merging to master results in a signficantly small rebuild set, ensuring that nixpkgs' channels stay relatively up-to-date and don't wait days for a rebuild.
[^6]: This is just their earliest git commit I could find - if the repositories are right, at some point they were using CVS, and probably hosted code elsewhere. There's a very real chance BIC-MNI's open source efforts predate me. 
[^7]: Having seen many researchers' code, this is not a given.
[^8]: A story for another post.
