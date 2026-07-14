# MEX binary compatibility across MATLAB versions and systems

How mip channels build MEX binaries so that one published build per
architecture loads correctly on end-user machines — across MATLAB releases,
OS versions, and system library versions. This is the overview; the deep
dives are [MATLAB-GCC.md](MATLAB-GCC.md), [MATLAB-GLIBC.md](MATLAB-GLIBC.md),
and [MEX-RUNTIME-LIBS.md](MEX-RUNTIME-LIBS.md).

## Three compatibility axes

A MEX binary (on Linux; macOS and Windows have analogues) must satisfy three
independent constraints, and the strategy for each is different:

| Axis | Pinned by | Direction |
|---|---|---|
| **MEX API** (`libmx`/`libmex` + MEX-file version stamp) | the MATLAB release you link against | forward-compatible only — runs on the build release and newer, never older |
| **Compiler runtime ABI** (`libstdc++`/`libgfortran`, which MATLAB ships its own copies of) | the GCC version you compile with | backward-compatible — build with old GCC, load in newer MATLAB |
| **System ABI** (glibc) | the build host's glibc | backward-compatible — build on old glibc, run on newer systems |

The effective floor is set by the highest of the three — and because the MEX
API axis is forward-compatible only, **the floor is the MATLAB release you
build with**. So the rule everywhere is: build at the oldest point of each
axis you intend to support.

A non-obvious wrinkle on the second axis: MATLAB's documented compiler
support overstates what its *bundled* `libstdc++` actually provides. R2023a
claims GCC 10 support but ships `libstdc++` 6.0.28, which only covers symbols
up to GCC 10.1 — and earlier releases are worse. The measured
release-by-release table is in [MATLAB-GCC.md](MATLAB-GCC.md).

## What the channel builds actually pin

- **Linux** — MATLAB **R2022a**, built inside the
  `mathworks/matlab-deps:r2022a-ubi8` container: glibc **2.28** (RHEL 8, the
  oldest glibc MathWorks supports for MATLAB containers) with the stock UBI 8
  **GCC 8.5**, whose `GLIBCXX`/`libgfortran` requirements are within what every
  MATLAB ≥ R2020b ships. A bare `ubuntu-latest` runner is not usable: its
  glibc floor silently rises with runner image updates
  ([MATLAB-GLIBC.md](MATLAB-GLIBC.md) records the failure that proved this).
- **macOS arm64** — MATLAB **R2023b**, the oldest Apple-Silicon-native
  release. The deployment floor on macOS is set by the statically-linked
  Homebrew bottles, not by `-mmacosx-version-min`
  ([MACOS-DEPLOYMENT-TARGET.md](MACOS-DEPLOYMENT-TARGET.md)).
- **Windows** — MATLAB **R2023a**, the oldest release that certifies the
  pinned MinGW-w64 8.1.0 toolchain ([MATLAB-MINGW.md](MATLAB-MINGW.md)).

So a published binary loads on its architecture's build release and
everything newer. In practice the MEX API's forward compatibility has held —
we have not hit a newer MATLAB refusing a binary built on these releases.

## Static linking and runtime libraries

Third-party dependencies (Boost, CGAL, Eigen, …) are **statically linked** —
end-user machines can't be assumed to have matching versions of anything.
Only OS-provided libraries stay dynamic. The exceptions are handled
deliberately:

- Libraries MATLAB itself ships (`libstdc++`, `libgfortran`, `libquadmath`,
  `libz`) are left to MATLAB — its copies are first on `LD_LIBRARY_PATH`, so
  a bundled copy would be shadowed anyway.
- Libraries MATLAB does *not* ship (notably `libgomp` for OpenMP code) are
  bundled next to the MEX with an `$ORIGIN` RPATH.

The full decision procedure, including why bundling is deliberately
non-recursive, is in [MEX-RUNTIME-LIBS.md](MEX-RUNTIME-LIBS.md).

## How this is verified

Every channel build is gated by tests designed to catch exactly the failures
this note is about:

- **Strip-then-test.** After bundling, CI uninstalls the entire build
  toolchain — compilers, and on Linux purges `libgfortran`/`libgomp`/
  `libquadmath` from the system with a verification step that fails the build
  if any remain — then installs the built `.mhl` and runs the package's test
  suite in that stripped environment. A MEX that quietly depends on a
  build-machine library fails to load, and the build goes red before anything
  is published.
- **Every MEX must be exercised.** The test harness walks the built package
  for MEX binaries and checks each one appears in `inmem` after the test
  script runs; a built-but-never-loaded MEX fails the build. The most common
  MEX failure mode is failing to *load* at all, so a test that never calls a
  MEX would prove nothing about it.

## Open items

- **Scheduled multi-version testing.** Build-time tests run on the (old)
  build release. A scheduled job that installs published packages on multiple
  MATLAB releases, OSes, and architectures — as an end user would — gives
  ongoing forward-compatibility coverage
  ([mip_integration_tests](https://github.com/mip-org/mip_integration_tests)).
- **Minimum MATLAB release metadata.** Compatibility isn't only about
  binaries — a pure-MATLAB package using newer language syntax needs a way to
  declare the oldest release it supports. Planned as a `mip.yaml` field.
- **SIMD-level variants.** Per-CPU-capability builds (beyond per-architecture)
  selected at install time.
