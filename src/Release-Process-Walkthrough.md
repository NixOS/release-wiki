# Release Process Walkthrough

## Release schedule

With RFC85, the release schedule is largely already dictated below:

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
| 0 Weeks | `release` | Release! |
| 0 Weeks | | ZHF Ends |

Clarification of terms:

| Term | Definition |
| --- | --- |
| `staging` | Git branch used to batch changes which cause large rebuilds (`500+` packages). This is to avoid straining infrastructure trying to build large package sets with each change. Since nix captures all changes in a dependency tree, a change to a fundamental package like glibc can affect 50,000+ packages. |
| `staging-next` | Git branch used to build all of the batched changes. This branch has a [hydra jobset](https://hydra.nixos.org/jobset/nixpkgs/staging-next) which is used to fix regressions before the changes will be merged into master. |
| `ZHF` | Zero Hydra Failures. A 2-4 week window before a release where there is a large attempt to stabilize failing builds. Also allows for additional "housekeeping" such as removing stale, abandoned, or unmaintained packages. |


The purpose of these changes are mainly to push "riskier" PRs to being merged earlier,
or after a release. Merging pull-requests which may have many downstream breakages should
be avoided right before ZHF, as ZHF should focus on stabilization of the nixpkgs package
set, and not distracted by large noise-to-signal build failures. Also, fixes to packages
which required the `staging` process will likely require additional `staging` cycles to
remedy the regressions.

## 8 Weeks before release

- Create an announcement on [Discourse](https://discourse.nixos.org)
  as a warning about upcoming beta “feature freeze” in a month. [See
  this post as an
  example](https://discourse.nixos.org/t/nixos-19-09-feature-freeze/3707).

- Discuss with Eelco Dolstra and the community (via Matrix) about
  what will reach the deadline. Any issue or Pull Request targeting
  the release should be included in the release milestone.

- Remove attributes that we know we will not be able to support,
  especially if there is a stable alternative. E.g. Check that our
  Linux kernels’ [projected
  end-of-life](https://www.kernel.org/category/releases.html) are
  after our release projected end-of-life.

## Zero Hyra Failure

1.  [Create an issue for tracking Zero Hydra Failures progress.
    ZHF is an effort to get build failures down to zero.](https://github.com/NixOS/nixpkgs/issues/13559)

2.  With RFC85, the [nixpkgs/trunk](https://hydra.nixos.org/jobset/nixpkgs/trunk) jobset will be used to track regressions.

## At beta release time, 1 week before release

1.  Merge current "Staging next" pull-request
2.  From the master branch run:

```sh
git checkout -b release-19.09
```

3.  [Bump the `system.defaultChannel` attribute in
    `nixos/modules/misc/version.nix`](https://github.com/NixOS/nixpkgs/commit/10e61bf5be57736035ec7a804cb0bf3d083bf2cf#diff-9c798092bac0caeb5c52d509be0ca263R69)

4.  [Update `versionSuffix` in
    `nixos/release.nix`](https://github.com/NixOS/nixpkgs/commit/10e61bf5be57736035ec7a804cb0bf3d083bf2cf#diff-831e8d9748240fb23e6734fdc2a6d16eR15)

To get the commit count, use the following command:

    git rev-list --count release-19.09

5.  Edit changelog at `nixos/doc/manual/release-notes/rl-1909.xml`.

    - Get all new NixOS modules:

      ```sh
      git diff release-19.03..release-19.09 nixos/modules/module-list.nix | grep ^+
      ```

    - Note systemd, kernel, glibc, desktop environment, and Nix
      upgrades.

6.  Tag the release:

```sh
git tag --annotate --message="Release 19.09-beta" 19.09-beta
git push upstream 19.09-beta

# create staging branches
git checkout -b staging-19.09
git push upstream staging-19.09

git checkout -b staging-next-19.09
git push upstream staging-next-19.09
```

7.  [On the `master` branch, increment the `.version`
    file](https://github.com/NixOS/nixpkgs/commit/01268fda85b7eee4e462c873d8654f975067731f#diff-2bc0e46110b507d6d5a344264ef15adaR1)

```sh
echo -n "20.03" > .version
```

8.  [Update `codeName` in
    `lib/trivial.nix`](https://github.com/NixOS/nixpkgs/commit/01268fda85b7eee4e462c873d8654f975067731f#diff-03f3d41b68f62079c55001f1a1c55c1dR137)
    This will be the name for the next release.
    
9.  [Create a new release notes file for the upcoming release +
     1](https://github.com/NixOS/nixpkgs/commit/01268fda85b7eee4e462c873d8654f975067731f#diff-e7ee5ff686cdcc513ca089d6e5682587R11),
    in our case this is `rl-2003.xml`.

10.  Tag master branch so that `git describe` shows the correct metadata.
```sh
git tag --annotate 20.03-pre
git push upstream 20.03-pre
```

11.  Contact the infrastructure team to create the necessary Hydra
    Jobsets.

12.  [Create a channel at https://nixos.org/channels by creating a PR to
    nixos-org-configurations, changing`channels.nix`](https://github.com/NixOS/nixos-org-configurations/blob/master/channels.nix)

## During Beta

- Monitor the master branch for bugfixes and minor updates and
  cherry-pick them to the release branch.

## Before the final release

- Re-check that the release notes are complete.

- Release Nix (currently only Eelco Dolstra can do that). [Make sure fallback is updated.](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/installer/tools/nix-fallback-paths.nix)

- [Update README.md with new stable NixOS version information.](https://github.com/NixOS/nixpkgs/commit/40fd9ae3ac8048758abdcfc7d28a78b5f22fe97e)

- Change `stableBranch` to `true` in Hydra and wait for the channel to
  update.

- Ensure the following items are sufficiently tested:
  - Graphical NixOS installer images (Can be found on nixos hydra jobset)
  - Minimal NixOS installer images (Can be found on nixos hydra jobset)

## At final release time

1.  Update [Upgrading NixOS](https://nixos.org/manual/nixos/stable/index.html#sec-upgrading) section of the manual to match new
    stable release version.

2.  Update `rl-1909.xml` with the release date.

3.  Tag the final release

```sh
git tag --annotate --message="Release 19.09" 19.09
git push upstream 19.09
```

4.  Update [nixos-homepage](https://github.com/NixOS/nixos-homepage) for
    the release.

    1.  [Update the `flake.nix` input `released-nixpkgs` to 19.09](https://github.com/NixOS/nixos-homepage/blob/47ac3571c4d71e841fd4e6c6e1872e762b9c4942/flake.nix#L10).

    2.  Run `./scripts/update.sh` (this updates flake.lock to updated channel).

    3.  [Add a compressed version of the NixOS logo for 19.09](https://github.com/NixOS/nixos-homepage/blob/a5626c71c03a2dd69086564e56f1a230a2bb177a/logo/nixos-logo-19.09-loris-lores.png).

    4.  [Compose a news item for the website RSSfeed](https://github.com/NixOS/nixos-homepage/commit/a5626c71c03a2dd69086564e56f1a230a2bb177a#diff-9cdc6434d3e4fd93a6e5bb0a531a7c71R5).

5.  Update [nixos-search](https://github.com/NixOS/nixos-search/)

    1.  Follow similar changes to [this PR](https://github.com/NixOS/nixos-search/pull/311/files).

    2.  Get in contact with Garbas to ensure proper deployment.

6.  Create a new topic on [the Discourse instance](https://discourse.nixos.org/) to announce the release.

You should include the following information:

- Number of commits for the release:

```sh
git log release-19.03..release-19.09 --format=%an | wc -l
```

- Commits by contributor:

```sh
git shortlog --summary --numbered release-19.03..release-19.09
```

Best to check how the previous post was formulated to see what needs to
be included.

## After Release

1.  [Update repology](https://github.com/repology/repology-updater/pull/1156/files)

2.  Update osinfo-db entries. [example](https://gitlab.com/libosinfo/osinfo-db/-/merge_requests/312)

```sh
nix-shell -p "python3.withPackages (p: [ p.lxml p.requests ])" -p cdrkit
./scripts/updates/nixos.py --release YY.MM --codename <codename> --release-date YYYY-MM-DD --next-release YY.MM
git add .
git commit -s -m "nixos: Add YY.MM release"

# run tests to validate your changes
nix-shell -p "python3.withPackages (p: [ p.lxml p.requests p.pytest ])" -p cdrkit osinfo-db-tools gettext --run "make check"
```

3.  Update ami images

  1.  Someone with s3 buck permssions will need to run `https://github.com/NixOS/nixpkgs/blob/master/nixos/maintainers/scripts/ec2/create-amis.sh` (usually AmineChikhaoui).

  2.  [Create PR adding it to NixOS configuration](https://github.com/NixOS/nixpkgs/pull/101720).
