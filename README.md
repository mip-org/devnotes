# mip devnotes

The developer knowledge base for the [mip](https://mip.sh) project — technical
background, design rationale, and deep dives that don't belong in the
user-facing docs at [mip.sh/docs](https://mip.sh/docs).

Conventions:

- One topic per file, flat directory, `SCREAMING-KEBAB.md` names.
- Commit directly — no PRs required for this repo.
- Keep each fact in one place; link between notes rather than duplicating.
  When something changes, fix it here, not in a copy elsewhere.

## Overview notes

Mid-level reading — start here; each links down to the deep dives.

| Note | What it covers |
|---|---|
| [MIP-SCOPE.md](MIP-SCOPE.md) | What mip is for — package/path/dependency management, channels, versioning; MEX distribution is one part |
| [MEX-COMPATIBILITY.md](MEX-COMPATIBILITY.md) | How mip builds MEX binaries that work across MATLAB versions, OSes, and system library versions |
| [MATLAB-BUILDTOOL.md](MATLAB-BUILDTOOL.md) | How mip packages build today and how that relates to MATLAB's `buildtool` |
| [FILE-EXCHANGE-INSTALLS.md](FILE-EXCHANGE-INSTALLS.md) | Installing File Exchange packages with mip, and the planned URL-first install syntax |

## Platform deep dives

MATLAB/toolchain/OS compatibility facts that hold independently of any one
repo. Originally developed in `mip_channel_tools/notes/`.

| Note | What it covers |
|---|---|
| [MATLAB-GCC.md](MATLAB-GCC.md) | MATLAB's bundled `libstdc++`/`libgfortran` vs the GCC versions it claims to support; the three compatibility axes; `LD_PRELOAD` escape hatch |
| [MATLAB-GLIBC.md](MATLAB-GLIBC.md) | The glibc axis: why the build host's glibc sets the Linux floor and why builds run in the `matlab-deps:r2022a-ubi8` (glibc 2.28) container |
| [MEX-RUNTIME-LIBS.md](MEX-RUNTIME-LIBS.md) | Which shared libraries get bundled next to a MEX and why bundling is deliberately non-recursive |
| [MACOS-DEPLOYMENT-TARGET.md](MACOS-DEPLOYMENT-TARGET.md) | The macOS floor comes from statically-linked Homebrew bottles, not `-mmacosx-version-min` |
| [MATLAB-MINGW.md](MATLAB-MINGW.md) | MATLAB's MinGW-w64 certification story on Windows and why the Windows MATLAB floor is R2023a |

Notes that document the channel build engine's own files and operation —
mexopts XML rationale (`MEXOPTS.md`, `MACOS-MEX-CPP-LINKER.md`,
`LINUX-LIBSTDCXX-ASNEEDED.md`), build-environment pitfalls
(`WINDOWS-TAR-XZ.md`), local builds (`LOCAL-BUILD.md`), and `builds:` schema
design (`MIP-YAML-BUILDS.md`) — live with that code in
[mip_channel_tools/notes](https://github.com/mip-org/mip_channel_tools/tree/main/notes).

## Related

- User docs: [mip.sh/docs](https://mip.sh/docs)
- Behavior reference: [`mip/docs/specification.md`](https://github.com/mip-org/mip/blob/main/docs/specification.md)
- Channel build engine: [`mip-org/mip_channel_tools`](https://github.com/mip-org/mip_channel_tools)
- Guide for adding a package to a channel: [`mip_channel_tools/adding_a_package.md`](https://github.com/mip-org/mip_channel_tools/blob/main/adding_a_package.md)
