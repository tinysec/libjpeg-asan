# libjpeg-asan

ASAN-instrumented fuzzer build of **Chrome's libjpeg-turbo** — the JPEG codec Chrome ships. Rebuilt automatically when the revision Chrome uses changes.

![build](https://github.com/tinysec/libjpeg-asan/actions/workflows/build.yml/badge.svg)
![track](https://github.com/tinysec/libjpeg-asan/actions/workflows/track.yml/badge.svg)
![release](https://img.shields.io/github/v/release/tinysec/libjpeg-asan?label=release)

**Engines:** libFuzzer (Linux + Windows) · AFL++ (Linux) · **Sanitizer:** ASAN · **Symbols:** Windows `.pdb` included

`track.yml` polls Chrome stable **daily** and resolves the libjpeg-turbo SHA Chrome actually ships (Chrome's inline `libjpeg_turbo.git@` pin in DEPS). Unchanged → nothing happens. Changed → bump + tag `chrome-<version>` → `build.yml` runs → new Release.

> No PAT or secret needed — the tracker runs on the default `GITHUB_TOKEN` and dispatches the build via `workflow_dispatch`.

```bash
git tag v1 && git push origin v1   # manual trigger
```

No harness is committed here — the build uses libjpeg-turbo's native `-DWITH_FUZZ=1` targets from the cloned source tree.
