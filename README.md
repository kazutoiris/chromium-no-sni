# Chromium without SNI

Removed SNI to test server handshake behavior without SNI support.

> [!CAUTION]
> You may be unable to complete the TLS handshake when the server uses SNI to select the SSL certificate!

----

# ungoogled-chromium-windows

Windows packaging for [ungoogled-chromium](//github.com/Eloston/ungoogled-chromium).

## Downloads

[Download binaries from the Contributor Binaries website](//ungoogled-software.github.io/ungoogled-chromium-binaries/).

Or install using `winget install --id=eloston.ungoogled-chromium -e`.

**Source Code**: It is recommended to use a tag via `git checkout` (see building instructions below). You may also use `master`, but it is for development and may not be stable.

## Building

Google only supports [Windows 10 x64 or newer](https://chromium.googlesource.com/chromium/src/+/refs/heads/main/docs/windows_build_instructions.md#system-requirements). These instructions are tested on Windows 10 Pro x64.

NOTE: The default configuration will build 64-bit binaries for maximum security (TODO: Link some explanation). This can be changed to 32-bit by setting `target_cpu` to `"x86"` in `flags.windows.gn`.

### Setting up the build environment

**IMPORTANT**: Please setup only what is referenced below. Do NOT setup other Chromium compilation tools like `depot_tools`, since we have a custom build process which avoids using Google's pre-built binaries.

#### Setting up Visual Studio

[Follow the "Visual Studio" section of the official Windows build instructions](https://chromium.googlesource.com/chromium/src/+/refs/heads/main/docs/windows_build_instructions.md#visual-studio).

* Make sure to read through the entire section and install/configure all the required components.
* If your Visual Studio is installed in a directory other than the default, you'll need to set a few environment variables to point the toolchains to your installation path. (Copied from [instructions for Electron](https://electronjs.org/docs/development/build-instructions-windows))
	* `vs2019_install = DRIVE:\path\to\Microsoft Visual Studio\2019\Community` (replace `2019` and `Community` with your installed versions)
	* `WINDOWSSDKDIR = DRIVE:\path\to\Windows Kits\10`
	* `GYP_MSVS_VERSION = 2019` (replace 2019 with your installed version's year)


#### Other build requirements

**IMPORTANT**: Currently, the `MAX_PATH` path length restriction (which is 260 characters by default) must be lifted in for our Python build scripts. This can be lifted in Windows 10 (v1607 or newer) with the official installer for Python 3.6 or newer (you will see a button at the end of installation to do this). See [Issue #345](https://github.com/Eloston/ungoogled-chromium/issues/345) for other methods for older Windows versions.

1. Setup the following:
    * 7-Zip
    * Python 3.8 - 3.10 (for build and packaging scripts used below); Python 3.11 and above is not supported.
    * If you don't plan on using the Microsoft Store version of Python:
        * Check "Add python.exe to PATH" before install.
        * At the end of the Python installer, click the button to lift the `MAX_PATH` length restriction.  
        * Check that your `PATH` does not contain the `python3` wrapper shipped by Windows, as it will only prompt you to install Python from the Microsoft Store and exit. See [this question on stackoverflow.com](https://stackoverflow.com/questions/57485491/python-python3-executes-in-command-prompt-but-does-not-run-correctly)
        * Ensure that your Python directory either has a copy of Python named "python3.exe" or a symlink linking to the Python executable.
    * Make sure to lift the `MAX_PATH` length restriction, either by clicking the button at the end of the Python installer or by [following these instructions](https://learn.microsoft.com/en-us/windows/win32/fileio/maximum-file-path-limitation?tabs=registry#:~:text=Enable,Later).
    * Git (to fetch all required ungoogled-chromium scripts)
        * During setup, make sure "Git from the command line and also from 3rd-party software" is selected. This is usually the recommended option.
    * The following additional python modules needs to be installed using `pip install`:
        * httplib2

### Building

Run in `Developer Command Prompt for VS` (as administrator):

```cmd
git clone --recurse-submodules https://github.com/ungoogled-software/ungoogled-chromium-windows.git
cd ungoogled-chromium-windows
# Replace TAG_OR_BRANCH_HERE with a tag or branch name
git checkout --recurse-submodules TAG_OR_BRANCH_HERE
python3 build.py
python3 package.py
```

A zip archive and an installer will be created under `build`.

**NOTE**: If the build fails, you must take additional steps before re-running the build:

* If the build fails while downloading the Chromium source code (which is during `build.py`), it can be fixed by removing `build\download_cache` and re-running the build instructions.
* If the build fails at any other point during `build.py`, it can be fixed by removing everything under `build` other than `build\download_cache` and re-running the build instructions. This will clear out all the code used by the build, and any files generated by the build.

An efficient way to delete large amounts of files is using `Remove-Item PATH -Recurse -Force`. Be careful however, files deleted by that command will be permanently lost.

## Developer info

### First-time setup

1. [Setup MSYS2](http://www.msys2.org/)
2. Run the following in a "MSYS2 MSYS" shell:

```sh
pacman -S quilt python3 vim tar dos2unix
# By default, there doesn't seem to be a vi command for less, quilt edit, etc.
ln -s /usr/bin/vim /usr/bin/vi
```

### Updating patches and pruning list

1. Start `Developer Command Prompt for VS` and `MSYS2 MSYS` shell and navigate to source folder
	1. `Developer Command Prompt for VS`
		* `cd c:\path\to\repo\ungoogled-chromium-windows`
	1. `MSYS2 MSYS`
		* `cd /path/to/repo/ungoogled-chromium-windows`
		* You can use Git Bash to determine the path to this repo
		* Or, you can find it yourself via `/<drive letter>/<path with forward slashes>`
1. Retrieve downloads
	**`Developer Command Prompt for VS`**
	* `mkdir "build\download_cache"`
	* `python3 ungoogled-chromium\utils\downloads.py retrieve -i downloads.ini -c build\download_cache`
1. Clone sources
	**`Developer Command Prompt for VS`**
	* `python3 ungoogled-chromium\utils\clone.py -o build\src`
1. Check for rust version change (see below)
1. Update pruning list
	**`Developer Command Prompt for VS`**
	* `python3 ungoogled-chromium\devutils\update_lists.py -t build\src --domain-regex ungoogled-chromium\domain_regex.list`
1. Unpack downloads
	**`Developer Command Prompt for VS`**
	* `python3 ungoogled-chromium\utils\downloads.py unpack -i downloads.ini -c build\download_cache build\src`
1. Apply ungoogled-chromium patches
	**`Developer Command Prompt for VS`**
	* `python3 ungoogled-chromium\utils\patches.py apply --patch-bin build\src\third_party\git\usr\bin\patch.exe build\src ungoogled-chromium\patches`
1. Update windows patches
	**`MSYS2 MSYS`**
	1. Setup shell to update patches
		* `source devutils/set_quilt_vars.sh`
	1. Go into the source tree
		* `cd build/src`
	1. Fix line breaks of files to patch
		* `grep -r ../../patches/ -e "^+++" | awk '{print substr($2,3)}' | xargs dos2unix`
	1. Use quilt to refresh patches. See ungoogled-chromium's [docs/developing.md](https://github.com/Eloston/ungoogled-chromium/blob/master/docs/developing.md#updating-patches) section "Updating patches" for more details
	1. Go back to repo root
		* `cd ../..`
	1. Sanity checking for consistency in series file
		* `./devutils/check_patch_files.sh`
1. Check for esbuild dependency changes in file `build/src/DEPS` and adapt `downloads.ini` accordingly
1. Check for commit hash changes of `src` submodule in `third_party/microsoft_dxheaders` (e.g. using GitHub https://github.com/chromium/chromium/tree/127.0.6533.72/third_party/microsoft_dxheaders) and adapt `downloads.ini` accordingly
1. Check for version changes of windows rust crate (`third_party/rust/windows_x86_64_msvc/`) and adapt `downloads.ini` and `patches/ungoogled-chromium/windows\windows-fix-building-with-rust.patch` accordingly
1. Use git to add changes and commit

### Update rust
1. Check `RUST_REVISION` constant in file `tools/rust/update_rust.py` in build root.
	1. Current revision is `ab71ee7a9214c2793108a41efb065aa77aeb7326`
1. Get date for nightly rust build from rust github page: `https://github.com/rust-lang/rust/commit/ab71ee7a9214c2793108a41efb065aa77aeb7326`
	1. In this case, the corresponding nightly build date is `2024-04-12`
	1. Adapt `downloads.ini` accordingly
1. Download nightly rust build from: https://static.rust-lang.org/dist/2024-04-12/rust-nightly-x86_64-pc-windows-msvc.tar.gz
	1. Extract archive
	1. Execute `rustc\bin\rustc.exe -V` to get rust version string
	1. Adapt `build.py` accordingly
	1. Adapt `patches\ungoogled-chromium\windows\windows-fix-building-with-rust.patch` accordingly

## License

See [LICENSE](LICENSE)
