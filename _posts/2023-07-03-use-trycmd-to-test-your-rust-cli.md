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

Recently we released [Rustup 1.26.0], which includes a bunch of new features and bug fixes. We also upgraded the [clap] version to 3.2.25, which is a major version upgrade. This upgrade is a bit tricky, because clap 3.0.0 has a lot of breaking changes. We need to make sure that the new version of Rustup works as expected. So before we upgrade clap we added [UI tests] for Rustup. We use [trycmd] to test the CLI of Rustup. In this post I will show you how to use `trycmd` to test your CLI.

[Rustup 1.26.0]: https://blog.rust-lang.org/2023/04/25/Rustup-1.26.0.html
[clap]: https://github.com/clap-rs/clap
[UI tests]: https://github.com/rust-lang/rustup/blob/master/tests/suite/cli_ui.rs
[trycmd]: https://github.com/assert-rs/trycmd

## Why trycmd?

Because it is very easy to use and well developed. It is also very easy to integrate with CI. We can use `cargo test` to run the test cases. Most importantly, it is recommended by the `clap` team. You can find it in the [clap upgrade guide].

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

We can have two types of test cases: a TOML file or a Markdown file. The TOML file is a bit more powerful than the markdown file. We can use the TOML file to test the command line arguments and the output of the command. We can also use the TOML file to test the exit code of the command. The Markdown file is a bit simpler. We can only use it to test the output of the command.

In clap documentation, we can find a [simple example] of trycmd. But it is only test a Markdown file. In this post, I will show you how to write a TOML file and a Markdown file test. You can find more examples from some real projects, like [typos tests], [clap tests] and [rustup tests].

[simple example]: https://github.com/assert-rs/trycmd/tree/main/examples/demo_trycmd
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

We can use `console` or `trycmd` code block to indicate test cases in markdown files. Here is an example:

~~~md
```console
$ command ...
```
Or
```trycmd
$ command ...
```
~~~

Sometimes, your test might include output that is generated at runtime. When that's the case, you can use variables to replace those values.

~~~md
```console
$ simple "blah blah runtime-value blah"
Hello blah blah [REPLACEMENT] blah!
```
~~~

In this example, we use `[REPLACEMENT]` to indicate the runtime value. We can use `trycmd::TestCases::new().case("README.md").insert_var("REPLACEMENT", "runtime-value")` to replace the runtime value.

```console
$ simple "blah blah runtime-value blah"
Hello blah blah runtime-value blah!
```

`trycmd` will replace the `[REPLACEMENT]` with the value of `REPLACEMENT` variable. If the output is `Hello blah blah runtime-value blah!`, the test will pass. Otherwise, the test will fail.

## One Real Example

### Create a CLI

1. Use `cargo new` to create a new CLI.

    ```sh
    cargo new --bin trycmd-example
    ```

2. Add clap as a dependency with `derive` feature.

    ```sh
    cargo add clap --features derive
    ```

    Check the `Cargo.toml` file. You should see the `clap` dependency.

    ```toml
    # Cargo.toml
    [package]
    name = "trycmd-example"
    version = "0.1.0"
    edition = "2021"

    # See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

    [dependencies]
    clap = { version = "4.3.10", features = ["derive"] }
    ```

3. Add a simple CLI.

    ```rust
    // src/main.rs
    use clap::Parser;

    /// Simple program to greet a person
    #[derive(Parser, Debug)]
    #[command(author, version, about, long_about = None)]
    struct Args {
        /// Name of the person to greet
        #[arg(short, long)]
        name: String,

        /// Number of times to greet
        #[arg(short, long, default_value_t = 1)]
        count: u8,
    }

    fn main() {
        let args = Args::parse();

        for _ in 0..args.count {
            println!("Hello {}!", args.name)
        }
    }
    ```

