# Release Process

These pages give an overview of the release process with as many details as there are available at the time of writing.

## Release Schedule

With [RFC85](https://github.com/NixOS/rfcs/blob/master/rfcs/0085-nixos-release-stablization.md), the release schedule is largely already dictated below:

| Weeks from Release | Branches Affected | Events |
| --- | --- | --- |
| -8 Weeks | | Ask ecosystems for desired changes, in "Feature Freeze" |
| -6 Weeks | `staging-next`, `staging` | Restrict breaking changes to Release Critical Packages |
| -4 Weeks | `staging-next`, `staging` | Restrict all breaking changes: allow only non-breaking updates and Desktop Manager changes |
| -3 Weeks | `master` | (Day before ZHF) merge `staging-next` into `master`, prep for ZHF |
| -3 Weeks | `master` | Begin ZHF, Focus on minimizing regressions in PRs targeting `master` |
| -2 Weeks | `master` | Merge first `staging-next` fixes into `master`; begin second `staging-next` cycle |
| -2 Weeks | `staging` | Unrestrict all breaking changes; new changes will not be present in `master` before branch-off |
| -1 Weeks | `master` | Merge second `staging-next` fix cycle |
| -1 Weeks | `staging-next` | Unrestrict all breaking changes; new changes will not be present in `master` before branch-off |
| -1 Weeks | `master`, `release` | Perform Branch-off, create release channels, create new beta / unstable tags |
| -1 Weeks | `master`, `release` | ZHF transitions to "backporting" workflow |
| -1 Weeks | `release` | Prepare for release, finish remaining issues |
| 0 Weeks | `release` | **Release!** |
| 0 Weeks | | ZHF Ends |
| +4 Weeks | | Release Retrospective and final cleanup |

Clarification of terms:

| Term | Definition |
| --- | --- |
| `staging` | Git branch used to batch changes which cause large rebuilds (`500+` packages). This is to avoid straining infrastructure trying to build large package sets with each change. Since nix captures all changes in a dependency tree, a change to a fundamental package like glibc can affect `50,000+` packages. |
| `staging-next` | Git branch used to build all of the batched changes. This branch has a [Hydra jobset](https://hydra.nixos.org/jobset/nixpkgs/staging-next) which is used to fix regressions before the changes will be merged into master. |
| `ZHF` | Zero Hydra Failures. A 2-4 week window before a release where there is a large attempt to stabilize failing builds. Also allows for additional "housekeeping" such as removing stale, abandoned, or unmaintained packages. |

The purpose of these changes are mainly to push "riskier" PRs to being merged earlier,
or after a release. Merging pull-requests which may have many downstream breakages should
be avoided right before ZHF, as ZHF should focus on stabilization of the nixpkgs package
set, and not distracted by large noise-to-signal build failures. Also, fixes to packages
which required the `staging` process will likely require additional `staging` cycles to
remedy the regressions.
