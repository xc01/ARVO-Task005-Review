# Task 006 Build Feasibility

## Build system retained

- `/src/assimp/CMakeLists.txt`: present.
- `/src/assimp/fuzz/assimp_fuzzer.cc`: present.
- `/src/build.sh` (ARVO-level build entry): present, contains:
  ```
  cmake CMakeLists.txt -G "Ninja" -DBUILD_SHARED_LIBS=OFF -DASSIMP_BUILD_ZLIB=ON ...
  cmake --build .
  $CXX $CXXFLAGS -std=c++11 -I$SRC/assimp/include fuzz/assimp_fuzzer.cc -o $OUT/assimp_fuzzer ...
  ```
- Clang/Clang++ 11: present at `/usr/local/bin/`.
- `compile` and `compile_libfuzzer`: present in `/usr/local/bin/`.

## `arvo compile` support

The `/bin/arvo` script supports `arvo compile` (dispatches to `/usr/local/bin/compile`), which invokes `/src/build.sh` via `BUILD_CMD="bash -eux $SRC/build.sh"`.

## Build cache

- No `/work/build`, no `/src/assimp/CMakeCache.txt`, no build directory retained. No reusable compilation cache.

## Host tools

- Host cmake 3.22.1 and clang available, but builds must run inside the container to match the original toolchain and ASan configuration.

## Additional downloads

All dependencies (zlib via `-DASSIMP_BUILD_ZLIB=ON`, IrrXML, clang toolchain) appear present in the image. No network-dependent downloads are expected for a rebuild from the existing `/src` tree. Whether `arvo compile` actually completes without errors is unverified (no compile was run per task constraints).

## Baseline interaction (important for rebuilds)

The build tree at `/src/assimp` is checked out at `575ce6136d4941a45c6ba894947d0e79fe032aff`, which is an ancestor of the official fix parent `10737165…` separated by 5 upstream commits. A rebuild from this tree reproduces the shipped vulnerable binary; it will not reproduce a binary built from the ARVO-recorded parent revision.

Because `chartype_table` does not exist at `575ce613`, the `isNumeric` hardening hunk of the official patch has no target in this tree. Any rebuild experiment must account for the baseline choice before applying patches.
