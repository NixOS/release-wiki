# Release Critical Packages

These packages are deemed "sufficiently complex" in the package ecosystem to
warrant treating them differently when stabilizing the package set leading up to
a stable release. Generally, theses packages will be closely related to `stdenv`
or fundamental to NixOS (e.g. `systemd`). The implications of 
[RFC 0085](https://github.com/NixOS/rfcs/blob/master/rfcs/0085-nixos-release-stablization.md)
are that these packages can not have breaking changes applied to them for six weeks
leading up to a release date. This delay allows for changes in these packages to be
sufficiently stabilized through usage on unstable, and allows for many staging
iterations for these fixes to be applied.

## List of Release Critical Packages:
- `binutils`
- `gcc`
- `glibc`
- `llvm`
- `systemd`

### Process for Modifying Release Critical Packages

A release retrospective will take place after a release occured; one of the
topics of discussion will be to modify the list of critical packages.
Packages may be added if they caused a significant amount of pain, or packages
may be removed if their updates cause little to no pain. Pain in this case
is defined as causing failing builds, regressions in downstream packages,
or regressions in user experience.
