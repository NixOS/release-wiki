# Release Managers

> When you are nixpkgs committer, you already have the skillset necessary for release management.
> You just need to apply it in a different way.
>
> — Jon Ringer

For each release there are two release managers. After each release the
release manager having managed two releases steps down and the release
management team of the last release appoints a new release manager.

This makes sure a release management team always consists of one release
manager who already has managed one release and one release manager
being introduced to their role, making it easier to pass on knowledge
and experience.

Release managers for the current NixOS release are tracked by GitHub
team
[`@NixOS/nixos-release-managers`](https://github.com/orgs/NixOS/teams/nixos-release-managers/members).

A release manager’s role and responsibilities are:

-   manage the release process

-   start discussions about features and changes for a given release

-   create a roadmap

-   decide which bug fixes, features, etc… get backported after a
    release

This team has special write permissions to certain repositories in NixOS.
It is **very** important that the new co-release-manager removes the previous manager
from the team when they assign a new one.

A release manager must have the commit bit in the nixpkgs repository.
