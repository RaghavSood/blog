---
layout: post
title: Using Nix flakes and fly.io without Docker
---

Admittedly, the title is not entirely accurate - we still use Docker, but successfully get away with not having to install it locally.

I've reached the point where I refuse to run Docker on my laptop. My typical workflow involves using a Nix flake for every project, usually with [devenv.sh](https://devenv.sh/) for some sane defaults.

This works great for development - I can easily pull in the languages and services I need, and even run postgres or redis. It's a clean way to manage different versions and dependencies without deviating far away from a simplistic Nix flake.

Recently, while building [burned.money](https://burned.money), I decided to try deploying to [fly.io](https://fly.io). Fly's done some great work in minimizing idle resources and still ensuring that you can respond in a timely manner when a request comes in. However, it also requires you to provide a Docker image to run on their Fly Machines - lightweight Firecracker based VMs.

Fly provides some nice supporting infrastructure around this - you can build your Docker image using Fly Builders, which are just Fly Machines used to build your Docker images. Support for this is built right into the `flyctl` CLI tool, which means it is possible to deploy to Fly without having to build your Docker image locally, as long as you can tell them how to build it (generally, by including a `Dockerfile` in your project).

I wanted to make this work well with my existing development workflow - Namely, I wanted to avoid having to maintain a separate build pipeline for Docker and my Nix flake.

This turned out to be relatively simple - While building the Docker image, I can use the Nix flake to provide dependencies and build the application. As Nix provides excellent support for binary closures, I can then copy the resulting binary and all of its dependencies into another, minimal layer for deployment.

## The Code

You can find the sample application for this post in [fly-flake-example](https://github.com/RaghavSood/fly-flake-example).

## The Nix flake

To get started, I put together a minimal Nix flake giving me access to the `fly` cli tool, and enabling Go as a language.

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixpkgs-unstable";
    systems.url = "github:nix-systems/default";
    devenv.url = "github:cachix/devenv/1e4701fb1f51f8e6fe3b0318fc2b80aed0761914";
  };

  outputs = { self, nixpkgs, devenv, systems, ... } @ inputs:
    let
      forEachSystem = nixpkgs.lib.genAttrs (import systems);
    in
    {
      devShells = forEachSystem
        (system:
          let
            pkgs = nixpkgs.legacyPackages.${system};
          in
          {
            default = devenv.lib.mkShell {
              inherit inputs pkgs;
              modules = [
                {
                  languages.go = {
                    enable = true;
                  };

                  packages = with pkgs; [ flyctl ];

                  enterShell = ''
                    echo "fly-flake-example shell activated!"
                  '';
                }
              ];
            };
          });
    };
}
```

With this, I can run `nix develop` to drop into a development environment. Of course, this can be a flake without `devenv` too - nothing about this is contingent on `devenv`.

## The Go application

With the development environment read, we can put together a very simple Go program to run a ping-pong server.

```go
package main

import (
	"net/http"

	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"message": "pong"})
	})
	r.Run()
}

```

This will serve as the stand-in for a real application.

We're using Gin instead of `net/http` so that we have some actual dependencies in our project - this is important for the next step.

## Building the program with Nix

So far, our flake only gives us a shell - before we can build the Dockerfile, we need to be able to have Nix build the binary itself.

Flakes support `nix build` and `nix run` - we're going to add some more scaffolding to our flake to allow for `nix build` to work.

The resulting flake looks like this:

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixpkgs-unstable";
    systems.url = "github:nix-systems/default";
    devenv.url = "github:cachix/devenv/1e4701fb1f51f8e6fe3b0318fc2b80aed0761914";
  };

  outputs = { self, nixpkgs, devenv, systems, ... } @ inputs:
    let
      forEachSystem = nixpkgs.lib.genAttrs (import systems);
    in
    {
      devShells = forEachSystem
        (system:
          let
            pkgs = nixpkgs.legacyPackages.${system};
          in
          {
            default = devenv.lib.mkShell {
              inherit inputs pkgs;
              modules = [
                {
                  languages.go = {
                    enable = true;
                  };

                  packages = with pkgs; [ flyctl ];

                  enterShell = ''
                    echo "fly-flake-example shell activated!"
                  '';
                }
              ];
            };
          });
      packages = forEachSystem
        (system:
          let
            pkgs = nixpkgs.legacyPackages.${system};
          in
          {
            default = pkgs.buildGoModule {
              name = "fly-flake-example";

              src = ./.;
              vendorHash = "sha256-Ef2XLxGq8TO3WVh9EvLE30Is2CBwH4pqXxkq1tcuR0Q=";

              doCheck = false;
            };
          });
    };
}
```

Here, we've defined a `packages` attribute which instructs `nix build` to build the Go program - the `vendorHash` is defined based on the contents of our `go.mod` and `go.sum` (this is why we used Gin - if we only use `stdlib` packages with no downloaded dependencies, this should be set to `null`)

To get the `vendorHash` for the first build, or on subsequent updates if your `go.sum` changes, you can replace it with an empty string `""`, run `nix build`, and copy the correct value from the resulting failure message.

