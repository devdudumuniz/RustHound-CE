# Native Windows recipe (x86_64-pc-windows-msvc)

This fork-maintained page complements [HELP.md](HELP.md), which documents
Linux, Docker and cross-compilation. It describes how to recover a working
checkout on a fresh Windows 10/11 x64 machine using only this public
repository. The upstream release workflow (`.github/workflows/build.yml`)
builds the same `x86_64-pc-windows-msvc` target on `windows-latest`.

The verification steps below never contact a domain controller. Running the
collector (`rusthound-ce -d ...`) performs real LDAP/ADWS collection and must
only be done against a directory you are explicitly authorized to assess.

## 1. Prerequisites

- Git for Windows.
- Visual Studio 2022 Build Tools with the "Desktop development with C++"
  workload (MSVC linker and Windows SDK).
- rustup with the MSVC toolchain. `Cargo.toml` declares `rust-version = "1.85"`;
  any newer stable toolchain works.

```powershell
winget install --id Git.Git -e
winget install --id Microsoft.VisualStudio.2022.BuildTools -e
winget install --id Rustlang.Rustup -e
rustup default stable-x86_64-pc-windows-msvc
rustup component add clippy rustfmt
```

Optional: set `CARGO_HOME`, `RUSTUP_HOME` and `CARGO_TARGET_DIR` to a cache
location outside the checkout before building, so no build output is written
into the source tree.

## 2. Clone

```powershell
git clone https://github.com/devdudumuniz/RustHound-CE.git
cd RustHound-CE
git remote add upstream https://github.com/g0h4n/RustHound-CE.git
git fetch upstream
git status --short --branch
```

## 3. Setup and verify

```powershell
cargo fetch --locked
cargo test --locked --all-targets
cargo clippy --locked --all-targets
```

`cargo fmt --all -- --check` currently reports formatting differences that
already exist upstream; check formatting only on the files you change
(`rustfmt --check <file>`).

## 4. Build

```powershell
cargo build --release --locked
```

The binary is written to `<target-dir>\release\rusthound-ce.exe`
(`target\release\rusthound-ce.exe` when `CARGO_TARGET_DIR` is not set).

## 5. Start (offline smoke check)

```powershell
& "<target-dir>\release\rusthound-ce.exe" --version
& "<target-dir>\release\rusthound-ce.exe" --help
```

Both commands print and exit without network access. See
[HELP.md](HELP.md) for collection options.

## 6. Keeping the fork in sync

The fork's `main` tracks `upstream/main`. Contribution branches start from
`upstream/main` and are proposed upstream via pull requests:

```powershell
git switch main
git fetch upstream
git merge upstream/main
git push origin main
```
