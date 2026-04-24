# nixos-flake-overlay-pattern

When to override a package via overlay versus via flake input — the two are not
interchangeable, and mixing them silently wins one or the other.

## Problem

You need `ripgrep` pinned to the latest release, not the channel's lagging
version. `inputs.ripgrep.url = "github:BurntSushi/ripgrep";` doesn't magically
replace `pkgs.ripgrep` — the overlay is what wires the input into the package
set. Skip the overlay and every downstream module still gets the old version.

## Snippet

`flake.nix`:

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    # Override: track a specific commit/release of an upstream repo.
    ripgrep-src = {
      url = "github:BurntSushi/ripgrep/14.1.1";
      flake = false;
    };
  };

  outputs = { self, nixpkgs, ripgrep-src, ... }: let
    system = "aarch64-darwin";
    overlays = [
      (final: prev: {
        ripgrep = prev.ripgrep.overrideAttrs (_: {
          src = ripgrep-src;
          version = "14.1.1";
        });
      })
    ];
    pkgs = import nixpkgs { inherit system overlays; };
  in {
    packages.${system}.default = pkgs.ripgrep;
  };
}
```

## Why

Overlays compose into the `pkgs` set, so every module that takes `pkgs` as an
argument gets the patched derivation — no plumbing required at the call sites.
Flake inputs alone only expose source; the overlay is the bridge that says
"build this source into the `ripgrep` attribute downstream consumers read."

## When NOT to use

Don't override via overlay when you only need the package in one module — use
`pkgs.ripgrep.overrideAttrs` inline at the call site, keep `pkgs` unpatched.
Overlays are transitive; one override can cascade a rebuild of every dependent
derivation. Verify with `nix-store --query --graph` before committing.

## Reference

- https://nixos.org/manual/nixpkgs/stable/#chap-overlays
