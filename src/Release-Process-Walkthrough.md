# Release Process Walkthrough

## At final release time
Create these PRs on master and backport to release-19.09.

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
git log upstream/release-19.03..upstream/release-19.09 --format=%an | wc -l
```

- Commits by contributor:

```sh
git shortlog --summary --numbered release-19.03..release-19.09
```

Best to check how the previous post was formulated to see what needs to
be included.

7. Update [nixos-org-configurations](https://github.com/NixOS/nixos-org-configurations/pull/192/files) with updated channel status

  - previous stable branches should be marked "deprecated"
  - beta branches should be marked "stable"

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

## A Month after Release

1. [EOL Channels should be marked unmaintained](https://github.com/NixOS/nixos-org-configurations/pull/201)
2. Increase the `oldestSupportedRelease` in `lib/trivial.nix` to match the oldest non-EOL release.
