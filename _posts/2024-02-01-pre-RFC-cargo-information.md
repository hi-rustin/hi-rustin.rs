---
title: 'pre-RFC: cargo-info for everyone'
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

```console
$ cargo info clap
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

## Detailed design

| Content                                                                    | Explanation                         | Why                                                                               |
|----------------------------------------------------------------------------|-------------------------------------|-----------------------------------------------------------------------------------|
| clap                                                                       | Name                                | The basic information.                                                            |
| #argument #cli #arg #parser #parse                                         | Keywords                            | It's more like a category, which you can use to search for relevant alternatives. |
| A simple to use, efficient, and full-featured Command Line Argument Parser | Description                         | The basic information.                                                            |
| version: 4.4.18                                                            | Version                             | The basic information.                                                            |
| license: MIT OR Apache-2.0                                                 | License                             | When choosing a crate, it is crucial to consider the license.                     |
| rust-version: 1.70.0                                                       | MSRV                                | When choosing a crate, it is crucial to make sure it can work with your MSRV.     |
| documentation: <https://docs.rs/clap/4.4.18>                               | Documentation Link                  | Use these links can find more docs and information.                               |
| repository: <https://github.com/clap-rs/clap>                              | Repo Link                           | Use these links can find more docs and information.                               |
| features:                                                                  | Default Features And Other Features | It helps for enabling features.                                                   |
| dependencies:                                                              | All dependencies                    | It indicates what it depends on.                                                  |
| owners:                                                                    | Owners                              | It indicates who maintains the crate.                                             |

# Motivation

The main motivation for this is to make it easier to get information about a crate from the command line. Currently, the way to get information about a crate is to go to the web and look it up on [crates.io] or find the crate's source code and look at the `Cargo.toml` file. This is not very convenient, especially not all information is displayed on the [crates.io] page.
This command also has been requested by the community for a long time. You can find more discussion about this in [cargo#948].

Another motivation is to make the workflow of finding and evaluating crates more efficient. In the current workflow, users can search for crates using `cargo search`, but then they have to go to the web to get more information about the crate. This is not very efficient, especially if the user is just trying to get a quick overview of the crate. This would allow users to quickly get information about a crate without having to leave the terminal.

[crates.io]: https://crates.io
[cargo#948]: https://github.com/rust-lang/cargo/issues/948

# Guide-level explanation

## Inspect a crate from crates.io outside of the workspace

Users can utilize the `cargo` info command to retrieve details in any directory. This command extracts information from crates.io and presents it in the terminal.

For example, let's run the `cargo info` command for the `home` crate.

```console
$ cargo info home
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

In a workspace directory, the `cargo info` command can also be used to gather details. While it retrieves information from crates.io for display in the terminal, it specifically selects the version used in the workspace.

Let's run the same command in the local [rustup repository]. Rustup uses the `home` crate as a dependency.

```console
$ cargo info home
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

[rustup repository]: https://github.com/rust-lang/rustup

## Inspect a local crate

To obtain information about a local crate, users can execute the `cargo info` command. This command will showcase details from the local Cargo.toml file.

Let's run the same command the local [cargo repository]. It manages the `home` crate.

```console
$ cargo info home
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

[cargo repository]: https://github.com/rust-lang/cargo

## Inspect a specific version of a crate

`cargo info` supports inspecting a specific version of a crate. It uses the same package ID specification as `cargo install` and other Cargo commands. You can find more information about the package ID specification in the [Cargo documentation].

Let's run the `cargo info` command for the `home` crate with the version `0.5.9`.

```console
$ cargo info home@0.5.9
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

[Cargo documentation]: https://doc.rust-lang.org/cargo/reference/pkgid-spec.html

# Reference-level explanation

## Downloading the crate from any Cargo compatible registry

The `cargo info` command will download the crate from any Cargo compatible registry. It will then extract the information from the `Cargo.toml` file and display it in the terminal.

If the crate is already in the local cache, it will not download the crate again. It will get the information from the local cache.

## Pick the correct version from the workspace

When executed in a workspace directory, the cargo info command chooses the version that the workspace is currently using.

If there's a lock file available, the version from this file will be used. In the absence of a lock file, the command attempts to select a version that is compatible with the Minimum Supported Rust Version (MSRV). And the lock file will be generated automatically.

The following hierarchy is used to determine the MSRV:

- First, the MSRV of the parent directory package is checked, if it exists.
- If the parent directory package does not specify an MSRV, the minimal MSRV of the workspace is checked.
- If neither the workspace nor the parent directory package specify an MSRV, the version of the current Rust compiler (rustc --version) is used.

# Prior art

## NPM

[npm] has a similar command called `npm info`.
For example:

```console
$ npm info lodash

lodash@4.17.21 | MIT | deps: none | versions: 114
Lodash modular utilities.
https://lodash.com/

keywords: modules, stdlib, util

dist
.tarball: https://registry.npmjs.org/lodash/-/lodash-4.17.21.tgz
.shasum: 679591c564c3bffaae8454cf0b3df370c3d6911c
.integrity: sha512-v2kDEe57lecTulaDIuNTPy3Ry4gLGJ6Z1O3vE1krgXZNrsQ+LFTGHVxVjcXPs17LhbZVGedAJv8XZ1tvj5FvSg==
.unpackedSize: 1.4 MB

maintainers:
- mathias <mathias@qiwi.be>
- jdalton <john.david.dalton@gmail.com>
- bnjmnt4n <benjamin@dev.ofcr.se>

dist-tags:
latest: 4.17.21

published over a year ago by bnjmnt4n <benjamin@dev.ofcr.se>
```

[npm]: https://www.npmjs.com/

## Poetry

[Poetry] has a similar command called `poetry show`.

For example:

```console
$ poetry show pendulum

name        : pendulum
version     : 1.4.2
description : Python datetimes made easy

dependencies
 - python-dateutil >=2.6.1
 - tzlocal >=1.4
 - pytzdata >=2017.2.2

required by
 - calendar >=1.4.0
```

[Poetry]: https://python-poetry.org/

# Unresolved questions

1. Report crates metrics? [cargo-information#20]

    Proposal:

    - recent download count (popularity)
    - last updated (give a feel for how active development is)
    - Only for crates.io.

2. What dependency fields might be relevant to indicate? [cargo-information#23]

    Proposal:

    - From @epage: Dependencies are mostly an implementation detail (except public) but people sometimes care, so I figure that holding off on private dependencies to --verbose might buy us more space.
    - From @hi-rustin: How about we show all the dependencies and only show the dev-dependencies and build-dependencies for a --verbose. I guess checking its dependencies before you use it in your project would always be considered.

3. How should we render features? [cargo-information#26]

    Proposal: Currently, it's a simple list of features and their dependencies. We could consider a tree view:

    ```console
    features:
      parent1
        child1
      parent2
        child1*
    ```

4. What version should we default to within a workspace? What if it isn't the direct dependency but is a transitive dependency? [cargo-information#29]

[cargo-information#20]: https://github.com/hi-rustin/cargo-information/issues/20
[cargo-information#23]: https://github.com/hi-rustin/cargo-information/issues/23
[cargo-information#26]: https://github.com/hi-rustin/cargo-information/issues/26
[cargo-information#29]: https://github.com/hi-rustin/cargo-information/issues/29
