# Food Safety Timer — v2.5 Design, Sync Repair

This build is based on v2.5's design and overlapping circular timers, **not v3**.

## Install

1. Export a full JSON backup from your existing site before replacing files.
2. Replace `index.html`, `app.js`, `style.css` on GitHub Pages. Hard-refresh the site.
3. Keep the existing `Code.gs` and Google Sheets deployment. Your /exec URL is preconfigured.
4. Select a date (today, past, or future): the five food types appear, even when there are no batches. Press Add batch under the food type.
5. Record one timestamp and use **Refresh & sync** in Settings. Inspect the corresponding row in the `Batches` sheet and `Events` sheet.

## Important sync details

GitHub Pages and Google Apps Script run on different origins. Browser JavaScript cannot reliably read a normal POST response from Apps Script. This build sends a text/plain POST with `mode:'no-cors'`, then independently requests Google Sheets history via JSONP and checks the exact batch fields. An opaque POST response is never treated as a successful save. Unverified records remain locally marked pending. It does not delete existing Google Sheets records.

A `Google Sheets read failed` message means the /exec URL is inaccessible, the Apps Script deployment is not available to the visitor, or Apps Script threw an error. Verify by opening `/exec?action=read` in a browser. It must return JSON with `ok:true`, `batches` and `events`. The provided `Code.gs` uses the spreadsheet to which the script is bound. The deployed web app must be redeployed after changes to Code.gs; the site update alone does not redeploy Apps Script.

### Sheets schema

Sheet named `Batches` with 12 columns `ID, Date, Food ID, Quantity, Sold Quantity, Finished ISO, Display ISO, Fridge ISO, Sold ISO, Discarded, Notes, Last Updated`.
Sheet named `Events` with 7 columns `Event ID, Batch ID, Action, Old, New, Reason, Recorded At`.

Using Google Sheets' visual **table formatting** generally does not interfere, as long as the exact sheet names and columns remain unchanged and the table is not filtered/protected in a way that prevents edits. Test with one record. Avoid changing the column order or putting unrelated data below the tables.

## Validation

JavaScript syntax, ZIP content, date/food rendering branches, and read-back comparison logic were checked locally. **Live deployment and actual Google Sheets writes were not verified from this environment.** Anyone-access Apps Script web apps are not authenticated; this is not an appropriate production security model for confidential records.
