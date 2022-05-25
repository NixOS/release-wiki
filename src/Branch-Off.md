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

1. Wait for the staging team to merge the final staging-next iteration
   ```shell
   git fetch upstream staging-next
   git merge upstream/staging-next
   ```

1. Fetch and check out the master branch

1. Update the [periodic-merge workflow](https://github.com/NixOS/nixpkgs/blob/master/.github/workflows/periodic-merge-24h.yml) to include the new branch, then commit and push to master

1. Create the release branch:

   ```shell
   git switch -c release-21.05
   ```

1. [Bump the `system.defaultChannel` attribute in
   `nixos/modules/misc/version.nix`](https://github.com/NixOS/nixpkgs/commit/10e61bf5be57736035ec7a804cb0bf3d083bf2cf#diff-b3379a98640b35a5fe4b046150cd2df1639995edf231d18bbad832be6a70b45f)

1. [Update `versionSuffix` in
   `nixos/release.nix`](https://github.com/NixOS/nixpkgs/commit/10e61bf5be57736035ec7a804cb0bf3d083bf2cf#diff-20da30ee012d7d87842fb7953237870493c5497c995cba1e6f6c3aa9268398ff)

   To get the commit count, use the following command:

   ```shell
   git rev-list --count release-21.05
   ```

1. Commit the changes from the previous steps

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

1. Switch back to the master branch and [increment the `.version`
   file](https://github.com/NixOS/nixpkgs/commit/01268fda85b7eee4e462c873d8654f975067731f#diff-2bc0e46110b507d6d5a344264ef15adaR1)

   ```sh
   git switch master
   echo -n "21.11" > .version
   ```

1. [Update `codeName` in
   `lib/trivial.nix`](https://github.com/NixOS/nixpkgs/commit/01268fda85b7eee4e462c873d8654f975067731f#diff-03f3d41b68f62079c55001f1a1c55c1dR137)
  This will be the name for the next release.

1. [Create a new release notes file for the upcoming release +
   1](https://github.com/NixOS/nixpkgs/commit/01268fda85b7eee4e462c873d8654f975067731f#diff-e7ee5ff686cdcc513ca089d6e5682587R11),
  in our case this is `rl-2111.section.md`. Don't forget to link the new file in the parent markdown file and to run `md-to-db.sh`

1. Commit the changes ([22.05 example](https://github.com/NixOS/nixpkgs/commit/bfdfe12c788d7474b88e7a7790b88b1c0f8e01b5) + [this additional commit](https://github.com/NixOS/nixpkgs/commit/953b5d19bca4d4ddfaef5625cca277c47b39f5e7))

1. Tag master branch so that `git describe` shows the correct metadata.
   ```shell
   git tag --annotate 21.11-pre
   git push upstream master 21.11-pre
   git describe HEAD # should yield 21.11-pre
   ```

1. Trigger an evaluation of the new jobsets in Hydra

1. Create a channel at `https://nixos.org/channels` by creating a PR to
   `nixos-org-configurations`. Update [`channels.nix`](https://github.com/NixOS/nixos-org-configurations/blob/master/channels.nix) to include the channel as a beta channel and notify the infrastructure team. [22.05 example](https://github.com/NixOS/nixos-org-configurations/pull/209)

1. Create the backport [labels](https://github.com/NixOS/nixpkgs/labels) for all new branches:
   - `backport staging-21.05`
   - `backport staging-21.05`

   Use the description `Backport PR automatically` and the color value `#0fafaa`

1. Update the flake input of nixos-search once the Pull Request in `nixos-org-configurations` is merged:
   - Clone [nixos-search](https://github.com/NixOS/nixos-search)
   - Update the flake inputs:
     ```shell
     nix --extra-experimental-features nix-command\ flakes flake update
     ```
   - Commit the result and create a Pull Request

1. Inform the [Marketing team](https://matrix.to/#/#marketing:nixos.org) about the upcoming release and the `nixos-serach` Pull Request

1. Comment onto the ZHF issue that the branch-off has been performed and issues now have to be backported ([22.05 example](https://github.com/NixOS/nixpkgs/issues/172160#issuecomment-1135112918))
