# mip and the MATLAB build tool

How mip packages build today, and how that relates to MATLAB's `buildtool`
(the declarative, task-based build framework introduced in R2022b, driven by
a `buildfile.m`; its built-in MEX task, `matlab.buildtool.tasks.MexTask`, was
added in R2024a).

## How mip packages build today

A package's `mip.yaml` declares one or more `builds:` entries, each keyed by
architecture, optionally naming a `compile_script` — an ordinary M-file that
runs in the package source directory:

```yaml
builds:
  - architectures: [linux_x86_64, macos_arm64, windows_x86_64]
    compile_script: compile.m
  - architectures: [any]
```

The script is arbitrary MATLAB code. For a simple package it's a few `mex`
calls; for heavier ones it drives CMake + vcpkg, patches upstream build
files, or post-processes binaries. It runs in channel CI when the package is
built for each architecture, and locally for editable installs (`mip install
-e`, re-runnable via `mip compile`).

This choice was deliberate:

- **It wraps existing upstream builds.** Most packages worth distributing
  already have a build (Makefiles, CMake, hand-written `mex` scripts). A
  script that invokes whatever exists is the lowest-friction adapter; a
  declarative format would need escape hatches for all of it anyway.
- **Architecture branching lives in `mip.yaml`.** Different platforms often
  need genuinely different build steps (see
  [MEXOPTS.md](MEXOPTS.md), [MATLAB-MINGW.md](MATLAB-MINGW.md),
  [WINDOWS-TAR-XZ.md](WINDOWS-TAR-XZ.md) for why); selecting a per-arch
  `builds:` entry keeps that explicit.
- **It works on every MATLAB release.** Channel builds intentionally run on
  old releases to maximize binary compatibility
  ([MEX-COMPATIBILITY.md](MEX-COMPATIBILITY.md)) — Linux builds on
  **R2022a**, which predates `buildtool` itself.

## Where `buildtool` fits

For MEX specifically, `MexTask` wraps the same `mex` command a compile
script would call, with two additions: declared inputs/outputs, so an
unchanged task is skipped as up-to-date on rebuild, and `MexTask.forEachFile`
(R2025a), which generates one task per source file. The incremental-build
machinery is genuinely useful for local development on a package with many
MEX targets; it buys little in channel CI, where every build starts in a
fresh container and compiles once.

Nothing prevents a package from using `buildtool` today: a one-line
`compile_script` that calls `buildtool mex` (or any task) works as-is,
provided the build runs on a release that has the tasks it uses.

Deeper integration is plausible and worth considering: if a source tree
ships a `buildfile.m` and no `compile_script`, mip could invoke `buildtool`
by convention — the same spirit as `mip init` auto-detecting paths. Two
things would need answers first:

1. **Release floor.** The channel build strategy pins old MATLAB releases,
   and all three pinned build releases (R2022a Linux, R2023a Windows,
   R2023b macOS arm64) predate `MexTask` (R2024a). Building channel MEX via
   `buildtool` would mean raising the build release — directly trading away
   binary-compatibility reach ([MEX-COMPATIBILITY.md](MEX-COMPATIBILITY.md)).
   Packages targeting the `any` architecture, or local/editable workflows on
   a modern MATLAB, don't have this constraint.
2. **Per-architecture variation.** A single `buildfile.m` would need to
   express the platform-specific steps that per-arch `builds:` entries
   handle today.

Position: `compile_script` stays the primitive; `buildtool` support would be
an opt-in convention on top, not a requirement.
