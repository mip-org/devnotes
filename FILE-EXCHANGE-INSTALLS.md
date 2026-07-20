# Installing File Exchange packages with mip

mip can install a package directly from a MATLAB File Exchange landing page —
the URL you'd copy from your browser. The URL is the positional argument:

```matlab
mip install https://www.mathworks.com/matlabcentral/fileexchange/23629-export_fig
```

mip infers the package name from the URL and asks you to confirm it. Pass
`--name` to set it non-interactively:

```matlab
mip install https://www.mathworks.com/matlabcentral/fileexchange/23629-export_fig --name export_fig
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
directory layout is keyed by it. A URL install doesn't come with a name, so
mip derives a default from the URL and asks you to confirm it.

## How the name is chosen

mip auto-detects a name from the URL. File Exchange URLs follow a parseable
`<5-digit-id>-<slug>` format; when the slug is a title rather than a clean
name (e.g. `2555-mesh2d-delaunay-based-unstructured-mesh-generation`), mip
falls back to other signals such as the entry's linked GitHub repository
name, or the zip filename for generic URLs. It then confirms interactively:

```
Package name [mesh2d]:
```

Press Enter to accept the default. `--name` overrides it non-interactively,
and `MIP_CONFIRM=y` accepts the default without prompting.

## Migration from the old `--url` flag

Before 1.1.0 the name was the positional argument and the URL was passed via
a `--url` flag (`mip install export_fig --url <url>`). That form now raises an
error pointing at the URL-first syntax — pass the URL directly instead.
