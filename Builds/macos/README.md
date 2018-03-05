# macos Build Instructions

## Prerequisites

You'll need macos 10.8 or later.

To clone the source code repository, create branches for inspection or
modification, build rmcd using clang, and run the system tests you will need
these software components:

* [XCode](https://developer.apple.com/xcode/)
* [Homebrew](http://brew.sh/)
* [Boost](http://boost.org/)
* other misc utilities and libraries installed via homebrew

## Install Software

### Install XCode

If not already installed on your system, download and install XCode using the
appstore or by using [this link](https://developer.apple.com/xcode/).

For more info, see "Step 1: Download and Install the Command Line Tools"
[here](http://www.moncefbelyamani.com/how-to-install-xcode-homebrew-git-rvm-ruby-on-mac)

The command line tools can be installed through the terminal with the command:

```
xcode-select --install
```

### Install Homebrew

> "[Homebrew](http://brew.sh/) installs the stuff you need that Apple didn’t."

Open a terminal and type:

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

For more info, see "Step 2: Install Homebrew"
[here](http://www.moncefbelyamani.com/how-to-install-xcode-homebrew-git-rvm-ruby-on-mac#step-2)

### Install Dependencies Using Homebrew

`brew` will generally install the latest stable version of any package, which
should satisfy the the minimum version requirements for rmcd.

```
brew update
brew install git cmake pkg-config protobuf openssl ninja
```

### Build Boost

We want to compile boost with clang/libc++

Download [a release](https://dl.bintray.com/boostorg/release/1.66.0/source/boost_1_66_0.tar.bz2)

Extract it to a folder, making note of where, open a terminal, then:

```
./bootstrap.sh
./b2 cxxflags="-std=c++14"
```

Create an environment variable `BOOST_ROOT` in one of your `rc` files, pointing
to the root of the extracted directory.

### Dependencies for Building Source Documentation

Source code documentation is not required for running/debugging rmcd. That
said, the documentation contains some helpful information about specific
components of the application. For more information on how to install and run
the necessary components, see [this document](../../docs/README.md)

## Build

### Clone the rmcd repository

From a shell:

```
git clone git@github.com:RussianMiningCoin/rmcd.git
cd rmcd
```

### Configure Library Paths

If you didn't persistently set the `BOOST_ROOT` environment variable to the
root of the extracted directory above, then you should set it temporarily.

For example, assuming your username were `Abigail` and you extracted Boost
1.66.0 in `/Users/Abigail/Downloads/boost_1_66_0`, you would do for any
shell in which you want to build:

```
export BOOST_ROOT=/Users/Abigail/Downloads/boost_1_66_0
```

### Generate and Build

For simple command line building we recommend using the *Unix Makefile* or
*Ninja* generator with cmake. All builds should be done in a separate directory
from the source tree root (a subdirectory is fine). For example, from the root
of the ripple source tree:

```
mkdir my_build
cd my_build
```

followed by:

```
cmake -G "Unix Makefiles" -Dtarget=clang.debug.unity ..
```

or

```
cmake -G "Ninja" -Dtarget=clang.debug.unity ..
```

The target variable can be adjusted as needed for `debug` vs. `release` and
`unity` vs. `nounity` builds. `unity` builds are typically faster to compile
but run the risk of ODR violations given that multiple compilation units are
merged together at compile time. `nounity` builds will take longer to compile
but align more closely with language standards. 

Once you have generated the build system, you can run the build via cmake:

```
cmake --build . -- -j 4
```

the `-j` parameter in this example tells the build tool to compile several
files in parallel. This value should be chosen roughly based on the number of
cores you have available and/or want to use for building.

When the build completes succesfully, you will have a `rmcd` executable in
the current directory, which can be used to connect to the network (when
properly configured) or to run unit tests.

If you prefer to have an XCode project to use for building, ask CMake to
generate that instead:

```
cmake -GXcode ..
```

After generation succeeds, the xcode project file can be opened and used to
build/debug. However, just as with other generators, cmake knows how to build
using the xcode project as well:

```
cmake --build . -- -jobs 4
```

This will invoke the `xcodebuild` utility to compile the project. See `xcodebuild
--help` for details about build options.

#### Options During Configuration:

There are a number of config variables that our CMake files support. These
can be added to the cmake generation command as needed:

* `-Dassert=ON` to enable asserts
* `-Djemalloc=ON` to enable jemalloc support for heap checking
* `-Dsan=thread` to enable the thread sanitizer with clang
* `-Dsan=address` to enable the address sanitizer with clang
