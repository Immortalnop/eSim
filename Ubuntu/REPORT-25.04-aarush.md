# Installing eSim 2.5 on Ubuntu 25.04: issues and fixes

**Author:** Aarush Mohan Mathur
**Task:** eSim Semester Long Internship, Autumn 2026, Screening Task 4 (eSim Upgradation)
**Branch:** [`installers`](https://github.com/Immortalnop/eSim/tree/installers) of the fork [Immortalnop/eSim](https://github.com/Immortalnop/eSim) (work done on `ubuntu25-fixes`)
**Tested on:** 20 November 2026 (finding and fixing), 23 November 2026 (clean install with all fixes)

## Summary

The eSim 2.5 release does not install on Ubuntu 25.04, and the newer scripts
in the `installers` branch did not install it either. This report lists
**12 issues** found while installing eSim 2.5 on Ubuntu 25.04. **9 are fixed**
in this fork, 1 had already been fixed upstream, and 2 are reported but not
fixed. With the fixes, the installer completes and eSim starts. A clean
install from scratch with all fixes, using the published branch, completed
unattended on 23 November 2026.

| # | Issue | Severity | Status | Commit |
| --- | --- | --- | --- | --- |
| 1 | `install-eSim.sh` in the `installers` branch has a syntax error from a bad merge | Critical | Fixed | [`7101a46`](https://github.com/Immortalnop/eSim/commit/7101a46e55e78fa10ed71d63faf553bc39db6f4a) |
| 2 | Ubuntu version is detected as blank on `x.y` releases such as 25.04 | Low | Fixed | [`7101a46`](https://github.com/Immortalnop/eSim/commit/7101a46e55e78fa10ed71d63faf553bc39db6f4a), [`5de53ce`](https://github.com/Immortalnop/eSim/commit/5de53cef1848c82a3a74aeebe4c4fc4645b6357b) |
| 3 | The eSim 2.5 release zip does not support Ubuntu 25.04 at all | Critical | Already fixed upstream in the `installers` branch | n/a |
| 4a | NGHDL installer has no Ubuntu 25.04 support, which stops the whole eSim install | Critical | Fixed | [`5de53ce`](https://github.com/Immortalnop/eSim/commit/5de53cef1848c82a3a74aeebe4c4fc4645b6357b) |
| 4b | GHDL 4.1.0 cannot be built with Ubuntu 25.04's default LLVM 20 | Critical | Fixed | [`5de53ce`](https://github.com/Immortalnop/eSim/commit/5de53cef1848c82a3a74aeebe4c4fc4645b6357b) |
| 4c | `libcanberra-gtk-module` no longer exists, so an NGHDL `apt install` line fails | Critical | Fixed | [`5de53ce`](https://github.com/Immortalnop/eSim/commit/5de53cef1848c82a3a74aeebe4c4fc4645b6357b) |
| 5 | `python3-virtualenv` is installed without `-y`, so the installer stops at a prompt | Medium | Fixed | [`85bdc47`](https://github.com/Immortalnop/eSim/commit/85bdc474658d1a07cd9f216c78e42ed1f5f48438) |
| 6 | eSim's symbol table is copied only to KiCad 6's config folder; KiCad 8 does not see the eSim symbols | High | Fixed | [`3fb715f`](https://github.com/Immortalnop/eSim/commit/3fb715fe5922ec0b263c8ce908f14c811a3a3d30) |
| 7 | When KiCad 8 is already installed, the installer exits early but reports success | High | Fixed | [`019dc91`](https://github.com/Immortalnop/eSim/commit/019dc91ea3cfe8e922711222e164c2e2de580ea5) |
| 8 | NGHDL's `model_generation.py` has invalid escape sequences (Python 3.12+ warning) | Low | Fixed | [`6306b3f`](https://github.com/Immortalnop/eSim/commit/6306b3f78d6646f74ca01600bb1082b35b4c9752) |
| 9 | The eSim Welcome page is blank because `Welcome.py` looks for `welcome.html` in the wrong folder | Low | Reported, not fixed | n/a |
| 10 | Two desktop-shortcut steps fail under WSL (not an eSim bug) | Low, WSL only | Reported, not fixed | n/a |

Severity follows the task's evaluation criteria:

- **Critical:** eSim (its main GUI) cannot be installed.
- **High:** the install completes or claims to, but a core part of eSim is
  missing or broken.
- **Medium:** the install needs manual input to continue.
- **Low:** cosmetic, a warning, or a future problem.

Issue 4 is split into three because each one alone stops the NGHDL install,
and so the eSim install. Only 4a showed up in a test run. 4b and 4c were
found by checking the next steps of the same script against Ubuntu 25.04's
packages and GHDL's source before running it (see issues 4b and 4c below).

## Contents

- [Environment](#environment)
- [Method](#method)
- [Test runs](#test-runs)
- [Issues and fixes](#issues-and-fixes)
- [Suspected issues that turned out not to be problems](#suspected-issues-that-turned-out-not-to-be-problems)
- [Result](#result)
- [What has not been tested yet](#what-has-not-been-tested-yet)
- [How to install eSim 2.5 on Ubuntu 25.04 with these fixes](#how-to-install-esim-25-on-ubuntu-2504-with-these-fixes)
- [Files changed](#files-changed)
- [Commits](#commits)
- [Evidence](#evidence)

## Environment

| Item | Value |
| --- | --- |
| Host | Windows laptop, WSL2 (kernel `6.18.33.2-microsoft-standard-WSL2`), GUI through WSLg |
| OS | Ubuntu 25.04 "plucky" (`lsb_release -a`: `Ubuntu 25.04`, `Release: 25.04`) |
| Python | 3.13.3 |
| GCC | 14.2.0 (`14.2.0-19ubuntu2`) |
| Default LLVM | 20 (`llvm 1:20.0-63ubuntu1`); LLVM 18.1.8 used for GHDL |
| GNAT (Ada) | 14 (`gnat 14.1ubuntu1`) |
| KiCad | 8.0.8 (`8.0.8+dfsg-1` from Ubuntu's `universe` repository) |
| eSim | `eSim-2.5.zip` from [esim.fossee.in/downloads](https://esim.fossee.in/downloads), SHA256 `b9860ae26f026930bae589520761f4a57b85d593a2e2cd1ac83d5e5db2f281ed` |

Ubuntu 25.04 is end-of-life and is not offered by `wsl --install`, so it was
built as its own WSL distribution with `debootstrap` from
`archive.ubuntu.com` (suite `plucky`), then imported with `wsl --import`.
The system was fully updated (`apt upgrade`) before testing. A snapshot of
this clean state (`wsl --export`) made it possible to go back to it.

## Method

1. **Two versions of the installer were tested.**
   - The official release, `eSim-2.5.zip`.
   - The newer scripts in the `installers` branch. The branch only holds
     installer scripts, not the eSim source, `library/` or `nghdl.zip`, so
     its `Ubuntu/install-eSim.sh` and `Ubuntu/install-eSim-scripts/` were
     copied over the extracted zip and run from there.
2. **Every run was recorded** with
   `script -c "./install-eSim.sh --install" <log>`, and the logs were kept.
3. **For each failure:** find the failing line in the log, trace it to the
   script, find the root cause, fix it in the repository, commit, copy the
   scripts into `~/eSim-2.5` and run again.
4. **Each fix was checked against the original.** For example, GHDL's own
   `configure` script, `apt-cache policy` for package availability,
   `bash -n` for syntax, and `git log` to find the commit that introduced a
   bug.
5. **Build checks without installing.** Before running the full installer,
   the NGHDL build (GHDL, Verilator, ngspice) was tried with the exact
   package versions from Ubuntu 25.04. The `.deb` packages were downloaded
   with `apt-get download` and unpacked into a private folder, so nothing was
   installed system-wide.
6. **KiCad 8 checks without the GUI.** `kicad-cli sch erc`, KiCad 8's
   command-line electrical rules check, reports a symbol library that is
   missing from the configuration. It was used with `KICAD_CONFIG_HOME`
   pointing at a test folder to compare configurations.
7. **Final clean install.** A separate WSL distribution was created from the
   clean snapshot (no eSim, KiCad or `python3-virtualenv`). The installer
   files were taken from the published branch with
   `git archive origin/ubuntu25-fixes Ubuntu/install-eSim.sh Ubuntu/install-eSim-scripts`,
   copied over a fresh unzip of `eSim-2.5.zip`, and run with `n` piped in as
   the answer to the proxy question and no other input. Any other prompt
   would have received end-of-input and failed the run. In that test copy
   only, `sudo` was set not to ask for a password so the run could be
   unattended.

## Test runs

| Run | Time (UTC, 20 Nov 2026) | What was run | Result | Log |
| --- | --- | --- | --- | --- |
| A | 10:17 | Release `eSim-2.5.zip`, unchanged | Stops at once: `Unsupported Ubuntu version: 25.04 ()` (issues 2, 3) | [`1-release-zip.log`](report-25.04-aarush/logs/1-release-zip.log) |
| B | 11:00 | `installers` branch scripts | Does not start: syntax error at line 70 (issue 1) | [`2-installers-branch-syntax-error.log`](report-25.04-aarush/logs/2-installers-branch-syntax-error.log) |
| C | 13:38 | Branch + fixes 1, 2 | Installs dependencies, Python packages and KiCad, then stops in NGHDL (issue 4a); prompt seen (issue 5) | [`3-after-syntax-fix-nghdl-fails.log`](report-25.04-aarush/logs/3-after-syntax-fix-nghdl-fails.log) |
| D | 16:24 | + NGHDL fix (4a–4c) | Stops after the KiCad step with exit code 0 (issue 7) | [`4-rerun-stops-after-kicad.log`](report-25.04-aarush/logs/4-rerun-stops-after-kicad.log) |
| E | 16:40 | + KiCad fix (7) | **Completes:** `eSim Installed Successfully`, exit code 0 | [`5-install-success.log`](report-25.04-aarush/logs/5-install-success.log) |
| F | 23 Nov, 15:49–16:22 | Clean install, all fixes (published branch) | Runs unattended until KiCad, then stops: Launchpad's PPA server returned `503 Service Unavailable` for one KiCad package (a server outage, not an eSim issue) | [`6-clean-install-launchpad-503.log`](report-25.04-aarush/logs/6-clean-install-launchpad-503.log) |
| G | 23 Nov, 16:22–16:47 | Same system, installer run again | **Completes:** `eSim Installed Successfully`, exit code 0, no prompts | [`7-clean-install-retry-success.log`](report-25.04-aarush/logs/7-clean-install-retry-success.log) |

Fixes 5, 6 and 8 were made after Run E. They were first checked on their
own (see each issue), then confirmed together in the clean install (Runs F
and G).

## Issues and fixes

### Issue 1: syntax error in `install-eSim.sh` (Critical, fixed)

**Symptom.** The `installers` branch installer does not start on any Ubuntu
version (Run B):

```
./install-eSim.sh: line 70: syntax error near unexpected token `}'
./install-eSim.sh: line 70: `}'
Script done on 2026-11-20 11:00:02+00:00 [COMMAND_EXIT_CODE="2"]
```

**Root cause.** In `run_version_script()`, the `if [[ -f "$SCRIPT" ]]` block
lost its `else` and `fi`. In their place were about 250 lines of old
functions (`installNghdl`, `installKicad`, `createDesktopStartScript`, …)
from the per-version scripts. Without `fi`, bash reaches the function's
closing `}` while still inside the `if`, and refuses to run the file.

Checking each commit that changed the file with `bash -n` shows where it
broke:

| Commit | `bash -n Ubuntu/install-eSim.sh` |
| --- | --- |
| `f4cede1` | parses |
| `5224460` | parses |
| `52d77f6` "Merge branch 'installers' into installers" (29 May 2026) | `line 70: syntax error near unexpected token '}'` |
| `a77d6b9` (branch head before this work) | same error |

**Fix.** Restore the end of the `if` block, and remove the pasted functions.
The launcher never calls them; each per-version script has its own copy.
The file goes from 342 lines to 93.

```diff
     if [[ -f "$SCRIPT" ]]; then
         echo "Running script: $SCRIPT $ARGUMENT"
         bash "$SCRIPT" "$ARGUMENT"
-    echo "[eSim]" >> $config_dir/$config_file
-    ... about 250 lines of pasted functions ...
+    else
+        echo "Script not found for Ubuntu version: $VERSION_ID ($FULL_VERSION)"
+        exit 1
+    fi
+}
```

**Verification.** `bash -n` passes. In Run C the launcher prints
`Running script: …/install-eSim-25.04.sh --install` (log line 4) and the
install continues.

### Issue 2: Ubuntu version detected as blank (Low, fixed)

**Symptom.** Run A, release zip:

```
Detected Ubuntu Version: 
Unsupported Ubuntu version: 25.04 ()
```

**Root cause.** The launcher reads the version with

```bash
FULL_VERSION=$(lsb_release -d | grep -oP '\d+\.\d+\.\d+')
```

The pattern needs three numbers (`22.04.4`). Ubuntu 25.04 reports
`Ubuntu 25.04`, so nothing matches and `FULL_VERSION` is empty. This
only affects the messages (`Detected Ubuntu Version:` and the `()` in the
error). The install script is chosen with `VERSION_ID` from
`/etc/os-release`, which is correct. The same line is in NGHDL's launcher
inside `nghdl.zip`.

**Fix.** Make the third number optional, keep only the first match, and fall
back to `VERSION_ID`:

```diff
-    FULL_VERSION=$(lsb_release -d | grep -oP '\d+\.\d+\.\d+')
+    FULL_VERSION=$(lsb_release -d | grep -oP '\d+\.\d+(\.\d+)?' | head -n1)
+    FULL_VERSION=${FULL_VERSION:-$VERSION_ID}
```

Applied to `Ubuntu/install-eSim.sh` (`7101a46`) and to the NGHDL launcher
(`5de53ce`, see issue 4a).

**Verification.** Run C log line 3 and Run E log line 170 (NGHDL) both show
`Detected Ubuntu Version: 25.04`.

### Issue 3: the release zip has no Ubuntu 25.04 support (Critical, already fixed upstream)

**Symptom.** Run A stops at once with `Unsupported Ubuntu version: 25.04 ()`
and installs nothing ([screenshot](report-25.04-aarush/images/01-release-zip-unsupported.png)).

**Root cause.** The release's `install-eSim.sh` only has cases for 22.04,
23.04 and 24.04, and `install-eSim-scripts/` has no 25.04 script.

**Status.** Already handled in the `installers` branch, which adds a
`"25.04"` case and `install-eSim-25.04.sh`. Credit goes to the upstream
contributors. It is listed here because a user who downloads eSim 2.5 today
hits it first.

### Issue 4a: NGHDL has no Ubuntu 25.04 support (Critical, fixed)

**Symptom.** With issues 1 and 2 fixed, Run C installs the dependencies, the
Python packages and KiCad, then fails inside NGHDL (log lines 2106–2166):

```
Installing NGHDL...........................
Archive:  nghdl.zip
...
Detected Ubuntu Version: 
Unsupported Ubuntu version: 25.04 ()
Script done on 2026-11-20 13:38:03+00:00 [COMMAND_EXIT_CODE="1"]
```

**Root cause.** `installNghdl` in `install-eSim-25.04.sh` unzips `nghdl.zip`
and runs `nghdl/install-nghdl.sh`. That launcher, from the eSim 2.5 release,
has the issue 2 version pattern and cases only for 22.04, 23.04 and 24.04.
Its `install-nghdl-scripts/` folder has no 25.04 script.

The eSim installer runs with `set -e`, so this failure ends the whole eSim
install. The SKY130 PDK and the desktop launcher are never installed, and
there is no `esim` command. eSim cannot be started at all.

The upstream [`REPORT.md`](REPORT.md) describes NGHDL changes for 25.04
(LLVM 18, `llvm-config-18`, dropping `libcanberra-gtk-module`). However, no
NGHDL 25.04 script was committed to the branch, and `nghdl.zip` is not in
the branch, so these changes never ran.

**Fix.** `nghdl.zip` is not part of the branch, so the 25.04 support is
added as files in a new folder,
[`install-eSim-scripts/nghdl-25.04/`](install-eSim-scripts/nghdl-25.04/).
`installNghdl` copies them into `nghdl/` after unzipping. This follows the
branch's existing one-script-per-Ubuntu-version layout.

| File | Copied to | Based on |
| --- | --- | --- |
| `install-nghdl.sh` | `nghdl/install-nghdl.sh` | the launcher in `nghdl.zip`, plus a `"25.04"` case and the issue 2 fix |
| `install-nghdl-25.04.sh` | `nghdl/install-nghdl-scripts/` | `install-nghdl-24.04.sh` in `nghdl.zip`, with the changes for 4b and 4c |
| `model_generation.py` | `nghdl/src/` | see issue 8 |

```diff
 # install-eSim-25.04.sh
 eSim_Home=`pwd`
 ngspiceFlag=0
+# Ubuntu 25.04 support files for NGHDL, kept next to this script
+nghdl_patch_dir="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)/nghdl-25.04"
 ...
     unzip -o nghdl.zip
+
+    # nghdl.zip from eSim 2.5 only supports Ubuntu 22.04 to 24.04.
+    # Add the 25.04 launcher and install script before running it.
+    cp "$nghdl_patch_dir/install-nghdl.sh" nghdl/
+    cp "$nghdl_patch_dir/install-nghdl-25.04.sh" nghdl/install-nghdl-scripts/
```

Every change in `install-nghdl-25.04.sh` is marked with an `Ubuntu 25.04:`
comment. The folder's [README](install-eSim-scripts/nghdl-25.04/README.md)
explains each file.

**Verification.** Run E, log lines 170–171:

```
Detected Ubuntu Version: 25.04
Running script: /home/amm/eSim-2.5/nghdl/install-nghdl-scripts/install-nghdl-25.04.sh --install
```

NGHDL then installs completely (see [Result](#result)).

### Issue 4b: GHDL 4.1.0 does not build with LLVM 20 (Critical, fixed)

**Symptom (would have been next).** The 24.04 NGHDL script installs the
default `llvm` package and configures GHDL with `/usr/bin/llvm-config`. On
Ubuntu 25.04 that is LLVM 20.

**Root cause.** GHDL 4.1.0's `configure` (in `nghdl/ghdl-4.1.0.tar.gz`) only
accepts LLVM versions up to 18.1:

```sh
  elif check_version  6.0 $llvm_version ||
       ...
       check_version 17.0 $llvm_version ||
       check_version 18.1 $llvm_version ||
       false; then
    echo "Debugging is enabled with llvm $llvm_version"
  else
    echo "Unhandled version llvm $llvm_version"
    exit 1
```

On Ubuntu 25.04, `apt-cache policy` shows `llvm` 20 as the default, with
`llvm-18` (1:18.1.8-17) also available.

**Fix** (in `install-nghdl-25.04.sh`, following the `llvm_version` variable
already used by the 23.04 script):

```diff
+llvm_version="18"
 ...
-    echo "Removing older LLVM........................................"
-    sudo apt remove -y llvm llvm-dev
-    echo "Installing LLVM........................................"
-    sudo apt install -y llvm llvm-dev
-    echo "Installing Clang.........................................."
-    sudo apt install -y clang
+    echo "Installing LLVM-${llvm_version}........................................"
+    sudo apt install -y llvm-${llvm_version} llvm-${llvm_version}-dev
+    echo "Installing Clang-${llvm_version}.........................................."
+    sudo apt install -y clang-${llvm_version}
 ...
-    ./configure --with-llvm-config=/usr/bin/llvm-config
+    ./configure CXX=clang++-${llvm_version} --with-llvm-config=/usr/bin/llvm-config-${llvm_version}
```

- GHDL's `configure` builds and links its LLVM backend with `clang++`
  (`CXX=${CXX:-clang++}`). The default `clang` package is clang 20, so
  `clang-18` is installed and named explicitly.
- The step that removed the system LLVM is dropped. LLVM 18 installs
  alongside LLVM 20, so removing it is unnecessary and could remove
  something the user needs.

**Verification.** Run E log line 14718:
`Debugging is enabled with llvm 18.1.8`; line 16050:
`GHDL installed successfully`. Afterwards `ghdl --version` reports
`GHDL 4.1.0 … Compiled with GNAT Version: 14.2.0 … llvm 18.1.8 code generator`.
A testbench for NGHDL's own example
`Example/combinational_logic/half_adder`, run with the installed `ghdl`,
gives the correct truth table:

```
a='0' b='0' sum='0' carry='0'
a='1' b='0' sum='1' carry='0'
a='0' b='1' sum='1' carry='0'
a='1' b='1' sum='0' carry='1'
```

### Issue 4c: `libcanberra-gtk-module` no longer exists (Critical, fixed)

**Root cause.** The 24.04 NGHDL script runs
`sudo apt install -y libcanberra-gtk-module libcanberra-gtk3-module`. On
Ubuntu 25.04, `apt-cache policy libcanberra-gtk-module` shows
`Candidate: (none)`: the GTK2 package was removed. `apt` fails the whole
command when one package is missing, and with `set -e` this stops the
install.

**Fix:**

```diff
-    sudo apt install -y libcanberra-gtk-module libcanberra-gtk3-module
+    sudo apt install -y libcanberra-gtk3-module
```

**Verification.** Run E log line 394 (`Installing Gtk Canberra modules`)
passes and the install continues.

**Also changed in the same script (not counted as issues).** Verilator needs
`FlexLexer.h`, which comes from `libfl-dev`. `flex` only *recommends*
`libfl-dev`, so the build fails on systems that do not install recommended
packages. A test build without it failed with
`fatal error: FlexLexer.h: No such file or directory`. `libfl-dev` is now
installed explicitly.

### Issue 5: installer stops at a `[Y/n]` prompt (Medium, fixed)

**Symptom.** Run C log line 83: `Continue? [Y/n] y`. The run only continued
because `y` was typed.

**Root cause.** In `install-eSim-25.04.sh`:
`sudo apt install python3-virtualenv` has no `-y`. Every other `apt`
install, remove and purge in the eSim and NGHDL 25.04 scripts already has
`-y`.

**Fix:**

```diff
-    sudo apt install python3-virtualenv
+    sudo apt install -y python3-virtualenv
```

**Verification.** This was the only `apt` command in the script without
`-y`. The prompt only appears when the package is not yet installed, so the
clean install (Run F) is the real test. There, `python3-virtualenv` was
installed with no input (log lines 13–513, virtual environment created at
line 513), and neither clean-run log contains a `[Y/n]` prompt.

### Issue 6: KiCad 8 does not see the eSim symbol libraries (High, fixed)

**Symptom.** After the install, `~/.config/kicad/` contains only
`6.0/sym-lib-table`.

**Root cause.** `copyKicadLibrary` copies eSim's symbol library table (which
lists 15 eSim libraries such as `eSim_Analog`, `eSim_Devices`,
`eSim_Sources`) only to `~/.config/kicad/6.0/`. Ubuntu 25.04 installs
KiCad 8, which reads its settings from `~/.config/kicad/8.0/`.

KiCad 8's own checker confirms this. A test schematic containing one eSim
symbol (`eSim_Analog:TRANSFO`) was checked with
`kicad-cli sch erc` with three configurations:

| KiCad 8 configuration | `kicad-cli sch erc` result |
| --- | --- |
| eSim table only in `6.0` (as installed) | `The current configuration does not include the library 'eSim_Analog'` |
| eSim table copied unchanged to `8.0` | library found and loaded |
| eSim table in `8.0` with a broken library path (control) | `Symbol 'TRANSFO' not found in symbol library 'eSim_Analog'` |

The table addresses libraries through `${KICAD6_SYMBOL_DIR}`. The second row
shows that KiCad 8 still resolves this variable, so the table does not need
editing. To confirm the library was really read, the symbol in the
schematic was altered, and KiCad 8 reported that it differs from the
library: `Symbol 'TRANSFO' has been modified in library 'eSim_Analog'`.

**Fix.** Also copy the table into the KiCad 8 folder. The 6.0 copy is kept
for KiCad 6. An existing KiCad 8 table may hold the user's own libraries, so
it is kept as a numbered backup instead of being overwritten.

```diff
     cp kicadLibrary/template/sym-lib-table ~/.config/kicad/6.0/
+
+    # KiCad 8 reads its settings from ~/.config/kicad/8.0, not 6.0, so
+    # copy the table there too. Any existing KiCad 8 table is kept as a
+    # numbered backup (sym-lib-table.~1~), since it may hold the user's
+    # own libraries.
+    mkdir -p ~/.config/kicad/8.0
+    cp --backup=numbered kicadLibrary/template/sym-lib-table ~/.config/kicad/8.0/
```

**Verification.** The updated `copyKicadLibrary` was run with a test home
folder and `sudo` stubbed out. The table was written to both `6.0` and
`8.0`, and `kicad-cli sch erc` then loaded `eSim_Analog` from it. Running it
a second time kept the earlier table as `sym-lib-table.~1~`.

In the clean install (Run G), the installer created
`~/.config/kicad/8.0/sym-lib-table` listing all 15 eSim libraries, and
`kicad-cli sch erc` using that real configuration loaded `eSim_Analog`.

### Issue 7: installer exits early when KiCad 8 is already installed (High, fixed)

**Symptom.** Run D stops right after the KiCad step and reports success. The
log is 124 lines long and ends with:

```
Installing KiCad...........................
Ubuntu 25.04 detected.
KiCad 8.0 is already installed.

Script done on 2026-11-20 16:24:47+00:00 [COMMAND_EXIT_CODE="0"]
```

`ghdl`, `verilator`, `ngspice`, `nghdl` and `esim` were not installed.

**Root cause.** In `installKicad`:

```bash
            else
                echo "KiCad 8.0 is already installed."
                exit 0
            fi
```

`exit 0` ends the whole installer, not just the function, and with a success
code. `copyKicadLibrary`, `installNghdl`, `installSky130Pdk` and
`createDesktopStartScript` are skipped. This affects anyone who already has
KiCad 8, or who runs the installer again after a failure. Because the exit
code is 0 and nothing looks wrong, it is easy to miss.

**Fix.** Return from the function instead, so the installer only skips the
KiCad install:

```diff
                 echo "KiCad 8.0 is already installed."
-                exit 0
+                # return (not exit) so the rest of the installer still runs
+                return 0
```

The `exit 1` a few lines above, used when the user declines to replace a
different KiCad version, is a deliberate abort and is unchanged.

**Verification.** Run E log lines 124–127: after
`KiCad 8.0 is already installed.` the installer continues with the library
copy and `Installing NGHDL...`, and completes.

### Issue 8: invalid escape sequences in NGHDL's `model_generation.py` (Low, fixed)

**Symptom.** With Python 3.13:

```
model_generation.py:46: SyntaxWarning: invalid escape sequence '\('
model_generation.py:47: SyntaxWarning: invalid escape sequence '\)'
```

**Root cause.** Two regular expressions are written as ordinary strings. The
upstream code had silenced the linter with `# noqa` instead of fixing them.
Python 3.12 and later warn about this, and a future version will reject it.

**Fix.** Use raw strings. The file is in `nghdl.zip`, so a corrected copy is
added to `nghdl-25.04/` and copied into `nghdl/src/` by `installNghdl`.
Behaviour does not change: `"\("` and `r"\("` are the same two characters.

```diff
-                item = re.sub("\(", " ", item, flags=re.I)      # noqa
-                item = re.sub("\)", " ", item, flags=re.I)      # noqa
+                item = re.sub(r"\(", " ", item, flags=re.I)
+                item = re.sub(r"\)", " ", item, flags=re.I)
```

**Verification.**
- With warnings treated as errors, the original file fails to compile and
  the fixed copy compiles.
- The old and new expressions give identical results on sample VHDL port
  lines.
- The module imports with the system Python used by the `nghdl` command.
- `installNghdl`, run in a test folder, puts the fixed file in place.
- In the clean install (Run G), the installed
  `~/eSim-2.5/nghdl/src/model_generation.py` is the fixed copy and compiles
  with warnings treated as errors.

All Python files in eSim's `src/` and NGHDL's `src/` were compiled with
Python 3.13, and these two lines are the only escape-sequence warnings.

### Issue 9: the eSim Welcome page is blank (Low, reported, not fixed)

**Symptom.** eSim starts, but the Welcome panel is empty, and the terminal
shows
([screenshot](report-25.04-aarush/images/03-esim-running.png)):

```
QTextBrowser: No document for file:///home/amm/eSim-2.5/src/browser/library/browser/welcome.html
```

**Root cause.** `src/browser/Welcome.py` builds the path relative to its own
folder when eSim is not a frozen executable:

```python
base_path = os.path.dirname(sys.executable) if getattr(sys, 'frozen', False) else os.path.dirname(__file__)
html_path = os.path.abspath(os.path.join(base_path, "library/browser/welcome.html"))
```

From source, this gives `src/browser/library/browser/welcome.html`, but the
file is at `~/eSim-2.5/library/browser/welcome.html`, two folders up.

**Status.** Not fixed here. This is a bug in the eSim source (`src/`, on the
`master` branch), not in the installer, and it is not specific to Ubuntu
25.04.

### Issue 10: desktop shortcut steps under WSL (Low, reported, not fixed)

Run E log line 24321: `gio: Setting attribute metadata::trusted not supported`.
`createDesktopStartScript` marks the desktop shortcut as trusted with `gio`,
which needs a desktop session. WSL has none, and `~/Desktop` only exists
because it was created by hand during setup. The script already ignores
errors around this command, so the install continues. This is a WSL
limitation, not an eSim bug, and needs no change for a normal Ubuntu
desktop.

### Minor observations (not counted)

- `install-eSim-25.04.sh` installs `hdlparse` twice: from the `pyhdlparser`
  GitHub tarball, then as `pip3 install hdlparse`, which is already
  satisfied (Run C log lines 1726–1739 and 1787–1788).
- The message `Instaling virtualenv` is misspelt.
- `rm: cannot remove '/usr/bin/ngspice': No such file or directory` (Run E
  log line 24313) is expected on a first install, and the NGHDL script
  ignores it on purpose.
- KiCad is installed from two sources at once. `kicad` is pinned to
  `8.0.8+dfsg-1`, which comes from Ubuntu's `universe` repository, but
  `kicad-libraries`, `kicad-symbols` and `kicad-templates` are not pinned,
  so they come from the KiCad PPA as `8.0.9~ubuntu25.04.1` (Run F log lines
  1902–1911). This worked in every run but mixes two KiCad releases.
- Because those packages come from the KiCad PPA, the install depends on
  Launchpad being available. In Run F, Launchpad returned
  `503 Service Unavailable` (log lines 1911–1915). The installer stopped
  with `Error! Kindly resolve above error(s) and try again.`, which is the
  correct behaviour, and running it again (Run G) worked.

## Suspected issues that turned out not to be problems

These were suspected from reading the 25.04 script before running it. Run C
showed they are not problems:

| Suspected issue | Result |
| --- | --- |
| The pinned `kicad=8.0.8+dfsg-1` may not exist for 25.04 | Installed from Ubuntu's own `universe` repository (Run C log line 1943); the KiCad PPA only offers 8.0.9 |
| `pip3 install hdlparse` may fail (`use_2to3` removed from setuptools) | The script first installs `pyhdlparser` from GitHub, which provides `hdlparse` 1.0.7 (log line 1739); the later `pip3 install hdlparse` is already satisfied |
| `pip3` may hit PEP 668 ("externally-managed-environment") | Not hit; all `pip3` calls run inside the virtual environment `~/.esim/env` |

## Result

With fixes 1, 2, 4a–4c and 7, Run E completes
([log](report-25.04-aarush/logs/5-install-success.log),
[screenshot](report-25.04-aarush/images/02-install-success.png)), and eSim
starts ([screenshot](report-25.04-aarush/images/03-esim-running.png)).

| Check | Result |
| --- | --- |
| Installer | `eSim Installed Successfully`, exit code 0 (log lines 24323, 24327) |
| GHDL | `/usr/local/bin/ghdl`: GHDL 4.1.0, GNAT 14.2.0, llvm 18.1.8 code generator; half-adder testbench gives the correct truth table |
| Verilator | 4.210 (`verilator --version`) |
| ngspice | `/usr/bin/ngspice` points to NGHDL's ngspice-35; XSPICE code models built, including `ghdl.cm` and `Ngveri.cm` |
| NGHDL | `/usr/local/bin/nghdl` link and `~/.nghdl/config.ini` created |
| SKY130 PDK | copied (log line 24317) |
| eSim | `/usr/bin/esim` and `esim.desktop` installed; `esim` opens the eSim window through WSLg |
| eSim Python code | the main `Application` module and the `kicadtoNgspice`, `maker`, `projManagement` and `ngspicetoModelica` packages import cleanly with Python 3.13.3, matplotlib 3.11.2 and PyQt5 5.15.11 (offscreen, no window). Command-line converter scripts that read their arguments on start-up were not import-tested. |
| Simulation backend | eSim's `Examples/RC/RC.cir.out`, run with the installed ngspice, gives `v(out)` = 4.966 V at 50 ms, as expected for R = 1 kΩ, C = 10 µF with a 5 V step |

### Clean install with all fixes (23 November 2026)

The final check installed eSim from scratch on a fresh copy of the clean
Ubuntu 25.04 snapshot, using the installer files from the published branch
(see [Method](#method), step 7). The first attempt (Run F) stopped when
Launchpad's server was unavailable. Running the installer again on the same
system (Run G) completed
([log](report-25.04-aarush/logs/7-clean-install-retry-success.log)).

| Check (clean install) | Result |
| --- | --- |
| Installer | `eSim Installed Successfully`, exit code 0 (Run G log lines 24540, 24544); no input other than the proxy answer, and no `[Y/n]` prompt in either log (issue 5) |
| NGHDL | `Detected Ubuntu Version: 25.04` and hand-off to `install-nghdl-25.04.sh` (lines 392–393); `Debugging is enabled with llvm 18.1.8` (line 14937); GHDL, Verilator and NGHDL installed (lines 16269, 20849, 24528) |
| Installed tools | GHDL 4.1.0 with GNAT 14.2.0 and llvm 18.1.8 code generator; Verilator 4.210; `ngspice` points to NGHDL's ngspice-35; `nghdl` and `esim` installed |
| KiCad 8 symbol table (issue 6) | `~/.config/kicad/8.0/sym-lib-table` created with all 15 eSim libraries; `kicad-cli sch erc` loads `eSim_Analog` from it |
| `model_generation.py` (issue 8) | Installed copy is the fixed file and compiles with warnings treated as errors |
| GHDL simulation | Half-adder testbench gives the correct truth table |
| eSim RC example | `v(out)` = 4.966 V at 50 ms |

## What has not been tested yet

- **KiCad 8's first-start dialog.** On first launch, KiCad 8 may offer to
  import settings from version 6.0. The fix for issue 6 works with the
  `kicad-cli` checker, but the dialog itself has not been checked in the
  GUI.
- **Simulating from the eSim GUI**, and NGHDL or NgVeri model generation
  from the GUI.

## How to install eSim 2.5 on Ubuntu 25.04 with these fixes

The `installers` branch only contains installer scripts, so they are copied
over the eSim 2.5 release:

```bash
# 1. Get the eSim 2.5 release (from https://esim.fossee.in/downloads) and unpack it
cd ~
unzip eSim-2.5.zip

# 2. Get the fixed installer scripts
git clone -b installers https://github.com/Immortalnop/eSim.git eSim-repo

# 3. Copy them over the release and run the installer
cp ~/eSim-repo/Ubuntu/install-eSim.sh ~/eSim-2.5/
cp -r ~/eSim-repo/Ubuntu/install-eSim-scripts ~/eSim-2.5/
cd ~/eSim-2.5
chmod +x install-eSim.sh
./install-eSim.sh --install

# 4. Start eSim
esim
```

The installer asks for your `sudo` password and whether you are behind a
proxy. The NGHDL step builds GHDL, Verilator and ngspice from source, which
takes several minutes.

## Files changed

| File | Change |
| --- | --- |
| [`install-eSim.sh`](install-eSim.sh) | Issue 1 (syntax error), issue 2 (version detection) |
| [`install-eSim-scripts/install-eSim-25.04.sh`](install-eSim-scripts/install-eSim-25.04.sh) | Issue 4a (copy NGHDL 25.04 files), 5 (`-y`), 6 (KiCad 8 table), 7 (`return`), 8 (copy fixed `model_generation.py`) |
| [`install-eSim-scripts/nghdl-25.04/install-nghdl.sh`](install-eSim-scripts/nghdl-25.04/install-nghdl.sh) | New: NGHDL launcher with a 25.04 case and the issue 2 fix |
| [`install-eSim-scripts/nghdl-25.04/install-nghdl-25.04.sh`](install-eSim-scripts/nghdl-25.04/install-nghdl-25.04.sh) | New: NGHDL install script for 25.04 (issues 4b, 4c) |
| [`install-eSim-scripts/nghdl-25.04/model_generation.py`](install-eSim-scripts/nghdl-25.04/model_generation.py) | New: NGHDL model generator with issue 8 fixed |
| [`install-eSim-scripts/nghdl-25.04/README.md`](install-eSim-scripts/nghdl-25.04/README.md) | New: what the NGHDL files are and why |
| `README.md`, this report, `report-25.04-aarush/` | Documentation and evidence |

## Commits

On top of `a77d6b9` (head of `installers` before this work):

| Commit | Description |
| --- | --- |
| [`7101a46`](https://github.com/Immortalnop/eSim/commit/7101a46e55e78fa10ed71d63faf553bc39db6f4a) | Fix syntax error in `install-eSim.sh` from bad merge (`52d77f6`); fix empty version detection (issues 1, 2) |
| [`727fe9d`](https://github.com/Immortalnop/eSim/commit/727fe9ddeb29fefeb24dd74665ceccf642427205), [`0e824dc`](https://github.com/Immortalnop/eSim/commit/0e824dc45ba7d4b002e48859b186e9103fabf3c0) | Remove text that was accidentally pasted from the nano editor into line 1 of `install-eSim.sh` while making the fix above |
| [`5de53ce`](https://github.com/Immortalnop/eSim/commit/5de53cef1848c82a3a74aeebe4c4fc4645b6357b) | Add Ubuntu 25.04 support to the NGHDL install (issues 4a–4c) |
| [`019dc91`](https://github.com/Immortalnop/eSim/commit/019dc91ea3cfe8e922711222e164c2e2de580ea5) | Do not exit the installer when KiCad 8 is already installed (issue 7) |
| [`85bdc47`](https://github.com/Immortalnop/eSim/commit/85bdc474658d1a07cd9f216c78e42ed1f5f48438) | Install `python3-virtualenv` with `-y` (issue 5) |
| [`3fb715f`](https://github.com/Immortalnop/eSim/commit/3fb715fe5922ec0b263c8ce908f14c811a3a3d30) | Copy eSim's symbol table to the KiCad 8 config folder (issue 6) |
| [`6306b3f`](https://github.com/Immortalnop/eSim/commit/6306b3f78d6646f74ca01600bb1082b35b4c9752) | Fix invalid escape sequences in NGHDL `model_generation.py` (issue 8) |
| [`cd6eda6`](https://github.com/Immortalnop/eSim/commit/cd6eda6f) | Add this report, logs and screenshots, and update the READMEs |

## Evidence

All files are in [`report-25.04-aarush/`](report-25.04-aarush/).

**Logs** (`logs/`). The original logs were written by `script`. These copies
have terminal colour codes and progress-bar redraws removed so they read as
plain text. Every line is kept, so line numbers in this report match.

| File | Run |
| --- | --- |
| [`1-release-zip.log`](report-25.04-aarush/logs/1-release-zip.log) | A: release zip |
| [`2-installers-branch-syntax-error.log`](report-25.04-aarush/logs/2-installers-branch-syntax-error.log) | B: `installers` branch, syntax error. This run's log file was later overwritten by Run C; this copy is the output recorded from the terminal. |
| [`3-after-syntax-fix-nghdl-fails.log`](report-25.04-aarush/logs/3-after-syntax-fix-nghdl-fails.log) | C: after fixes 1 and 2, fails in NGHDL |
| [`4-rerun-stops-after-kicad.log`](report-25.04-aarush/logs/4-rerun-stops-after-kicad.log) | D: stops after the KiCad step |
| [`5-install-success.log`](report-25.04-aarush/logs/5-install-success.log) | E: complete install |
| [`6-clean-install-launchpad-503.log`](report-25.04-aarush/logs/6-clean-install-launchpad-503.log) | F: clean install with all fixes, stopped by a Launchpad server outage |
| [`7-clean-install-retry-success.log`](report-25.04-aarush/logs/7-clean-install-retry-success.log) | G: clean install, run again, complete |

**Screenshots** (`images/`):

| File | Shows |
| --- | --- |
| [`01-release-zip-unsupported.png`](report-25.04-aarush/images/01-release-zip-unsupported.png) | Run A: SHA256 check of the zip, then `Unsupported Ubuntu version: 25.04 ()` |
| [`02-install-success.png`](report-25.04-aarush/images/02-install-success.png) | Run E: end of the log, `eSim Installed Successfully` |
| [`03-esim-running.png`](report-25.04-aarush/images/03-esim-running.png) | eSim running on Ubuntu 25.04 through WSLg, and the blank Welcome panel (issue 9) |
