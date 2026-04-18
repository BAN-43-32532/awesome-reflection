# Awesome Reflection

A collection of C++26 reflection materials, _with_ a tutorial for **building [clang-p2996](https://github.com/bloomberg/clang-p2996/tree/p2996) on Windows**.

## List of Examples

Unless otherwise specified, all examples are compiled with the latest [clang-p2996](https://github.com/bloomberg/clang-p2996/tree/p2996) implementation, with compile flags `-std=c++26 -freflection-latest -fexpansion-statements`. 

### From [C++ meetup Brno | Herb Sutter: Reflection - C++´s Decade-Defining Rocket Engine](https://youtu.be/H4NiR9xe1XA):

- [List data members with template-for](https://godbolt.org/z/WeaKzWc56)

- [Compile-time JSON parser with aggregate class generation](https://godbolt.org/z/3jzehW1bv)

  Use `#embed` with compile flag `-Wno-c23-extensions`.

- [Polymorphism code generation](https://godbolt.org/z/aP4dKfqjo)

  A pre-C++26 implementation: <https://godbolt.org/z/MEPvEMbqP>

- [Class interface code generation](https://godbolt.org/z/Y75Wr3nGe)

- [Class interface code injection](https://godbolt.org/z/7jv9aj8Mh)

  Based on Experimental EDG Reflection Implementation (<https://github.com/cern-nextgen/reflmempp>)

### From [reflection for C++26如期而至](https://zhuanlan.zhihu.com/p/1919860255870948496):

- [Enum to string](https://godbolt.org/z/8a6W9vjzz)

  Change `enum Color` in line 17 to e.g. `enum class Color` or `enum Color: int`.

- [An implementation of tuple](https://godbolt.org/z/b6K6YvddP)

- [Argument parser](https://godbolt.org/z/YovYWsx17)

### Experimental JSON Builder based on reflection:

<https://github.com/simdjson/experimental_json_builder>

### Compile-Time TOML parser (WIP):

<https://github.com/BAN-43-32532/toml26>

## A Tutorial for Building [Clang-P2996](https://github.com/bloomberg/clang-p2996/tree/p2996) on Windows

### 1. Install MSYS2

Download and install [MSYS2](https://www.msys2.org/). It is recommended to install it at the default path (`C:\msys64`) which the following assumes.

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

1. Open MSYS2 MSYS. Run `pacman -Syu`. Then close and reopen the terminal, and run `pacman -Syu` again.
2. Open MSYS2 CLANG64 and install the prerequisites:

```bash
pacman -S mingw-w64-x86_64-toolchain mingw-w64-clang-x86_64-toolchain mingw-w64-clang-x86_64-cmake mingw-w64-clang-x86_64-ninja git
```
Run `clang --version` etc. to verify installation and check the clang version.

3. Add `C:\msys64\clang64\bin` and `C:\msys64\mingw64\bin` to Windows system path.

### 4. Fetch [Clang-P2996](https://github.com/bloomberg/clang-p2996/tree/p2996) source

Clone the specific p2996 branch. We use a shallow clone to save download time and disk space.

```bash
git clone --depth 1 --branch p2996 https://github.com/bloomberg/clang-p2996.git
cd clang-p2996
```

### 4.5. Workaround for C++ module support

If you intend to use C++20 modules or `std` module with CMake and `clang-p2996`, open `clang-p2996/clang/lib/Frontend/CompilerInvocation.cpp` and apply the following changes:

- Locate `getExceptionHandlingName` function definition. Add the following to the switch block:

```cpp
case LangOptions::ExceptionHandlingKind::WinEH:
return "seh";
```

- Locate `ParseLangArgs` function definition. Make the following modification:

```cpp
.Case("sjlj", LangOptions::ExceptionHandlingKind::SjLj)
.Case("seh", LangOptions::ExceptionHandlingKind::WinEH) // ADD THIS LINE
.Case("wineh", LangOptions::ExceptionHandlingKind::WinEH)
```

The effectiveness may vary with future updates to `clang-p2996` and `CMake`. See [this](https://github.com/BAN-43-32532/cmake-import-std) for a practical use of C++ modules. 

### 5. Build Clang-P2996

Run the following scripts in MSYS2 CLANG64. Note that `-Wno-error` is required for building `clang-p2996` (based on Clang 21.0.0) with Clang 22 installed from Step 3. 

```bash
cmake -S llvm -B build -G Ninja \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_INSTALL_PREFIX="$PWD/install" \
  -DLLVM_ENABLE_PROJECTS="clang;lld" \
  -DLLVM_ENABLE_RUNTIMES="libcxx;libcxxabi;libunwind" \
  -DLLVM_TARGETS_TO_BUILD="X86" \
  -DLLVM_DEFAULT_TARGET_TRIPLE=x86_64-w64-windows-gnu \
  -DCMAKE_C_COMPILER=clang \
  -DCMAKE_CXX_COMPILER=clang++ \
  -DLIBCXX_ENABLE_SHARED=OFF \
  -DLIBCXXABI_ENABLE_SHARED=OFF \
  -DLIBUNWIND_ENABLE_SHARED=OFF \
  -DLIBCXX_USE_COMPILER_RT=OFF \
  -DLIBCXXABI_USE_COMPILER_RT=OFF \
  -DLIBCXXABI_USE_LLVM_UNWINDER=ON \
  -DLIBCXX_INSTALL_MODULES=ON \
  -DCMAKE_C_FLAGS="-Wno-error" \
  -DCMAKE_CXX_FLAGS="-Wno-error" \
  -DRUNTIMES_CMAKE_ARGS="-DCMAKE_SYSROOT=$MSYSTEM_PREFIX"

ninja -C build install
```

### 6. Build Your C++26 Code

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
        "CMAKE_CXX_FLAGS": "-D_UCRT -IC:/path/to/clang-p2996/install/include/c++/v1 -O3 -stdlib=libc++ -fparameter-reflection -Wno-c23-extensions -freflection-latest -fexpansion-statements",
        "CMAKE_EXE_LINKER_FLAGS": "-LC:/path/to/clang-p2996/install/lib -static -fuse-ld=lld -lc++abi -lunwind -lucrt",
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

add_executable(test main.cpp)
```

Run

```bash
cmake --preset p2996
cmake --build build --target main
```
