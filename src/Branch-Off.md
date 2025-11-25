# Branch-off

For these steps "24.05" represents the current release tag and "24.11" represents the next
(6 months in the future) release tag. "23.11" is the last release which was released 6 months
ago.

## Jobset creation

The [infrastructure team](https://matrix.to/#/#infra:nixos.org) can create the necessary jobsets
in advance. They will not evaluate because the release branch doesn't exist but it allows the
infrastructure team and the release team to work more asynchronously. Reach out to them 1-2 days
before release to create the necessary Hydra Jobsets. You can link them this section.

- [https://hydra.nixos.org/project/nixos](https://hydra.nixos.org/project/nixos)
  - release-24.05
  - release-24.05-small
  - staging-next-24.05-small
- [https://hydra.nixos.org/project/nixpkgs](https://hydra.nixos.org/project/nixpkgs)
  - nixpkgs-24.05-darwin
  - staging-next-24.05
  - staging-24.05

The easiest way to create the new jobsets is to clone the ones from the previous release and rewrite the version number.

Example configuration: [nixos:release-25.11](https://hydra.nixos.org/jobset/nixos/release-25.11#tabs-configuration)

|Field|Value|
|-|-|
|State|Enabled|
|Description|NixOS 24.05 release branch|
|Nix expression|`nixos/release-combined.nix` in input `nixpkgs`|
|Check interval|151200|
|Scheduling shares|5000000 (8.32% out of 60071636 shares)|
|Enable Dynamic RunCommand Hooks:|No (not enabled by server)|
|Number of evaluations to keep|1|

### Inputs for `nixos/release-24.05`, `nixos/release-24.05-small`:

|Input name|Type|Values|
|-|-|-|
|`nixpkgs`|Git checkout|`https://github.com/NixOS/nixpkgs.git release-24.05`|
|`stableBranch`|Boolean|`false`|
|`supportedSystems`|Nix expression|`[ "x86_64-linux" "aarch64-linux" ]`|

`stableBranch` influences the `versionSuffix` in NixOS and the channel tarball.

  - `true` leads to `24.05.1234.0abs3fe`
  - `false` leads to `24.05pre1234.0abs3fe`

Note that changing this leads to a rebuild of most NixOS tests!

### Inputs for `nixpkgs/nixpkgs-24.05-darwin`:

|Input name|Type|Values|
|-|-|-|
|`nixpkgs`|Git checkout|`https://github.com/NixOS/nixpkgs.git release-24.05`|
|`officialRelease`|Boolean|`false`|
|`supportedSystems`|Nix expression|`[ "x86_64-darwin" "aarch64-darwin" ]`|

`officialRelease` influences the `versionSuffix` of the release tarball

  - `true` leads to `24.05.1234.0abs3fe`
  - `false` leads to `24.05pre1234.0abs3fe`

## Nixpkgs branch protection ruleset

The same as for the jobsets applies to Nixpkgs branch protection rulesets, which can be updated by NixOS org owners in advance.
You can propose the changes as a Pull Request to the NixOS/org repository.

New release branches should be added to the `includes` list in:
- https://github.com/NixOS/org/blob/main/rulesets/nixpkgs/require-merge-queue.json

## Actual branch-off

Set NEWVER to the new release version (i.e. the version you release):

```bash
export NEWVER=24.05
```

### On the master branch

Pull in the final changes before performing the actual branch-off.

1. Wait for the staging team to merge the final staging-next iteration

1. Fetch and check out the master branch

1. Create the release branch:

   ```bash
   git switch -c release-$NEWVER
   ```

### On the release branch

Update metadata on the release branch, create its staging branches and tag the release.

1. Update the `system.defaultChannel` option default in [`nixos/modules/config/nix-channel.nix`](https://github.com/NixOS/nixpkgs/commit/3c80acabe4eef35d8662733c7e058907fa33a65d#diff-14008678e5ceeef1edd491de85bca2bded89daea16793fe10c4eb724097b7f12)

1. Update the `versionSuffix` attribute in [`nixos/release.nix`](https://github.com/NixOS/nixpkgs/commit/3c80acabe4eef35d8662733c7e058907fa33a65d#diff-20da30ee012d7d87842fb7953237870493c5497c995cba1e6f6c3aa9268398ff)

   To get the commit count, use the following command:

   ```bash
   git rev-list --count release-$NEWVER
   ```

1. Add `SUPPORT_END = "YYYY-MM-DD";` to `osReleaseContents` in [`nixos/modules/misc/version.nix`](https://github.com/NixOS/nixpkgs/commit/3c80acabe4eef35d8662733c7e058907fa33a65d#diff-b3379a98640b35a5fe4b046150cd2df1639995edf231d18bbad832be6a70b45f).

1. Run treefmt
   ```bash
   nix-shell --run treefmt
   ```

1. Test that the `tested` "job" still evals
   ```bash
   nix eval -f nixos/release-combined.nix tested
   ```

1. Commit the changes from the previous steps

   ```bash
   git commit -m "$NEWVER beta release" -S
   ```

1. Update the staging branches

   ```bash
   git checkout staging-$NEWVER
   git merge release-$NEWVER
   git checkout staging-next-$NEWVER
   git merge release-$NEWVER
   ```

1. Tag the release and push everything

   ```bash
   git switch release-$NEWVER
   git tag --annotate --message="Release $NEWVER-beta" $NEWVER-beta
   git push upstream release-$NEWVER $NEWVER-beta staging-$NEWVER staging-next-$NEWVER
   ```

1. Create jobsets on hydra by contacting the infrastructure team and start the evaluation on all new jobsets.

1. Switch back to the master branch

   ```bash
   git switch master
   ```

1. Create the backport [label](https://github.com/NixOS/nixpkgs/labels) for the new release branch:
   - `backport release-24.05`

   Use the description `Backport PR automatically` and the color value `#0fafaa`

### Back on the master branch

Now we prepare the master branch for the next release after this one. We do this in two steps, error-prone changes thorugh a PR and direct to allow for proper tagging.

Set NEXTVER to the release number after the one you released (i.e. when you release 24.05, `NEXTVER=24.11`).

```bash
export NEXTVER=24.11
```

#### PR changes 1

1. Checkout a branch you want for the first PR, e.g.

   ```bash
   git switch -c rm-branchoff-documentation-changes
   ```

1. Add the release name to [`nixos/doc/manual/release-notes/rl-$NEXTVER.section.md`](https://github.com/NixOS/nixpkgs/commit/bf470a4fddca6315d1b7c256a873c90f755b02d9#diff-417feb28ebc7ab0109746fef515d6cccdcd5af5c7386e1ce0b186db9b8e5677b), [`doc/release-notes/rl-2411.section.md`](https://github.com/NixOS/nixpkgs/commit/bf470a4fddca6315d1b7c256a873c90f755b02d9#diff-c5ba8855198f95e7fdb39bcd1106bb29c96b85c14bdfc4030b235a02806bf9b3)

1. Include `rl-$NEXTVER.section.md` in [`nixos/doc/manual/release-notes/release-notes.md`](https://github.com/NixOS/nixpkgs/commit/bf470a4fddca6315d1b7c256a873c90f755b02d9#diff-9b75bf997f6c13cb4a15145ef9e758a28addeeff4a3a5cb893a5c23a976b3a1a), [`doc/release-notes/release-notes.md`](https://github.com/NixOS/nixpkgs/commit/bf470a4fddca6315d1b7c256a873c90f755b02d9#diff-89abd55b6169384ba08083097a7ac5f5c30ea35d432a0ba2d1a76887f50a4f49)

1. Update [`nixos/manual/doc/redirects.json`](https://github.com/NixOS/nixpkgs/commit/bf470a4fddca6315d1b7c256a873c90f755b02d9#diff-8e9034e569678f9c67bf82087ff7dced3b18de5491707988a43fb8979a82794f), [`doc/redirects.json`](https://github.com/NixOS/nixpkgs/commit/bf470a4fddca6315d1b7c256a873c90f755b02d9#diff-9499ad98fd024845445ff2f32b424d29f632932b4bbe15dfd9d0f447890bbae6) and to include the new sections.

1. Go through `.github/ISSUE_TEMPLATE`, and edit them to include the new version as beta.

   Example [25.11](https://github.com/NixOS/nixpkgs/commit/c69a76cf803096fb8d1b3732fea57d69d6d301f1)

1. Commit the changes and create a PR.
   Wait for the CI to finish and merge.

#### PR changes 2

You can create this PR while waiting for the CI of the first one.

1. Checkout a branch you want for the second PR, e.g.

   ```bash
   git switch -c rm-branchoff-ci-changes
   ```

1. Update [.github/workflows/periodic-merge-24h.yml](https://github.com/NixOS/nixpkgs/commit/bf470a4fddca6315d1b7c256a873c90f755b02d9#diff-a4f6ea695ede268916c760fe782e9645a8cab5b27747e4baa994bf59f3e4e07b) so that
   - `release-$NEWVER` (instead of `master`) gets merged into `staging-next-$NEWVER`

1. Update [.github/labeler-no-sync.yml](https://github.com/NixOS/nixpkgs/commit/fb0fb7420fcafaaced083c6767da35617f2837f3) so that it includes `backport release-25.11`.

#### Direct changes

These changes should be done when the two PRs are already merged.

1. Switch back to the `master` branch
  ```bash
  git switch master
  ```

1. Update the branch to get the changes from the two PRs
  ```bash
  git pull
  ```

1. Increment the [`lib/.version`](https://github.com/NixOS/nixpkgs/commit/01268fda85b7eee4e462c873d8654f975067731f#diff-2bc0e46110b507d6d5a344264ef15adaR1)
   file. This file must **not** end with a new line!

   ```bash
   # The release after $NEWVER (24.05 -> 24.11)
   echo -n "$NEXTVER" > lib/.version
   ````

1. Update the `codeName` attribute in [`lib/trivial.nix`](https://github.com/NixOS/nixpkgs/commit/2c28f1de7cdc10be556d2106108411dd2482794b#diff-29c71aa8261b14b1cad6e6fa28486fed7295050db4eeb32ba205672ba91d40e1)
   This will be the name for the next release.

1. Commit the changes as `$NEXTVER is <codeName>`([25.11 example](https://github.com/NixOS/nixpkgs/commit/2493002b10ccef0880f72d7720538f91fb4f7434))

1. Tag the master branch, so that `git describe` shows the new version as the base for commits.

   ```bash
   git tag --annotate $NEXTVER-pre
   git push upstream master $NEXTVER-pre
   git describe HEAD # should yield $NEXTVER-pre
   ```

### And afterwards

Now that everything on git is done, we are still missing the channels.

1. Create the necessary channels for `https://channels.nixos.org` in beta status, by updating
   [`channels.nix`](https://github.com/NixOS/infra/blob/master/channels.nix) in `infra`
   and nag the infrastructure team to get these changes deployed.

   Example: [22.11](https://github.com/NixOS/infra/commit/9a0b3674a11b445c973334c78e8ca0eda36775e4)

1. Update the ZHF issue, that now that the branch-off has been performed, fixes have to be backported.
   Examples: [22.05](https://github.com/NixOS/nixpkgs/issues/172160#issuecomment-1135112918)

### Once the channel is available

The following steps should be done after the channels have become available on [channels.nixos.org](https://channels.nixos.org).

1. Update the flake input on the `nixos-search` repository by running [the update-flake-lock](https://github.com/NixOS/nixos-search/actions/workflows/update-flake-lock.yml) and merging the new created PR afterwards.

1. Give the [Marketing team](https://matrix.to/#/#marketing:nixos.org) a heads-up about the upcoming release

1. Get in contact with [Arian van Putten](https://github.com/arianvp)

   1. They will need to update [upload-legacy-ami.yml](https://github.com/NixOS/amis/blob/main/.github/workflows/upload-legacy-ami.yml) to point to the new jobset. This will automatically upload the AMIs from the jobset to AWS
   1. A PR is also welcome :)

1. Make sure the release editors have started finalizing the release notes. Only 7 days left until release!

1. Add release to LXC/Incus image server

   Ping [Adam C. Stephens](https://github.com/adamcstephens) or open a Pull Request to [lxc/lxc-ci](https://github.com/lxc/lxc-ci/blob/720a50e23f9a122694056d7394226476ae24f973/jenkins/jobs/image-nixos.yaml#L19-L21)

## For Release Editors

Following a branch-off, Release Editors should keep in mind that the release notes between `master`
and the release branches may fall out of sync. This is usually due to PRs adding release note
entries being merged post branch-off. Entries for changes not actually present in the branched
release may appear in `master` due to this, and backporting restructuring of the notes can be
troublesome.
