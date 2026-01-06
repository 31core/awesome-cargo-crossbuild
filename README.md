# Awesome cargo crossbuild

## FreeBSD target

Install rust standard library for `x86_64-unknown-freebsd` if you've not installed it before.

```shell
rustup target add x86_64-unknown-freebsd
```

Download FreeBSD root, here we extract it into `~/freebsd-sysroot`.

```shell
wget https://download.freebsd.org/ftp/releases/amd64/15.0-RELEASE/base.txz
mkdir ~/freebsd-sysroot
tar -xf base.txz -C ~/freebsd-sysroot
```

Set up linker `~/.cargo/config.toml`.
```toml
[target.x86_64-unknown-freebsd]
linker = "clang"
rustflags = [
  "-C", "link-arg=--sysroot=/home/xxx/freebsd-sysroot",
  "-C", "link-arg=-Wl,--dynamic-linker=/libexec/ld-elf.so.1",
]
```

Set up environment variables.
```shell
export TARGET_CC=clang
export TARGET_CXX=clang++
export TARGET_CFLAGS="-target x86_64-unkown-freebsd --sysroot $HOME/freebsd-sysroot"
export TARGET_CXXFLAGS="-target x86_64-unkown-freebsd --sysroot $HOME/freebsd-sysroot"
```

Start building!
```shell
cargo build --release --target x86_64-unknown-freebsd
```

## macOS target

Install rust standard library for `aarch64-apple-darwin`.

```shell
rustup target add aarch64-apple-darwin
```

Set up [osxcross](https://github.com/tpoechtrager/osxcross) toolchain.

Set up linker in `~/.cargo/config.toml`.
```toml
[target.aarch64-apple-darwin]
linker = "aarch64-apple-darwin25.2-cc"
```

Set up environment variables.
```shell
export TARGET_CC=aarch64-apple-darwin25.2-cc
export TARGET_CXX=aarch64-apple-darwin25.2-c++
```

Start building!
```shell
cargo build --release --target aarch64-apple-darwin
```

## Windows target
### MSVC toolchain

Install rust standard library for `x86_64-pc-windows-msvc`.

```shell
rustup target add x86_64-pc-windows-msvc
```

Install cargo-xwin, it will download C/C++ headers and set up environment automatically.
```shell
cargo install cargo-xwin
```

Start building!
```shell
cargo xwin build --release --target x86_64-pc-windows-msvc
```

### GNU toolchain (also similar to \*-unknown-linux-gnu\* targets)

Install rust standard library for `x86_64-pc-windows-gnu`.

```shell
rustup target add x86_64-pc-windows-gnu
```

Install cross-platform GCC.
```shell
sudo apt install gcc-mingw-w64-x86-64-win32
```

Set up linker in `~/.cargo/config.toml`.
```toml
[target.x86_64-pc-windows-gnu]
linker = "x86_64-w64-mingw32-gcc"
```

Set up environment variables.
```shell
export TARGET_CC=x86_64-w64-mingw32-gcc
export TARGET_CXX=x86_64-w64-mingw32-g++
```

Start building!
```shell
cargo build --release --target x86_64-pc-windows-gnu
```

# Troubleshooting
## No host OpenSSL development files found when building openssl-sys

Error message:
```
  Could not find directory of OpenSSL installation, and this `-sys` crate cannot
  proceed without this knowledge. If OpenSSL is installed and this crate had
  trouble finding it,  you can set the `OPENSSL_DIR` environment variable for the
  compilation process.
```

### Solution

Add `openssl-sys` as `vendored`.
```shell
cargo add openssl-sys -F vendored
```

## Host linker has been used when building aws-lc-sys for macOS

Error message:
```
  Supported emulations: elf_x86_64 elf32_x86_64 elf_i386 elf_iamcu i386pep i386pe
  clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

### Solution

Override system `ld` temporarily.
```shell
ln -s [OSXCROSS_DIR]/MacOSX26.2.sdk/bin/aarch64-apple-darwin25.2-ld ~/.local/bin/ld
```

Remove it after building.
```shell
rm ~/.local/bin/ld
```
