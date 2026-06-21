# Screenlapse — measurement (privacy-clean, zero in-app telemetry)

The app never phones home: no account, no usage tracking. Its only network call is Sparkle's
appcast fetch (for updates). **Every metric below lives at the distribution layer** — the
website, the release host, and the payments dashboard. Nothing is ever added to the app.

## The funnel

| Stage | Source | Status |
|------|--------|--------|
| Visits to screenlapse.net | Cloudflare Web Analytics (cookieless) | ⏳ paste token (see below) |
| Downloads (new installs) | GitHub Releases per-asset `download_count` | ✅ live |
| Active installs (proxy) | Sparkle appcast fetch count (Cloudflare in front of appcast.xml) | ⏳ optional, later |
| Sales / conversion | Merchant-of-Record dashboard (Lemon Squeezy / Paddle) | ⏳ after MoR is chosen |

Approx **trial → paid conversion** = MoR purchases ÷ GitHub downloads (computed entirely off-device).

## Downloads — live

The website "Start free trial" buttons point at a **stable** GitHub Releases URL:

    https://github.com/denixoid/screenlapse-releases/releases/latest/download/Screenlapse.pkg

`latest/download/Screenlapse.pkg` always serves the newest release's installer — each release
uploads its installer under the stable name `Screenlapse.pkg`, so the button never needs editing
and GitHub counts every download.

### See the numbers anytime

    # Total downloads across all releases:
    gh api repos/denixoid/screenlapse-releases/releases --jq '[.[].assets[]?.download_count] | add // 0'

    # Per release:
    gh api repos/denixoid/screenlapse-releases/releases \
      --jq '.[] | "\(.tag_name): \([.assets[]?.download_count] | add // 0)"'

Sparkle in-app **updates** still pull the `.zip` from screenlapse.net (Pages) — deliberately
left untouched so the updater keeps working. Updates are an existing-user metric, distinct from
new installs.

## Web analytics — activate when you're back

A cookieless beacon placeholder already sits in `index.html` (commented, just before `</body>`).
To turn it on:

1. Create a free Cloudflare account → **Web Analytics** → add site `screenlapse.net`.
2. Copy the beacon token, paste it into the `data-cf-beacon` `TOKEN` in `index.html`,
   uncomment the `<script>`, then commit + push.

Cloudflare Web Analytics is cookieless and stores no PII (GDPR-clean). Equally privacy-respecting
alternatives: Plausible / Fathom (paid, EU-hosted), GoatCounter (free).

## Active installs (optional, later)

To approximate active installs with no app change: put Cloudflare in front of
`screenlapse.net/appcast.xml` (proxied DNS) and read the request count — every running copy polls
the appcast daily. Needs your Cloudflare account + DNS, so it's deferred.

## How releases publish (automated)

`scripts/release.sh` (in the app repo) uploads each new installer to GitHub Releases under the
stable `Screenlapse.pkg` name and marks it `--latest`, so the download buttons and the count keep
working release-to-release with no manual edits. The `.zip` + `appcast.xml` continue to publish to
screenlapse.net for Sparkle.
