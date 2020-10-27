# NixOS Release Wiki
Home for the NixOS Release Wiki.
This is deployed to GitHub pages at https://nixos.github.io/release-wiki/.

# Contributing
This project is built with [mdbook](https://github.com/rust-lang/mdBook)

You can built the documentation with
```bash
mdbook build
```

or have it get rebuild when you make edits with:
```bash
mdbook watch
```

and view the `book/index.html`.

You can also serve it over `localhost` with:
```bash
mdbook serve
```

and open http://localhost:3000. The serve command functions in the same way as `watch`.


Also see the [mdbook User Guide](https://rust-lang.github.io/mdBook/) for a more exhaustive
walkthrough.
