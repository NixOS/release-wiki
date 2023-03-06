# Branch-off

For these steps "21.05" represents the current release tag and "21.11" represents the next
(6 months in the future) release tag. "20.11" is the last release which was released 6 months
ago.

## Jobset creation

The [infrastructure team](https://matrix.to/#/#infra:nixos.org) can create the necessary jobsets
in advance. They will not evaluate because the release branch doesn't exist but it allows the
infrastructure team and the release team to work more asynchronously. Reach out to them 1-2 days
before release to create the necessary Hydra Jobsets. You can link them this section.

- [https://hydra.nixos.org/project/nixos](https://hydra.nixos.org/project/nixos)
    - release-21.05	NixOS 21.05
    - release-21.05-aarch64	NixOS 21.05
    - release-21.05-small	NixOS 21.05
- [https://hydra.nixos.org/project/nixpkgs](https://hydra.nixos.org/project/nixpkgs)
    - nixpkgs-21.05-darwin
    - staging-next-21.05

For example: [21.05 configuration](https://hydra.nixos.org/jobset/nixos/release-21.05#tabs-configuration)

|Field|Value|
|-|-|
|State|Enabled|
|Description|NixOS 21.05 release branch|
|Nix expression|`nixos/release-combined.nix` in input `nixpkgs`|
|Check interval|5000000|
|Scheduling shares|500|
|Number of evaluations to keep|1|

Inputs:

|Input name|Type|Values|
|-|-|-|
|`nixpkgs`|Git checkout|`https://github.com/NixOS/nixpkgs.git release-21.05`|
|`stableBranch`|Boolean|`false`|
|`supportedSystems`|Nix expression|`[ "x86_64-linux" "aarch64-linux" ]`|

## Actual branch-off

### On the master branch

Pull in the final changes before performing the actal branch-off.

1. Wait for the staging team to merge the final staging-next iteration

1. Fetch and check out the master branch

1. Create the release branch:

   ```shell
   git switch -c release-21.05
   ```

### On the release branch

Update metadata on the release branch, create its staging branches and tag the release.

1. Update the `system.defaultChannel` attribute in [`nixos/modules/misc/version.nix`](https://github.com/NixOS/nixpkgs/commit/bb029673bface2fc9fb807f209f63ca06478a72d)

1. Update the `versionSuffix` attribute in [`nixos/release.nix`](https://github.com/NixOS/nixpkgs/commit/7ae60dd7068478db5d936a3850b6df859aec21d0)

   To get the commit count, use the following command:

   ```shell
   git rev-list --count release-21.05
   ```


1. Add `SUPPORT_END=YYYY-MM-DD` to `osReleaseContents` in `nixos/modules/misc/version.nix`.

1. Commit the changes from the previous steps

   ```shell
   git commit -m "$NEWVER beta release" -S
   ```

1. Create the staging branches
   ```shell
   git branch staging-21.05
   git branch staging-next-21.05
   ```

1. Tag the release and push everything

   ```shell
   git tag --annotate --message="Release 21.05-beta" 21.05-beta
   git push upstream master release-21.05 21.05-beta staging-21.05 staging-next-21.05
   ```

1. Create jobsets on hydra by contacting the infrastructure team and start the evaluation on all new jobsets.

1. Switch back to the master branch

   ```shell
   git switch master
   ```

### Back on the master branch

Now we prepare the master branch for the next release after this one.


1. Update the [periodic-merge workflow](https://github.com/NixOS/nixpkgs/blob/master/.github/workflows/periodic-merge-24h.yml) to include the new release.

1. Increment the [`.version`](https://github.com/NixOS/nixpkgs/commit/01268fda85b7eee4e462c873d8654f975067731f#diff-2bc0e46110b507d6d5a344264ef15adaR1)
   file in the repository root.

   ```shell
   echo -n "21.11" > .version
   ````

1. Update the `codeName` attribute in [`lib/trivial.nix`](https://github.com/NixOS/nixpkgs/commit/01268fda85b7eee4e462c873d8654f975067731f#diff-03f3d41b68f62079c55001f1a1c55c1dR137)
   This will be the name for the next release.

1. Create a new [release notes file](https://github.com/NixOS/nixpkgs/commit/01268fda85b7eee4e462c873d8654f975067731f#diff-e7ee5ff686cdcc513ca089d6e5682587R11)
   for the next release

1. Commit the changes ([22.05 example](https://github.com/NixOS/nixpkgs/commit/bfdfe12c788d7474b88e7a7790b88b1c0f8e01b5) + [this additional commit](https://github.com/NixOS/nixpkgs/commit/953b5d19bca4d4ddfaef5625cca277c47b39f5e7))

1. Tag the master branch, so that `git describe` shows the new version as the base for commits..

   ```shell
   git tag --annotate 21.11-pre
   git push upstream master 21.11-pre
   git describe HEAD # should yield 21.11-pre
   ```

### And afterwards

Now that everything on git is done, we are still missing the channels.

1. Create the necessary channels for `https://channels.nixos.org` in beta status, by updating
   [`channels.nix`](https://github.com/NixOS/nixos-org-configurations/blob/master/channels.nix) in `nixos-org-configurations`
   and nag the infrastructure team to get these changes deployed.

   The channels since NixOS 22.11 are:

     - nixos-22.11
     - nixos-22.11-small
     - nixos-22.11-darwin

   Example: [22.11](https://github.com/NixOS/nixos-org-configurations/commit/9a0b3674a11b445c973334c78e8ca0eda36775e4)

1. Create the backport [labels](https://github.com/NixOS/nixpkgs/labels) for all new branches:
   - `backport staging-21.05`
   - `backport staging-21.05`

   Use the description `Backport PR automatically` and the color value `#0fafaa`

### Once the channel is available

The following steps should be done after the channels have become available on [channels.nixos.org](https://channels.nixos.org).

1. Update the flake input on the `nixos-search` repository` and create a pull request:

   ```shell
   git clone git@github.com:nixos/nixos-search`
   nix --extra-experimental-features nix-command flakes flake update nixos-org-configurations
   ```

1. Give the [Marketing team](https://matrix.to/#/#marketing:nixos.org) a heads-up about the upcoming release

1. Update the ZHF issue, that now that the branch-off has been performed, fixes have to be backported.
   Examples: [22.05](https://github.com/NixOS/nixpkgs/issues/172160#issuecomment-1135112918)
