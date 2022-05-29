# Release Process Walkthrough

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
