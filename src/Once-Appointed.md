# What to do once appointed

You have been appointed as the new Release Manager? Congratulations!
This is a loose list of things you could start doing now.
These tasks are not urgent but can be done as early as the appointment which should start to get you into your new role.
While not urgent, these task need to be completed at some point before the release process starts.
The release manager of the previous release will get in contact with you and help you along the way.

This is not an actual part of the release process but rather preparations for starting the process.

- Get in contact with the previous release manager and schedule a meeting
- Join the [release management room](https://matrix.to/#/#nixos-release-management:nixos.org) on Matrix
	- Request moderator permissions on the room, this allows to manage the topic and create video calls
- Gain access to the [release-manager GitHub Team](https://github.com/orgs/NixOS/teams/nixos-release-managers/members) by asking any of the existing people with "Manager" rights
	- Also make sure there are only people left as members who are managing this release
- Ensure you have a [Hydra](https://hydra.nixos.org/) account which will come in handy
	- Ensure that all release managers have at least `restart-jobs`, `bump-to-front`, and `cancel-build` privileges (see the top-right "Preferences" button). If they are missing, reach out to the Infrastructure team
- Create a blockers [project](https://github.com/orgs/NixOS/projects) in the organization for your release and the next one
	- Also close old ones while you're at it and consider migrating remaining open issues to the current project
- Figure out a codename for the next release (not the one you are releasing but the next one) and write it down somewhere for the branch-off
- Find someone to create a [logo](https://github.com/NixOS/nixos-artwork/tree/master/releases) or create one yourself for the current release and PR it into the repository
	- Usually done by creating an issue on the `nixos-artwork` repository pinging artists who created previous logos
- Check if the process can be improved this release
	- Find comments on the Discourse posts, GitHub issues, … of the previous release and see if the comments suggest doing something different this time
	- Read the Discourse post of the last retrospective
	- Check the Issues and Pull Requests of the [release-wiki repository](https://github.com/NixOS/release-wiki)
- Figure out concrete dates for the week offsets outlined in schedule
- Publish a "Let's have a great XX.XX release cycle!" post on Discourse, outlining the plans for this release. [Example of 22.05](https://discourse.nixos.org/t/lets-have-a-great-22-05-release-cycle/18357)
	- This links to the release schedule issue ([22.05 example](https://github.com/NixOS/nixpkgs/issues/165792)) which must be created before posting to Discourse
