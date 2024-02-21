---
title: 'Feedback on `cargo-info` to prepare it for merging'
layout: post

categories: post
tags:
- Rust
- Cargo
- CLI
---

I am working on merging the `cargo-info` subcommand into Cargo. There are a few points that need to be addressed before merging. I would like to get feedback on these points.
You can find the key points of concern at the end of this post.

### Background

This adds a new subcommand to Cargo, `cargo info`. This subcommand would allow users to get information about a crate from the command line, without having to go to the web.

The main motivation for this is to make it easier to get information about a crate from the command line. Currently, the way to get information about a crate is to go to the web and look it up on [crates.io] or find the crate's source code and look at the `Cargo.toml` file. This is not very convenient, especially not all information is displayed on the [crates.io] page.
This command also has been requested by the community for a long time. You can find more discussion about this in [cargo#948].

Another motivation is to make the workflow of finding and evaluating crates more efficient. In the current workflow, users can search for crates using `cargo search`, but then they have to go to the web to get more information about the crate. This is not very efficient, especially if the user is just trying to get a quick overview of the crate. This would allow users to quickly get information about a crate without having to leave the terminal.

[crates.io]: https://crates.io
[cargo#948]: https://github.com/rust-lang/cargo/issues/948

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
Questions we expect users will be asking when wanting to run this:
- Is this the dependency I intended?
- What version should I use?
- What `features` can I use?  What are there impacts?
- Where are the docs?
- Where can I open an issue or create a PR?
- Is this compatible with my project wrt
  - licensing
  - MSRV
 
Behavior notes:
- Information sources
  - Index: relies on cargo's caches
  - `.crate`: relies on cargo's caches
  - crates.io API
- Implicit version selection
  - Prefers version already in your lockfile when run inside of a project
  - Prefers a version with a `rust-version` that is the same or older as your `rust-version` when run inside of a project
  - Prefers a version with a `rust-version` that is the same or older than your toolchain
- A note will be added to the version field if the latest is not selected

#### Prior art

##### `npm info`

[npm] has a similar command called `npm info`.

<details summary="For example:">
    
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

</details>


[npm]: https://www.npmjs.com/

##### `poetry show`

[Poetry] has a similar command called `poetry show`.

<details summary="For example:">

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


</details>

[Poetry]: https://python-poetry.org/

### Known Points of Concern

**What other questions would people want to ask?**

e.g. [cargo-information#20] is about other crates.io metrics would be useful.

Proposal:
- recent download count (popularity)
- last updated (give a feel for how active development is)

**What would people like to get out of dependencies?**

We have [cargo-information#23] about what dependency fields are relevant to show but there is a broader question of what dependencies themselves are relevant to show.
Dependencies can take up a lot of space and make it harder to browse what content is already there.

Ideas
- Only show public dependencies, show normal and build dependencies on `--verbose` and maybe also dev dependencies on `--verbose --verbose`
- Show all normal and build dependencies, relegating dev dependencies to `--verbose`

**What do people want to get out of the features view?**

[cargo-information#26] asks how we should render dependencies.

Currently, we show a flat list with all activations and highlight what gets enabled by `default`.
In  [rust-lang/cargo#10681](https://github.com/rust-lang/cargo/issues/10681), people have asked to see a visual tree.
In doing so, we might lose dependency activations.
We could have the dependency view specially render activated and inactive optional dependencies but we wouldn't get cause and effect.

**What dependency version from your lockfile would be most useful?**

We currently select the latest.  [cargo-information#29] calls out that we might want to do it differently, including
- depth in the tree
- only use direct dependencies from lockfile and get the absolute latest if its a transitive dependency

[cargo-information#20]: https://github.com/hi-rustin/cargo-information/issues/20
[cargo-information#23]: https://github.com/hi-rustin/cargo-information/issues/23
[cargo-information#26]: https://github.com/hi-rustin/cargo-information/issues/26
[cargo-information#29]: https://github.com/hi-rustin/cargo-information/issues/29
