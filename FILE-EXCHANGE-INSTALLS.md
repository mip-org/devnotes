# Installing File Exchange packages with mip

> **Planned change (1.1.0):** the syntax documented here is slated to flip —
> you will paste the File Exchange (or zip) URL directly as the argument and
> mip will infer the package name, instead of naming the package and passing
> the URL via `--url`. See
> [Planned change: URL-first syntax](#planned-change-url-first-syntax-110)
> below.

mip can install a package directly from a MATLAB File Exchange landing page —
the URL you'd copy from your browser:

```matlab
mip install export_fig --url https://www.mathworks.com/matlabcentral/fileexchange/23629-export_fig
```

## How it works

1. The landing-page URL is resolved to the underlying zip download by
   appending `?download=true` and following the redirect to the CDN URL.
2. The zip is downloaded and extracted.
3. File Exchange entries don't ship a `mip.yaml`, so mip generates one via
   `mip init`, which walks the tree and makes a best guess at which
   directories belong on the MATLAB path. The resolved zip URL is recorded in
   the generated `repository` field.
4. The package installs into the managed store under the `fex/` source type
   (`fex/<name>`), separate from channel-published packages, and then loads
   like any other package.

Limitations: only zip-distributed entries work (no `.mltbx`); and since the
original archive isn't preserved, `mip update` skips these — re-run the
install to pick up a new upstream version. The generated path guesses can be
adjusted by editing the `mip.yaml` or with `mip load --addpath`/`--rmpath`.

## Why an install needs a name

mip separates installing from loading, and loading is by name (`mip load
export_fig`) — so every installed package needs one, and the package-store
directory layout is keyed by it. That's the reason the current syntax takes a
positional name with the URL in a flag.

## Planned change: URL-first syntax (1.1.0)

The current syntax has the emphasis backwards — the natural user action is
pasting the URL, not inventing a name:

```matlab
% current (1.0.x)
mip install export_fig --url https://www.mathworks.com/matlabcentral/fileexchange/23629-export_fig

% planned (1.1.0)
mip install https://www.mathworks.com/matlabcentral/fileexchange/23629-export_fig
```

The plan:

- Accept a File Exchange landing-page URL (or generic zip URL) as the
  positional argument.
- Auto-detect the name. File Exchange URLs follow a parseable
  `<5-digit-id>-<slug>` format; when the slug is a title rather than a clean
  name (e.g. `2555-mesh2d-delaunay-based-unstructured-mesh-generation`),
  fall back to other signals such as the entry's linked GitHub repository
  name, or the zip filename for generic URLs.
- Confirm interactively (`Package name [mesh2d]:`), with `--name` to
  override non-interactively.

This is a breaking change to the `--url` form, scheduled for 1.1.0 while the
feature has effectively no users.
