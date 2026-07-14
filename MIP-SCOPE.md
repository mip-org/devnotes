# mip's scope: more than MEX distribution

MEX distribution was one of the founding motivations for mip — getting
compiled C/C++/Fortran code into the hands of MATLAB users who don't have (and
shouldn't need) a compiler. But mip is a general package manager, and most of
what it does applies equally to pure-MATLAB code.

## Path management

MATLAB's path is global, session-scoped state, and managing it by hand is the
root of a lot of pain: shadowed functions, stale `addpath` lines in
`startup.m`, packages that step on each other. mip separates **installing** a
package (downloading it into a managed store) from **loading** it (putting its
declared directories on the path for the current session):

```matlab
mip install chebfun     % download into the package store
mip load chebfun        % add its paths (and its dependencies') for this session
mip unload chebfun      % remove them again, pruning unused dependencies
```

Each package declares exactly which of its directories belong on the path, so
loading is precise — no `genpath` over a whole source tree. Unloading removes
those entries and sweeps any residual ones, leaving the session clean.

## Versioning and updates

Channels publish versioned releases. Users can request a version explicitly
(`mip install chebfun@1.2.0`), pin packages against updates (`mip pin`), and
update everything else (`mip update --all`). Following a branch (installing
`@main`) is respected by `mip update` — it refreshes the branch build rather
than silently switching to a numeric release.

## Dependencies

Packages declare dependencies; mip resolves them recursively, installs them in
topological order, loads them before the package that needs them, and prunes
them automatically when nothing needs them anymore. Direct installs are
tracked separately from transitively-installed dependencies, so cleanup never
removes something the user asked for by name.

## Self-hosted channels

A channel is a package index hosted on GitHub Pages; anyone can host one to
distribute their own packages, and users can subscribe (`mip channel add
mylab/dev`) so bare-name installs resolve against it. The default channel is
`mip-org/core`. All the CI machinery that builds, tests, and publishes
packages is shared tooling ([mip_channel_tools](https://github.com/mip-org/mip_channel_tools))
that any channel reuses.

## Installs from outside channels

mip also installs from local directories (including editable, in-place
installs for development), from zip URLs, from
[File Exchange landing pages](FILE-EXCHANGE-INSTALLS.md), and from `.mhl`
bundle files passed around directly.

## The MEX story

For packages with compiled code, channels build per-architecture binaries
(Linux, macOS Intel/ARM, Windows, WASM) and `mip install` downloads the one
matching the user's platform — no compiler needed. Making those binaries
actually load on end-user machines across MATLAB releases and OS versions is
its own engineering problem; see [MEX-COMPATIBILITY.md](MEX-COMPATIBILITY.md).

## Roadmap

- **Virtual environments and project workflows** — uv-style project
  manifests (`mip project add` / `mip project run`) and disposable
  environments.
- **mip-cli** — a standalone command-line binary that runs the same installed
  mip without starting MATLAB (interpreting the mip MATLAB source via numbl).
- **Minimum MATLAB release metadata** — a `mip.yaml` field so a package that
  needs newer language features can refuse to install on older releases.
- **SIMD-level variants** — per-CPU-capability builds selected at install
  time, alongside the existing per-architecture builds.
