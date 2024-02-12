# ZERO Hydra Failures

## Hydra scaling

The activity during ZHF is usually higher, so more build capacity is a good
investment, to keep up with the pace of contributions.

There is an autoscaler in place for aarch64-linux and x86_64-linux while the
darwin allocations are pretty much static.

If capacity issues are apparent, contact the infrastructure team on the
`infra` repository (example: [21.11 example](https://github.com/NixOS/infra/issues/186)),
and start a discussion in [#infra:nixos.org](https://matrix.to/#/#infra:nixos.org).

## ZHF issue

- Create an issue for tracking Zero Hydra Failures progress. ZHF is an effort
  to get build failures down to zero. Examples:
  [22.11](https://github.com/NixOS/nixpkgs/issues/199919), [22.05](https://github.com/NixOS/nixpkgs/issues/172160)

- The [nixpkgs/trunk](https://hydra.nixos.org/jobset/nixpkgs/trunk) jobset is ideal
  to track regressions, since it builds for all platforms.

- Subscribe to the issue and monitor the conversation for new blockers.

- Get in contact with the staging team through their Matrix room at [#staging:nixos.org](https://matrix.to/#/#staging:nixos.org)
  to coordinate the staging-next merges.
