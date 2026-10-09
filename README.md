# Food Safety Timer — v2: historical calendar + two-way sync

Mobile-first GitHub Pages frontend + Google Sheets / Apps Script backend.

## Upgrade from v1.1 (important)

1. Keep your existing Google Sheet. **Do not create another spreadsheet.**
2. Open **Extensions → Apps Script** in that Sheet.
3. Replace the old `Code.gs` with the `Code.gs` included in this ZIP and save.
4. Choose `setupFoodTimer` and click **Run**. Authorise when prompted. Existing `Batches` and `Events` rows are preserved.
5. Select **Deploy → Manage deployments → Edit (pencil) → Version: New version → Deploy**. This is essential: simply saving the code does not update the published web app.
6. Copy the Web app `/exec` URL (it may be unchanged).
7. Replace the GitHub Pages files (`index.html`, `app.js`, `style.css`) with the new versions. Keep the files at the repository root or update GitHub Pages source settings.
8. Open the website → Settings → paste the `/exec` URL → Save connection → Refresh & sync.
9. Test on one device, then on a second device. Confirm records appear on both and in Sheets.

## What's new

- Read the **entire history** of batches and event corrections from Google Sheets on first load, calendar navigation, manual refresh and periodic refresh (approximately every 60 seconds).
- Merge records by immutable batch ID; preserve unsynced local records rather than silently overwriting them.
- Optimistic concurrency: every saved batch has a unique version in `Last Updated`. If a different device saved a newer version, an old client write is refused rather than overwriting it. Export local JSON backup and reconcile the conflict manually.
- Existing batch and event sheet column layouts are preserved; no migration is required.
- Timestamped event history remains in the `Events` tab. CSV export and print operate on the selected date.
- Offline/browser-only mode is available if no backend is configured, but it is not a shared compliance register.

## Important limitations before production use

**Security:** GitHub Pages is public and an Apps Script web app deployed as **Anyone** exposes the read and write endpoint to anybody with the URL. JSONP history reads are inherently publicly accessible. This version is a working prototype, **not** a secure production system for regulated records. Restrict access with an authenticated backend before live use. Never embed an API secret in public GitHub code.

**Connectivity:** The frontend saves edits to localStorage immediately and attempts to POST to Apps Script. If a request fails, the UI marks it as not synced. Browser localStorage is not a durable offline queue; clear browser data only after verifying the spreadsheet. Apps Script cross-origin POST behaviour depends on deployment and browser: verify POST and GET in your actual GitHub Pages origin before relying on it.

**Conflicts:** Conflicts are intentionally not auto-merged because a blind merge could corrupt food safety evidence. Use Export full JSON backup and inspect the conflicting record in Google Sheets. Changes in Sheets made manually will not produce automatic `Events` audit entries.

**Food safety:** The elapsed-time warning is a prompt, not a legal compliance determination. Cumulative time out of temperature control, applicable NSW 2-hour/4-hour guidance, cold-chain checks, and discards require a reviewed operating procedure. The current four timestamp model cannot fully reconstruct repeated display/refrigeration cycles. Sold quantities are captured as batch totals, not individual item preparation events.

**Time zone:** Uses Australia/Sydney for display. Check timestamps around daylight-saving transitions, particularly manual corrections.

**Data volume:** The full history is read on refresh. For a large dataset, implement paginated/date-filtered reads before operational scale.

## Quick verification

1. Create a test batch on device A and tap Finished. Confirm the row in `Batches` and an entry in `Events`.
2. On device B, connect to the same `/exec` endpoint and click Refresh & sync. Open the same date. Confirm that batch appears.
3. Choose an older calendar day on B; confirm its batches and timestamps load.
4. Correct a time with a reason on A. Refresh B; check the new time and event log.
5. Temporarily disconnect the internet, change a test quantity, and confirm `Not synced`. Reconnect, refresh/sync and verify Sheets.
6. Make conflicting edits on two devices before syncing. Verify the older edit is rejected rather than silently overwriting the newer one.

