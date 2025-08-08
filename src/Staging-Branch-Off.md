# Staging Branch-off

**Requirement:** The last `staging-next` cycle before the release started and `staging` is now unrestricted.

For these steps "24.05" represents the current release tag (the version you want to release).

## Git branches

Set NEWVER to the new release version:

```bash
export NEWVER=24.05
```

Now create the branches:

1. Fetch and check out the `staging-next` branch.

1. Create the actual branches
    ```bash
    git branch staging-$NEWVER
    git branch staging-next-$NEWVER
    ```

1. Push the changes

    ```bash
    git push upstream staging-$NEWVER staging-next-$NEWVER
    ```

## Incorporate new branches

We now incorporate the new branches into the workflows. The suggestion is to do these changes via a PR.

1. Create new [NixOS and Nixpkgs release notes file](https://github.com/NixOS/nixpkgs/commit/95cc97659c813256dc17d2905904144c1588e6ad)

1. Update [periodic-merge workflow](https://github.com/NixOS/nixpkgs/commit/86de96dbc1a8e65bc5f02b98d708315c0ec0fc02) so that
    - master gets merged into staging-next-24.05
    - staging-next-24.05 gets merged into staging-24.05

1. Create the backport [label](https://github.com/NixOS/nixpkgs/labels) for the new staging branch:
   - `backport staging-24.05`

   Use the description `Backport PR automatically` and the color value `#0fafaa`
