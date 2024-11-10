# Final Release

Set the following environment variables to execute the commands below

```bash
# The version of the previous stable release
export OLDVER=22.11
# The version that is about to be released
export NEWVER=23.05
```

## Before the final release

- Two days before expected release date, change `stableBranch` to `true` in Hydra and wait for the channel to update

- Re-check that the release editors have completed their work on the release notes

- Merge the Pull Request that marks broken packages as broken

- Update [README.md](https://github.com/NixOS/nixpkgs/commit/40fd9ae3ac8048758abdcfc7d28a78b5f22fe97e) on master with
  new stable NixOS version information. Then backport the change to the release branch.

- Update [Upgrading NixOS](https://nixos.org/manual/nixos/stable/index.html#sec-upgrading) section of the manual to
  match new stable release version and backport this as well. This way the manual will already reflect the new version
  on release. Example: [22.05](https://github.com/nixos/nixpkgs/commit/cbaacfb8dfa2ddadfb152fa8ef163b40db9041af)

- Ensure the installer images are sufficiently tested on both aarch64 and x86_64 linux:
  - Graphical (GNOME/KDE)
  - Minimal

  All ISO images can be found on the `nixos/release-$NEWVER` hydra jobset.

  If you don't have any hardware to test the aarch64-linux on, [QEMU
  emulation](QEMU-aarch64.md) should allow to boot the images in a
  reasonable time to check the installer starts and is usable.

- Gather some information about the release for the final announcement

  - Number of non-merge commits for the release:

    ```bash
    git log upstream/release-$OLDVER..upstream/release-$NEWVER --no-merges --format=%an | wc -l
    ```

  - Number of contributors for this release:

    ```bash
    git log upstream/release-$OLDVER..upstream/release-$NEWVER --no-merges --format=%an | sort | uniq | wc -l
    ````

  - New/updated/removed packages:

    ```bash
    git switch release-$OLDVER
    NIX_PATH= nix-env -f $PWD -qaP --json --out-path > old.json

    git switch release-$NEWVER
    NIX_PATH= nix-env -f $PWD -qaP --json --out-path > new.json
    ```

    ```python
    #!/usr/bin/env python3

    import json

    with open('old.json') as f:
        old = json.load(f)
    with open('new.json') as f:
        new = json.load(f)

    n_removed = 0
    n_new = 0
    n_updated = 0
    for pkgname,pkg in new.items():
        if pkgname not in old:
            n_new += 1
            continue
        if pkg["version"] != old[pkgname]["version"]:
            n_updated += 1
    for pkgname in old.keys():
        if pkgname not in new:
            n_removed += 1

    print(f"New: {n_new}")
    print(f"Rem: {n_removed}")
    print(f"Upd: {n_updated}")
    ```

    Best to check how the previous announcement post was formulated to see what needs to
    be included.

  - Added/removed modules:

    ```bash
    git diff release-$OLDVER..release-$NEWVER nixos/modules/module-list.nix | grep ^+ | wc -l
    git diff release-$OLDVER..release-$NEWVER nixos/modules/module-list.nix | grep ^- | wc -l
    ```

  - Added/removed options:

    Download the result of the `nixos.options` job in Hydra from both your release and the previous release, then do:

    ```bash
    # removed options:
    comm -23 <(jq -r 'keys[]' old.json | sort) <(jq -r 'keys[]' new.json | sort) | wc -l
    # new options:
    comm -13 <(jq -r 'keys[]' old.json | sort) <(jq -r 'keys[]' new.json | sort) | wc -l
    ```

## At final release time

### On the master branch

1. Create these PRs on master and backport to the release branch:

   1. Update `rl-$NEWVER.section.md` with the final release date.

   1. Commit and push all changes.
   
      ```bash
      git commit -m "Release NixOS $NEWVER" -S
      git push upstream master
      ```

### On the release branch

1. Switch to the release branch

   ```bash
   git switch release-$NEWVER
   ```

1. Cherry-pick the release commit from master

1. Push the commit to the release branch

   ```bash
   git push
   ````

1. Find the commit id and tag the release **on the release branch**:

   ```bash
   git tag --annotate --message="Release $NEWVER" "branch-off-$NEWVER" <COMMIT_ID>
   git push upstream "branch-off-$NEWVER"
   ```

1. Update [nixos-homepage](https://github.com/NixOS/nixos-homepage) for the release.

   Examples: [24.05](https://github.com/NixOS/nixos-homepage/pull/1453)

   This step requires the released ISOs, from the hydra evaluation with `stableBranch` enabled, to be available.

   1. Update the [`flake.nix`](https://github.com/NixOS/nixos-homepage/blob/5587a754b21aa34310cb1752ddc03da232e2e9d2/flake.nix#L11)
      input `released-nixpkgs-stable` to $NEWVER and then run:

   1. Run `nix flake update` (this updates flake.lock to updated channel).

   1. Add a compressed version of the NixOS logo ([24.05 example](https://github.com/NixOS/nixos-homepage/blob/e17aa94fff8f808ab6331d77b40a787e06e866b5/src/assets/logo/nixos-logo-24.05-uakari-lores.png)). The logo should have a width of 100px.

   1. Write the announcement under `src/content/blog/announcements/` ([24.05 example](https://github.com/NixOS/nixos-homepage/blob/e43b7f46a3c3b924844799fac48347e1813ec457/src/content/blog/announcements/2024-05-31_nixos-24.05.md)).

   1. Create a pull request and poke the marking team. Make sure to
      push to a branch on the `nixos-homepage` repository, not your
      own fork, so the CI actions work properly.

   The website is hosted by a CDN so you may occasionally see the old site for a couple of minutes/hours (?) after your changes

1. Update [infra](https://github.com/NixOS/infra) to reflect the `stable` channel
   status for $NEWVER and the `deprecated` status for $OLDVER.
   
   Examples: [22.11](https://github.com/NixOS/infra/pull/228), [22.05](https://github.com/NixOS/infra/pull/210), [21.11](https://github.com/NixOS/infra/pull/192)

1. Create a new topic on [Discourse](https://discourse.nixos.org/) to announce the release.

   Examples: [22.11](https://discourse.nixos.org/t/nixos-22-11-released/23637), [22.05](https://discourse.nixos.org/t/nixos-22-05-released/19404)

1. Once the Pull Request in `infra` is merged, update [nixos-search](https://github.com/NixOS/nixos-search/) to mark the channel as released.
   This is the same process as for the creation of the beta channel in the project.

## After Release

1. Add the new release to repology. Don't remove old releases, they want to keep them around.

   Examples: [22.11](https://github.com/repology/repology-updater/pull/1289), [22.05](https://github.com/repology/repology-updater/pull/1156)

1. Update osinfo-db entries.

   Examples: [22.11](https://gitlab.com/libosinfo/osinfo-db/-/merge_requests/539), [22.05](https://gitlab.com/libosinfo/osinfo-db/-/merge_requests/457), [21.11](https://gitlab.com/libosinfo/osinfo-db/-/merge_requests/385)

   ```bash
   nix-shell -p "python3.withPackages (p: [ p.lxml p.requests ])" -p cdrkit
   ./scripts/updates/nixos.py --release YY.MM --codename <codename> --release-date YYYY-MM-DD --next-release YY.MM
   git add .
   git commit -s -m "nixos: Add YY.MM release"

   # run tests to validate your changes
   nix-shell -p "python3.withPackages (p: [ p.lxml p.requests p.pytest ])" -p cdrkit osinfo-db-tools gettext --run "make check"
   ```

1. Check that the AMI image references have been updated in `nixos/modules/virtualisation/amazon-ec2-amis.nix`.

1. Close the [milestone](https://github.com/NixOS/nixpkgs/milestones)

1. Close the [blockers project](https://github.com/orgs/NixOS/projects)

1. Close all issues you opened for the release and unpin them if necessary

1. Ask someone with the appropriate permissions to update the description of the NixOS and NixOS Dev Matrix rooms to include the new release

1. Create an announcement for the retrospective, some days after the old channel is discontinued.
 
   Examples: [22.11](https://discourse.nixos.org/t/nixos-22-11-retrospective/23858), [22.05](https://discourse.nixos.org/t/22-05-retrospective/19413)
