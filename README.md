# Superior Ballistics — published build

The public build of the emafd external-ballistics solver, served at
**[ballistics.emafd.com](https://ballistics.emafd.com/)**.

This repository holds the deployed artefact and nothing else. `index.html` is
the entire application in one self-contained file: no build step, no framework,
no CDN. It solves entirely in the browser and runs from `file://`, so a copy on
a phone or a USB stick works where there is no signal.

## This build

```
sha256  c4c7322e0cf92a12b9156df62a6d3777068848f5eb530a8f4c5fe485893bf477
bytes   1203236
stamp   1.5.0 · 2026-08-21 20:57 UTC   (shown in the page footer)
```

**Match on the whole stamp, never on the version number.** Four separate builds
shipped as `1.1.0` — 2026-08-13 17:49, 2026-08-14 02:22, 2026-08-14 20:09 and
2026-08-15 23:27 — and only the timestamp separates them, so a bug report
citing "1.1.0" identifies four different programs. From 1.2.0 the number moves
with the content, and `SuperiorBallistics.version` agrees with it; before that
it reported 1.1.0 from inside 1.2.x builds.

**Some numbers are ambiguous and always will be.** Three distinct programs
carry `1.4.1` — `2026-08-19 03:39`, `04:12` and `2026-08-20 01:32` — and five
carry `1.5.0`: `2026-08-20 02:15`, `02:55`, `2026-08-21 19:16`, `20:13` and
`20:57`.
Ask for the whole stamp; a bare version number is not an answer. It matters
here: `02:55` shipped with five screens invisible and `19:16` is the repair, so
those two are not interchangeable in a bug report.

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

Since 1.4.1 the wind toggle starts on **HOLD YOURSELF** rather than WIND IN
DOPE. Saved sessions keep whatever the shooter chose; only a fresh visitor sees
the new default. It changes nothing in the solver and everything in the data:
`wind_in_dope = false` becomes the common case, and those records read as
enormous windage errors unless they are filtered, because the correction was
meant to be held on the reticle rather than dialled.

Both require the page to be served over **HTTPS**: geolocation is blocked on
insecure origins, and an `https` call from an `http` page is blocked as mixed
content. This deployment enforces HTTPS, so both work. Everything else keeps
working regardless.

From the 1.2.0 build the app **uploads each logged shot as it is saved**, to a
Supabase edge function. It is the third and last thing that leaves the device:

- **Upload the shot log** — a POST to the project's `shot` function.

A failed upload is not a lost shot. The record is already in `localStorage`
(`superior-ballistics-shotlog-v1`), stays marked unsent, and
`SuperiorBallistics.log.retryUnsent()` sweeps the backlog. Nothing about it
blocks the interface, and manual JSON/CSV export from the Output screen still
works exactly as before.

The endpoint is set by `window.SB_LOG_ENDPOINT` in `src/index.html`, before the
app scripts run, so switching it off means rebuilding from source rather than
editing what is served. It targets an edge function rather than the database
API because the app sends only `Content-Type` and cannot add the `apikey` and
`Authorization` headers the table API requires; the function also answers the
CORS preflight and discards longitude before storing.

### Shared presets

From 1.5.0 the app also publishes and reads **shared presets** — rifle, bullet,
sight, or all three at once — through a second edge function. Presets always
save to the shooter's own browser; the shared list is what travels.

**That list is deliberately open.** No accounts, no authentication: anyone may
read it and anyone may publish to it. That was a considered choice, and
moderation is deletion by the owner. The function authenticates nobody; it
validates only shape — format, kind, a payload, and a 64 KB body cap — because
a malformed or enormous record would spoil the list for everyone. The table
carries a `hidden` flag so a preset can be withdrawn from the list without
being destroyed.

What the client can defend it cannot fix: remote text is written with
`textContent` and never as markup, lengths are capped and control characters
stripped, and a browser may delete only what it authored. Impersonation — the
author field is free text — and flooding remain open.

**The endpoint is open by construction.** A static page cannot hold a secret,
so anyone with the URL can insert rows. The table is append-only from outside —
row-level security is on with no policies, so the anon key can neither read nor
write, and only the function's service role reaches it. Treat the contents as
untrusted input rather than as a guarantee that every row came from a real
shot.

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
