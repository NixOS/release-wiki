# Feature Freeze announcement and pre-release cleanup

The feature freeze announcement should be done 8 weeks before the release to
inform subsystem maintainers about the upcoming feature freeze and to discuss
potential blockers in the respective subsystems. This announcement is done on
GitHub ([22.05
example](https://github.com/NixOS/nixpkgs/issues/167025)). It's also a good
idea to cross-post this to Discourse to raise awareness about the upcoming
freeze ([22.05
example](https://discourse.nixos.org/t/22-05-feature-freeze/18453)).

People and teams will be pinged in this issue. The list of people and teams
pinged is generated from the teams list in nixpkgs. The script to generate
the pings is: `./maintainers/feature-freeze-teams.pl` in nixpkgs. The script
needs a GitHub username and a token to read the teams from the NixOS org.
Instructions are provided as a comment in the script.

You might need to add a comment that pings people again because GitHub seems
to limit the amount of people that can be pinged per post.

## Pre-release cleanup

The pre-release cleanup consists of multiple small cleanups to keep the release
secure throughout its lifecycle while also maintaining some level of
cleanliness.

- Remove attributes that we know we will not be able to support,
  especially if there is a stable alternative. E.g. Check that our
  Linux kernels’ [projected
  end-of-life](https://www.kernel.org/category/releases.html) are
  after our release projected end-of-life. Also check these package sets:
	- MariaDB
	- PHP
	- PostgreSQL

- Attributes that can also be safely removed are packages that were broken for
  more than 2 years.

- Remove old aliases that have been `throw`s for at least one release. There is
  a script in the nixpkgs tree to do that:
  `maintainers/scripts/remove-old-aliases.py --file
  ./pkgs/top-level/aliases.nix --year 2019 --month 6`
  The conversion to `throw`s is usually done in one big Pull Request so you can
  remove all aliases from the date prior to this Pull Request. [Example of the
  2022 throw conversion](https://github.com/NixOS/nixpkgs/pull/161146).
