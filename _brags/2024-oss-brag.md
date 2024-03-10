---
title: '2024 Open Source Brag'
layout: post
---

### Goals for this year

1. Continue contributing to Cargo and crates.io. Identify specific areas of interest and actively work on them with the goal of becoming a maintainer.
2. Deepen my understanding of asynchronous runtime internals by actively contributing to projects like Tokio's console and Golang's pprof, focusing on understanding their architecture and design.

### Projects

#### Cargo

##### Weekly Summary

- 2024-01-07:
  - Cargo-Information
    - It can pick the MSRV-compatible version of the crate now. [cargo-information#74](https://github.com/hi-rustin/cargo-information/pull/74)
    - Released 0.4.0
    - Added a new feature `vendored-openssl` to allow users to use vendored OpenSSL. [cargo-information#90](https://github.com/hi-rustin/cargo-information/pull/90)
    - Discussed with crates.io team about showing the crates.io metrics [cargo-information#20](https://github.com/hi-rustin/cargo-information/issues/20)
  - Cargo
    - Tried to fix [cargo#13127](https://github.com/rust-lang/cargo/issues/13127) in [cargo#13204](https://github.com/rust-lang/cargo/pull/13204)
    - Tried to analyze [cargo#13235](https://github.com/rust-lang/cargo/issues/13235)
- 2024-01-14:
  - Cargo-Information
    - No Progress
  - Cargo
    - Only inherit workspace package table if the new package is a member of the workspace [cargo#13261](https://github.com/rust-lang/cargo/pull/13261)
- 2024-01-21:
  - Cargo-Information
    - Support display non-registries dependencies [cargo-information#100](https://github.com/hi-rustin/cargo-information/pull/100)
  - Cargo
    - Tried to analyze [cargo#13310](https://github.com/rust-lang/cargo/issues/13310)
- 2024-02-11:
  - Cargo-Information
    - Worked on the pre-RFC for the `cargo info` command [hi-rustin.rs#108](https://github.com/hi-rustin/hi-rustin.rs/pull/108)
- 2024-02-18:
  - Cargo-Information
    - Bumped a bunch of dependencies
    - Released 0.4.2
  - crates.io
    - Added prettier as a CI action [crates.io#8140](https://github.com/rust-lang/crates.io/pull/8140)
    - Review couple of PRs
- 2024-02-25:
  - Cargo-Information
    - Publish the internals post [Feedback on `cargo-info` to prepare it for merging](https://internals.rust-lang.org/t/feedback-on-cargo-info-to-prepare-it-for-merging/20369)
  - crates.io
    - No Progress
- 2024-03-10:
  - Cargo-Information
    - Used the latest `snapbox` to verify the style of the `cargo-info` [hi-rustin.rs#119](https://github.com/hi-rustin/cargo-information/pull/119)
    - Added crates.io link to the `cargo info` command [hi-rustin.rs#122](https://github.com/hi-rustin/cargo-information/pull/122)
  - Cargo
    - Discussed the `yanking message` with the team 😆
  - crates.io
    - Discussed the `yanking message` with the team 😆
- 2024-03-17:
  - Goals
    - Discussed the `yanking message` with the team again
    - Added some test cases to the `cargo info`
    - Finish the crates.io email PR
  - Achieved
    - TBD

#### Tokio

##### Weekly Summary

- 2024-01-07:
  - console-web
    - Used the correct timestamp from backend [console-web#68](https://github.com/hi-rustin/console-web/pull/68)
    - Added the console anchor to make it can switch between the tasks and resources [console-web#71](https://github.com/hi-rustin/console-web/pull/71)
    - Added the resources page [console-web#75](https://github.com/hi-rustin/console-web/pull/75)
    - Marked the current route path in the anchor [console-web#76](https://github.com/hi-rustin/console-web/pull/76)
  - Tokio
    - Made the `io::copy` cooperative [tokio#6265](https://github.com/tokio-rs/tokio/pull/6265)
- 2024-01-14:
  - console-web
    - Only watch the update stream once [console-web#89](https://github.com/hi-rustin/console-web/pull/89)
  - Tokio
    - No Progress
- 2024-01-21:
  - console-web
    - Stop the task stream when the task page is not active [console-web#91](<https://github.com/hi-rustin/console-web/pull/91>)
  - console
    - Added flags and configurations for warnings [console#493](https://github.com/tokio-rs/console/pull/493)
    - Added some comments to the `h2` dependency [console#499](https://github.com/tokio-rs/console/pull/499)
- 2024-02-11:
  - console
    - Added `--allow` flag to the `tokio-console` [console#513](https://github.com/tokio-rs/console/pull/513)
    - Finished `grpc-web` support [console#498](https://github.com/tokio-rs/console/pull/498)
- 2024-02-18:
  - console
    - Merged `grpc-web` support [console#498](https://github.com/tokio-rs/console/pull/498)
    - Added a `grpc-web` nodejs example [console#526](https://github.com/tokio-rs/console/pull/526)
- 2024-02-25:
  - console
    - Joined the team!
    - Reviewed a couple of PRs
  - console-web
    - Updated the `console-web` to use the latest `console` version [console-web#102](https://github.com/hi-rustin/console-web/pull/102)
- 2024-03-10:
  - console
    - Did a tiny refactor [console#533](https://github.com/tokio-rs/console/pull/533)
  - console-web
    - Added the `tokio-console-web` CLI [console-web#129](https://github.com/hi-rustin/tokio-console-web/pull/129)
    - Used `cargo-dist` to build the `tokio-console-web` [console-web#129](https://github.com/hi-rustin/tokio-console-web/pull/129)
    - Added backoff to the `tokio-console-web` [console-web#153](https://github.com/hi-rustin/tokio-console-web/pull/153)
    - Added logs and comments to the tasks page [console-web#155](https://github.com/hi-rustin/tokio-console-web/pull/155)
    - Used the correct last wake time [console-web#159](https://github.com/hi-rustin/tokio-console-web/pull/159)
- 2024-03-17:
  - Goals
    - Finish the task details page refactoring
    - Finish the resource table page refactoring
    - Read the tokio runtime source code
  - Achieved
    - TBD
