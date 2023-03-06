# ZERO Hydra Failures

## Hydra scaling

The activity during ZHF is usually higher, so more build capacity is a good
investment, to keep up with the pace of contributions.

There is an autoscaler in place for aarch64-linux and x86_64-linux while the
darwin allocations are pretty much static.

If capacity issues are apparent, contact the infrastructure team on the
`nixos-org-configurations` repository (example: [21.11 example](https://github.com/NixOS/nixos-org-configurations/issues/186)),
and start a discussion in [#infra:nixos.org](https://matrix.to/#/#infra:nixos.org).

## ZHF issue

- Create an issue for tracking Zero Hydra Failures progress. ZHF is an effort
  to get build failures down to zero.
  [22.05 example](https://github.com/NixOS/nixpkgs/issues/172160)

- With RFC85, the
  [nixpkgs/trunk](https://hydra.nixos.org/jobset/nixpkgs/trunk) jobset will be
  used to track regressions.

- Monitor the issue and linked Pull Requests and check if new blockers come up.

- Get in contact with the staging team (for example via their Matrix room) in
  the last days before branch-off to coordinate the staging-next merges.
