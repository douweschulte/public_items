# cargo public-api

You probably want to use the convenient `cargo public-api` subcommand rather than this low-level library. The `cargo public-api` subcommand performs syntax highlighting of the output, among other things. See https://github.com/Enselic/cargo-public-api.

# public-api

List and diff the public API of Rust library crates by analyzing rustdoc JSON output files from the nightly toolchain.

# Usage

The library comes with a thin bin wrapper that can be used to explore the capabilities of this library.

```bash
# Build and install the thin bin wrapper with a recent stable Rust toolchain
cargo install public-api

# Install nightly-2022-03-14 or later so you can build up-to-date rustdoc JSON files
rustup install nightly
```

## List public API

To list all items that form the public API of your Rust library:

```bash
# Generate rustdoc JSON for your own Rust library
% cd ~/src/your_library
% RUSTDOCFLAGS='-Z unstable-options --output-format json' cargo +nightly doc --lib --no-deps

# List all items in the public API of your Rust library
% public-api ./target/doc/your_library.json
pub mod public_api
pub fn public_api::Options::clone(&self) -> Options
pub fn public_api::Options::default() -> Self
pub fn public_api::PublicItem::clone(&self) -> PublicItem
pub fn public_api::public_api_from_rustdoc_json_str(rustdoc_json_str: &str, options: Options) -> Result<Vec<PublicItem>>
pub struct public_api::Options
pub struct public_api::PublicItem
pub struct field public_api::Options::sorted: bool
pub struct field public_api::Options::with_blanket_implementations: bool
...
```

## Diff public API

It is frequently of interest to know how the public API of a crate has changed. You can find this out by doing a diff between different versions of the same library. Again, [`cargo public-api`](https://github.com/Enselic/cargo-public-api) makes this more convenient, but it is straightforward enough without it.

```bash
# Generate two different rustdoc JSON files for two different versions of your library
# and then pass both files to the bin to make it print the public API diff
% public-api ./target/doc/your_library.old.json  ./target/doc/your_library.json
Removed:
(nothing)

Changed:
-pub fn public_api::sorted_public_api_from_rustdoc_json_str(rustdoc_json_str: &str) -> Result<Vec<PublicItem>>
+pub fn public_api::sorted_public_api_from_rustdoc_json_str(rustdoc_json_str: &str, options: Options) -> Result<Vec<PublicItem>>

Added:
+pub fn public_api::Options::clone(&self) -> Options
+pub fn public_api::Options::default() -> Self
+pub struct public_api::Options
+pub struct field public_api::Options::with_blanket_implementations: bool
```

# Expected output

In general, output aims to be character-by-character identical to the textual parts of the regular `cargo doc` HTML output. For example, [this item](https://docs.rs/bat/0.20.0/bat/struct.PrettyPrinter.html#method.input_files) has the following textual representation in the rendered HTML:

```
pub fn input_files<I, P>(&mut self, paths: I) -> &mut Self
where
    I: IntoIterator<Item = P>,
    P: AsRef<Path>,
```

and `public-api` represent this item in the following manner:

```
pub fn bat::PrettyPrinter::input_files<I, P>(&mut self, paths: I) -> &mut Self where I: IntoIterator<Item = P>, P: AsRef<Path>
```

If we normalize by removing newline characters and adding some whitespace padding to get the alignment right for side-by-side comparison, we can see that they are exactly the same, except an irrelevant trailing comma:

```
pub fn                     input_files<I, P>(&mut self, paths: I) -> &mut Self where I: IntoIterator<Item = P>, P: AsRef<Path>,
pub fn bat::PrettyPrinter::input_files<I, P>(&mut self, paths: I) -> &mut Self where I: IntoIterator<Item = P>, P: AsRef<Path>
```

# Blanket implementations

By default, blanket implementations such as `impl<T> Any for T`, `impl<T> Borrow<T> for T`, and `impl<T, U> Into<U> for T where U: From<T>` are omitted from the list of public items of a crate. For the vast majority of use cases, blanket implementations are not of interest, and just creates noise.

If you want to include items of blanket implementations in the output, set [`Options::with_blanket_implementations`](https://docs.rs/public-api/latest/public_api/struct.Options.html#structfield.with_blanket_implementations) to true if you use the library, or pass `--with-blanket-implementations` to `public_api`.

# Library documentation

Documentation can be found at [docs.rs](https://docs.rs/public-api/latest/public-api/) as usual. There are also some simple [examples](https://github.com/Enselic/public-api/tree/main/examples) on how to use the library. The code for the [thin bin wrapper](https://github.com/Enselic/public-api/blob/main/src/main.rs) might also be of interest.

# Target audience

Maintainers of Rust libraries that want to keep track of changes to their public API.

# Limitations

See [`[limitation]`](https://github.com/Enselic/public-api/labels/limitation)
labeled issues.

# Compatibility matrix

| public-api    | Understands the rustdoc JSON output of  |
| ------------- | --------------------------------------- |
| v0.10.x       | nightly-2022-03-14 —                    |
| v0.5.x        | nightly-2022-02-23 — nightly-2022-03-13 |
| v0.2.x        | nightly-2022-01-19 — nightly-2022-02-22 |
| v0.0.5        | nightly-2021-10-11 — nightly-2022-01-18 |

# Development

See [development.md](./doc/development.md).

# Maintainers

- [Enselic](https://github.com/Enselic)
- [douweschulte](https://github.com/douweschulte)
