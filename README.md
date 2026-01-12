# Awesome Reflection

A collection of C++26 reflection materials.

## List of Examples

Unless otherwise specified, all examples are compiled with the latest [clang-p2996](https://github.com/bloomberg/clang-p2996/tree/p2996) implementation, with compile flags `-std=c++26 -freflection-latest -fexpansion-statements`. 

### From [C++ meetup Brno | Herb Sutter: Reflection - C++´s Decade-Defining Rocket Engine](https://youtu.be/H4NiR9xe1XA):

- [List data members with template-for](https://godbolt.org/z/WeaKzWc56)

- [Compile-time JSON parser with aggregate class generation](https://godbolt.org/z/3jzehW1bv)

  Use `#embed` with compile flag `-Wno-c23-extensions`.

- [Polymorphism code generation](https://godbolt.org/z/aP4dKfqjo)

  Pre-C++26 implementation: <https://godbolt.org/z/MEPvEMbqP>

- [Class interface code generation](https://godbolt.org/z/Y75Wr3nGe)

- [Class interface code injection](https://godbolt.org/z/7jv9aj8Mh)

  Based on Experimental EDG Reflection Implementation (<https://github.com/cern-nextgen/reflmempp>)

### From [reflection for C++26如期而至](https://zhuanlan.zhihu.com/p/1919860255870948496):

- [Enum to string](https://godbolt.org/z/8a6W9vjzz)

  Change `enum Color` in line 17 to e.g. `enum class Color` or `enum Color: int`.

- [An implementation of tuple](https://godbolt.org/z/b6K6YvddP)

- [Argument parser](https://godbolt.org/z/YovYWsx17)

### Experimental JSON Builder based on reflection

<https://github.com/simdjson/experimental_json_builder>

## A Tutorial for Building [Clang-P2996](https://github.com/bloomberg/clang-p2996/tree/p2996) on Windows

### 1. Install MSYS2

Download and install [MSYS2](https://www.msys2.org/). It is recommended to install it at the default path (`C:\msys64`). This tutorial assumes the default path.

### 2. (Optional) Configure Windows Terminal

Integrate MSYS2 terminals into Windows Terminal for a better experience:

1. Open Windows Terminal -> Settings -> Open JSON file.
2. Add the following to the `"profiles"` → `"list"` array:

```json
{
    "commandline": "C:\\msys64\\msys2_shell.cmd -defterm -here -no-start -clang64",
    "icon": "C:\\msys64\\clang64.ico",
    "name": "MSYS2 CLANG64",
    "startingDirectory": "%USERPROFILE%"
},
{
    "commandline": "C:\\msys64\\msys2_shell.cmd -defterm -here -no-start -msys",
    "icon": "C:\\msys64\\msys2.ico",
    "name": "MSYS2 MSYS",
    "startingDirectory": "%USERPROFILE%"
}
```

### 3. Configure the Environment in MSYS2

1. Open MSYS2 MSYS. Run `pacman -Syu`. Then close the terminal and reopen it. Run `pacman -Syu` again.
2. Open MSYS2 CLANG64 and install the prerequisites:

```bash
pacman -S mingw-w64-clang-x86_64-toolchain mingw-w64-clang-x86_64-cmake mingw-w64-clang-x86_64-ninja git python base-devel

# Verify installation
clang --version
cmake --version
ninja --version
```

### 4. Fetch [Clang-P2996](https://github.com/bloomberg/clang-p2996/tree/p2996) source

Clone the specific p2996 branch. We use a shallow clone to save download time and disk space.

```bash
git clone --depth 1 --branch p2996 https://github.com/bloomberg/clang-p2996.git
cd clang-p2996
```

### 5. Build

```bash
cmake -S llvm -B build \
  -G Ninja \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_INSTALL_PREFIX="$(pwd)/install" \
  -DLLVM_ENABLE_PROJECTS="clang;lld" \
  -DLLVM_ENABLE_RUNTIMES="libcxx;libcxxabi;libunwind" \
  -DLLVM_TARGETS_TO_BUILD="X86" \
  -DLLVM_DEFAULT_TARGET_TRIPLE=x86_64-w64-windows-gnu \
  -DLLVM_HOST_TRIPLE=x86_64-w64-windows-gnu \
  -DCMAKE_C_COMPILER=clang \
  -DCMAKE_CXX_COMPILER=clang++ \
  -DCMAKE_C_COMPILER_TARGET=x86_64-w64-windows-gnu \
  -DCMAKE_CXX_COMPILER_TARGET=x86_64-w64-windows-gnu \
  -DLIBCXX_ENABLE_SHARED=OFF \
  -DLIBCXXABI_ENABLE_SHARED=OFF \
  -DLIBUNWIND_ENABLE_SHARED=OFF \
  -DLIBCXX_USE_COMPILER_RT=OFF \
  -DLIBCXXABI_USE_COMPILER_RT=OFF \
  -DLIBCXXABI_USE_LLVM_UNWINDER=ON \
  -DLIBCXX_INSTALL_MODULES=ON \
  -DLLVM_ENABLE_TERMINFO=OFF \
  -DLLVM_ENABLE_ZLIB=OFF \
  -DLLVM_ENABLE_ZSTD=OFF \
  -DRUNTIMES_CMAKE_ARGS="-DCMAKE_SYSROOT=$(cygpath -m $MSYSTEM_PREFIX)"

ninja -C build install
```

### 6. Compile

Suppose you have a C++ source file `./main.cpp`.

Create `./CMakePresets.json`:

```json
{
  "version": 4,
  "configurePresets": [
    {
      "name": "p2996",
      "generator": "Ninja",
      "binaryDir": "${sourceDir}/build",
      "cacheVariables": {
        "CMAKE_BUILD_TYPE": "Release",
        "CMAKE_CXX_COMPILER": "C:/path/to/clang-p2996/install/bin/clang++.exe",
        "CMAKE_C_COMPILER": "C:/path/to/clang-p2996/install/bin/clang.exe",
        "CMAKE_CXX_FLAGS": "-O3 -stdlib=libc++ -fparameter-reflection -Wno-unused -Wno-c23-extensions -freflection-latest -fexpansion-statements -IC:/path/to/clang-p2996/install/include/c++/v1",
        "CMAKE_EXE_LINKER_FLAGS": "-fuse-ld=lld -LC:/path/to/clang-p2996/install/lib -stdlib=libc++ -static -lc++abi -lunwind -lucrt",
        "CMAKE_TRY_COMPILE_TARGET_TYPE": "STATIC_LIBRARY"
      }
    }
  ]
}
```

Create `./CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 4.0)
project(ReflectionTest LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 26)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(main main.cpp)
```

Run

```bash
cmake --preset p2996
cmake --build build --target main
```
