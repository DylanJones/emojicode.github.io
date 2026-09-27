# Installing Emojicode 1.0 beta 2

>!N **AI-edited:** Parts of this page were written or revised by AI (Claude) to document changes in this fork of
>!N Emojicode, and have not been fully reviewed by a human. If something here disagrees with the compiler, the
>!N compiler is right.

>!N The prebuilt binaries that the magic installation and the other
>!N installation methods on this page download are releases of the original
>!N Emojicode from [emojicode/emojicode](https://github.com/emojicode/emojicode/releases).
>!N They don’t contain the changes of this fork that this documentation
>!N describes, such as the `emojicode-lsp` language server or the `c` package.
>!N To use this fork, [build it from source](#building-from-source).

## Prerequisites

Before you install Emojicode make sure you have a C++ compiler and linker
installed. `clang++` or `g++` are fine, for instance. The Emojicode compiler can
only link binaries if such a compiler is available.

## Magic Installation

The magic installation is the easiest way to install Emojicode. Select your
operating system and copy’n’paste the below commands into your
shell. The script will download the appropriate Emojicode binaries and run the
installer.

The installer will tell you what it is about to do and will prompt you for
confirmation. It copies the compiler `emojicodec` and, in distributions built
from this fork, the language server `emojicode-lsp` to `/usr/local/bin`, the
packages to `/usr/local/EmojicodePackages` and the headers for
[implementing functions in C++](api.html) to `/usr/local/include/emojicode`.
You can pass other locations to `install.sh` like so:

```bash
./install.sh [binary location] [packages location] [include location]
```

If you're on Windows 10, you can use
[Bash on Ubuntu on Windows 10](https://msdn.microsoft.com/en-us/commandline/wsl/install_guide)
to install and use Emojicode. From there, simply select `Linux` as your OS and
proceed as specified above.

<div class="magic-install-sw">
  <div class="magic-install-sw-box">
    <label class="magic-install-sw-label">
      Version
    </label>
    <select id="magic-install-version"></select>
    <div class="magic-install-sw-help">We recommend that you choose the latest.</div>
  </div>
  <div class="magic-install-sw-box center">
    <label class="magic-install-sw-label">
      Operating System
    </label>
    <select id="magic-install-os">
      <option value="darwin">macOS</option>
      <option value="linux">Linux</option>
    </select>
  </div>
  <div class="magic-install-sw-box">
    <label class="magic-install-sw-label">
      Downloader
    </label>
    <select id="magic-install-http">
      <option value="curl">curl</option>
      <option value="wget">wget</option>
    </select>
    <div class="magic-install-sw-help">If you’re unsure, just leave it like it is.</div>
  </div>
</div>
<pre><code id="magic-install-code"></code></pre>

## Try Emojicode without Installing

You can even try Emojicode by only downloading it. [Download the prebuilt binaries](https://github.com/emojicode/emojicode/releases) and extract the tar file and navigate into the extracted directory:

```bash
tar -xzf Emojicode-VERSION-YOUR-PLATFORM.tar.gz
cd Emojicode-VERSION-YOUR-PLATFORM
```

You’re ready to go! Try this, for example:

```bash
echo '🏁 🍇
  😀 🔤Hello World!🔤❗️
🍉' > hello.emojic
./emojicodec hello.emojic  # Compile it
./hello  # Run it!
```

## Installing for Arch Linux

Install the [`emojicode`](https://aur.archlinux.org/packages/emojicode) package from the AUR using your favourite AUR helper or the manual `git`/`makepkg` method.

## Troubleshooting

If you see a message like the one below

```
sh: 1: c++: not found
```

try exporting the name of your C++ compiler like in the below example:

```bash
export CXX=clang++  # or g++ or whatever your compiler is
```

## Manual Installation

1. [Download the prebuilt binaries](https://github.com/emojicode/emojicode/releases) for your
  system and extract the tar file. For instance:

  ```bash
  tar -xzf Emojicode-VERSION-YOUR-PLATFORM.tar.gz
  ```

2.  Run the `install.sh` script in the extracted directory:

  ```bash
  cd Emojicode-VERSION-YOUR-PLATFORM
  ./install.sh
  ```

  You can find more information about the installer above.

### Very Manual Installation

If the installer doesn’t work for you or you simply don‘t want to use it, you
can also copy things into place yourself.

1. [Download the prebuilt binaries](https://github.com/emojicode/emojicode/releases)
  for your system and extract the tar file.

2. Copy `emojicodec` (the compiler executable) to the place you keep executables.
   If there is an `emojicode-lsp` (the [language server](editors.html)), copy
   it there too.

3. Copy the contents of the  `packages` somewhere where the
   Emojicode Compiler can find it.

   One of these locations is `/usr/local/EmojicodePackages`. See [Package Search Paths](../reference/compiler.html#package-search-paths) for more information.
4. Finally, you should copy the contents of `include` into a directory named
   `emojicode` in your C++ compiler’s search path.

   The installer, for example, copies it to `/usr/local/include/emojicode`.

## Building from Source

To use this fork of Emojicode, you have to build it from source. The source code
is in the [DylanJones/emojicode](https://github.com/DylanJones/emojicode)
repository on GitHub. Building it also builds the language server
`emojicode-lsp` and all packages, including the `c` package.

### Requirements

- A C++17 compiler, e.g. clang 21 or newer or GCC 13 or newer
- CMake 3.20 or newer and, preferably, Ninja
- LLVM 21 or newer (tested with LLVM 21, 22 and 23)
- Python 3.8 or newer to run the tests

On Ubuntu 24.04 you can install these from [apt.llvm.org](https://apt.llvm.org):

```bash
wget -qO- https://apt.llvm.org/llvm-snapshot.gpg.key | sudo tee /etc/apt/trusted.gpg.d/apt.llvm.org.asc
echo "deb http://apt.llvm.org/noble/ llvm-toolchain-noble-23 main" | sudo tee /etc/apt/sources.list.d/llvm.list
sudo apt update
sudo apt install clang-23 llvm-23-dev cmake ninja-build python3 rsync zlib1g-dev libzstd-dev
```

On macOS, `brew install llvm cmake ninja` provides everything you need.
Homebrew’s LLVM is not on CMake’s default search path, so pass
`-DLLVM_DIR="$(brew --prefix llvm)/lib/cmake/llvm"` to CMake in step 2 below.

### Building

1. Clone the repository (or download the source code and extract it) and
   navigate into it:

   ```bash
   git clone https://github.com/DylanJones/emojicode
   cd emojicode
   ```

2. Create a `build` directory and run CMake in it:

   ```bash
   mkdir build
   cd build
   cmake .. -GNinja
   ```

   If CMake does not pick up the right version of LLVM, point it to LLVM’s CMake
   directory, e.g. `-DLLVM_DIR=/usr/lib/llvm-23/lib/cmake/llvm`.

   To change the default package search path, which is
   `/usr/local/EmojicodePackages`, pass `-DdefaultPackagesDirectory=` followed
   by the path you want. See
   [Package Search Paths](../reference/compiler.html#package-search-paths).

3. Build the compiler, the language server and the packages:

   ```bash
   ninja
   ```

4. Run the tests:

   ```bash
   ninja tests
   ```

   The tests run in parallel, one per core. To run fewer at once, set the
   environment variable `EMOJICODE_TEST_JOBS` to the number you want, e.g.
   `EMOJICODE_TEST_JOBS=4 ninja tests`.

5. Install Emojicode with the installer:

   ```bash
   ninja magicinstall
   ```

   This installs `emojicodec`, `emojicode-lsp`, the packages and the headers into
   the default locations described in [Magic Installation](#magic-installation).

   Alternatively, `ninja dist` puts everything that would be installed, including
   `install.sh`, into a directory named like `Emojicode-1.0-beta.2-Linux-x86_64`
   in the build directory. You can then run its `install.sh` with other
   locations, or copy the files yourself as described in
   [Very Manual Installation](#very-manual-installation). To pack that
   directory into a `.tar.gz` archive, run `python3 ../dist.py archive`.

### Building with Docker

The repository also contains a `Dockerfile` that builds and installs Emojicode
in an Ubuntu 24.04 image with LLVM 23. Build the image in the repository and
run the tests in it:

```bash
docker build -t emojicode-build -f docker/clang .
docker run --rm emojicode-build
```

To use the image, start it with one of your directories mounted into it, e.g.

```bash
docker run --rm -v "$(pwd)/code:/workspace" -it emojicode-build /bin/bash
```

and compile and run your code inside the container:

```bash
emojicodec /workspace/hello.🍇 && /workspace/hello
```

<script src="/static/js/magicinstall.js"></script>
