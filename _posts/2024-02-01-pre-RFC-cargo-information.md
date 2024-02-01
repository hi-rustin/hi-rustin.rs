---
title: 'pre-RFC: `cargo-info` for everyone'
layout: post

categories: post
tags:
- Rust
- Cargo
- CLI
---

# Summary

This adds a new subcommand to Cargo, `cargo info`. This subcommand would allow users to get information about a crate from the command line, without having to go to the web.

Example usage:

```sh
cargo info clap
    Updating crates.io index
clap #argument #cli #arg #parser #parse
A simple to use, efficient, and full-featured Command Line Argument Parser
version: 4.4.18
license: MIT OR Apache-2.0
rust-version: 1.70.0
documentation: https://docs.rs/clap/4.4.18
repository: https://github.com/clap-rs/clap
features:
  default         = [std, color, help, usage, error-context, suggestions]
  std             = [clap_builder/std]
  color           = [clap_builder/color]
  help            = [clap_builder/help]
  usage           = [clap_builder/usage]
  error-context   = [clap_builder/error-context]
  suggestions     = [clap_builder/suggestions]
  cargo           = [clap_builder/cargo]
  debug           = [clap_builder/debug, clap_derive?/debug]
  deprecated      = [clap_builder/deprecated, clap_derive?/deprecated]
  derive          = [dep:clap_derive]
  env             = [clap_builder/env]
  string          = [clap_builder/string]
  unicode         = [clap_builder/unicode]
  unstable-doc    = [clap_builder/unstable-doc, derive]
  unstable-styles = [clap_builder/unstable-styles]
  unstable-v5     = [clap_builder/unstable-v5, clap_derive?/unstable-v5, deprecated]
  wrap_help       = [clap_builder/wrap_help]
dependencies:
  clap_builder@=4.4.18
  clap_derive@=4.4.7
owners:
  kbknapp (Kevin K.)
  github:rust-cli:maintainers (Maintainers)
  github:clap-rs:admins (Admins)
```

# Motivation

The main motivation for this is to make it easier to get information about a crate from the command line. Currently, the way to get information about a crate is to go to the web and look it up on [crates.io] or find the crate's source code and look at the `Cargo.toml` file. This is not very convenient, especially not all information is displayed on the [crates.io] page.
This command also has been requested by the community for a long time. You can find more discussion about this in [cargo#948].

Another motivation is to make the workflow of finding and evaluating crates more efficient. In the current workflow, users can search for crates using `cargo search`, but then they have to go to the web to get more information about the crate. This is not very efficient, especially if the user is just trying to get a quick overview of the crate. This would allow users to quickly get information about a crate without having to leave the terminal.

[crates.io]: https://crates.io
[cargo#948]: https://github.com/rust-lang/cargo/issues/948

# Guide-level explanation

## Inspect a crate from crates.io outside of the workspace

Users can run `cargo info` command to get information in any directory. This command will fetch the information from the crates.io index and display it in the terminal.

```sh
cargo info home
    Updating crates.io index
home
Shared definitions of home directories.
version: 0.5.9
license: MIT OR Apache-2.0
rust-version: 1.70.0
documentation: https://docs.rs/home
repository: https://github.com/rust-lang/cargo
dependencies:
  windows-sys@0.52
owners:
  alexcrichton (Alex Crichton)
  brson (Brian Anderson)
  LucioFranco (Lucio Franco)
  ehuss (Eric Huss)
  kinnison (Daniel Silverstone)
  rust-lang-owner
```

It will display the name, description, version, license, rust version, documentation, repository, and dependencies of the crate. It will also display the owners of the crate if available.

## Inspect a crate from crates.io inside of the workspace

Users can run `cargo info` command to get information in a workspace directory. This command will fetch the information from the crates.io index and display it in the terminal. But it will pick the version used in the workspace.

Let's run the same command in the local [rustup repository]. Rustup uses the `home` crate as a dependency.

```sh
cargo info home
    Updating crates.io index
home
Shared definitions of home directories.
version: 0.5.9
license: MIT OR Apache-2.0
rust-version: 1.70.0
documentation: https://docs.rs/home
repository: https://github.com/rust-lang/cargo
dependencies:
  windows-sys@0.52
owners:
  alexcrichton (Alex Crichton)
  brson (Brian Anderson)
  LucioFranco (Lucio Franco)
  ehuss (Eric Huss)
  kinnison (Daniel Silverstone)
  rust-lang-owner
note: to see how you depend on home, run `cargo tree --invert --package home@0.5.9`
```

As you can see, it displays the same information as the previous example, but it also displays a note to see how you depend on the crate.

## Inspect a local crate

Users can run `cargo info` command to get information about a local crate. This command will display the information from the local `Cargo.toml` file.

Let's run the same command the local [cargo repository]. It manages the `home` crate.

```sh
cargo info home
home
Shared definitions of home directories.
version: 0.5.11 (from ./crates/home)
license: MIT OR Apache-2.0
rust-version: 1.73
documentation: https://docs.rs/home
homepage: https://github.com/rust-lang/cargo
repository: https://github.com/rust-lang/cargo
dependencies:
  windows-sys@0.52
```

As you can see, it inspects the local crate and displays the information from the `Cargo.toml` file. Because we get the information from the `Cargo.toml` file, it doesn't display the owners of the crate.

# Reference-level explanation

# Prior art

# Unresolved questions
