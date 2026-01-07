---
title: Rust
icon: fontawesome/brands/rust
---

## Ecosystem

```console
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Now you have `cargo` etc. available. Update: `rustup update`.

If you are OOMing during compilation, you can try using `cargo-binstall`, which installs precompiled binaries of Rust crates.

```console
curl -L --proto '=https' --tlsv1.2 -sSf https://raw.githubusercontent.com/cargo-bins/cargo-binstall/main/install-from-binstall-release.sh | bash
```

### Cargo

In Python terms Cargo is `pip`, `venv`, `make`, `pytest` and `twine` all in one.

```console
cargo install <crate>       # install a binary crate
cargo binstall <crate>      # install a binary crate from precompiled binaries
```

```console
cargo new <project-name>  # create a new project
cargo add <crate>         # add a dependency to Cargo.toml
cargo test                # run tests
cargo build               # build the project
cargo run                 # compile and run the project
cargo check               # check the code
cargo fmt                 # format code 
cargo clippy              # lint code
cargo doc --open
```
