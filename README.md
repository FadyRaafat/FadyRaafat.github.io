# PillPal developer site (`fadyraafat.github.io`)

This directory mirrors the GitHub Pages repository that serves:

| File | Served at | Purpose |
|---|---|---|
| `app-ads.txt` | `https://fadyraafat.github.io/app-ads.txt` | AdMob authorized-seller verification (must sit at the domain root) |
| `config.json` | `https://fadyraafat.github.io/config.json` | Remote app config (billing kill-switch, minimum supported version, notice) — the URL baked into `core:data` BuildConfig |
| `config-drill.json` | `https://fadyraafat.github.io/config-drill.json` | **Drill document** (spec 009): same shape, read only by a drill build with `-PpillpalConfigUrl=…`. The `kill-switch-drill` job flips it every release candidate to measure real propagation. Never read by shipped apps; automation never writes `config.json` |
| `privacy.html` | `https://fadyraafat.github.io/privacy.html` | Privacy policy URL for the Play listing |

The contact address in `privacy.html` is the real support email
(`fedo.raafat@gmail.com` — replaced during spec 008; the collateral
placeholder gate keeps any placeholder from coming back).

These URLs are guarded by `scripts/check_site_health.sh` (config URL taken
from `core/data/build.gradle.kts`, never hardcoded): it runs nightly and on
URL/site-touching PRs via `.github/workflows/site-health.yml`, and as a hard
gate in `release-readiness.yml`. `AppConfigUrlConsistencyTest` (`core:data`
unit tests) additionally pins the baked-in URL to this mirror's layout on
every CI run.

## Publish (3 commands)

```bash
# 1. Create the user-pages repo (once): https://github.com/new → name it exactly fadyraafat.github.io, then:
git clone https://github.com/FadyRaafat/fadyraafat.github.io && cd fadyraafat.github.io

# 2. Copy this directory's contents into it (from the PillPal repo root):
cp -R /path/to/PillPal/site/. .

# 3. Push — GitHub Pages serves user-pages repos from main automatically:
git add -A && git commit -m "Publish PillPal site (app-ads.txt, config, privacy)" && git push origin main
```

## After publishing

- Verify `https://fadyraafat.github.io/app-ads.txt`, `.../config.json`,
  and `.../privacy.html` all load (Pages can take a few minutes on
  first publish).
- In the **Play Console listing**, set `https://fadyraafat.github.io` as the
  developer website and `https://fadyraafat.github.io/privacy.html`
  as the privacy policy URL.
- In **AdMob → app settings → app-ads.txt**, re-check the verification tab
  ~24 h after publishing (crawls are not instant).

## Changing the remote config

Edit `config.json` in the Pages repo and push. Two windows apply — origin propagation
(minutes; measured every release by the kill-switch drill and budgeted in
`config/rollout-thresholds.json`) and client pickup, which happens on **every app open**
(`AppConfigRepositoryImpl.refresh` fetches unconditionally, so there is no 24-hour device
timer). Full procedure: `docs/release/rollback-runbook.md`.

- `billing_enabled` — purchase-path kill-switch (false = Premium screen shows
  "coming soon", no purchase or product query runs).
- `min_supported_version` — devices with a lower `versionCode` show a
  full-screen "Update required" blocker linking to the Play listing. Raise
  with extreme care; never above the versionCode currently live on Play.
- `notice` — reserved; keep `""`.