Now, we can run `nix build` to build the Go program. This produces a Nix closure and puts it in our Nix store, symlinked to the `result` symlink in the project directory. We can then run our built program with `./result/bin/fly-flake-example`, even without being inside the Go development environment!

```bash
$ nix build
$ ls -al result
lrwxr-xr-x 1 raghavsood staff 61 Jun 14 12:38 result -> /nix/store/ccmb3k660k2pkiq27jbnc92kx3a79cfx-fly-flake-example
$ ./result/bin/fly-flake-example
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /ping                     --> main.main.func1 (3 handlers)
[GIN-debug] [WARNING] You trusted all proxies, this is NOT safe. We recommend you to set a value.
Please check https://pkg.go.dev/github.com/gin-gonic/gin#readme-don-t-trust-all-proxies for details.
[GIN-debug] Environment variable PORT is undefined. Using port :8080 by default
[GIN-debug] Listening and serving HTTP on :8080
[GIN] 2024/06/14 - 12:40:07 | 200 |          85µs |             ::1 | GET      "/ping"
^C
$ which go
go not found
```

Now, we're ready to assemble the Dockerfile and set up the fly.io project.

## fly launch

We can let Fly go through its default launch flow by just running `fly launch` - for many projects, this will auto detect various settings and provide a good starting point. I didn't modify any of the defaults here, but you may have to tweak them based on your project.

```bash
$ fly launch
```

Fly will auto detect common stacks such as Go, Rails, etc. and generate an appropriate `fly.toml` - in this case, we get a fairly straightforward one.

```toml
# fly.toml app configuration file generated for fly-flake-example on 2024-06-14T12:45:50+08:00
#
# See https://fly.io/docs/reference/configuration/ for information about how to use this file.
#

app = 'fly-flake-example'
primary_region = 'sin'

[build]
  [build.args]
    GO_VERSION = '1.22.3'

[env]
  PORT = '8080'

[http_service]
  internal_port = 8080
  force_https = true
  auto_stop_machines = true
  auto_start_machines = true
  min_machines_running = 0
  processes = ['app']

[[vm]]
  memory = '1gb'
  cpu_kind = 'shared'
  cpus = 1
```

This also generates a `Dockerfile`, based on our project's stack and the `.gitignore` contents. The auto-generated Dockerfile can be sufficient to build the project, and would look something like this:

```Dockerfile
ARG GO_VERSION=1
FROM golang:${GO_VERSION}-bookworm as builder

WORKDIR /usr/src/app
COPY go.mod go.sum ./
RUN go mod download && go mod verify
COPY . .
RUN go build -v -o /run-app .


FROM debian:bookworm

COPY --from=builder /run-app /usr/local/bin/
CMD ["run-app"]
```

## Using Nix within the Dockerfile

To keep our `nix build` and `Dockerfile` in sync, we can use the Nix flake to build the binary and then copy it into the Docker image.

We can swap out the `bookworm` base image with the `nixos/nix` image, and then use `nix build` to build the binary, then copy the entire closure over into a smaller `scratch` image.

```Dockerfile
# Use the NixOS base image
FROM nixos/nix as builder

# Set up the Nix environment
COPY . /src
WORKDIR /src

RUN nix \
    --extra-experimental-features "nix-command flakes" \
    --option filter-syscalls false \
    build

RUN mkdir /tmp/nix-store-closure
RUN cp -R $(nix-store -qR result/) /tmp/nix-store-closure

# Copy the built application to the runtime container
FROM scratch

WORKDIR /app

# Copy /nix/store
COPY --from=builder /tmp/nix-store-closure /nix/store
COPY --from=builder /src/result /app
CMD ["/app/bin/fly-flake-example"]
```

The magic sauce here is the `nix-store -qR result/` command, which gives us the closure of the `result` symlink. We then copy this closure into a temporary directory, and then copy it into the `scratch` image. This gives us all of the dependencies required to run the binary, without having to install them in the image itself.

Of course, the `scratch` image may be a little too minimal. If you intend to use features like `fly console ssh`, you probably want to use a slightly larger base image like `alpine` or `debian`.

## Deploying

With everything in place, a deployment is as simple as running:

```bash
$ fly deploy --remote-only
```

This instructs fly.io to build the image using a remote builder (which is nice, since I don't have Docker installed locally), and then subsequently deploy it.

We can then make any changes to our Nix flake for additional dependencies as the project progresses (such as adding in `sqlite`, or `tailwindcss`, or anything else we may need), and not have to mess around with the `Dockerfile` - as long as `nix build` produces the binary, the Docker image will be built correctly.

## Using Nix to build Docker images

It is, of course, possible to use Nix to build a Docker image directly, as opposed to running `nix build` during the Docker build. However, given that I don't run Docker locally, this is a little less useful for me. 

If this seems like a better fit for you, you may be interested in reading up on [dockerTools.buildImage](https://nix.dev/tutorials/nixos/building-and-running-docker-images.html){:target="_blank"}. Note that this only builds the image, and doing anything useful with it, such as pushing it to a registry, will require you to run Docker locally and interact with it like a regular image.

### Notes/References

A special thanks to [Tricia Tan](https://blog.trishtzy.com/){:target="_blank"} for reviewing this post and providing feedback.
