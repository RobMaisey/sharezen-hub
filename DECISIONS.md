# DECISIONS — sharezen-hub

A running record of this site's migration and the decisions made along the way.
Written as we work (this first entry was captured right after completion, on 2026-06-01).
This doubles as the **template** for future migrations (next up: sharezenpro.com).

> Note: Claude never changes DNS. Rob makes all DNS changes at the registrar.
> The "CNAME" file in the repo is a GitHub Pages config file, NOT a DNS record — same
> name, different thing.

---

## Summary
- **Site:** ShareZen hub page (existing single-file page, reused unchanged — no redesign).
- **Domain:** sharezen.com (apex).
- **Repo:** RobMaisey/sharezen-hub (public, default branch `main`).
- **Host:** GitHub Pages (build from `main`, root folder).
- **Outcome:** Live at https://sharezen.com with a valid TLS certificate and HTTP→HTTPS enforced.
- **Date completed:** 2026-06-01.

## Source content
- `index.html` pulled UNCHANGED from:
  `https://raw.githubusercontent.com/RobMaisey/RobMaisey.github.io/main/sharezen/index.html`
  (7,577 bytes, title "ShareZen"). Self-contained single file; no other assets needed.

---

## DNS records

### Changed — apex `sharezen.com`
- **After:** four A records pointing at GitHub Pages:
  - `185.199.108.153`
  - `185.199.109.153`
  - `185.199.110.153`
  - `185.199.111.153`
- **Before:** NOT captured before the change.
  **LESSON:** for the next migration, record the prior apex records BEFORE editing so this
  section is a true before/after.
- (Optional, not confirmed added this time: AAAA records `2606:50c0:8000::153`,
  `...8001::153`, `...8002::153`, `...8003::153` for IPv6.)

### Changed — `www.sharezen.com`
- **After:** CNAME → `robmaisey.github.io`
- **Before:** not captured.

### Deliberately LEFT UNTOUCHED (and why)
- **`app.sharezen.com`** — a separate subdomain running Rob's live software platform.
  Must stay intact. It is its own DNS entry; changing the apex/www records does not affect
  it. Explicitly not modified.
- **Any MX / email / other existing records** — left alone. Only the apex + www were part
  of this migration.

---

## Order of operations (the repeatable pattern)
1. **Repo:** create it — public, with a README, default branch `main`. Verify.
2. **Content:** add to the repo root: `index.html` (unchanged) + a `CNAME` file containing
   the apex domain on one line (here: `sharezen.com`). Commit + push to `main`. Verify both
   files are present in the repo.
3. **Pages:** enable GitHub Pages, source = `main`, root folder. Confirm the custom domain
   registers (`pages.cname == sharezen.com`). **Do NOT enable Enforce HTTPS yet** — this is
   the DNS handoff point.
4. **DNS (Rob):** update records at the registrar — apex A records → the four GitHub IPs;
   `www` CNAME → `robmaisey.github.io`; leave `app` and email records untouched. Wait for
   propagation.
5. **HTTPS:** once DNS has propagated, GitHub requests the certificate. When
   `pages.https_certificate.state` is `approved`/`issued`, enable Enforce HTTPS. Verify
   `https://` loads with a valid cert and `http://` 301-redirects to `https://`.

---

## Decisions & notes
- **Reuse the existing hub page exactly** — no redesign.
- **CNAME file = the APEX domain** (`sharezen.com`), not `www`.
- **Repo created WITH a README**, so we cloned the repo into the local working folder and
  added files on top, rather than `git init` (avoids a divergence/merge tangle).
- **Enforce HTTPS stays OFF until the certificate is issued.**
- **Timing lesson (carried over from the earlier sharezen.ca / workfit-site migration):**
  if Pages/cert is requested BEFORE DNS is ready, the certificate request fails and must be
  re-triggered (remove + re-add the custom domain in Pages settings). For sharezen.com the
  DNS was ready first, so the certificate issued cleanly on the first attempt. **Prefer:
  have DNS ready before (or at the moment) Pages is enabled.**
- **Guardrails honored:** no DNS changed by Claude; `app.sharezen.com` untouched; other
  repos (`workfit-site`, `RobMaisey.github.io`) untouched; nothing deleted; no force-push;
  every command run from the repo's working folder.

## Final verification (2026-06-01)
- `pages.cname` = `sharezen.com`; `https_enforced` = `true`; `cert_state` = `approved`.
- `https://sharezen.com` → HTTP 200, valid certificate, serves the hub ("ShareZen").
- `http://sharezen.com` → 301 redirect to `https://sharezen.com`.
  ## 404 handling (2026-06-06)
- Added `404.html` to the repo root. GitHub Pages serves it automatically for any
  unmatched path. Styled to match `index.html` (same fonts, colour vars, mark, animations).
- Behaviour: friendly "this page has moved" message, 4-second redirect to
  https://sharezen.com/ via `<meta http-equiv="refresh">`, plus a manual button.
- `<meta name="robots" content="noindex">` so the page stays out of search results.
- Why: retired Weebly paths (e.g. /sharing-software.html) now 404 and bounce to the
  homepage instead of hitting the generic grey GitHub 404.
- Template note: replicate this 404.html for sharezenpro.com, swapping the redirect
  target to that apex.
