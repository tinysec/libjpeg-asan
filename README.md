# libjpeg-asan

ASAN-instrumented build of **Chrome's libjpeg-turbo** — the JPEG codec Chrome ships — plus ready-to-run libFuzzer harnesses. Rebuilt automatically whenever the revision Chrome uses changes.

![build](https://github.com/tinysec/libjpeg-asan/actions/workflows/build.yml/badge.svg)
![track](https://github.com/tinysec/libjpeg-asan/actions/workflows/track.yml/badge.svg)
![release](https://img.shields.io/github/v/release/tinysec/libjpeg-asan?label=release)

## Staying current

`chrome.lock` pins the exact libjpeg-turbo revision a specific Chrome stable
version ships (Chrome pins libjpeg-turbo inline in its DEPS). The
[`track-chrome`](.github/workflows/track.yml) workflow runs every 6 hours: it
resolves the latest Chrome stable, extracts the libjpeg-turbo SHA from Chrome
DEPS, and when it has changed it bumps `chrome.lock`, tags `chrome-<version>`,
and triggers [`build`](.github/workflows/build.yml). Each `chrome-<version>` tag
becomes a GitHub release.

The build compiles the **upstream libjpeg-turbo release** that matches the
Chrome-vendored version (read from the fork's `jconfig.h`), using its native
CMake — the Chrome fork ships only BUILD.gn.

## Release artifacts

Each release is published at its `chrome-<version>` tag as one **zip per
platform and build mode**:

- **ASAN build** (`*-asan-*`): the ASAN **static** libjpeg + libturbojpeg
  (`libjpeg.a` / `jpeg-static.lib`, `libturbojpeg.a` / `turbojpeg-static.lib`)
  and **dynamic** libs (`libjpeg.so.*` / `jpeg62.dll`, `libturbojpeg.so.*` /
  `turbojpeg.dll`), the public headers (`jpeglib.h`, `jmorecfg.h`, `jerror.h`,
  `turbojpeg.h`, generated `jconfig.h`), and on Windows the ASAN runtime DLL.
- **libFuzzer build** (`*-libfuzzer-*`, Linux only): self-contained libFuzzer
  binaries (`cjpeg_fuzzer`, `compress*_fuzzer`, `transform_fuzzer`,
  `libjpeg_turbo_fuzzer`, …). The upstream harnesses `#include <unistd.h>`,
  which clang-cl/MSVC does not ship, so libFuzzer binaries are built on Linux
  only; Windows still ships the ASAN static + dynamic libraries.

## ASAN runtime model (Windows)

The Windows LLVM toolchain ships **dynamic-only** ASAN. Every Windows library
here imports `__asan_*` from `clang_rt.asan_dynamic-x86_64.dll`, which is
shipped inside each Windows zip. **That DLL must sit beside your executable /
DLL at runtime.** Linux links ASAN statically and needs no extra runtime.

## Fuzzing on Windows

- **Run the shipped libFuzzer binaries directly** (Linux): e.g.
  `cjpeg_fuzzer corpus/`.
- **AFL++ / WinAFL (Windows):** link the ASAN **static** library into a harness
  exposing a target function over a memory buffer, then point WinAFL at that
  function. The static lib + shipped ASAN DLL is the same combination the
  libFuzzer binaries use.

> These builds provide the ASAN-instrumented library to target. They do not
> bundle the AFL++ / WinAFL runner itself (a separate DynamoRIO-based
> toolchain on Windows).
