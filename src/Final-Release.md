# Final Release

## Before the final release

- Two days before expected release date, change `stableBranch` to `true` in Hydra and wait for the channel to update

- Re-check that the release notes are complete

- Merge the Pull Request that marks broken packages as broken

- [Update README.md with new stable NixOS version information.](https://github.com/NixOS/nixpkgs/commit/40fd9ae3ac8048758abdcfc7d28a78b5f22fe97e) (for `master` then backport to the release branch)

- Ensure the following items are sufficiently tested:
  - Graphical NixOS installer images (Can be found on nixos hydra jobset)
  - Minimal NixOS installer images (Can be found on nixos hydra jobset)
