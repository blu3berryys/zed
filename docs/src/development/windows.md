# Building Zed for Windows

> The following commands may be executed in any shell.

## Repository

Clone down the [Zed repository](https://github.com/zed-industries/zed).

## Dependencies

1. Install [Rust](https://www.rust-lang.org/tools/install).
2. Install [Visual Studio](https://visualstudio.microsoft.com/downloads/) with the optional components:
   - `MSVC v*** - VS YYYY C++ x64/x86 build tools`
   - `MSVC v*** - VS YYYY C++ x64/x86 Spectre-mitigated libs (latest)`
   
   (`v***` is your VS version and `YYYY` is the year when your VS was released. Pay attention to the architecture and change it to yours if needed.)

3. Install the [Windows 11 or 10 SDK](https://developer.microsoft.com/windows/downloads/windows-sdk/) (at least `Windows 10 SDK version 2104 (10.0.20348.0)`).
4. Install [CMake](https://cmake.org/download) (required by [a dependency](https://docs.rs/wasmtime-c-api-impl/latest/wasmtime_c_api/)).

## Backend Dependencies

> This section is still in development. The instructions are not yet complete.

If you are developing collaborative features of Zed, you'll need to install the dependencies for Zed's `collab` server:

1. Install [Postgres](https://www.postgresql.org/download/windows/).
2. Install [Livekit](https://github.com/livekit/livekit-cli) and [Foreman](https://theforeman.org/manuals/3.9/quickstart_guide.html).

Alternatively, if you have [Docker](https://www.docker.com/) installed, you can bring up all the `collab` dependencies using Docker Compose:

```sh
docker compose up -d
```

## Building from Source

Once the dependencies are installed, you can build Zed using [Cargo](https://doc.rust-lang.org/cargo/).

- For a debug build:

    ```sh
    cargo run
    ```

- For a release build:

    ```sh
    cargo run --release
    ```

- To run the tests:

    ```sh
    cargo test --workspace
    ```

## Installing from MSYS2

[MSYS2](https://msys2.org/) distribution provides Zed as a package `mingw-w64-zed`. The package is available for UCRT64, MINGW64, and CLANG64 repositories. To download it, run:

```sh
pacman -Syu
pacman -S $MINGW_PACKAGE_PREFIX-zed
```

Then you can run `zed` in a shell.

You can see the [build script](https://github.com/msys2/MINGW-packages/blob/master/mingw-w64-zed/PKGBUILD) for more details on the build process.

> Please report any issues in the [msys2/MINGW-packages/issues](https://github.com/msys2/MINGW-packages/issues?q=is%3Aissue+is%3Aopen+zed) repository.

## Troubleshooting

### Can't Compile Zed

Before reporting the issue, make sure that you have the latest `rustc` version by running:

```sh
rustup update
```

### Cargo Errors Claiming That a Dependency is Using Unstable Features

Try running the following commands:

```sh
cargo clean
cargo build
```

### `STATUS_ACCESS_VIOLATION`

This error may occur if you are using the "rust-lld.exe" linker. Consider trying a different linker.

If you're using a global config, try moving the Zed repository to a nested directory and adding a `.cargo/config.toml` with a custom linker config in the parent directory.

For more information, see this [issue](https://github.com/zed-industries/zed/issues/12041).

### Invalid RC Path Selected

Sometimes, you may encounter the following error while compiling Zed:

```
error: failed to run custom build command for `zed(C:\Users\USER\src\zed\crates\zed)`
...
Selected RC path: 'bin\x64\rc.exe'
...
The system cannot find the path specified. (os error 3)
```

To fix this, manually set the `ZED_RC_TOOLKIT_PATH` environment variable to the RC toolkit path. It is typically:

```
C:\Program Files (x86)\Windows Kits\10\bin\<SDK_version>\x64
```

For more information, see this [issue](https://github.com/zed-industries/zed/issues/18393).

### Build Fails: Path Too Long

You may receive an error like the following:

```
error: failed to get `pet` as a dependency of package `languages v0.1.0 (D:\a\zed-windows-builds\zed-windows-builds\crates\languages)`
...
path too long: 'C:/Users/runneradmin/.cargo/git/checkouts/python-environment-tools-903993894b37a7d2/ffcbf3f/crates/pet-conda/tests/unix/conda_env_without_manager_but_found_in_history/some_other_location/conda_install/conda-meta/python-fastjsonschema-2.16.2-py310hca03da5_0.json'
```

To solve this, enable long-path support for both Git and Windows:

For Git:

```sh
git config --system core.longpaths true
```

For Windows (in PowerShell):

```powershell
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD -Force
```

For more information, see the [win32 docs](https://learn.microsoft.com/windows/win32/fileio/maximum-file-path-limitation?tabs=powershell).

(Note: You'll need to restart your system after enabling long-path support.)
