# Superior Ballistics — published build

The public build of the emafd external-ballistics solver, served at
**[ballistics.emafd.com](https://ballistics.emafd.com/)**.

This repository holds the deployed artefacts and nothing else. `index.html` is
the entire application in one self-contained file: no build step, no framework,
no CDN. It solves entirely in the browser and runs from `file://`, so a copy on
a phone or a USB stick works where there is no signal.

## This build

```
sha256  3ceb6a9a922a4a0ddda600ffb4fea85ec96c6baf33f10fdcb9a8b14fe23a63f9
bytes   2700058
stamp   1.13.1 · 2026-10-18 06:18 UTC   (shown in the page footer)
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
`20:57`. Ask for the whole stamp; a bare version number is not an answer. It
matters here: `02:55` shipped with five screens invisible and `19:16` is the
repair, so those two are not interchangeable in a bug report.

**From 1.5.1 the number rises on every release**, so a version identifies a
build again. The stamp is still the safer thing to quote.

**1.5.3 is a correctness boundary for the shot log.** Before it, HOLD YOURSELF
asked for too much hold — the whole spin drift, 0.3 MIL at 800 m with a 15 m/s
crosswind, always to the same side — and the log pre-filled `dialled_wind` with
that drift instead of zero. Records with `wind_in_dope = false` from earlier
builds carry the bias and mean something different by that column. Filter on
`build` before pooling them.

**1.6.0 is a second boundary, and a wider one.** Until it, the app solved the
shot twice — once through the wind and once through still air — and the wind
toggle chose which of the two filled the table, so with `wind_in_dope = false`
the `pred_*` columns described a flight that never happened and `dialled_elev`
was pre-filled from it. From 1.6.0 there is one trajectory, the physical one,
and `pred_*` is always that. The horizontal difference is the whole point and
is large; the vertical one is small but not zero — a pure crosswind moves the
drop by under a hundredth of a MIL, while 15 m/s straight down or up the range
at 800 m moves it by about a quarter of one. `wind_in_dope` keeps its name and
its meaning: whether the correction was dialled or held. Filter on `build`
before pooling across it.

**1.7.0 is not a boundary at all**, and that is worth saying because the two
above train you to look for one. It changes what a preset `kind` means, not
what any shot-log column holds. Nothing in the data moved; do not go hunting
for a step at that stamp.

**1.8.0 is a boundary again, and the worst kind: one no column reveals.**
Before it, a wind fetched from Open-Meteo was converted from a true bearing to
a relative one at the azimuth of that moment, and never again. Set the azimuth
afterwards and `wind_dir_rel_deg` stayed where the old azimuth had put it. A
west wind fetched while facing north and then fired east — a tailwind — was
recorded and solved as a full crosswind from the left: for the M852 at 800 m in
5 m/s, 2.55 MIL of wrong drift, 204 cm, measured on the build that fixes it.
The azimuth persists between sessions, and typing only the speed afterwards
relabelled the source as `typed`, so nothing in an old row says whether it was
hit. Treat every pre-1.8.0 row with wind in it as horizontally suspect wherever
the weather was ever fetched. From 1.8.0 the JSON record carries
`provenance.windFromTrue`, the bearing as fetched; it is not a CSV column, so
the column count did not change.

**1.10.0 is a boundary, and it changes what the log can be used for.** It
carries the changes of 1.9.0, which was built but never served, so no record
is stamped 1.9.0. From this build a shot is logged in one tap or from the table
row, and either way it is a hit: no miss, no impact, no note. A new 66th CSV
column, `entry`, says which way it came in. Rows from 1.10.0 on therefore
measure nothing about the model; only rows from 1.8.0 and earlier carry an
impact. Three other changes move predictions without a column to show it. The
built-in Sellier & Bellot 6.5 Creedmoor goes from G7 0.24831 to 0.247 and from
813.8 to 810 m/s, which adds 0.22 MIL of drop at 1000 m — 22 cm — for the Tikka
starting rifle; `bc` and `mv_mps` show which numbers a row used. Catalogue
bullets now borrow the lift and spin curves of the nearest built-in profile
instead of always the M852's, so most bullets under 6 mm fly with the M855's:
a Sierra MatchKing 77 gr loses 0.07 MIL of spin drift at 1000 m, and its drop
moves by less than 0.005 MIL. And thirteen catalogue bullets had the wrong
calibre, which earlier rows still carry in `calibre_m`. The azimuth can now
come from the phone's compass, so it may be a few degrees off where it used to
be typed. Filter on `build` before pooling across this stamp.

**1.11.0 changes what one flag means.** From this build the shooter can type a
bullet's inertias, and typing either one sets `inertia_estimated` to `false`.
Before it, `false` meant a built-in profile's measured pair; now it can also
mean typed, and a pair with only one value typed still says `false` while the
other is an estimate. Nothing else in the log changed: still 66 columns, and
the new bullet drawing and rifle photo record nothing.

**1.12.0 is not a boundary.** It carries 1.11.1, which was never served — a
new way to create a rifle — and adds a catalogue of scopes and their reticles.
Choosing a scope sets the reticle, the click value and, where the table reads
MIL or MOA, the turret unit, so later rows may show a different `dialled_unit`
or `dialled_click`; both mean what they always did. The log has no reticle
column, and nothing it records changes meaning.

**1.13.0 is a boundary for the wind columns, and no column shows it.** The
wind can now be given by segments along the range. When it is, `wind_mps` and
`wind_dir_rel_deg` hold only the first segment, the wind at the shooter, while
every `pred_*` column comes from the flight through the whole profile; the
whole profile is in the JSON record as `wind.profile`. Measured on this build
with the default M852 at azimuth 40°: 5 m/s from 3 o'clock to 300 m, then
10 m/s from 3 o'clock, records `wind_mps = 5` and a drift of −2.11 MIL at
600 m, against −1.57 for 5 m/s all the way; the same with the far wind from
9 o'clock gives +0.07. Join the JSON before reading any row from 1.13.0 on as
a constant wind.

**1.13.1 is not a boundary.** It makes the solver about twice as fast and
changes no result: every number it computes is bit-identical to 1.13.0's.
Rows from either side of it pool freely.

The build stamp is at the bottom of the page. Quote it when reporting anything,
otherwise there is no way to tell whether you were looking at a build that has
already been fixed.

The file must not be reformatted, minified, or passed through any optimiser. It
carries CRLF line endings and `.gitattributes` marks it `-text` so Git leaves
them alone; normalising them would not change how the solver runs, but it would
invalidate the hash above.

## Network access

The solver needs no connection. From 1.8.0 one button on the Environment screen
reaches the network, and only when pressed:

- **Get my position and weather** — the browser's geolocation API, then a
  request to `api.open-meteo.com` carrying the coordinates, for the air and the
  height of the ground there.

Until 1.8.0 those were two buttons, **Use my location** and **Fetch live
weather**. When the browser refuses a position the button says what the browser
can tell about why and fetches for coordinates already typed in. There is no
IP-address fallback, on purpose: it puts a phone in the wrong city often enough
to feed a wrong station pressure into the drop. The shot log and the shared
presets, described below, are the other two things that leave the device.

From 1.10.0 the azimuth can also be read from the phone's compass. That uses
the device's own sensors and NOAA's World Magnetic Model 2025, which is
compiled into the page; nothing is sent. The factory-ammunition list is part
of the page too, so choosing a load fetches nothing.

From 1.11.0 a rifle can carry a photo. It is stored only in the browser
(IndexedDB, redrawn at most 1000 px across, which also drops the location a
phone camera writes into the file), never uploaded, and leaves the device only
inside a rifle-library file the shooter exports themselves.

The scope catalogue added in 1.12.0 is part of the page as well. Each reticle
names the maker's document it was drawn from, but nothing is fetched from the
makers' sites: the page loads no image or file from anywhere.

Since 1.4.1 the wind toggle starts on **HOLD YOURSELF** rather than the mode
now labelled DRIFT INCLUDED — WIND DRIFT INCLUDED from 1.5.1 to 1.7.0 and WIND
IN DOPE before that. Only the label changed each time, and the `wind_in_dope`
column kept its name. Saved sessions
keep whatever the shooter chose; only a fresh visitor sees the new default. It
changes nothing in the solver and everything in the data:
`wind_in_dope = false` becomes the common case, and those records read as
enormous windage errors unless they are filtered, because the correction was
meant to be held on the reticle rather than dialled.

From 1.6.0 that toggle no longer changes a displayed figure. The column is
Drift in both modes and always carries the whole deflection, wind and spin
drift together; the switch — now in the top right of the reticle rather than
the readout — decides only where the correction goes, on the windage turret or
held in the reticle. It moves the aim dot, and it still sets `wind_in_dope`,
which keeps its name and its meaning for every record written before and
after.

The position needs the page served over **HTTPS**: every browser refuses
geolocation to an insecure origin, and on plain http the button can only fetch
the weather for typed coordinates. This deployment enforces HTTPS, so both
halves work. Everything else keeps working regardless.

From the 1.2.0 build the app **uploads each logged shot as it is saved**, to a
Supabase edge function. It is the second thing that leaves the device:

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

From 1.5.0 the app also publishes and reads **shared presets** through a second
edge function. There are three kinds: a `rifle` — which from 1.7.0 is the whole
rifle, its barrel and zero together with the bullet it shoots and the sight
bolted to it — and a `bullet` or a `sight` on its own, so one can be swapped
without the other. Presets always save to the shooter's own browser; the shared
list is what travels.

**Presets published before 1.7.0 keep working, and the endpoint needed no
migration.** Until then `rifle` meant the barrel and zero alone and the
three-in-one version was filed under `setup`. Both shapes are still read and
both appear under Rifle; only the wide one is written now. The client tells
them apart by the shape of the payload rather than by its kind or a version
number, so an old narrow record loads its barrel and zero and leaves the bullet
and sight untouched.

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

## Studio — withdrawn

An experimental second interface over the same engine was served at
`/studio/` from 2026-09-08 and withdrawn on 2026-10-17; that address now
answers with this site's 404 page. Its last build is recorded here so that a
report quoting it can still be matched:

```
studio/index.html   (withdrawn)
sha256  42dc17e00f1ad22c3101bc47bd6e72c47628d07d5f9605d2edb73c13c55ab5b5
bytes   1296157
stamp   1.7.0-studio.3 · 2026-09-09 04:31 UTC
```

It ran Field 1.7.0's engine and data, sent nothing anywhere — no shot, no
preset — and kept its browser storage under its own
`superior-ballistics-studio-*` keys, which Field never reads. Those keys may
linger in the browsers of people who tried it; they are harmless.

## Validation

The engine is a port of the Python solver it was derived from, and is checked
against it rather than trusted. The suites were re-run against the bundle
above, stamped with the build they were run against, before it was published:

- **22 of 22** cross-validation cases pass per cell across the whole range
  card. The canonical 7.62 mm M852 168 gr reference reads −1357.3097 cm of drop
  at 1000 m, a deviation of 1.72e-5 cm against a 0.02 cm tripwire.
- Numeric primitives — every drag table and spline — are **bit-identical to
  SciPy**, 0 ULP.
- From 1.10.0, the magnetic declination behind the compass agrees with all
  **12** of NOAA's published WMM2025 test values, the worst field component
  within 0.049 nT.

In 1.13.1 the engine was optimised for speed for the first time since the
port. No formula and no order of operations changed. The outputs were checked
bit for bit against the previous engine — 228 solves and a fuzz of the
numerical primitives, some 35 million numbers compared exactly — and the
suites above give the same figures as before. On publication, six whole range
cards from the published 1.13.0 and from 1.13.1 were compared again, 5,786
numbers each, with no difference.

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
redeploys within about a minute. If `src/engine/` was touched, re-run the test
pages first — they are the only thing separating a plausible number from a
correct one. If the push is refused as non-fast-forward, `git pull --rebase`:
never force it, because GitHub keeps its own commit of `CNAME` here and a
forced push would delete it and take the domain down without an error.

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
