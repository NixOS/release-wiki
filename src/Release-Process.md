# Release Process

These pages give an overview of the release process with as many details as there are available at the time of writing.

## Release Schedule

With [RFC85](https://github.com/NixOS/rfcs/blob/master/rfcs/0085-nixos-release-stablization.md), the release schedule is largely already dictated below:

| Weeks from Release | Branches Affected | Events |
| --- | --- | --- |
| -8 Weeks | | Start discussion about "Feature Freeze & Release Blockers" with ecosystems maintainers |
| -6 Weeks | `staging-next`, `staging` | Restrict breaking changes to Release Critical Packages |
| -4 Weeks | `staging-next`, `staging` | Restrict all breaking changes with the exception of updates to desktop environment |
| -3 Weeks | `master` | Wait for `staging-next` merge into `master`, begin first `staging-next` cycle |
| -3 Weeks | `master` | Begin Zero Hydra Failures campaign |
| -2 Weeks | `master` | Wait for first `staging-next` merge into `master`; begin second `staging-next` cycle |
| -2 Weeks | `staging` | Unrestrict all breaking changes; new changes will not be present in the release |
| -1 Weeks | `master` | Wait for second `staging-next` merge into `master` |
| -1 Weeks | `staging-next` | Unrestrict all breaking changes; new changes will not be present in the release |
| -1 Weeks | `master`, `release` | Perform Branch-off, create release channels, create new beta / unstable tags |
| -1 Weeks | `master` | Mark failing packages as broken |
| -1 Weeks | `master`, `release` | Create the release branch and tags |
| -1 Weeks | `release` | Create release channels |
| -1 Weeks | `master`, `release` | Fixes need to use the "backport" workflow to arrive in the release |
| -1 Weeks | `release` | Prepare for release, finish remaining issues |
| 0 Weeks | `release` | **Release!** Marks the end of the Zero Hydra Failures campaign. |
| +1 Weeks | | Release Retrospective, everyone is invited to give feedback! |
| +4 Weeks | | End of life cleanup |


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
