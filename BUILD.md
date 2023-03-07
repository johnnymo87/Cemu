# Build Instructions

## Table of Contents

- [Windows](#windows)
- [Linux](#linux)
   - [Dependencies](#dependencies)
      - [For Arch and derivatives:](#for-arch-and-derivatives)
      - [For Debian, Ubuntu and derivatives](#for-debian-ubuntu-and-derivatives)
      - [For Fedora and derivatives:](#for-fedora-and-derivatives)
   - [Build Cemu](#build-cemu)
      - [CMake and Clang](#cmake-and-clang)
      - [GCC](#gcc)
      - [Debug Build](#debug-build)
      - [Troubleshooting Steps](#troubleshooting-steps)
         - [Compiling Errors](#compiling-errors)
         - [Building Errors](#building-errors)
- [macOS](#macos)
   - [Installing brew](#installing-brew)
   - [Installing Tool Dependencies](#installing-tool-dependencies)
   - [Installing Library Dependencies](#installing-library-dependencies)
   - [Build Cemu using CMake](#build-cemu-using-cmake)
- [Updating Cemu and source code](#updating-cemu-and-source-code)

## Table of Contents

* [Windows](#windows)
* [Linux](#linux)
* [Mac](#macos)

## Windows

Prerequisites:
- git
- A recent version of Visual Studio 2022 with the following additional components:
   - C++ CMake tools for Windows
   - Windows 10/11 SDK

Instructions for Visual Studio 2022:

1. Run `git clone --recursive https://github.com/cemu-project/Cemu`
2. Open the newly created Cemu directory in Visual Studio using the "Open a local folder" option
3. In the menu select Project -> Configure CMake. Wait until it is done, this may take a long time
4. You can now build, run and debug Cemu

Any other IDE should also work as long as it has CMake and MSVC support. CLion and Visual Studio Code have been confirmed to work.

## Linux

To compile Cemu, a recent enough compiler and STL with C++20 support is required! Clang-15 or higher is what we recommend.

### Dependencies

#### For Arch and derivatives:
`sudo pacman -S --needed base-devel clang cmake freeglut git glm gtk3 libgcrypt libpulse libsecret linux-headers llvm nasm ninja systemd unzip zip`

#### For Debian, Ubuntu and derivatives:
`sudo apt install -y cmake curl clang-15 freeglut3-dev git libgcrypt20-dev libglm-dev libgtk-3-dev libpulse-dev libsecret-1-dev libsystemd-dev libtool nasm ninja-build`

You may also need to install `libusb-1.0-0-dev` as a workaround for an issue with the vcpkg hidapi package.

At Step 3 in [Build Cemu using cmake and clang](#build-cemu-using-cmake-and-clang), use the following command instead:
   `cmake -S . -B build -DCMAKE_BUILD_TYPE=release -DCMAKE_C_COMPILER=/usr/bin/clang-15 -DCMAKE_CXX_COMPILER=/usr/bin/clang++-15 -G Ninja -DCMAKE_MAKE_PROGRAM=/usr/bin/ninja`

#### For Fedora and derivatives:
`sudo dnf install clang cmake cubeb-devel freeglut-devel git glm-devel gtk3-devel kernel-headers libgcrypt-devel libsecret-devel libtool libusb1-devel llvm nasm ninja-build perl-core systemd-devel zlib-devel zlib-static`

### Build Cemu

#### CMake and Clang

```
git clone --recursive https://github.com/cemu-project/Cemu
cd Cemu
cmake -S . -B build -DCMAKE_BUILD_TYPE=release -DCMAKE_C_COMPILER=/usr/bin/clang -DCMAKE_CXX_COMPILER=/usr/bin/clang++ -G Ninja
cmake --build build
```

#### GCC

If you are building using GCC, make sure you have g++ installed:
- Installation for Arch and derivatives: `sudo pacman -S gcc`
- Installation for Debian, Ubuntu and derivatives: `sudo apt install g++`
- Installation for Fedora and derivatives: `sudo dnf install gcc-c++`

```
git clone --recursive https://github.com/cemu-project/Cemu
cd Cemu
cmake -S . -B build -DCMAKE_BUILD_TYPE=release -DCMAKE_C_COMPILER=/usr/bin/gcc -DCMAKE_CXX_COMPILER=/usr/bin/g++ -G Ninja
cmake --build build
```

#### Debug Build

```
git clone --recursive https://github.com/cemu-project/Cemu
cd Cemu
cmake -S . -B build -DCMAKE_BUILD_TYPE=debug -DCMAKE_C_COMPILER=/usr/bin/clang -DCMAKE_CXX_COMPILER=/usr/bin/clang++ -G Ninja
cmake --build build
```

If you are using GCC, replace `cmake -S . -B build -DCMAKE_BUILD_TYPE=debug -DCMAKE_C_COMPILER=/usr/bin/clang -DCMAKE_CXX_COMPILER=/usr/bin/clang++ -G Ninja` with `cmake -S . -B build -DCMAKE_BUILD_TYPE=debug -DCMAKE_C_COMPILER=/usr/bin/gcc -DCMAKE_CXX_COMPILER=/usr/bin/g++ -G Ninja`

#### Troubleshooting Steps

##### Compiling Errors

This section refers to running `cmake -S...` (truncated).

* `vcpkg install failed`
   * Run the following in the root directory and try running the command again (don't forget to change directories afterwards):
      * `cd dependencies/vcpkg && git fetch --unshallow`
* `Please ensure you're using the latest port files with git pull and vcpkg update.`
   * Either:
      * Update vcpkg by running by the following command:
         * `git submodule update --remote dependencies/vcpkg`
      * If you are sure vcpkg is up to date, check the following logs:
         * `Cemu/dependencies/vcpkg/buildtrees/wxwidgets/config-x64-linux-out.log`
         * `Cemu/dependencies/vcpkg/buildtrees/libsystemd/config-x64-linux-dbg-meson-log.txt.log`
         * `Cemu/dependencies/vcpkg/buildtrees/libsystemd/config-x64-linux-dbg-out.log`
* Not able to find Ninja.
   * Add the following and try running the command again:
      * `-DCMAKE_MAKE_PROGRAM=/usr/bin/ninja`
* Compiling failed during the boost-build dependency.
   * It means you don't have a working/good standard library installation. Check the integrity of your system headers and making sure that C++ related packages are installed and intact.
* Compiling failed during rebuild after `git pull` with an error that mentions RPATH
   * Add the following and try running the command again:
      * `-DCMAKE_BUILD_WITH_INSTALL_RPATH=ON`
* If you are getting a random error, read the [package-name-and-platform]-out.log and [package-name-and-platform]-err.log for the actual reason to see if you might be lacking the headers from a dependency.


If you are getting a different error than any of the errors listed above, you may either open an issue in this repo or try using [GCC](#gcc). Make sure your standard library and compilers are updated since Cemu uses a lot of modern features!


##### Building Errors

This section refers to running `cmake --build build`.

* `main.cpp.o: in function 'std::__cxx11::basic_string...`
   * You likely are experiencing a clang-14 issue. This can only be fixed by either lowering the clang version or using GCC, see [GCC](#gcc).
* `fatal error: 'span' file not found`
   *  You're either missing `libstdc++` or are using a version that's too old. Install at least v10 with your package manager, eg `sudo apt install libstdc++-10-dev`. See [#644](https://github.com/cemu-project/Cemu/issues/644).
* `undefined libdecor_xx`
   * You are likely experiencing an issue with sdl2 package that comes with vcpkg. Delete sdl2 from vcpkg.json in source file and recompile.

If you are getting a different error than any of the errors listed above, you may either open an issue in this repo or try using [GCC](#gcc). Make sure your standard library and compilers are updated since Cemu uses a lot of modern features!

## macOS

To compile Cemu, a recent enough compiler and STL with C++20 support is required! LLVM 13 and below
don't support the C++20 feature set required, so either install LLVM from Homebrew or make sure that
you have a recent enough version of Xcode. Xcode 15 is known to work. The OpenGL graphics API isn't
supported on macOS, so Vulkan must be used through the Molten-VK compatibility layer.

The rest of this section will walk you through the process of setting up and building Cemu on a Mac, whether it's an Intel or Apple Silicon machine.

### First time setup before compiling Cemu for Apple Silicon Macs

1. Install Rosetta 2.

   ```bash
   softwareupdate --install-rosetta
   ```

1. Run an x86_64 shell.

   ```bash
   arch -x86_64 zsh
   ```

1. Unload your arm64-specific brew from your `PATH` so that it doesn't confuse your x86_64-specific installation of brew.

   ```bash
   export PATH=`printf '%s:' $(echo $PATH | tr ':' '\n' | grep -iv "^\/opt\/homebrew\/")`
   ```

   * **Note:** This step is only necessary if you have an arm64-specific brew installed. To check, do `echo $PATH | tr ':' '\n'`, and check if there's any mention of `/opt/homebrew`.
1. Now continue onto the `First time setup for Intel Macs` section. Note that you should definitely do the `Install brew` step **even if** you have an arm64-specific brew already installed. This will install a second one, specific to x86_64.

### First time setup before compiling Cemu for either kind of Mac

1. Install brew.

   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

1. Initialize brew.

   ```bash
   eval "$(/usr/local/bin/brew shellenv)"
   ```

1. Install dependencies.

   ```bash
   brew install boost git cmake llvm nasm ninja pkg-config molten-vk
   ```

1. Clone the Cemu repository with the `--recursive` flag to also clone the dependencies that it submodules.

   ```bash
   git clone --recursive https://github.com/cemu-project/Cemu
   ```

1. Change into the cloned repository directory.

   ```bash
   cd Cemu
   ```

1. Run cmake to generate the build files.

   ```bash
   cmake -S . -B build -DCMAKE_BUILD_TYPE=release -DCMAKE_C_COMPILER=/usr/local/opt/llvm/bin/clang -DCMAKE_CXX_COMPILER=/usr/local/opt/llvm/bin/clang++ -G Ninja
   ```

### Every time setup before compiling Cemu for either kind of Mac

Some of the following steps should be done every time you wish to compile Cemu, others should be done on an as-needed basis. The latter are marked with a :soap: emoji.

1. If you have an Apple Silicon Mac, run an x86_64 shell.

   ```bash
   arch -x86_64 zsh
   ```

1. If you have an Apple Silicon Mac, and you have an arm64-specific brew installed, unload your arm64-specific brew from your `PATH` so that it doesn't confuse your x86_64-specific installation of brew.

   ```bash
   export PATH=`printf '%s:' $(echo $PATH | tr ':' '\n' | grep -iv "^\/opt\/homebrew\/")`
   ```

1. Initialize brew.

   ```bash
   eval "$(/usr/local/bin/brew shellenv)"
   ```

1. :soap: Update dependencies.

   ```bash
   brew update && brew upgrade
   ```

   * **Note:** This step is optional and only needs to be done if there are updates to the dependencies installed via brew.
1. :soap: Update the Cemu repository and its submodules.

   ```bash
   git pull --recurse-submodules
   ```

   * **Note:** This step is optional and only needs to be done if there are updates to the Cemu repository or its submodules.

1. :soap: If during the previous step you see that you pulled in changes to `CMakeLists.txt`, rerun cmake to regenerate the build files.

   ```bash
   cmake -S . -B build -DCMAKE_BUILD_TYPE=release -DCMAKE_C_COMPILER=/usr/local/opt/llvm/bin/clang -DCMAKE_CXX_COMPILER=/usr/local/opt/llvm/bin/clang++ -G Ninja
   ```

### Compile Cemu using cmake and clang for either kind of Mac

1. Run cmake to build Cemu using clang.

   ```bash
   cmake --build build
   ```

1. You should now have a Cemu executable file in the /bin folder, which you can run using:

   ```bash
   ./bin/Cemu_release
   ```

## Updating Cemu and source code
1. To update your Cemu local repository, use the command `git pull --recurse-submodules` (run this command on the Cemu root).
    - This should update your local copy of Cemu and all of its dependencies.
2. Then, you can rebuild Cemu using the steps listed above, according to whether you use Linux or Windows.

If CMake complains about Cemu already being compiled or another similar error, try deleting the `CMakeCache.txt` file inside the `build` folder and retry building.

