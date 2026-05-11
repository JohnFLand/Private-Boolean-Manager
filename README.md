# Private Boolean Manager

A [Hubitat Elevation](https://hubitat.com/) app that scans all Rule Machine (RM) and Button Controller (BC) rules, displays each rule's **Private Boolean state** and **Last Run time** in a sortable table, and provides tools for setting Private Boolean values individually or in bulk — manually or on a daily schedule.

---

## Screenshot

Private-Boolean-Manager screenshot](Screenshot_2026-05-10_164422.png)

---

## Installation

1. In the Hubitat web UI, go to **Apps Code → + New App** and paste in the app's Groovy source.
2. Save and then go to **Apps → + Add User App** and select **Private Boolean Manager**.
3. The app will attempt to create an OAuth token automatically on first open. If it does not, enable OAuth in Apps Code for this app and re-open it.
4. Click **Scan All Rules** to perform the initial scan.

---

## Usage

### Scanning

Click **Scan All Rules** to start a scan. The app queries Hubitat's internal endpoints asynchronously, rule by rule. When the scan finishes, the table renders automatically and a timestamp with scan duration appears below the button.

Re-opening the app after a scan re-renders the table instantly from cached data — no rescan is needed for display setting changes such as hiding columns or adjusting the filter.

---

### Rule State Table

The table lists every discovered RM and BC rule with the following columns:

| Column | Description |
|--------|-------------|
| **Set TRUE** | Checkbox — mark this rule to have its PB set TRUE on the next Apply |
| **Set FALSE** | Checkbox — mark this rule to have its PB set FALSE on the next Apply |
| **Rule ID** | The Hubitat internal app ID for the rule |
| **Rule** | Rule name, linked directly to its configuration page |
| **App Type** | `RM` (Rule Machine) or `BC` (Button Controller) |
| **Current PB State** | The rule's Private Boolean value — **TRUE** in bold blue, FALSE in grey, or — if unreadable |
| **Last Run** | Date and time of the most recent trigger event (`yyyy-MM-dd HH:mm`) |

**Sorting** — click any column header to sort by that column; click again to reverse.

**Filter** — type in the Filter box to show only rules whose names contain the entered text (case-insensitive substring match). Use `*` and `?` as wildcards for pattern matching (e.g., `Lights*` shows all rules starting with "Lights"). The filter is applied immediately as you type.

**Hide columns** — the **Rule ID**, **App Type**, **Current PB State**, and **Last Run** column buttons toggle those columns on and off. Visibility preferences are saved immediately without pressing Done.

**Paused rules** — rules that are currently paused are shown with *(Paused)* appended to their name in orange.

---

### Setting Private Booleans

#### Checkbox columns — bulk apply

Each table row has a **Set TRUE** and a **Set FALSE** checkbox:

- Checking one clears the other for that row automatically (mutual exclusion).
- Leaving both unchecked means "no change" for that rule.

Three bulk-action buttons operate on **visible rows only** (filtered-out rows are skipped):

| Button | Action |
|--------|--------|
| **All TRUE** | Checks Set TRUE and clears Set FALSE for every visible row |
| **All FALSE** | Checks Set FALSE and clears Set TRUE for every visible row |
| **All Clear** | Clears all Set TRUE and Set FALSE checkboxes for all rows |

Click **Apply selected PB changes** to send all pending changes to Hubitat in a single request. The table updates immediately to reflect the new values without requiring a rescan.

Checkbox selections persist across page refreshes and Done/reopen cycles — they are saved to the hub each time a checkbox is changed.

> **Note:** Changes are issued via `RMUtils.sendAction()` and immediately reflected in the table, but are **not verified** by re-reading the affected rules. If a rule's PB value appears unchanged after a rescan, Rule Machine may have silently rejected the action (e.g. wrong RM version, rule deleted since last scan).

#### Current PB State column — single-rule toggle

Click any **Current PB State** cell to toggle that rule's Private Boolean immediately:

- The cell updates in-place without a rescan.
- Cells showing **—** could not read the rule's PB state and are not clickable.
- The same fire-and-forget caveat applies — the change is not verified by a read-back.

---

### Scheduled PB Apply

In the **Controls** section, set a time under **Apply PB changes daily at:** and press Done. Each day at that time the app applies whichever Set TRUE / Set FALSE checkboxes are currently saved — exactly as if you had clicked **Apply selected PB changes** manually.

- The last run timestamp and rule counts (e.g. *TRUE: 3, FALSE: 2*) appear below the time field after the schedule has fired at least once.
- To remove the schedule, clear the time field and press Done.
- The same fire-and-forget caveat applies to scheduled runs.

---

### Reports

In the **Controls** section (after a scan):

- **Open Printable Report** — opens a formatted HTML page suitable for printing or saving, listing all rules with their current PB state and last-run time.
- **Download CSV** — downloads a comma-separated file of the same data for use in a spreadsheet.

Both reports reflect the data from the most recent scan.

---

## Controls Section

| Control | Description |
|---------|-------------|
| **App instance name** | Rename this app instance |
| **Open Printable Report** | HTML report of all scanned rules (requires a completed scan) |
| **Download CSV** | CSV export of all scanned rules |
| **Apply PB changes daily at:** | Optional daily schedule for automatic bulk PB apply |
| **Enable debug logging** | Turns on verbose debug output to the Hubitat log; auto-disables after 30 minutes |

---

## Technical Notes

The app uses the following Hubitat local/internal endpoints:

| Endpoint | Purpose |
|----------|---------|
| `/hub2/appsList` | Discover all RM and BC rules |
| `/installedapp/statusJson/{appId}` | Read per-rule Private Boolean state and last-run time |
| `/apps/api/{appId}/setPB` | In-table single-rule PB toggle (OAuth) |
| `/apps/api/{appId}/bulkPB` | Bulk PB apply from checkboxes (OAuth) |
| `/apps/api/{appId}/setpref` | Persist column-hide and checkbox preferences (OAuth) |
| `/apps/api/{appId}/report` | Printable HTML report (OAuth) |
| `/apps/api/{appId}/RM-BC_Rules.csv` | CSV export (OAuth) |

> **Warning:** The `/hub2/appsList` and `/installedapp/statusJson/` endpoints are internal Hubitat APIs with no formal public specification. They could change in a future Hubitat platform release.

Private Boolean changes use `RMUtils.sendAction()` targeting **Rule Machine version 5.0**. Rules created under earlier RM versions will display their PB state correctly but the toggle and bulk-apply actions may not work.

---

## Limitations

- **No write verification.** PB changes are fire-and-forget. The table is updated optimistically based on the requested value, not the confirmed new state. Run **Scan All Rules** after applying changes to confirm the actual state from the hub.
- **RM 5.0 only.** Rules from earlier Rule Machine versions are displayed but PB changes may be silently ignored by Rule Machine.
- **Basic Button Controller excluded.** BBC rules use a fundamentally different internal structure and are not included in the scan.
- **Rule Machine internal API.** If Hubitat changes the internal JSON endpoint formats, the scan or PB detection may break until the app is updated.

---

## Credits

Designed initially by John Land. Built by Claude AI, with an assist by ChatGPT. The in-table clickable-cell PB toggle technique was adapted from the work of **hubitrep**.
