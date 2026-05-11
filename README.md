  Overview

                This app has two functions: it scans Rule Machine (RM) and Button Controller
                (BC) rules and displays their Private Boolean state and Last Run time in a
                table; and it lets you bulk-set Private Boolean values across multiple rules at once.
                

                Scanning

                Click Scan All Rules to start a scan. The table updates automatically when
                the scan finishes. Clicking Done and reopening the app re-renders the table
                instantly from cached data — no rescan needed for display setting changes.
                

                Table

                Shows Rule ID, Rule name (linked to its config page), App Type (RM or BC),
                Private Bool, and Last Run. Column headers are clickable to sort. The
                Hide columns buttons and wildcard name filter above the table persist
                without pressing Done.
                

                Setting Private Booleans (checkbox columns)

                Each table row has a Set TRUE and Set FALSE checkbox.
                Checking one automatically clears the other for that row (mutual exclusion).
                Leaving both unchecked means no change for that rule.
                Use All TRUE, All FALSE, or All Clear to bulk-select visible rows
                (filtered rows are skipped). 
                
                Click Apply selected PB changes to send all
                pending changes to the hub in a single request via the /bulkPB OAuth
                endpoint. Changes are issued via RMUtils.sendAction() and immediately
                reflected in the table, but are not verified by re-reading the affected rules.
                If a rule's PB value appears unchanged after a rescan, Rule Machine may have
                silently rejected the action (e.g. wrong RM version, rule deleted since last scan).
                

                Current PB State column (in-table toggle)

                Click any Current PB State cell to toggle a rule's value in-place via OAuth.
                TRUE is shown in bold blue; FALSE in grey. Cells showing "—" mean the
                state could not be read and are not clickable.

                As with the bulk apply, the toggle uses RMUtils.sendAction() (targeting
                RM version 5.0), updates the table immediately, and does not re-read the
                rule to confirm the change took effect.
                

                Last Run column

                The date and time of the most recent trigger event for each rule (yyyy-MM-dd HH:mm).
                A blank cell means the rule has never been triggered since last installed.
                

                Controls section

                App instance rename, printable HTML report, CSV export, debug logging toggle,
                and scheduled daily PB apply.
                

                Scheduled PB Apply

                Set a time in the Controls section and press Done. Each day at that time the
                app applies whichever Set TRUE / Set FALSE checkboxes are currently saved
                (the same state used by Apply selected PB changes). The last run
                timestamp and TRUE/FALSE counts appear below the time field after the first
                run. Clear the time field and press Done to remove the schedule. Changes are
                fire-and-forget via RMUtils.sendAction(), with the same caveats
                as the manual apply (not verified by re-reading the rule state).
                

                WARNING

                This app uses Hubitat local/internal JSON endpoints that are not a formal public
                API and could change in a future platform update.
