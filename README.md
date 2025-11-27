# Install Compiler GitHub Action

Install and activate a C/C++ compiler toolchain (GCC, Clang, or MSVC) on GitHub-hosted Linux or Windows runners, and expose the selected compiler paths as outputs.

## Features
- Installs GCC (default version from `GCC_VERSION`, default `14`) on Linux
- Installs Clang (`CLANG_VERSION`, default `20`) with optional libc++ and extras on Linux
- Configures MSVC build environment on Windows with selectable `arch`
- Sets up Ninja on all platforms
- Emits `c_compiler` and `cxx_compiler` outputs for downstream jobs

## Inputs
- `compiler` (required): One of `gcc`, `clang`, `msvc`
- `arch` (optional, Windows/MSVC only): Target architecture. Default: `x64`

## Environment variables
- `GCC_VERSION`: GCC major version to install (e.g., `14`). Default: `14`
- `CLANG_VERSION`: Clang major version to install (e.g., `20`). Default: `20`
- `INSTALL_EXTRAS`: When `true`, also installs `clang-format` and `clang-tidy` for the specified version. Default: `true`

## Outputs
- `c_compiler`: Path/name of the selected C compiler
  - GCC: `/usr/bin/gcc`
  - Clang: `/usr/bin/clang`
  - MSVC: `cl`
- `cxx_compiler`: Path/name of the selected C++ compiler
  - GCC: `/usr/bin/g++`
  - Clang: `/usr/bin/clang++`
  - MSVC: `cl`

## Supported runners
- Linux: `ubuntu-*`
- Windows: `windows-*` (MSVC only)

## Usage

### GCC on Ubuntu
```yaml
jobs:
  build-gcc:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v4
      - name: Install GCC toolchain
        uses: StormBytePP/install-toolchain@master
        with:
          compiler: gcc
        env:
          GCC_VERSION: "14"  # optional, defaults to 14
      - name: Print compilers
        run: |
          echo "CC: ${{ steps.tool.outputs.c_compiler }}"
          echo "CXX: ${{ steps.tool.outputs.cxx_compiler }}"
        id: show
```

### Clang on Ubuntu (with extras)
```yaml
jobs:
  build-clang:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v4
      - name: Install Clang toolchain
        id: tool
        uses: StormBytePP/install-toolchain@master
        with:
          compiler: clang
        env:
          CLANG_VERSION: "20"      # optional, defaults to 20
          INSTALL_EXTRAS: "true"   # optional, installs clang-format & clang-tidy
      - name: Use outputs
        run: |
          echo "CC: ${{ steps.tool.outputs.c_compiler }}"
          echo "CXX: ${{ steps.tool.outputs.cxx_compiler }}"
```

### MSVC on Windows
```yaml
jobs:
  build-msvc:
    runs-on: windows-2022
    steps:
      - uses: actions/checkout@v4
      - name: Configure MSVC environment
        id: tool
        uses: StormBytePP/install-toolchain@master
        with:
          compiler: msvc
          arch: x64  # x86, arm64 also supported
      - name: Verify cl
        shell: bash
        run: |
          where cl || where.exe cl
```

### Consuming outputs
To reliably consume the action outputs, set an `id` on the step and read from `steps.<id>.outputs.*`:
```yaml
- name: Install GCC
  id: tool
  uses: StormBytePP/install-toolchain@master
  with:
    compiler: gcc

- name: Configure build vars
  run: |
    echo "CC=${{ steps.tool.outputs.c_compiler }}" >> $GITHUB_ENV
    echo "CXX=${{ steps.tool.outputs.cxx_compiler }}" >> $GITHUB_ENV
```

## Notes
- On Linux, the action uses `apt` and `update-alternatives` to select the requested compiler version.
- For Clang, when `INSTALL_EXTRAS` is `true`, `clang-format-<ver>` and `clang-tidy-<ver>` are installed.
- On Windows, MSVC environment setup is performed via `ilammy/msvc-dev-cmd@v1` using the requested `arch`.
- Ninja is set up via `ashutoshvarma/setup-ninja@master` for faster CMake builds.
- The action validates `compiler` and fails fast on unsupported values.

## Troubleshooting
- If `apt` fails, ensure the runner image supports the requested versions.
- If `clang` or `gcc` commands don’t reflect the expected version, check that `GCC_VERSION`/`CLANG_VERSION` are set correctly and that `update-alternatives` ran without errors.
- For MSVC, ensure you’re on a Windows runner and using a supported `arch`.
