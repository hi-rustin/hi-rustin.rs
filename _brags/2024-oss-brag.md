---
title: '2024 Open Source Brag'
layout: post
---

### Goals for this year

1. Keep contributing to Cargo and become a member of the Cargo team
2. Learn async Rust and contribute to Tokio

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
