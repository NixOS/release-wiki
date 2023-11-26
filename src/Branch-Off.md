# Branch-off

For these steps "23.05" represents the current release tag and "23.11" represents the next
(6 months in the future) release tag. "22.11" is the last release which was released 6 months
ago.

## Jobset creation

The [infrastructure team](https://matrix.to/#/#infra:nixos.org) can create the necessary jobsets
in advance. They will not evaluate because the release branch doesn't exist but it allows the
infrastructure team and the release team to work more asynchronously. Reach out to them 1-2 days
before release to create the necessary Hydra Jobsets. You can link them this section.

- [https://hydra.nixos.org/project/nixos](https://hydra.nixos.org/project/nixos)
    - release-22.11
    - release-22.11-small
- [https://hydra.nixos.org/project/nixpkgs](https://hydra.nixos.org/project/nixpkgs)
    - nixpkgs-22.11-darwin
    - staging-next-22.11

Example configuration: [nixos:release-22.11](https://hydra.nixos.org/jobset/nixos/release-22.11#tabs-configuration)

|Field|Value|
|-|-|
|State|Enabled|
|Description|NixOS 22.11 release branch|
|Nix expression|`nixos/release-combined.nix` in input `nixpkgs`|
|Check interval|86400|
|Scheduling shares|5000000 (8.32% out of 60071636 shares)|
|Enable Dynamic RunCommand Hooks:|No (not enabled by server)|
|Number of evaluations to keep|1|

Inputs:

|Input name|Type|Values|
|-|-|-|
|`nixpkgs`|Git checkout|`https://github.com/NixOS/nixpkgs.git release-22.11`|
|`stableBranch`|Boolean|`false`|
|`supportedSystems`|Nix expression|`[ "x86_64-linux" "aarch64-linux" ]`|

## Actual branch-off

Set NEWVER to the new release version:

```bash
export NEWVER=23.05
```

### On the master branch

Pull in the final changes before performing the actal branch-off.

1. Wait for the staging team to merge the final staging-next iteration

1. Fetch and check out the master branch

1. Create the release branch:

   ```bash
   git switch -c release-$NEWVER
   ```

### On the release branch

Update metadata on the release branch, create its staging branches and tag the release.

1. Update the `system.defaultChannel` attribute in [`nixos/modules/config/nix-channel.nix`](https://github.com/NixOS/nixpkgs/commit/bb029673bface2fc9fb807f209f63ca06478a72d)

1. Update the `versionSuffix` attribute in [`nixos/release.nix`](https://github.com/NixOS/nixpkgs/commit/7ae60dd7068478db5d936a3850b6df859aec21d0)

   To get the commit count, use the following command:

   ```bash
   git rev-list --count release-$NEWVER
   ```


1. Add `SUPPORT_END=YYYY-MM-DD` to `osReleaseContents` in `nixos/modules/misc/version.nix`.

1. Commit the changes from the previous steps

   ```bash
   git commit -m "$NEWVER beta release" -S
   ```

1. Create the staging branches
   ```bash
   git branch staging-$NEWVER
   git branch staging-next-$NEWVER
   ```

1. Tag the release and push everything

   ```bash
   git tag --annotate --message="Release $NEWVER-beta" $NEWVER-beta
   git push upstream master release-$NEWVER $NEWVER-beta staging-$NEWVER staging-next-$NEWVER
   ```

1. Create jobsets on hydra by contacting the infrastructure team and start the evaluation on all new jobsets.

1. Switch back to the master branch

   ```bash
   git switch master
   ```

### Back on the master branch

Now we prepare the master branch for the next release after this one.


1. Update the [periodic-merge workflow](https://github.com/NixOS/nixpkgs/blob/master/.github/workflows/periodic-merge-24h.yml) to include the new release.

1. Increment the [`.version`](https://github.com/NixOS/nixpkgs/commit/01268fda85b7eee4e462c873d8654f975067731f#diff-2bc0e46110b507d6d5a344264ef15adaR1)
   file in the repository root.

   ```bash
   # The release after $NEWVER (23.05 -> 23.11)
   echo -n "23.11" > .version
   ````

1. Update the `codeName` attribute in [`lib/trivial.nix`](https://github.com/NixOS/nixpkgs/commit/01268fda85b7eee4e462c873d8654f975067731f#diff-03f3d41b68f62079c55001f1a1c55c1dR137)
   This will be the name for the next release.

1. Create a new [release notes file](https://github.com/NixOS/nixpkgs/commit/01268fda85b7eee4e462c873d8654f975067731f#diff-e7ee5ff686cdcc513ca089d6e5682587R11)
   for the next release

1. Update the release versions in [`PULL_REQUEST_TEMPLATE.md`](https://github.com/NixOS/nixpkgs/blob/master/.github/PULL_REQUEST_TEMPLATE.md)
   on master.

   Examples: [22.11](https://github.com/NixOS/nixpkgs/commit/f1b9cc23aa8b1549dd7cb53dbe9fc950efc97646#diff-18813c86948efc57e661623d7ba48ff94325c9b5421ec9177f724922dd553a35)

1. Update [`CONTRIBUTING.md`](https://github.com/NixOS/nixpkgs/blob/master/CONTRIBUTING.md) on master

   Examples: [22.11](https://github.com/NixOS/nixpkgs/commit/f1b9cc23aa8b1549dd7cb53dbe9fc950efc97646#diff-eca12c0a30e25b4b46522ebf89465a03ba72a03f540796c979137931d8f92055)

1. Commit the changes ([22.05 example](https://github.com/NixOS/nixpkgs/commit/bfdfe12c788d7474b88e7a7790b88b1c0f8e01b5) + [this additional commit](https://github.com/NixOS/nixpkgs/commit/953b5d19bca4d4ddfaef5625cca277c47b39f5e7))

1. Tag the master branch, so that `git describe` shows the new version as the base for commits..

   ```bash
   git tag --annotate $NEWVER-pre
   git push upstream master $NEWVER-pre
   git describe HEAD # should yield 23.05-pre
   ```

### And afterwards

Now that everything on git is done, we are still missing the channels.

1. Create the necessary channels for `https://channels.nixos.org` in beta status, by updating
   [`channels.nix`](https://github.com/NixOS/nixos-org-configurations/blob/master/channels.nix) in `nixos-org-configurations`
   and nag the infrastructure team to get these changes deployed.

   Example: [22.11](https://github.com/NixOS/nixos-org-configurations/commit/9a0b3674a11b445c973334c78e8ca0eda36775e4)

1. Create the backport [labels](https://github.com/NixOS/nixpkgs/labels) for all new branches:
   - `backport staging-21.05`
   - `backport release-21.05`

   Use the description `Backport PR automatically` and the color value `#0fafaa`
   
1. Update the ZHF issue, that now that the branch-off has been performed, fixes have to be backported.
   Examples: [22.05](https://github.com/NixOS/nixpkgs/issues/172160#issuecomment-1135112918)

### Once the channel is available

The following steps should be done after the channels have become available on [channels.nixos.org](https://channels.nixos.org).

1. Update the flake input on the `nixos-search` repository` and create a pull request:

   ```bash
   git clone git@github.com:nixos/nixos-search
   nix --extra-experimental-features "nix-command flakes" flake lock --update-input nixos-org-configurations
   ```

1. Give the [Marketing team](https://matrix.to/#/#marketing:nixos.org) a heads-up about the upcoming release

1. Get in contact with [Amine Chikhaoui](https://github.com/AmineChikhaoui) on the infrastructure room on Matrix, so
   they can get the AMIs updated in time for the release.

   1. They will need to run [`create-amis.sh`](https://github.com/NixOS/nixpkgs/blob/master/nixos/maintainers/scripts/ec2/create-amis.sh),
      which requires write permissions to the S3 bucket.

   1. Create PR adding it to NixOS configuration. Examples:
      - [22.11](https://github.com/NixOS/nixpkgs/pull/204014)
      - [20.09](https://github.com/NixOS/nixpkgs/pull/101720)

1. Make sure the release editors have started finalizing the release notes. Only 7 days left until release!
