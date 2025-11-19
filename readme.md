# Embedded Rust STM32 Template Repository

# Tools to install

1. [nix](https://nixos.org/download/) for reproducible development environments, no more "it works on my machine"
   - After installing nix, enable the `command` and `flakes` features by running
     `echo "experimental-features = nix-command flakes" > ~/.config/nix/nix.conf`
   - Then reboot, if you haven't done this yet
   - Now you should be able to enter the dev shell with `nix develop`
2. [direnv](https://direnv.net/) for automatic sourcing of the nix dev shell
   - After installation, run `direnv allow` in this repository root
   - From now on you automatically enter the nix dev shell when `cd`-ing into this repo

## Build & Dependency Management

### `Cargo.toml`

This is Rust's package manifest file. It defines:

- **Package metadata**: Name (`motor`), version, and Rust edition
- **Dependencies**: All the libraries your project needs, such as:
  - `embassy-*` crates for async embedded development
  - `defmt` for efficient logging on embedded systems
  - `cortex-m` for ARM Cortex-M processor support
  - Hardware-specific drivers like `ssd1309` (display) and `tb6600` (motor driver)

### `build.rs`

A build script that runs before compiling your project. It:

- Copies the `memory.x` linker script to where the compiler can find it
- Configures linker arguments for embedded targets
- Ensures the build reruns when memory configuration changes

## Cargo Configuration

### `.cargo/config.toml`

Configures Cargo's behavior for this project:

- **Target**: Sets default compilation target to `thumbv7em-none-eabihf` (ARM Cortex-M7 with hardware floating point)
- **Runner**: Uses `probe-rs` to flash and run code on the STM32H753ZITx chip
- **Alias**: Defines `cargo attach` command for debugging
- **Environment**: Sets logging level to "trace"
- **Profiles**: Optimizes debug builds for size while keeping symbols

## Memory Configuration

### `memory.x`

Linker script defining the STM32H753's memory layout:

- **FLASH**: 2MB starting at `0x08000000` (where your program code lives)
- **RAM**: 512KB starting at `0x24000000` (main working memory)
- **RAM_D3**: 64KB starting at `0x38000000` (special fast memory region)

The linker uses this to place code and data in the correct memory regions.

## Development Environment

### `flake.nix`

A Nix flake that defines a reproducible development environment. It provides:

- Rust toolchain with embedded support (via Fenix)
- Development tools: `nil` (Nix LSP), `alejandra` (formatter), `rust-analyzer`
- Embedded tools: `probe-rs`, `openocd`, `tio` (serial monitor)

To use: Run `nix develop` to enter the development shell with all tools available.

### `flake.lock`

Automatically generated lockfile ensuring everyone gets the exact same versions of all Nix dependencies. Don't edit manually.

### `.envrc`

Used by [direnv](https://direnv.net/) to automatically load the Nix development environment when you enter the project directory. Just contains `use flake` which tells direnv to use `flake.nix`.

### `rust-toolchain.toml`

Specifies the exact Rust version and components:

- **Channel**: Rust 1.89.0 (pinned version)
- **Components**: Tools like `rust-analyzer`, `rustfmt`, `clippy`
- **Targets**: Cross-compilation target for STM32 (`thumbv7em-none-eabihf`)

## Build Artifacts

### `.gitignore`

Tells Git which files to ignore:

- `target/`: Compiled binaries and intermediate build files
- `.direnv/`: Direnv cache directory

These are generated files that shouldn't be committed to version control.

## Task Automation

### `justfile`

A command runner (like Make) defining common tasks:

- `just build`: Compile the project
- `just run CRATE`: Build and flash a specific binary
- `just attach TARGET`: Debug a running program
- `just openocd`: Start OpenOCD debug server

This simplifies repetitive commands during development.
