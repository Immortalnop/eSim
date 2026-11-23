# NGHDL on Ubuntu 25.04

`nghdl.zip` in the eSim 2.5 release only has install scripts for Ubuntu
22.04, 23.04 and 24.04. On 25.04 its launcher stops with
`Unsupported Ubuntu version: 25.04 ()`. Because the eSim installer runs with
`set -e`, the whole eSim install stops there, before the SKY130 PDK and the
`esim` launcher are installed.

`nghdl.zip` is not part of this branch, so the 25.04 support lives in this
folder. `installNghdl` in `../install-eSim-25.04.sh` copies these files into
`nghdl/` after unzipping and before running the NGHDL installer.

| File | Copied to | Based on |
| --- | --- | --- |
| `install-nghdl.sh` | `nghdl/install-nghdl.sh` | the launcher in `nghdl.zip` |
| `install-nghdl-25.04.sh` | `nghdl/install-nghdl-scripts/` | `install-nghdl-24.04.sh` in `nghdl.zip` |
| `model_generation.py` | `nghdl/src/model_generation.py` | the same file in `nghdl.zip` |

## Changes

`install-nghdl.sh`:

- Adds a `25.04` case that runs `install-nghdl-25.04.sh`.
- Fixes the version detection. The old pattern `\d+\.\d+\.\d+` needs three
  numbers, so `25.04` came out blank. The third number is now optional, with
  `VERSION_ID` as a fallback.

`install-nghdl-25.04.sh` (each change is marked with an `Ubuntu 25.04:`
comment):

- **LLVM 18 instead of the default LLVM 20.** GHDL 4.1.0's `configure`
  accepts LLVM up to 18.1 and stops with `Unhandled version llvm 20...`
  otherwise. Installs `llvm-18` and `llvm-18-dev`, and configures GHDL with
  `--with-llvm-config=/usr/bin/llvm-config-18`.
- **`clang-18` instead of `clang`.** GHDL builds its LLVM backend with
  `clang++`; the default `clang` package is clang 20. GHDL is configured with
  `CXX=clang++-18`.
- **No longer removes the system LLVM.** The 24.04 script runs
  `apt remove llvm llvm-dev`; LLVM 18 installs alongside LLVM 20, so this is
  not needed.
- **`libcanberra-gtk-module` dropped.** The GTK2 package no longer exists on
  25.04, and requesting it made the whole `apt install` line fail. Only
  `libcanberra-gtk3-module` is installed.
- **`libfl-dev` added.** Verilator needs `FlexLexer.h`. `flex` only
  recommends `libfl-dev`, so the build fails wherever recommended packages
  are turned off.

`model_generation.py` (NGHDL's model generator, used from the `nghdl` GUI):

- Two regular expressions were ordinary strings containing `\(` and `\)`.
  Python 3.12 and later, including 25.04's Python 3.13, report
  `SyntaxWarning: invalid escape sequence '\('`, and a future Python
  version will reject them. They are now raw strings. The upstream code had
  silenced the linter with `# noqa`, which is removed. Behaviour does not
  change: `"\("` and `r"\("` are the same two characters.

  ```diff
  -                item = re.sub("\(", " ", item, flags=re.I)      # noqa
  -                item = re.sub("\)", " ", item, flags=re.I)      # noqa
  +                item = re.sub(r"\(", " ", item, flags=re.I)
  +                item = re.sub(r"\)", " ", item, flags=re.I)
  ```

## Tested

**Full install.** On Ubuntu 25.04 under WSL2, the eSim installer with these
files completed on 20 November 2026 (`eSim Installed Successfully`). GHDL
4.1.0 (`llvm 18.1.8 code generator`), Verilator 4.210, NGHDL's ngspice and
the `nghdl` command were installed. On 23 November 2026, a clean install
from scratch with all fixes, including the `model_generation.py` fix,
completed as well. Details and logs are in
[`../../REPORT-25.04-aarush.md`](../../REPORT-25.04-aarush.md).

**Build checks before the full install.** On Ubuntu 25.04 (GCC 14.2,
GNAT 14.2, LLVM 18.1.8), using the sources from `nghdl.zip`:

- GHDL 4.1.0 builds against LLVM 18 and reports
  `llvm 18.1.8 code generator`; a testbench for
  `Example/combinational_logic/half_adder` gives the correct truth table.
- Verilator 4.210 builds with GCC 14 and compiles and runs a small Verilog
  model.
- ngspice (`nghdl-simulator-source`) builds with XSPICE and X11 support, and
  its code models, including `ghdl.cm` and `Ngveri.cm`, are built.
