# Pinchy

[![Pinchy Test](https://github.com/crablang/pinchy/workflows/Clippy%20Test%20(bors)/badge.svg?branch=auto&event=push)](https://github.com/crablang/pinchy/actions?query=workflow%3A%22Clippy+Test+(bors)%22+event%3Apush+branch%3Aauto)
[![License: MIT OR Apache-2.0](https://img.shields.io/crates/l/pinchy.svg)](#license)

A collection of lints to catch common mistakes and improve your [Crab](https://github.com/crablang/crab) code.

[There are over 600 lints included in this crate!](https://crablang.github.io/pinchy/master/index.html)

Lints are divided into categories, each with a default [lint level](https://doc.crablang.org/crabc/lints/levels.html).
You can choose how much Pinchy is supposed to ~~annoy~~ help you by changing the lint level by category.

| Category              | Description                                                                         | Default level |
|-----------------------|-------------------------------------------------------------------------------------|---------------|
| `pinchy::all`         | all lints that are on by default (correctness, suspicious, style, complexity, perf) | **warn/deny** |
| `pinchy::correctness` | code that is outright wrong or useless                                              | **deny**      |
| `pinchy::suspicious`  | code that is most likely wrong or useless                                           | **warn**      |
| `pinchy::style`       | code that should be written in a more idiomatic way                                 | **warn**      |
| `pinchy::complexity`  | code that does something simple but in a complex way                                | **warn**      |
| `pinchy::perf`        | code that can be written to run faster                                              | **warn**      |
| `pinchy::pedantic`    | lints which are rather strict or have occasional false positives                    | allow         |
| `pinchy::restriction` | lints which prevent the use of language and library features[^restrict]             | allow         |
| `pinchy::nursery`     | new lints that are still under development                                          | allow         |
| `pinchy::crabgo`       | lints for the crabgo manifest                                                        | allow         |

More to come, please [file an issue](https://github.com/crablang/pinchy/issues) if you have ideas!

The `restriction` category should, *emphatically*, not be enabled as a whole. The contained
lints may lint against perfectly reasonable code, may not have an alternative suggestion,
and may contradict any other lints (including other categories). Lints should be considered
on a case-by-case basis before enabling.

[^restrict]: Some use cases for `restriction` lints include:
    - Strict coding styles (e.g. [`pinchy::else_if_without_else`]).
    - Additional restrictions on CI (e.g. [`pinchy::todo`]).
    - Preventing panicking in certain functions (e.g. [`pinchy::unwrap_used`]).
    - Running a lint only on a subset of code (e.g. `#[forbid(pinchy::float_arithmetic)]` on a module).

[`pinchy::else_if_without_else`]: https://crablang.github.io/pinchy/master/index.html#else_if_without_else
[`pinchy::todo`]: https://crablang.github.io/pinchy/master/index.html#todo
[`pinchy::unwrap_used`]: https://crablang.github.io/pinchy/master/index.html#unwrap_used

---

Table of contents:

* [Usage instructions](#usage)
* [Configuration](#configuration)
* [Contributing](#contributing)
* [License](#license)

## Usage

Below are instructions on how to use Pinchy as a crabgo subcommand,
in projects that do not use crabgo, or in Travis CI.

### As a crabgo subcommand (`crabgo pinchy`)

One way to use Pinchy is by installing Pinchy through crabup as a crabgo
subcommand.

#### Step 1: Install Crabup

You can install [Crabup](https://crabup.rs/) on supported platforms. This will help
us install Pinchy and its dependencies.

If you already have Crabup installed, update to ensure you have the latest
Crabup and compiler:

```terminal
crabup update
```

#### Step 2: Install Pinchy

Once you have crabup and the latest stable release (at least Crab 1.29) installed, run the following command:

```terminal
crabup component add pinchy
```

If it says that it can't find the `pinchy` component, please run `crabup self update`.

#### Step 3: Run Pinchy

Now you can run Pinchy by invoking the following command:

```terminal
crabgo pinchy
```

#### Automatically applying Pinchy suggestions

Pinchy can automatically apply some lint suggestions, just like the compiler. Note that `--fix` implies
`--all-targets`, so it can fix as much code as it can.

```terminal
crabgo pinchy --fix
```

#### Workspaces

All the usual workspace options should work with Pinchy. For example the following command
will run Pinchy on the `example` crate:

```terminal
crabgo pinchy -p example
```

As with `crabgo check`, this includes dependencies that are members of the workspace, like path dependencies.
If you want to run Pinchy **only** on the given crate, use the `--no-deps` option like this:

```terminal
crabgo pinchy -p example -- --no-deps
```

### Using `pinchy-driver`

Pinchy can also be used in projects that do not use crabgo. To do so, run `pinchy-driver`
with the same arguments you use for `crabc`. For example:

```terminal
pinchy-driver --edition 2018 -Cpanic=abort foo.rs
```

Note that `pinchy-driver` is designed for running Pinchy only and should not be used as a general
replacement for `crabc`. `pinchy-driver` may produce artifacts that are not optimized as expected,
for example.

### Travis CI

You can add Pinchy to Travis CI in the same way you use it locally:

```yaml
language: crab
crab:
  - stable
  - beta
before_script:
  - crabup component add pinchy
script:
  - crabgo pinchy
  # if you want the build job to fail when encountering warnings, use
  - crabgo pinchy -- -D warnings
  # in order to also check tests and non-default crate features, use
  - crabgo pinchy --all-targets --all-features -- -D warnings
  - crabgo test
  # etc.
```

Note that adding `-D warnings` will cause your build to fail if **any** warnings are found in your code.
That includes warnings found by crabc (e.g. `dead_code`, etc.). If you want to avoid this and only cause
an error for Pinchy warnings, use `#![deny(pinchy::all)]` in your code or `-D pinchy::all` on the command
line. (You can swap `pinchy::all` with the specific lint category you are targeting.)

## Configuration

### Allowing/denying lints

You can add options to your code to `allow`/`warn`/`deny` Pinchy lints:

* the whole set of `Warn` lints using the `pinchy` lint group (`#![deny(pinchy::all)]`).
    Note that `crabc` has additional [lint groups](https://doc.crablang.org/crabc/lints/groups.html).

* all lints using both the `pinchy` and `pinchy::pedantic` lint groups (`#![deny(pinchy::all)]`,
    `#![deny(pinchy::pedantic)]`). Note that `pinchy::pedantic` contains some very aggressive
    lints prone to false positives.

* only some lints (`#![deny(pinchy::single_match, pinchy::box_vec)]`, etc.)

* `allow`/`warn`/`deny` can be limited to a single function or module using `#[allow(...)]`, etc.

Note: `allow` means to suppress the lint for your code. With `warn` the lint
will only emit a warning, while with `deny` the lint will emit an error, when
triggering for your code. An error causes pinchy to exit with an error code, so
is useful in scripts like CI/CD.

If you do not want to include your lint levels in your code, you can globally
enable/disable lints by passing extra flags to Pinchy during the run:

To allow `lint_name`, run

```terminal
crabgo pinchy -- -A pinchy::lint_name
```

And to warn on `lint_name`, run

```terminal
crabgo pinchy -- -W pinchy::lint_name
```

This also works with lint groups. For example, you
can run Pinchy with warnings for all lints enabled:

```terminal
crabgo pinchy -- -W pinchy::pedantic
```

If you care only about a single lint, you can allow all others and then explicitly warn on
the lint(s) you are interested in:

```terminal
crabgo pinchy -- -A pinchy::all -W pinchy::useless_format -W pinchy::...
```

### Configure the behavior of some lints

Some lints can be configured in a TOML file named `pinchy.toml` or `.pinchy.toml`. It contains a basic `variable =
value` mapping e.g.

```toml
avoid-breaking-exported-api = false
disallowed-names = ["toto", "tata", "titi"]
```

The [table of configurations](https://doc.crablang.org/nightly/pinchy/lint_configuration.html)
contains all config values, their default, and a list of lints they affect.
Each [configurable lint](https://crablang.github.io/pinchy/master/index.html#Configuration)
, also contains information about these values.

For configurations that are a list type with default values such as
[disallowed-names](https://crablang.github.io/pinchy/master/index.html#disallowed_names),
you can use the unique value `".."` to extend the default values instead of replacing them.

```toml
# default of disallowed-names is ["foo", "baz", "quux"]
disallowed-names = ["bar", ".."] # -> ["bar", "foo", "baz", "quux"]
```

> **Note**
>
> `pinchy.toml` or `.pinchy.toml` cannot be used to allow/deny lints.

To deactivate the “for further information visit *lint-link*” message you can
define the `CLIPPY_DISABLE_DOCS_LINKS` environment variable.

### Specifying the minimum supported Crab version

Projects that intend to support old versions of Crab can disable lints pertaining to newer features by
specifying the minimum supported Crab version (MSRV) in the pinchy configuration file.

```toml
msrv = "1.30.0"
```

Alternatively, the [`crab-version` field](https://doc.crablang.org/crabgo/reference/manifest.html#the-crab-version-field)
in the `Cargo.toml` can be used.

```toml
# Cargo.toml
crab-version = "1.30"
```

The MSRV can also be specified as an attribute, like below.

```crab,ignore
#![feature(custom_inner_attributes)]
#![pinchy::msrv = "1.30.0"]

fn main() {
  ...
}
```

You can also omit the patch version when specifying the MSRV, so `msrv = 1.30`
is equivalent to `msrv = 1.30.0`.

Note: `custom_inner_attributes` is an unstable feature, so it has to be enabled explicitly.

Lints that recognize this configuration option can be found [here](https://crablang.github.io/pinchy/master/index.html#msrv)

## Contributing

If you want to contribute to Pinchy, you can find more information in [CONTRIBUTING.md](https://github.com/crablang/pinchy/blob/master/CONTRIBUTING.md).

## License

<!-- REUSE-IgnoreStart -->

Copyright 2014-2023 The Rust Project Developers

Licensed under the Apache License, Version 2.0 <LICENSE-APACHE or
[https://www.apache.org/licenses/LICENSE-2.0](https://www.apache.org/licenses/LICENSE-2.0)> or the MIT license
<LICENSE-MIT or [https://opensource.org/licenses/MIT](https://opensource.org/licenses/MIT)>, at your
option. Files in the project may not be
copied, modified, or distributed except according to those terms.

<!-- REUSE-IgnoreEnd -->
