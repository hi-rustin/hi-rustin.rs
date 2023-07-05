---
title: 'How to use trycmd to test your Rust CLI?'
layout: post

categories: post
tags:
- Rust
- trycmd
- testing
- CLI
- Rustup
---

Recently we released [Rustup 1.26.0], which includes a bunch of new features and bug fixes. We also upgraded the [clap] version to 3.2.25, which is a major version upgrade. This upgrade is a bit tricky, because clap 3.0.0 has a lot of breaking changes. We need to make sure that the new version of Rustup works as expected. So before we upgrade clap we added UI tests for Rustup. We use [trycmd] to test the CLI of Rustup. In this post I will show you how to use `trycmd` to test your CLI.

[Rustup 1.26.0]: https://blog.rust-lang.org/2023/04/25/Rustup-1.26.0.html
[clap]: https://github.com/clap-rs/clap
[trycmd]: https://github.com/assert-rs/trycmd

## Why trycmd?

Because it is very easy to use and well developed. It is also very easy to integrate with CI. We can use `cargo test` to run the test cases. Most importantly, it is recommended by the clap team. You can find it in the [clap upgrade guide].

[clap upgrade guide]: https://github.com/clap-rs/clap/blob/master/CHANGELOG.md#migrating

## What is trycmd?

From the [README] of `trycmd`: "`trycmd` is a test harness that will enumerate test case files and run them to verify the results, taking inspiration from [trybuild] and [cram]."

Here is an example:

```rust
// tests/cli_tests.rs
#[test]
fn cli_tests() {
    trycmd::TestCases::new()
        .case("tests/cmd/*.toml")
        .case("README.md");
}
```

We can use `trycmd::TestCases::new()` to create a new test case. Then we can use `case()` to add test cases. The argument of `case()` is a glob pattern. It will enumerate all the files that match the pattern and run them as test cases.

In this example, we use `tests/cmd/*.toml` to match all the files in `tests/cmd` that end with `.toml`. We also use `README.md` as a test case. The `README.md` file is a markdown file, but trycmd will treat it as a test case. It will run the command in the code block and verify the output and it is a very useful feature when you want to test the examples in your README.

We can treat this test case as a normal test case and run it with `cargo test`.

[README]: https://github.com/assert-rs/trycmd/blob/main/README.md
[trybuild]: https://crates.io/crates/trybuild
[cram]: https://bitheap.org/cram/

## How to write a test case?

We can have two types of test cases: a TOML file or a markdown file. The TOML file is a bit more powerful than the markdown file. We can use the TOML file to test the command line arguments and the output of the command. We can also use the TOML file to test the exit code of the command. The markdown file is a bit simpler. We can only use it to test the output of the command.

In clap documentation, we can find a [simple example] of trycmd. But it is only test a markdown file. In this post, I will show you how to write a TOML file and a markdown file. You can find more examples from some real projects, like [typos tests], [clap tests] and [rustup tests].

[typos tests]: https://github.com/crate-ci/typos/blob/master/crates/typos-cli/tests/cli_tests.rs
[clap tests]: https://github.com/clap-rs/clap/tree/master/tests/ui
[rustup tests]: https://github.com/rust-lang/rustup/blob/master/tests/suite/cli_ui.rs

### TOML file

Here is an example of a TOML file:

```toml
# tests/cmd/help.toml
bin.name = "YOUR_BIN_NAME"
args = ["--help"]
status.code = 0
stdout = """
HELP MESSAGE
"""
stderr = ""
```

We can use `bin.name` for the binary name, `args` for command arguments, `status.code` for the exit code, and `stdout` and `stderr` for command output.

If your CLI will read some files, you can use a same name directory with `.in` suffix to store the input files. For example, if your CLI will read a file named `input.txt` and your test case is `tests/cmd/input.toml`, you can put the `input.txt` file in `tests/cmd/input.in/input.txt`.

```toml
# tests/cmd/input.toml
bin.name = "YOUR_BIN_NAME"
status.code = 0
stdout = ""
stderr = ""
```

```console
tree tests/cmd
.
├── input.toml
└── input.in
    └── input.txt
```

If your CLI will write some files, you can use a same name directory with `.out` suffix to store the output files. For example, if your CLI read the `input.txt` file and write the `output.txt` file. You can put the `output.txt` file in `tests/cmd/input.out/output.txt`.

```console
tree tests/cmd
.
├── input.toml
├── input.in
│   └── input.txt
└── input.out
    ├── input.txt
    └── output.txt
```

`trycmd` will compare the output file with the expected output file. If they are different, the test will fail.

### Markdown file

## One Real Example

## Summary
