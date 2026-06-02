# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file static web app (`index.html`) that displays a public "open games" board for the Tennessee Soccer League (TSL) Middle Tennessee region. Referees browse uncovered game slots, select one or more, and send an assignment request to the assignor via a **pre-filled email (`mailto:`) or text (`sms:`)** opened in their own device's app. No backend receives the request and no referee data is ever persisted server-side — the user's name/email only exist inside the message they send.

## Architecture

Everything lives in `index.html` — HTML, CSS (in `<style>`), and JS (in `<script>`). There is no build, no dependencies, no package manager, and no tests. Edit the file directly and open it in a browser (or push to deploy) to see changes.

Data flow:
1. On load, `loadGames()` fetches JSON from a Google Apps Script Web App endpoint (`CONFIG.apiUrl`). That script reads the assignor's Google Sheet of open slots and returns an array of row objects.
2. Keys are lowercased/normalized; each row gets a `_id` for selection tracking.
3. `renderBoard()` groups games by venue and renders cards. Selection state lives in the `selectedIds` Set.
4. `buildEmailBody()` / `buildTextBody()` compose the request message; `sendClaim()` / `sendText()` hand off to `mailto:` / `sms:`.

### Key conventions

- **`CONFIG` object (top of `<script>`)** is the only thing meant to be edited per-deployment: `apiUrl` (Apps Script endpoint), `assignorEmail`, `assignorPhone`, `boardTitle`.
- **`gf(game, ...keys)`** is the resilient field accessor — it matches sheet column names ignoring case/spaces/`_`/`-`. The Google Sheet's column headers can change wording; add new alias keys to `gf()` calls rather than assuming an exact header. Existing aliases handle things like `start date/time` vs `startdatetime`, `age group` vs `agegroup`, `open roles` vs `roles`.
- **`formatDayTime(dt)`** parses several date formats (US `MM/DD/YYYY`, ISO, time-only, and full JS `Date.toString()` output) and deliberately builds dates with the local-time constructor / string extraction to avoid UTC timezone shifting. Preserve that behavior when touching it — times must display in Central time as entered.
- **Role display**: a sheet value of `CR` renders as "Referee"; anything else (e.g. `AR`) renders as "AR".
- All sheet-derived strings are passed through `esc()` before insertion into HTML.

## Deployment

Hosted from the `615jess/TSLOpenGames` GitHub repo (GitHub Pages). Deploy by committing to `main` and pushing. The Google Apps Script backend is separate and not in this repo; changing what data appears on the board means editing that script / the underlying Sheet, not this file.