4. Build the CLI to make sure it works.

    ```sh
    cargo build
    ./target/debug/trycmd-example --help
    ```

    Get the output:

    ```console
    Simple program to greet a person

    Usage: trycmd-example [OPTIONS] --name <NAME>

    Options:
    -n, --name <NAME>    Name of the person to greet
    -c, --count <COUNT>  Number of times to greet [default: 1]
    -h, --help           Print help
    -V, --version        Print version
    ```

    Now we have a simple CLI. Let's add some test cases.

### Add a TOML test case

1. Create a `tests/cmd` directory.

    ```sh
    mkdir -p tests/cmd
    ```

    ```console
    tree . --gitignore
    .
    ├── Cargo.lock
    ├── Cargo.toml
    ├── src
    │   └── main.rs
    └── tests
        └── cmd
    ```

2. Create a `tests/cmd/help.toml` file.

    ```sh
    touch tests/cmd/help.toml
    ```

    ```toml
    # tests/cmd/help.toml
    bin.name = "trycmd-example"
    args = ["--help"]
    status.code = 0
    stdout = ""
    stderr = ""
    ```

3. Add `trycmd` as a dev dependency.

    ```sh
    cargo add trycmd --dev
    ```

    Check the `Cargo.toml` file. You should see the `trycmd` dependency.

    ```toml
    # Cargo.toml
    [package]
    name = "trycmd-example"
    version = "0.1.0"
    edition = "2021"

    # See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

    [dependencies]
    clap = { version = "4.3.10", features = ["derive"] }

    [dev-dependencies] # <-- Add this section
    trycmd = "0.14.16" # <-- Add this line
    ```

4. Add a Rust test case.

    ```rust
    // tests/cmd.rs
    #[test]
    fn test_cmd() {
        let t = trycmd::TestCases::new();
        let trycmd_example_binary = trycmd::cargo::cargo_bin("trycmd-example");
        t.register_bin("trycmd-example", &trycmd_example_binary);
        t.case("tests/cmd/*.toml");
    }
    ```

    In this test case, we use `trycmd::TestCases::new()` to create a new test case. Then we use `trycmd::cargo::cargo_bin("trycmd-example")` to get the path of the `trycmd-example` binary. Finally, we use `t.case("tests/cmd/*.toml")` to run all the test cases in the `tests/cmd` directory.

5. Run the test case.

    ```sh
    cargo test
    ```

    The test case will fail because we don't have right output in the `tests/cmd/help.toml` file.

    ```console
    running 1 test
    Testing tests/cmd/help.toml ... failed
    Exit: success

    ---- expected: stdout
    ++++ actual:   stdout
            1 + Simple program to greet a person
            2 +
            3 + Usage: trycmd-example [OPTIONS] --name <NAME>
            4 +
            5 + Options:
            6 +   -n, --name <NAME>    Name of the person to greet
            7 +   -c, --count <COUNT>  Number of times to greet [default: 1]
            8 +   -h, --help           Print help
            9 +   -V, --version        Print version
    stderr:

    Update snapshots with `TRYCMD=overwrite`
    Debug output with `TRYCMD=dump`
    test test_cmd ... FAILED
    ```

### Overwrite the output

AS you can see from the output, we can use `TRYCMD=overwrite` to overwrite the output.

```sh
TRYCMD=overwrite cargo test
```

`trycmd` will overwrite the output in the `tests/cmd/help.toml` file.

After that, we can run the test case again and it will pass.

### Add a Markdown test case

Usually, we use this feature to test the `README.md` or other example files.

1. Create a README.md file.

    ~~~markdown
    # trycmd-example

    ```console
    $ trycmd-example --help
    ```
    ~~~

2. Add README.md as a test case.

    ```rust
    // tests/cmd.rs
    #[test]
    fn test_cmd() {
        let t = trycmd::TestCases::new();
        let trycmd_example_binary = trycmd::cargo::cargo_bin("trycmd-example");
        t.register_bin("trycmd-example", &trycmd_example_binary);
        t.case("tests/cmd/*.toml");
        t.case("README.md"); // <-- Add this line
    }
    ```

