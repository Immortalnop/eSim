eSim Installer (Ubuntu OS)
====

It contains the documentation to package eSim for Ubuntu OS.


## Ubuntu 25.04

The eSim 2.5 release does not install on Ubuntu 25.04. The scripts in this
branch fix that. [`REPORT-25.04-aarush.md`](REPORT-25.04-aarush.md) lists
each problem found, its cause, the fix and how it was tested, with logs and
screenshots in [`report-25.04-aarush/`](report-25.04-aarush/).

To install eSim 2.5 on Ubuntu 25.04, copy this folder's `install-eSim.sh`
and `install-eSim-scripts/` over the extracted `eSim-2.5` release, then run
`./install-eSim.sh --install` from there. The exact steps are in the
report, under
[How to install eSim 2.5 on Ubuntu 25.04 with these fixes](REPORT-25.04-aarush.md#how-to-install-esim-25-on-ubuntu-2504-with-these-fixes).

`nghdl.zip` from the eSim 2.5 release has no Ubuntu 25.04 support. The
missing NGHDL files are in
[`install-eSim-scripts/nghdl-25.04/`](install-eSim-scripts/nghdl-25.04/),
and `install-eSim-25.04.sh` copies them into `nghdl/` during the install.
When packaging a new release, these changes should go into `nghdl.zip`
itself (see the [NGHDL packaging notes](https://github.com/fossee/nghdl/tree/installers/Ubuntu/README.md)),
after which this folder is no longer needed.

The earlier [`REPORT.md`](REPORT.md) by upstream contributors describes the
first 25.04 changes to this branch.

> Note: If planning to freeze the eSim source code for a target platform (Ubuntu OS), then refer this [documentation](executable.md). Remember to update the installer script to work with this executable!


## How to package eSim for Ubuntu OS?

1. Take the `master` branch containing the source code. Rename the folder to `eSim-<version>`.

2. Add the installer script `install-eSim.sh` from `installers` branch to `eSim-<version>` folder.

3. Add the eSim executable (if available) in `eSim-<version>` folder. Also, remove following files from this folder:
	- `.git` folder
	- `.github` folder
	- `code` folder
	- `src` folder (Applicable only if eSim executable is used)
	- conf.py
	- setup.py
	- index.rst
	- requirement.txt
	- .gitignore
	- .travis.yml
    - `library/browser/User-Manual/figures` folder
    - library/browser/User-Manual/eSim.html

4. Add eSim user manual `eSim_Manual_<version>.pdf` at location `library/browser/User-Manual`.

5. Add the zip file of `NGHDL` (`nghdl.zip`) in the eSim folder.

> Note: Refer this [documentation](https://github.com/fossee/nghdl/tree/installers/Ubuntu/README.md) on packaging of NGHDL for Ubuntu OS.

6. Compress `kicadLibrary` folder to a `tar.xz` format and then remove that folder.

7. Compress `eSim-<version>` to a zip format for distribution.
