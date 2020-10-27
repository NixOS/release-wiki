# Release Process Walkthrough

Going through an example of releasing NixOS 19.09:

## One month before the beta

- Create an announcement on [Discourse](https://discourse.nixos.org)
  as a warning about upcoming beta “feature freeze” in a month. [See
  this post as an
  example](https://discourse.nixos.org/t/nixos-19-09-feature-freeze/3707).

- Discuss with Eelco Dolstra and the community (via IRC, ML) about
  what will reach the deadline. Any issue or Pull Request targeting
  the release should be included in the release milestone.

- Remove attributes that we know we will not be able to support,
  especially if there is a stable alternative. E.g. Check that our
  Linux kernels’ [projected
  end-of-life](https://www.kernel.org/category/releases.html) are
  after our release projected end-of-life.

## At beta release time

1.  From the master branch run:

```sh
git checkout -b release-19.09
```

2.  [Bump the `system.defaultChannel` attribute in
    `nixos/modules/misc/version.nix`](https://github.com/NixOS/nixpkgs/commit/10e61bf5be57736035ec7a804cb0bf3d083bf2cf#diff-9c798092bac0caeb5c52d509be0ca263R69)

3.  [Update `versionSuffix` in
    `nixos/release.nix`](https://github.com/NixOS/nixpkgs/commit/10e61bf5be57736035ec7a804cb0bf3d083bf2cf#diff-831e8d9748240fb23e6734fdc2a6d16eR15)

To get the commit count, use the following command:

    git rev-list --count release-19.09

1.  Edit changelog at `nixos/doc/manual/release-notes/rl-1909.xml`.

    - Get all new NixOS modules:

      ```sh
      git diff release-19.03..release-19.09 nixos/modules/module-list.nix | grep ^+
      ```

    - Note systemd, kernel, glibc, desktop environment, and Nix
      upgrades.

2.  Tag the release:

```sh
git tag --annotate --message="Release 19.09-beta" 19.09-beta
git push upstream 19.09-beta
```

3.  [On the `master` branch, increment the `.version`
    file](https://github.com/NixOS/nixpkgs/commit/01268fda85b7eee4e462c873d8654f975067731f#diff-2bc0e46110b507d6d5a344264ef15adaR1)

```sh
echo -n "20.03" > .version
```

4.  [Update `codeName` in
    `lib/trivial.nix`](https://github.com/NixOS/nixpkgs/commit/01268fda85b7eee4e462c873d8654f975067731f#diff-03f3d41b68f62079c55001f1a1c55c1dR137)
    This will be the name for the next release.

5.  [Create a new release notes file for the upcoming release +
     1](https://github.com/NixOS/nixpkgs/commit/01268fda85b7eee4e462c873d8654f975067731f#diff-e7ee5ff686cdcc513ca089d6e5682587R11),
    in our case this is `rl-2003.xml`.

6.  Contact the infrastructure team to create the necessary Hydra
    Jobsets.

7.  [Create a channel at https://nixos.org/channels by creating a PR to
    nixos-org-configurations, changing`channels.nix`](https://github.com/NixOS/nixos-org-configurations/blob/master/channels.nix)

8.  Get all Hydra jobsets for the release to have their first
    evaluation.

9.  [Create an issue for tracking Zero Hydra Failures progress.
    ZHF is an effort to get build failures down to zero.](https://github.com/NixOS/nixpkgs/issues/13559)

## During Beta

- Monitor the master branch for bugfixes and minor updates and
  cherry-pick them to the release branch.

## Before the final release

- Re-check that the release notes are complete.

- Release Nix (currently only Eelco Dolstra can do that). [Make sure fallback is updated.](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/installer/tools/nix-fallback-paths.nix)

- [Update README.md with new stable NixOS version information.](https://github.com/NixOS/nixpkgs/commit/40fd9ae3ac8048758abdcfc7d28a78b5f22fe97e)

- Change `stableBranch` to `true` in Hydra and wait for the channel to
  update.

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

    1.  [Update `NIXOS_SERIES` in the `Makefile`](https://github.com/NixOS/nixos-homepage/blob/47ac3571c4d71e841fd4e6c6e1872e762b9c4942/Makefile#L1).

    2.  [Update `nixos-release.tt` with the new NixOS version](https://github.com/NixOS/nixos-homepage/blob/47ac3571c4d71e841fd4e6c6e1872e762b9c4942/nixos-release.tt#L1).

    3.  [Update the `flake.nix` input `released-nixpkgs` to 19.09](https://github.com/NixOS/nixos-homepage/blob/47ac3571c4d71e841fd4e6c6e1872e762b9c4942/flake.nix#L10).

    4.  Run `./update.sh` (this updates flake.lock to updated channel).

    5.  [Add a compressed version of the NixOS logo for 19.09](https://github.com/NixOS/nixos-homepage/blob/a5626c71c03a2dd69086564e56f1a230a2bb177a/logo/nixos-logo-19.09-loris-lores.png).

    6.  [Compose a news item for the website RSSfeed](https://github.com/NixOS/nixos-homepage/commit/a5626c71c03a2dd69086564e56f1a230a2bb177a#diff-9cdc6434d3e4fd93a6e5bb0a531a7c71R5).

5.  Create a new topic on [the Discourse instance](https://discourse.nixos.org/) to announce the release.

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