3. Run the test case.

    ```sh
    cargo test
    ```

    The test case will fail because we don't have right output in the `README.md` file.

    ```console
    running 1 test
    Testing tests/cmd/help.toml ... ok
    Testing README.md:4 ... failed
    Exit: success

    ---- expected: stdout
    ++++ actual:   stdout
            1 + Simple program to greet a person
            2 +
            3 + Usage: trycmd-example [OPTIONS] --name <NAME>
            4 +
            5 + Options:
            6 +   -n, --name <NAME>    Name of the person to greet
            7 +   -c, --count <COUNT>  Number of times to greet [default: 1]
            8 +   -h, --help           Print help
            9 +   -V, --version        Print version
    stderr:

    Update snapshots with `TRYCMD=overwrite`
    Debug output with `TRYCMD=dump`
    test test_cmd ... FAILED
    ```

4. Overwrite the output.

    ```sh
    TRYCMD=overwrite cargo test
    ```

    `trycmd` will overwrite the output in the `README.md` file.

    After that, we can run the test case again and it will pass.

### Use directory to store input and output files

We can use a directory to store the input and output files.

Right now, we print the output to the console. If we want to print the output to a file, we also can test it with `trycmd`.

1. Change the `main.rs` file.

    ```rust
    // src/main.rs
    use std::io::Write; // <-- Add this line

    use clap::Parser;

    /// Simple program to greet a person
    #[derive(Parser, Debug)]
    #[command(author, version, about, long_about = None)]
    struct Args {
        /// Name of the person to greet
        #[arg(short, long)]
        name: String,

        /// Number of times to greet
        #[arg(short, long, default_value_t = 1)]
        count: u8,
    }

    fn main() {
        let args = Args::parse();

        // Print the greeting to a file. <-- Add this line
        let mut file = std::fs::File::create("greeting.txt").unwrap(); // <-- Add this line
        for _ in 0..args.count { // <-- Add this line
            writeln!(file, "Hello, {}!", args.name).unwrap(); // <-- Add this line
        } // <-- Add this line
    }
    ```

2. Add a new TOML test case.

    ```toml
    # tests/cmd/greeting.toml
    bin.name = "trycmd-example"
    args = ["--name", "foo"]
    status.code = 0
    stdout = ""
    stderr = ""
    ```

3. Add a output directory.

    ```sh
    mkdir tests/cmd/greeting.out
    ```

4. Add an output file.

    ```sh
    touch tests/cmd/greeting.out/greeting.txt
    ```

5. Run the test case.

    ```sh
    cargo test
    ```

    The test case will fail because we don't have right output in the `tests/cmd/greeting.out/greeting.txt` file.

    ```console
    running 1 test
    Testing README.md:4 ... ok
    Testing tests/cmd/help.toml ... ok
    Testing tests/cmd/greeting.toml ... ok
    Testing tests/cmd/greeting.toml:teardown ... failed
    Failed: Files left in unexpected state

    tests/cmd/greeting.out: is good

    ---- expected: tests/cmd/greeting.out/greeting.txt
    ++++ actual:   /private/var/folders/76/zkdsk83x0dl3qydmhxf9dj3h0000gn/T/.tmpW1Aqys/greeting.txt
            1 + Hello, foo!
    Update snapshots with `TRYCMD=overwrite`
    Debug output with `TRYCMD=dump`
    test test_cmd ... FAILED
    ```

6. Overwrite the output.

    ```sh
    TRYCMD=overwrite cargo test
    ```

    `trycmd` will overwrite the output in the `tests/cmd/greeting.out/greeting.txt` file.

    After that, we can run the test case again and it will pass.

## Summary

In this tutorial, we learned how to use `trycmd` to test a CLI program. We also learned how to use `TRYCMD=overwrite` to overwrite the output in the test case files. This feature is very useful when we want to update the output in the test case files. Hope you enjoy this tutorial and use `trycmd` to test your CLI programs.

You can find the source code of this tutorial in [this repository](https://github.com/hi-rustin/trycmd-example).

## Reference

- [trycmd](https://github.com/assert-rs/trycmd)
- [clap](https://github.com/clap-rs/clap)
- [rustup](https://github.com/rust-lang/rustup)
