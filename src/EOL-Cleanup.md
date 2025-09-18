# EOL cleanup

The old release reaches its end of life one month after the new release
gets released. At that point a few cleanup tasks need to be done.

1. Set the EOL channel status from `deprecated` to `unmaintained` in `infra`. Examples:
    - [channels: Deprecate NixOS 22.05](https://github.com/NixOS/infra/pull/229)
    - [channels: Mark 21.11 as unmaintained](https://github.com/NixOS/infra/pull/211)
    - [channels:21.05 is unmaintained](https://github.com/NixOS/infra/pull/201)

1. Once this is merged, update the [`nixos-search`](https://github.com/NixOS/nixos-search)
   repository to reflect that status on [search.nixos.org](https://search.nixos.org).

   ```bash
   nix --extra-experimental-features "nix-command flakes" flake lock --update-input nixos-infra
   ```

1. Close all pull requests that target one of the the old release- or staging-branches

1. Prune old [backport labels](https://github.com/NixOS/nixpkgs/labels?q=backport)

Now create a PR that contains the following changes:

1. Remove the old release from the [periodic merge workflows](https://github.com/NixOS/nixpkgs/commit/8befefd1a72da597bdb1d01e97127e0c9866912e)

1. Remove the references to the old release in `.github/PULL_REQUEST_TEMPLATE.md`

   Examples: [25.05](https://github.com/NixOS/nixpkgs/commit/23454de4554d675d144a5cbeaf0be391a43e5f7f)

1. Remove the references to the old release in `.github/ISSUE_TEMPLATE/*`

   Examples: [25.05](https://github.com/NixOS/nixpkgs/commit/7a31ddea37d998b8f0d5813e71abda0a6a6e75c0)

1. Remove the old release branch from auto-labeler in `.github/labeler-no-sync.yml`

   Examples: [25.05](https://github.com/NixOS/nixpkgs/commit/e83d9fbf169c90bfd9d784f11c8fcc303181e388)

1. Increase the `oldestSupportedRelease` in `lib/trivial.nix` to match
   the oldest supported release.

   Note: This may need a seperate PR as it can be breaking.

   Examples: [22.05](https://github.com/NixOS/nixpkgs/pull/180152)

1. Remove old release from LXC/Incus image server

   Ping [Adam C. Stephens](https://github.com/adamcstephens) or open a Pull Request to [lxc/lxc-ci](https://github.com/lxc/lxc-ci/blob/720a50e23f9a122694056d7394226476ae24f973/jenkins/jobs/image-nixos.yaml#L19-L21)

Last but not least, update the Nixpkgs branch protection rules and remove the old branch from the Merge Queue rule at:
https://github.com/NixOS/org/blob/main/rulesets/nixpkgs/require-merge-queue.json
