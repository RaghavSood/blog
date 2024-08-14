---
title: Contributing to nixpkgs - Part 1 - Basic Contributions
tags: nixpkgs
---

For many, contributing to [nixpkgs](https://github.com/NixOS/nixpkgs) can be a daunting task - the repository is extremely active, with thousands of PRs going in every week. With the breadth of what's included in nixpkgs spanning everything from bootstrapping an OS install to npm packages, it can be hard to know where to start.

This is the first in a series of posts on how to effectively contribute to nixpkgs. It helps you avoid common pitfalls and approach things in a manner that are easier to review and less likely to break things.

## How is nixpkgs structured?

Nixpkgs is a monorepo, containing all the packages and infrastructure needed to build the NixOS and nixpkgs ecosystem. It's structured in a way that makes it easy to add new packages and update existing ones.

The root of the repo contains only a handful of directories:

{% highlight shell %}
$ tree -d -L 1
.
├── doc
├── lib
├── maintainers
├── nixos
├── pkgs
{% endhighlight %}

As a first-time contributor, you don't need to go too deep. Just know that:

 - `doc` contains the documenation used to generate the [nixpkgs manual](https://nixos.org/manual/nixpkgs/stable/). As someone updating and adding packages, you generally don't need to change anything here.
 - `lib` contains helpful functions and utilities used to compose packages and other information about them. Once again, not a common entrypoint for first-time contributors.
 - `maintainers` contains a list of known-maintainers who look after certain package or areas of the repository, as well as helpful scripts for common maintenance tasks such as updates and listings. If you wish to add a new package, or adopt an existing one, you would need to create an entry for yourself in `maintainers/maintainer-list.nix`.
 - `nixos` contains the NixOS module system, which is used to configure NixOS systems. Some packages have associated NixOS modules that may require updates when the package is updated.
 - `pkgs` is where the magic happens. This directory contains all the packages in nixpkgs, organized by category. This is where you'll be spending most of your time.

## Finding something to contribute

There are a few ways to make a meaningful contribution. Most folks start out by updating a package they use, or fixing a bug they're running into.

If you already have a contribution in mind, you can skip this section. If you're looking for something to work on, here are a few ways to find something:

- **Look for issues**: The [nixpkgs issue tracker](https://github.com/NixOS/nixpkgs/issues) has frequent requests for new package additions or updates. Scrolling through this list can provide some opportunities.[[If you pick an open issue, make sure to mention `Closes #123` in your PR description to automatically close the issue when the PR is merged.::lsn]]
- **Check failing builds**: Visit Hydra's latest build for [nixpkgs#trunk](https://hydra.nixos.org/jobset/nixpkgs/trunk/evals) and take a look at the `Newly Failing Jobs` and `Still Failing Jobs` to see if anything looks interesting.
- **Contribute without code**: There are countless opportunities for things such as documenation improvements, PR review, and other general tasks. We won't touch upon these in this post, but you can filter pull requests by tags such as [has: documenation](https://github.com/NixOS/nixpkgs/pulls?q=is%3Aopen+is%3Apr+label%3A%228.has%3A+documentation%22) to see how others are making such contributions.

Once you've identified something to work on, it's time to get started. For this post, we'll be updating the `flyctl` package to the latest version.

## Getting set up

_We're going to assume you already have [Nix installed](https://nix.dev/install-nix.html), or are using NixOS._

The first step is to fork the nixpkgs repository. This is done by visiting the [nixpkgs repository](https://github.com/NixOS/nixpkgs) and clicking the `Fork` button in the top right corner.

Forking can take a moment as it is a large repository.

Once you have your fork, you can clone it locally. We're also going to want to add the main nixpkgs repository as a remote, so we can pull in changes from upstream.

{% highlight shell %}
$ git clone git@github.com/<your-username>/nixpkgs.git          # This can be slow since it's a multi-gigabyte repository
$ cd nixpkgs
$ git remote add upstream git@github.com:NixOS/nixpkgs.git
$ git remote -v
$ git pull upstream master                                      # Pull in the latest changes from upstream - do this each time before you start something new
{% endhighlight %}

To ensure everything is in order, let's run a quick build command:

{% highlight shell %}
$ nix-build -A flyctl
{% endhighlight %}

This may not actually build anything for you - nixpkgs is heavily cached, and since we haven't updated the version yet this will likely just pull the existing binary from the cache.

## Basic contribution workflow

When contributing to a large, living repository like nixpkgs, there's a few things to watch out for:

1. Follow the commit and style standards - most repositories have these defined in the README or CONTRIBUTING file, or in a wiki for the project.
2. Every commit is one unit of work - It should be easy to revert a single change without undoing an entire PR worth of work, especially if a PR touches multiple things.
3. There may be tooling you can use - nixpkgs, and the broader Nix ecosystem, does have a variety of bots and other tooling that can be used to simplify or cross-check your work.
4. When in doubt, ask - it's always easier to show your approach and ask for feedback, than to post an open ended "How do I X?" question. Members will be able to give more focused and helpful answers, as well as specific feedback on why your approach wasn't the preferred option.
5. Read [CONTRIBUTING.md](https://github.com/NixOS/nixpkgs/blob/master/CONTRIBUTING.md), [pkgs/README.md](https://github.com/NixOS/nixpkgs/blob/master/pkgs/README.md), and the respective section of the manual for whichever language/stack you're updating packages in (for example, for this post, that's the [Go section](https://nixos.org/manual/nixpkgs/stable/#ssec-language-go)).

## [Optional] Becoming a maintainer

If you plan on becoming a listed maintainer for any package, it's best to do this step first - maintainer tracking in nixpkgs is done by evaluating a Nix expression, so you must be declared in the maintainer list before being added to a package or team.[[Listed maintainers are automatically tagged for Code Review by much of the automated tooling, such as GitHub and OfBorg.::rmn]]

To do this, you need to add an entry to `maintainers/maintainer-list.nix`. This file is a list of maintainers, with each maintainer having a name, email, and a contain method.

{% highlight nix %}
handle = {
  name = "Your Name";                       # Required
  github = "your-github-username";          # Required
  email = "hello@example.com";              # At least one
  githubId = "12345678";                    # At least one
  matrixId = "@your-matrix-id:matrix.org";  # At least one
  keys = [{                                 # Optional
    fingerprint = "1234 5678 9ABC DEF0 1234 5678 9ABC DEF0 1234 5678";
  }];
};
{% endhighlight %}

You can fetch your `githubId` by visiting [https://api.github.com/users/\<your-github-username\>](https://api.github.com/users/<your-github-username>). This will return a JSON object with your user details, including your `id`.

`handle` does not have to match your GitHub username, but conventionally it does.

The entries in the file are sorted alphabetically by the `handle` field, so make sure to insert your entry in the correct place.

Once you've added your entry, it's time to commit!

{% highlight shell %}
$ git checkout -b maintainers/add-<handle>
$ git add maintainers/maintainer-list.nix
$ git commit -m "maintainers: add <handle>"
$ git push origin HEAD
{% endhighlight %}

Now you can open a PR to add yourself as a maintainer. It's also possible to do this as one commit that is part of a larger PR. However, if you take that route, ensure:

1. The commit adding yourself as a maintainer is the first commit in the PR.
2. The focus of the PR is on the package being updated or added.

A good PR which includes maintainer updates will look something like:

```
maintainers: add RaghavSood
flyctl: 0.0.1 -> 0.0.2
flyctl: add RaghavSood as maintainer
```

By splitting each change into its own, clearly delineated commit, it is easier for reviewers to evaluate. Additionally, in the event that the package update needs to be reverted, it can be done without affecting the maintainer changes.

## Updating the package

Within nixpkgs many wrappers and build systems exist to make it easier to package different kinds of applications. For basic packages, the `stdenv.mkDerivation` function is used. Go has `buildGoModule`, Python has `buildPythonPackage`, and so on.

A trivial update to a package will generally follow the same recipe, regardless of the language or build system used:

1. Update the version or revision of the package to point to the desired version.
2. Update any hashes (within `src`, or for build requirements like go modules or cargo crates) to match their new state.
3. If required, update any dependencies or inputs.
4. Fix any tests, build issues, or other problems that arise from the update.
5. If required, do some housekeeping by removing any inputs, patches, or other now-useless parts of the derivation.[[A nix derivation is a Nix program that describes how to achieve a specific task, such as building a package.::rmn]]

In this post, we're simply going to make a trivial update that requires no fixes.

As of writing, the `flyctl` package in nixpkgs is at version 0.2.52, while the latest upstream version is 0.2.55. It's a Go package, so we need to:

1. Update the version
2. Update the `src` hash
3. Update the go module hash

Updating the version is easy enough - we simply change the `version` field in the derivation:

{% highlight diff %}
diff --git a/pkgs/development/web/flyctl/default.nix b/pkgs/development/web/flyctl/default.nix
index 8ca6eca5268b..24f523242cea 100644
--- a/pkgs/development/web/flyctl/default.nix
+++ b/pkgs/development/web/flyctl/default.nix
@@ -2,7 +2,7 @@

 buildGo122Module rec {
   pname = "flyctl";
-  version = "0.2.52";
+  version = "0.2.55";

   src = fetchFromGitHub {
     owner = "superfly";
{% endhighlight %}

To get the updated `src` hash, we have a few options. For most derivations, including `flyctl`, the source is downloaded from an archive hosted on GitHub or Gitlab or other service. These have a predictable, defined URL, which can be used to fetch the source and calculate the hash using the `nix-prefetch-url` command. Since the derivation uses the unpacked code, we must ask `nip-prefetch-url` to unpack the source:

{% highlight shell %}
$ nix-prefetch-url --unpack https://github.com/superfly/flyctl/archive/v0.2.55.tar.gz
117zl6pg9flr2ffhfjfy8a8svdc2gwd4s81vdgkb0lilmlbml968
{% endhighlight %}

This gives us the hash in the old, base32 format. However, current standards[^4] require that hashes in nixpkgs follow the SRI format. We can convert the hash using the `nix-hash` command:[[`nixpkgs` is an incredibly large, complex intersection of thousands of contributors. Standards and styles change often, and it is common to find code in the repo that is written in an old manner. To ensure you're following the latest standards, it's best to refer to the latest PRs and commits in the repository that touch similar areas and read the description and reviews.::rmn]]

{% highlight shell %}
$ nix-hash --to-sri --type sha256 117zl6pg9flr2ffhfjfy8a8svdc2gwd4s81vdgkb0lilmlbml968
sha256-yCRaF600UrDmazsgTRp/grWtkULeSQedE5m69K6h/4Q=
{% endhighlight %}

While this approach works, some people find it cumbersome - moreover, it doesn't work well if you are cloning the repository over git, or have another source type that isn't just a simple file. 

A more common, albeit slightly "hacky" approach is to intentionally put the wrong hash in the derivation, and let `nix-build` throw an error and fail. This will give you the correct hash to use. To do this, we can replace the `src.hash` attribute with an empty string `""` or `lib.fakeHash`:

{% highlight diff %}
diff --git a/pkgs/development/web/flyctl/default.nix b/pkgs/development/web/flyctl/default.nix
index 8ca6eca5268b..244802befee4 100644
--- a/pkgs/development/web/flyctl/default.nix
+++ b/pkgs/development/web/flyctl/default.nix
@@ -2,13 +2,13 @@
 
 buildGo122Module rec {
   pname = "flyctl";
-  version = "0.2.52";
+  version = "0.2.55";
 
   src = fetchFromGitHub {
     owner = "superfly";
     repo = "flyctl";
     rev = "v${version}";
-    hash = "sha256-BCnMXyS94tuD+Un1DLqs3mdGi7XrVBoZGJ/XkpACOQI";
+    hash = lib.fakeHash;
   };
{% endhighlight %}

Now, running `nix-build -A flyctl` will produce an error and provide us with the correct hash:

{% highlight shell %}
$ nix-build -A flyctl
these 3 derivations will be built:
  /nix/store/31l3mhp8rvgphz5gn1q62x7j03ryrcpd-source.drv
  /nix/store/3ygzq80kxg7nk9fkk3lp7mq06dmg9ax7-flyctl-0.2.55-go-modules.drv
  /nix/store/x0i067rpc1h5mvhrb1qadwh1vrx3mnal-flyctl-0.2.55.drv
building '/nix/store/31l3mhp8rvgphz5gn1q62x7j03ryrcpd-source.drv'...

trying https://github.com/superfly/flyctl/archive/v0.2.55.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 1662k    0 1662k    0     0   547k      0 --:--:--  0:00:03 --:--:--  993k
unpacking source archive /private/tmp/nix-build-source.drv-0/v0.2.55.tar.gz
error: hash mismatch in fixed-output derivation '/nix/store/31l3mhp8rvgphz5gn1q62x7j03ryrcpd-source.drv':
         specified: sha256-AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=
            got:    sha256-yCRaF600UrDmazsgTRp/grWtkULeSQedE5m69K6h/4Q=
{% endhighlight %}

Go modules are fixed against the `vendorHash` attribute in `buildGoModule` derivations. Unfortunately, there is no equivalent to `nix-prefetch-url` for go modules, so the easiest way to get the new hash is to set it to `lib.fakeHash` and try a build.

{% highlight diff %}
diff --git a/pkgs/development/web/flyctl/default.nix b/pkgs/development/web/flyctl/default.nix
index 8ca6eca5268b..03d1b79f1810 100644
--- a/pkgs/development/web/flyctl/default.nix
+++ b/pkgs/development/web/flyctl/default.nix
@@ -2,16 +2,16 @@

 buildGo122Module rec {
   pname = "flyctl";
-  version = "0.2.52";
+  version = "0.2.55";

   src = fetchFromGitHub {
     owner = "superfly";
     repo = "flyctl";
     rev = "v${version}";
-    hash = "sha256-BCnMXyS94tuD+Un1DLqs3mdGi7XrVBoZGJ/XkpACOQI";
+    hash = "sha256-yCRaF600UrDmazsgTRp/grWtkULeSQedE5m69K6h/4Q=";
   };

-  vendorHash = "sha256-eTiY65VGFBgGzCOrnp/WbOo9Lbdk4PYwT7CppjsZ4WE=";
+  vendorHash = lib.fakeHash;

   subPackages = [ "." ];
{% endhighlight %}

Once again, `nix-build -A flyctl` helpfully tells us the expected hash:

{% highlight shell %}
$ nix-build -A flyctl
# some output omitted
error: hash mismatch in fixed-output derivation '/nix/store/z6xdjn6aikaaw3z364jrixr5z9jx07zl-flyctl-0.2.55-go-modules.drv':
         specified: sha256-AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=
            got:    sha256-1hlWyr41t8J4naN5QbEtfCv3npe/kvMH5NKKaxYvLYk=
{% endhighlight %}

And that's it! We now have everything we need for a updated, successful build:

{% highlight diff %}
diff --git a/pkgs/development/web/flyctl/default.nix b/pkgs/development/web/flyctl/default.nix
index 8ca6eca5268b..b0bf00051840 100644
--- a/pkgs/development/web/flyctl/default.nix
+++ b/pkgs/development/web/flyctl/default.nix
@@ -2,16 +2,16 @@

 buildGo122Module rec {
   pname = "flyctl";
-  version = "0.2.52";
+  version = "0.2.55";

   src = fetchFromGitHub {
     owner = "superfly";
     repo = "flyctl";
     rev = "v${version}";
-    hash = "sha256-BCnMXyS94tuD+Un1DLqs3mdGi7XrVBoZGJ/XkpACOQI";
+    hash = "sha256-yCRaF600UrDmazsgTRp/grWtkULeSQedE5m69K6h/4Q=";
   };

-  vendorHash = "sha256-eTiY65VGFBgGzCOrnp/WbOo9Lbdk4PYwT7CppjsZ4WE=";
+  vendorHash = "sha256-1hlWyr41t8J4naN5QbEtfCv3npe/kvMH5NKKaxYvLYk=";

   subPackages = [ "." ];
{% endhighlight %}

... or do we?

Although this was supposed to be a trivial update, it turns out we have a failing test

{% highlight shell %}
Running phase: checkPhase
?       github.com/superfly/flyctl      [no test files]
?       github.com/superfly/flyctl      [no test files]
?       github.com/superfly/flyctl/agent        [no test files]
?       github.com/superfly/flyctl/agent/internal/proto [no test files]
?       github.com/superfly/flyctl/agent/server [no test files]
?       github.com/superfly/flyctl/cmd/audit    [no test files]
?       github.com/superfly/flyctl/doc  [no test files]
?       github.com/superfly/flyctl/flyctl       [no test files]
?       github.com/superfly/flyctl/flypg        [no test files]
?       github.com/superfly/flyctl/gql  [no test files]
ok      github.com/superfly/flyctl/helpers      0.393s
--- FAIL: TestToTestMachineConfig (0.00s)
panic: runtime error: invalid memory address or nil pointer dereference [recovered]
        panic: runtime error: invalid memory address or nil pointer dereference
[signal SIGSEGV: segmentation violation code=0x2 addr=0xa8 pc=0x1034f24cc]

goroutine 38 [running]:
testing.tRunner.func1.2({0x1038acac0, 0x1041d6000})
        /nix/store/460vdyz0ghxh8n5ibq3fgc3s63is68cd-go-1.22.2/share/go/src/testing/testing.go:1631 +0x1c4
testing.tRunner.func1()
        /nix/store/460vdyz0ghxh8n5ibq3fgc3s63is68cd-go-1.22.2/share/go/src/testing/testing.go:1634 +0x33c
panic({0x1038acac0?, 0x1041d6000?})
        /nix/store/460vdyz0ghxh8n5ibq3fgc3s63is68cd-go-1.22.2/share/go/src/runtime/panic.go:770 +0x124
github.com/superfly/flyctl/internal/appconfig.(*Config).ToTestMachineConfig(0x14000421980, 0x140003971d0, 0x0)
        /private/tmp/nix-build-flyctl-0.2.55.drv-1/source/internal/appconfig/machines.go:153 +0x59c
github.com/superfly/flyctl/internal/appconfig.TestToTestMachineConfig(0x14000403040)
        /private/tmp/nix-build-flyctl-0.2.55.drv-1/source/internal/appconfig/machines_test.go:210 +0x51c
testing.tRunner(0x14000403040, 0x1039e59a0)
        /nix/store/460vdyz0ghxh8n5ibq3fgc3s63is68cd-go-1.22.2/share/go/src/testing/testing.go:1689 +0xec
created by testing.(*T).Run in goroutine 1
        /nix/store/460vdyz0ghxh8n5ibq3fgc3s63is68cd-go-1.22.2/share/go/src/testing/testing.go:1742 +0x318
FAIL    github.com/superfly/flyctl/internal/appconfig   0.355s
{% endhighlight %}

It turns out that [one of](https://github.com/superfly/flyctl/pull/3430) the changes between 0.2.52 and 0.2.55 added some tests that require network access. Since we're building in a sandboxed environment.[[Derivations in `nixpkgs` are built in a sandbox with no network or filesystem access. Anything required by your program, such as dependencies (NPM packages, go modules, config files, etc.) must be explicitly provided. `buildGoModules` already provides a wrapper to download go moduels from the internet and pin them using `vendorHash`. However, if tests or other parts of the build process require network access, they will have to be skipped or patched to avoid it.::rmn]]

We can fix this by disabling the tests. `buildGoModule` helpfully provides [a few options](https://github.com/NixOS/nixpkgs/blob/d616253e70a3476874d86974e9ef1d20629f6ab0/doc/languages-frameworks/go.section.md?plain=1#L255C21-L255C40) to remove tests from the checkPhase.

Unfortunately, the test setup for go packages doesn't quite support the common `go test ./...` pattern, so we also need to override the `checkPhase` step of the build process.[[A build consists of [multiple phases](https://nixos.org/manual/nixpkgs/stable/#sec-stdenv-phases).::lsn]]

By disabling all of the `TestToTestMachineConfig` tests, we can get a successful build.

{% highlight diff %}
diff --git a/pkgs/development/web/flyctl/default.nix b/pkgs/development/web/flyctl/default.nix
index 8ca6eca5268b..2683203a79ca 100644
--- a/pkgs/development/web/flyctl/default.nix
+++ b/pkgs/development/web/flyctl/default.nix
@@ -2,16 +2,16 @@

 buildGo122Module rec {
   pname = "flyctl";
-  version = "0.2.52";
+  version = "0.2.55";

   src = fetchFromGitHub {
     owner = "superfly";
     repo = "flyctl";
     rev = "v${version}";
-    hash = "sha256-BCnMXyS94tuD+Un1DLqs3mdGi7XrVBoZGJ/XkpACOQI";
+    hash = "sha256-yCRaF600UrDmazsgTRp/grWtkULeSQedE5m69K6h/4Q=";
   };

-  vendorHash = "sha256-eTiY65VGFBgGzCOrnp/WbOo9Lbdk4PYwT7CppjsZ4WE=";
+  vendorHash = "sha256-1hlWyr41t8J4naN5QbEtfCv3npe/kvMH5NKKaxYvLYk=";

   subPackages = [ "." ];

@@ -34,8 +34,20 @@ buildGo122Module rec {
     HOME=$(mktemp -d)
   '';

-  postCheck = ''
-    go test ./... -ldflags="-X 'github.com/superfly/flyctl/internal/buildinfo.buildDate=1970-01-01T00:00:00Z'"
+  checkFlags = [
+    # these tests require network
+    "-skip=TestToTestMachineConfig"
+  ];
+
+  # We override checkPhase to be able to test ./... while using subPackages
+  checkPhase = ''
+    runHook preCheck
+    # We do not set trimpath for tests, in case they reference test assets
+    export GOFLAGS=''${GOFLAGS//-trimpath/}
+
+    buildGoDir test ./...
+
+    runHook postCheck
   '';

   postInstall = ''
{% endhighlight %}

To see the full breakdown of why these changes are necessary, you can refer to the [PR description](https://github.com/NixOS/nixpkgs/pull/313157).

With the update read, it's a simple commit:

{% highlight shell %}
$ git checkout -b flyctl/0.2.55
$ git add pkgs/development/web/flyctl/default.nix
$ git commit -m "flyctl: 0.2.52 -> 0.2.55"
$ git push origin HEAD
{% endhighlight %}

## Housekeeping

`flyctl` is a package I've found myself using more and more, so I'm going to adopt it as one of the maintainers - since I already exist in `maintainers/maintainer-list.nix`, it's a simple change:

{% highlight diff %}
diff --git a/pkgs/development/web/flyctl/default.nix b/pkgs/development/web/flyctl/default.nix
index 2683203a79ca..6ab6ad9057d2 100644
--- a/pkgs/development/web/flyctl/default.nix
+++ b/pkgs/development/web/flyctl/default.nix
@@ -69,7 +69,7 @@ buildGo122Module rec {
     downloadPage = "https://github.com/superfly/flyctl";
     homepage = "https://fly.io/";
     license = lib.licenses.asl20;
-    maintainers = with lib.maintainers; [ adtya jsierles techknowlogick ];
+    maintainers = with lib.maintainers; [ adtya jsierles techknowlogick RaghavSood ];
     mainProgram = "flyctl";
   };
 }
{% endhighlight %}

We can then commit that, ensuring we do it separately from the version update itself:

{% highlight shell %}
$ git add pkgs/development/web/flyctl/default.nix
$ git commit -m "flyctl: add RaghavSood as maintainer"
{% endhighlight %}

Lastly, I'm going to define an `updateScript`. A significant portion of `nixpkgs` updates are automated and various tooling exists to support this. However, it doesn't work on `flyctl` as the upstream repository has non-standard release tags, and no `updateScript` is defined to inform the tooling on how to process these tags. We can make use of existing tooling options to define such a script:

{% highlight diff %}
diff --git a/pkgs/development/web/flyctl/default.nix b/pkgs/development/web/flyctl/default.nix
index 6ab6ad9057d2..12483d4ef5b3 100644
--- a/pkgs/development/web/flyctl/default.nix
+++ b/pkgs/development/web/flyctl/default.nix
@@ -1,4 +1,4 @@
-{ lib, buildGo122Module, fetchFromGitHub, testers, flyctl, installShellFiles }:
+{ lib, buildGo122Module, fetchFromGitHub, testers, flyctl, installShellFiles, gitUpdater }:

 buildGo122Module rec {
   pname = "flyctl";
@@ -58,6 +58,14 @@ buildGo122Module rec {
     ln -s $out/bin/flyctl $out/bin/fly
   '';

+  # Upstream tags every PR merged with release tags like
+  # v2024.5.20-pr3545.4. We ignore all revisions containing a '-'
+  # to skip these releases.
+  passthru.updateScript = gitUpdater {
+    rev-prefix = "v";
+    ignoredVersions = "-";
+  };
+
   passthru.tests.version = testers.testVersion {
     package = flyctl;
     command = "HOME=$(mktemp -d) flyctl version";
{% endhighlight %}

Again, we want to commit this as a separate unit of work from our version update and maintainer list change:

{% highlight shell %}
$ git add pkgs/development/web/flyctl/default.nix
$ git commit -m "flyctl: add updateScript"
{% endhighlight %}

And with that, we're done!

{% highlight shell %}
$ git log --oneline
54a85a69a597 flyctl: add updateScript
df4c4e9183ae flyctl: add RaghavSood as maintainer
b57260da32c6 flyctl: 0.2.52 -> 0.2.55
{% endhighlight %}

