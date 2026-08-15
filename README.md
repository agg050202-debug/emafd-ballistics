# Superior Ballistics — published build

The public build of the emafd external-ballistics solver, served at
**[ballistics.emafd.com](https://ballistics.emafd.com/)**.

This repository holds the deployed artefact and nothing else. `index.html` is
the entire application in one self-contained file: no build step, no framework,
no CDN. It solves entirely in the browser and runs from `file://`, so a copy on
a phone or a USB stick works where there is no signal.

## This build

```
sha256  696b05292535edcea8309ab8d44d2222cc3028bbef4eb996300f3a89caf9c3f1
bytes   1147908
stamp   1.1.0 · 2026-08-15 23:27 UTC   (shown in the page footer)
```

**Match on the whole stamp, never on the version number.** Four separate builds
have now shipped as `1.1.0` — 2026-08-13 17:49, 2026-08-14 02:22, 2026-08-14
20:09 and 2026-08-15 23:27 — and only the timestamp separates them. A bug report
citing "1.1.0" identifies four different programs.

The build stamp is at the bottom of the page. Quote it when reporting anything,
otherwise there is no way to tell whether you were looking at a build that has
already been fixed.

The file must not be reformatted, minified, or passed through any optimiser. It
carries CRLF line endings and `.gitattributes` marks it `-text` so Git leaves
them alone; normalising them would not change how the solver runs, but it would
invalidate the hash above.

## Network access

The solver needs no connection. Two optional buttons on the Environment screen
are the only things that ever leave the device, and neither fires unless it is
pressed:

- **Use my location** — the browser's geolocation API.
- **Fetch live weather** — a request to `api.open-meteo.com` carrying the
  coordinates.

Both require the page to be served over **HTTPS**: geolocation is blocked on
insecure origins, and an `https` call from an `http` page is blocked as mixed
content. This deployment enforces HTTPS, so both work. Everything else keeps
working regardless.

From the 2026-08-15 build the app can also upload its shot log, but **that is
switched off in this deployment and sends nothing.** The upload only happens if
`window.SB_LOG_ENDPOINT` is defined before the app loads, and nothing here
defines it — shots stay in the browser's own `localStorage`
(`superior-ballistics-shotlog-v1`) and are exported by hand from the Output
screen as JSON or CSV.

Turning it on is not a deployment change. The bundle has no configuration hook:
`src/index.html` loads its scripts directly, so the global has to be set in the
source and the bundle rebuilt. Editing the shipped file to inject it would work
and would also invalidate the sha256 above, which is the one thing this
repository exists to keep honest.

## Validation

The engine is a port of the Python solver it was derived from, and is checked
against it rather than trusted. Both suites were re-run against this exact
build before it was published:

- **22 of 22** cross-validation cases pass per cell across the whole range
  card. The canonical 7.62 mm M852 168 gr reference reads −1357.3097 cm of drop
  at 1000 m, a deviation of 1.72e-5 cm against a 0.02 cm tripwire.
- Numeric primitives — every drag table and spline — are **bit-identical to
  SciPy**, 0 ULP.

## Releasing a new build

```bash
# from the source project, only if src/ changed
python tools/build.py src/index.html dist/index.html

# then, here
cp <source>/dist/index.html index.html
sha256sum index.html          # must match what the source project reports
git commit -am "Superior Ballistics X.Y.Z"
git push
```

Update the hash, size and stamp in this file at the same time. GitHub Pages
redeploys within about a minute. If `src/engine/` was touched, re-run both test
pages first — they are the only thing separating a plausible number from a
correct one.

## Scope

Superior Ballistics is an engineering-analysis and research tool, published for
numerical study, model comparison and validation work. It is not an operational
aid, and every result should be verified against measured data before it is
relied upon.

## Licence

Copyright © 2026 emafd — Engineering Consulting Associates. **All rights
reserved.** Public visibility is not a licence: see [LICENSE](LICENSE). No
permission is granted to copy, modify, redistribute or create derivative works
without prior written permission.
