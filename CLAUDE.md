# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-file browser tool (`index.html`) that lets authenticated Gmail users send bulk personalised emails using data from an Excel file. No build step, no server, no dependencies installed locally — everything loads from CDN at runtime.

## Deployment

The site is hosted on GitHub Pages at `https://nguyetquynh309.github.io/gmail-bulk-emailer/`. To deploy, commit and push to `master` — Pages rebuilds automatically.

```
git add index.html
git commit -m "..."
git push
```

## Architecture

Everything lives in `index.html` in three sections: `<style>`, HTML markup, and a `<script>` block.

**External dependencies (CDN)**
- [SheetJS](https://sheetjs.com/) (`xlsx.full.min.js`) — parses `.xls`/`.xlsx` in the browser
- [Google Identity Services](https://accounts.google.com/gsi/client) — OAuth2 token flow for Gmail API

**Key runtime constants**
- `GOOGLE_CLIENT_ID` — OAuth2 client ID (safe to expose; scoped to `gmail.send` only)
- `HEADER_ROW = 6` — the 1-based Excel row number that contains column headers
- `TARGET_SHEET = 'tổng hợp'` — the sheet name to read; falls back to the first sheet

**Data flow**
1. User uploads a file → SheetJS reads it → headers extracted from row 6 of sheet "tổng hợp" → data rows stored in `rows[]`
2. Column chips insert `{{ColumnName}}` tokens into the Email address / Subject / Body fields
3. On preview or send, `substitute(template, row)` replaces tokens with row values; `formatVal()` auto-formats numbers ≥ 1000 with `.` as the thousands separator (Vietnamese style) and rounds floating-point noise
4. Send loop calls the Gmail REST API (`/gmail/v1/users/me/messages/send`) with a base64url-encoded RFC 2822 message, throttled at ~400 ms per email

**Excel parsing note**
`sheet_to_json(ws, { header: 1 })` returns arrays starting from `sheetRange.s.r` (the first *used* row), not from row 1. The header array index is computed as `(HEADER_ROW - 1) - sheetRange.s.r` to stay correct regardless of how many blank rows precede the header in the file.

## Google Cloud setup (required once per deployment origin)

The OAuth client must list the Pages origin (`https://nguyetquynh309.github.io`) as an **Authorized JavaScript origin** in [Google Cloud Console → Credentials](https://console.cloud.google.com/apis/credentials). The Gmail API must also be enabled in the same project.
