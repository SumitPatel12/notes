_____
**Created**: 19-10-2024 08:49 am
**Status**: Completed
**Tags**: [[Rust]]
**References**: 
______

Cargo uses the `--crate-type lib` options. It tells **rustc** not to look for a main file and instead compile the file and produce a `.rlib` file containing that code. The `.rilb` file can then be used to create *binaries* and other *rlib* files.

When *compiling* a program, Cargo uses the `--create-type bin` option which results in a binary executable for the target platform.
With each **rustc** command, Cargo passes `--extern` options giving the filename of each library the crate will use. This helps **rustc** identify the name of a crate and with help of Cargo find it on the disk. For example `image::png::PNGEncoder` can be identified by **rustc** as using the `image` crate.

The `.rlib` file contains the compiled code of the library which rust will statically link that code into the final executable. It also contains the type information so rust can check if what we are using actually exists in the crate or not. It also maintains a copy of the crate's public inline functions, generics, and macros, features that can't be fully compiled to machine code until Rust sees how we use them.

### Build Profiles

| Command Line          | Cargo.toml section used |
| --------------------- | ----------------------- |
| cargo build           | \[profile.dev\]         |
| cargo build --release | \[profile.release\]     |
| cargo test            | \[profile.test\]        |

