# EOL cleanup

Once a month has passed after the release was released, the old release goes end of life.

1. [EOL Channels should be marked unmaintained](https://github.com/NixOS/nixos-org-configurations/pull/201)
1. Once this is merged, update [nixos-search as well](https://github.com/NixOS/nixos-search/pull/495)
1. Inform the infrastructure team so they can scale Hydra back down
1. Close all Pull Requests that are still opened against one of the the old release- or staging-branches
1. Delete the old [backport labels](https://github.com/NixOS/nixpkgs/labels?q=backport)
1. Remove the old [automerge workflows](https://github.com/NixOS/nixpkgs/commit/8befefd1a72da597bdb1d01e97127e0c9866912e)
1. Increase the `oldestSupportedRelease` in `lib/trivial.nix` to match the oldest non-EOL release ([22.05 example](https://github.com/NixOS/nixpkgs/pull/180152))
