# Nexmon and Rust

Nexmon uses custom Makefiles for compiling and flashing patches. To integrate Rust code we have to modify some parts. From my understanding, the Nexmon Makefile first compiles C code to `.obj` files and then links them with the rest of the firmware. Some helper files are generated along the way `.pre`, not sure what their purpose is. But we should be able to replace `gcc` with `rust` and generate object files manually for Rust.

[This](https://users.rust-lang.org/t/how-create-obj-file-from-rust-code-without-core-panic/40441/16) thread should answer how to generate `.obj` from Rust. Several possibilities exist:

- The simplest solution would be to use following command: `rustc lib.rs --emit=obj`. For more complex code this is probably not feasible because you do not benefit from the Rust library management and have to specify every source file yourself, just like with C.

- To use `cargo`, the build tool for Rust, one would use: `cargo build --release` with following `Cargo.toml` content:
```toml
[lib]
crate-type = [ "staticlib" ] # to generate a `lib<package_name>.a`
                             # i.e. an archive of `.o` files

[package]
name = "foo"
version = "0.1.0"
edition = "2018"

[dependencies] # you may depend on libc if you so wish:
# libc = { version = "*", default-features = false }

```
Afterwards, one would need to adjust the Makefile to link with `libfoo.a` instead of multiple object files.

To cross compile Rust for ARM, the `Cargo.toml` file in this repository contains a few extra configuration options. To find the correct toolchain for your platform, look at [](https://doc.rust-lang.org/nightly/rustc/platform-support.html). You then have to install it with `rustup target add <toolchain>` and adjust `target=` in `Cargo.toml`.

An interesting resource for Rust on embedded systems is the following book: [](https://docs.rust-embedded.org/book/intro/index.html)
