# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file, static HTML/CSS/JS vacation-request portal ("Leave Portal"). There is no build system, no
package manager, and no server-side code — `index.html` is the entire application. Open it directly in a
browser (or serve the folder with any static file server) to run it; there is nothing to compile.

This repo is the client-only prototype. A parallel repo, `leave-ledger`, contains the same UI wired up to a
real Supabase backend instead of an in-memory array — treat that as the "next stage" of this app if asked to
add persistence here.

## Development workflow

- Edit `index.html` directly; all markup, CSS (in a `<style>` block), and JS (in a `<script>` block) live in
  that one file.
- There are no tests, linters, or build/watch commands. Verify changes by opening `index.html` in a browser
  and exercising the UI manually (submit a request, switch role, toggle language, export to Excel).
- Since everything is inline, keep an eye on scope: CSS custom properties are defined once in `:root`
  (and overridden under `html[dir="rtl"]`), and JS is not modularized — new functions/state should follow the
  existing flat structure rather than introducing bundlers or module systems.

## Architecture

**State is in-memory only.** The `requests` array (seeded with two sample rows) lives in a page-level JS
variable and resets on every reload — there is no backend, database, or localStorage. The only ways to
persist data out of the page are:
- **Export to Excel** — uses the `xlsx` library (loaded from a CDN `<script>` tag) to download a `.xlsx` snapshot of `requests`.
- **Power Automate / OneDrive sync (optional, unconfigured by default)** — if a Power Automate HTTP-trigger URL is pasted into the `FLOW_URL` constant near the top of the `<script>` block, new requests and manual "Sync now" clicks POST JSON to that URL. With `FLOW_URL` empty (the default), sync is a no-op that shows a toast telling the user it isn't configured.

**Bilingual UI (en/ar).** All user-facing strings live in the `T` object (`T.en`, `T.ar`). `applyTranslations()`
walks a DOM-id → string map to update text content, flips `document.documentElement.dir` to `rtl` for Arabic
(switching the display font to Cairo via the `html[dir="rtl"]` CSS block), and re-renders the dynamic
sections. When adding a new user-facing string, add it to **both** language objects and wire it through
`applyTranslations()` (or the relevant `renderX()` function) rather than hardcoding text in the DOM.

**Role-based view (employee/manager).** A `role` variable (`employee` | `manager`) toggled via the segmented
control in the header controls whether the "New request" form is shown and whether Approve/Reject buttons
render per row in `renderTable()`. There is no auth — this is a pure UI toggle, not a security boundary.

**Render functions are manually invoked, not reactive.** There's no framework — `renderFilters()`,
`renderGauges()`, `renderTable()`, and `updateDaysHint()` are called explicitly after any state mutation
(submitting a request, approving/rejecting, changing filter/language/role, editing balance inputs). When
adding a new state mutation, remember to call the relevant render function(s) afterward, the same way the
existing event handlers do.

**Working-day calculation.** `workingDays(startStr, endStr)` excludes Friday/Saturday as the weekend (not
Saturday/Sunday) — this matches the target locale (Arabic/Middle East region). Keep this convention if
touching date logic.

**XSS-safety.** Table cells built from user input (`name`, `dept`, `reason`) are passed through `escapeHtml()`
before being interpolated into `innerHTML`. Preserve this when adding new fields that render user-supplied text.
