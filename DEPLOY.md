# Control Deck — Deploy notes (India Moi two-sided ledger)

## What changed
India Moi Details page only:
- New received columns: Recv: Nandhini Marriage, Recv: Raja Marriage, Recv: Varnavi Kathu Kuthu / Saree
- New gave slots: Gave F1, Gave F2, Gave F3 (function, date, amount each; max 3 per family)
- Calculated columns (not stored): Recv Total, Gave Total, Balance
- New Gold Details column
- Totals cards: per-function received, Received total, Gave total, Net, Families to return (follows the search box)
- Add/Edit form: sections for Received / Gave / Gold; Gave F2 and F3 appear via "+ Add Gave F…"
- Excel export includes the new raw fields, so export → edit → import round-trips cleanly

## Existing data safety
- No migration and no data rewrite. All original columns (Function Name, Date, Amount, Gold,
  Received, Gave, New Moi or Return, custom columns like Village / Tamil name, Notes) are unchanged.
- Saves use Firestore merge writes; fields not on the form are never removed.
- Calculated columns are never written to Firestore or imported from Excel.

## Deploy
Code-only change. Firestore and Storage rules are unchanged (included for completeness).

    firebase deploy --only hosting

Recommended first: Home → full backup to Excel (or Export to Excel on the Moi page).

## After deploy
New columns appear at the end of the Moi table. Use "Table columns" to reorder or hide them.
To add a 4th function of ours later: add one entry to MOI_OUR_FUNCTIONS in index.html.

---

# Update 2 — Gift Details (US) two-sided family ledger

## What changed
Gift Details page only (the India Moi changes above are included too):
- One row per family: new Family / Person column
- Gave G1–G5 and Received R1–R5 slots, each with Occasion, For (VSR/SR/Family suggestions), Date, Gift, Value ($)
- Calculated columns (not stored): Gave Total, Recv Total, Balance ("to return" / "settled" / "gave extra")
- Totals cards: Gave total, Received total, Net, this year's spend, Families to return (follow the search box)
- Edit form: slot 2 and 3 appear via "+ Add Gave G2 / Received R2"

## Existing data safety
- No data is rewritten. Event, Gift, Received, Gave, Amount, Date, Who gave?, To whom and Notes stay as-is.
- Old entries are displayed in slot 1 (Gave or Received from the tick) on screen only, marked "from original entry".
  Values are copied into the new fields only when you Edit and Save that entry.
- Event is no longer a required field, so family-only rows can be saved.

## Deploy
Code-only: firebase deploy --only hosting  (rules unchanged)

---

# Update 3 — Add column → Delete now really deletes

## What changed
- **Delete** on an original column is no longer the same as Hide. It:
  1. erases that field from every record on the page (read straight from Firestore, batched writes),
  2. then records the column in a new `deleted` list (settings/customFields),
  3. and removes it from the Original columns list, table, Add/Edit form, Table columns and exports — no Show button.
- If any record can't be updated, the column is NOT marked deleted, so pressing Delete again retries.
- Calculated columns (Recv Total, Gave F1–F3, Gave Total, Balance) store no data; Delete just removes them.
- Bug fix: adding or removing a custom column no longer wipes hidden columns, renames,
  table-column order and row-button settings (the save now merges instead of overwriting).

## Existing data safety
- Columns currently tagged "hidden" (e.g. Received, Gave) are still just hidden. Their data is untouched
  until you press Delete on them after this deploy.
- Export to Excel before deleting anything you might need.

## Deploy
Code-only: firebase deploy --only hosting  (rules unchanged)

---

# Update 4 — Family and Friends page upgrade + import from "Names & Places.xlsx"

## What changed (Family and Friends page only)
- First column is now "Family name (primary person)".
- New fields: Family members, Relationship, Side (Raja's / Pradeepa's), Country (India / US / Other),
  Circle / Group, Type (Person / Family, Place, School / Class, Note), Phone 1, Phone 2, Email,
  Address / Place 1–3, Native place, Work / Company, Client / Project, Kids' school / class, Years / When, Source.
- Form is split into sections: Family, Contact, Addresses, Work & school, Notes.
- Quick filters above the table: Relationship, Side, Country, Type chips + a Circle dropdown. They combine.
- Search on this page looks inside every field (notes, addresses, family members…), not just table columns.

## Existing data safety
- Original fields (Name, Meet on, Last called, Next call date, Notes, Notes 1–3) keep the same internal names,
  so the existing entry keeps all its data.

## Deploy
Code-only: firebase deploy --only hosting  (rules unchanged)

## Then import
Family and Friends → Import from Excel → "Family and Friends - import.xlsx" (711 entries; Moi note names are excluded — they stay on India Moi Details).
Import once only — importing the same file twice adds the rows twice.

---

# Update 5 — Fixed table header + always-visible horizontal scrollbar (all pages)
- Each table now scrolls inside its own box sized to the screen (below the pinned page title bar).
- The column header row stays visible while you scroll rows.
- The horizontal scrollbar sits at the bottom of that box, so it's on screen without scrolling to the end.
- Print and Excel export are unchanged.

Code-only: firebase deploy --only hosting  (rules unchanged)

---

# Update 6 — Arizona Events: track up to 5 visits per event
- New fields: Category, Status (Want to go / Planned / Attended / Skip), Happens (Every year / Seasonal / One-time),
  Usual month, City, Website / tickets, Cost.
- Attended 1–5: each a date + "Who went / notes". Form shows Attended 1, then "+ Add Attended 2…".
- Table shows calculated columns: Attended years (hover a year for the full date + note), Times attended, Last attended.
- Filters: Status, Category, Happens, City chips + Usual month dropdown. Search looks in every field.
- "Date" is relabelled "Next / planned date" (same field, so the existing entry and the Calendar are unaffected).
- Excel export includes all 10 visit fields, so export → edit → import round-trips.

Code-only: firebase deploy --only hosting  (rules unchanged)

---

# Update 7 — Readable dropdowns (all pages)
- Dropdown lists now open dark with light text (color-scheme: dark + explicit option colours); the selected
  option is highlighted amber. Previously the list was white with light text, so only the highlighted value showed.
- Applies to form dropdowns, inline table dropdowns, filter dropdowns and access settings. Print view unchanged.

Code-only: firebase deploy --only hosting  (rules unchanged)

---

# Update 8 — Rows per page options + Arizona Events repeat rules

## Rows per page (all pages)
- Options are now 5, 10, 20, 50, 100, 200 and All (was 50, 100, 200, 500, All). Default stays 50.
- Your choice is still remembered per page on each device. A previously saved 500 falls back to 50.
- "All" shows every row on one page and hides Prev/Next.

## Arizona Events — repeating events
- "Happens" now also offers Every week and Every month (existing Every year / Seasonal / One-time values unchanged).
- New fields under a "🔁 Repeats" form section: Week of month (1st–5th, Last) and Day (Sunday–Saturday).
  Every year uses Week + Day + Usual month (e.g. 3rd Saturday of October).
- New calculated columns (not stored): Repeats (e.g. "Every month · 1st Saturday") and Next occurrence
  (date + weekday + "in N days"). Sorting by Next occurrence works across years.
- New filters: Week of month and Day, alongside Happens. Example: Happens = Every month + Day = Saturday.
- On save (form or inline dropdown), if Next / planned date is empty or already past, it is filled with the
  next occurrence so the Calendar and Group by month show it. A future date you typed is never overwritten.

## Existing data safety
- No migration. Existing entries are untouched until you edit them; the new fields start blank.
- If you saved a custom Table columns order, Repeats and Next occurrence appear at the end — reorder via Table columns.

## Deploy
Code-only: firebase deploy --only hosting  (Firestore and Storage rules unchanged)

---

# Update 9 — New page: USA Trips

## What's on it
- Menu: USA Trips (🗺️), under Personal. Move it with "Move to group" if you prefer another group.
- Fields: Destination / place, State (all 50 + D.C. suggested), Status (Want to go / Planned / Booked / Went / Skip),
  Travel by (Car / Flight / Flight + rental car / RV / Camper / Train / Cruise), Best month, Trip month, Trip year,
  Start date, End date, Days, Estimated cost ($), Actual spend ($), Who went / going, Stayed at, Website / itinerary link,
  Notes + Notes 1–3.
- Calculated column (not stored): Over / under = Actual − Estimate (amber "over", green "under", "on budget").
- Summary cards (follow the filters and search): Trips taken, Want to go / planned, States visited, Days travelled,
  Actual spend (went), this year's spend, Estimate for upcoming.
- Filters: Status, Travel by, Best month, Trip month, Trip year, State.
- On save, blanks are filled from the dates: Days (start to end, inclusive), Trip month and Trip year from Start date.
  Anything typed by hand is kept.
- Trips with a Start date show on the Calendar and in Group by month.
- Money fields accept $1,200 / 1200 / 1.2k.

## Access
- You (owner) see it immediately. Family members with no per-page restrictions see it too.
- A family member who HAS per-page restrictions won't see USA Trips until you grant it in Who has access
  (new pages default to no access for restricted members — by design in firestore.rules).

## Deploy
Code-only: firebase deploy --only hosting  (Firestore and Storage rules unchanged)

---

# Update 10 — Fixed row height with full text on hover / click (all pages)
- Every table cell now shows at most 3 lines, so long notes and addresses no longer stretch rows.
  Cut-off cells fade at the bottom and show a zoom cursor.
- Hover a cut-off cell (about a third of a second) to see its full content in a popup. You can move into the
  popup to scroll or copy text; links in it work.
- Click a cut-off cell to open the whole row (works on phones too). Click again to edit as before.
- Click the row's # number to open or close the row. Open rows stay open after an edit refreshes the table.
- Inline editing of long notes opens a taller box (up to 320px).
- Dropdown cells, checkboxes and buttons are not clamped. Print and Excel export are unchanged.

Code-only: firebase deploy --only hosting  (rules unchanged)

---

# Update 11 — Calendar + Today on every date field (all pages)
- Every date field in Add / Edit forms has 📅 (calendar) and Today buttons; you can still type a date.
- Clicking a date in a table opens a small editor with the same 📅 and Today buttons.
- The calendar is drawn by the app (not the browser's picker): all 42 day cells visible in the dark theme,
  today outlined in amber, the current value highlighted, ‹ › for months, Month and Year lists to jump, plus
  Today / Clear / Close. Escape or a click outside closes it. Fits on screen on phones.
- Picked dates are written in the format that field already uses on that page (MM/DD/YYYY by default;
  YYYY-MM-DD where that is what the page already holds), so a column never mixes formats.
- Bug fixes:
  - Clicking an MM/DD/YYYY date in a table used to show an empty box, and clicking away then saved the
    date as blank. The editor now shows the stored value, and an unchanged date is not re-saved.
  - Date columns now sort by real date (12/01/2025 before 01/05/2026); blanks sort last.
  - YYYY-MM-DD dates no longer show a day early on the Calendar / Group by month (they were read as UTC).
  - The Calendar's "today" highlight used UTC and jumped to tomorrow after 5 PM in Arizona.

Code-only: firebase deploy --only hosting  (rules unchanged)

---

# Update 12 — Daily Expenses: by month, top shops, need type, bought for others
- New fields: Need type (Basic need / Can be avoided / Unwanted), Bought for (Our family / Others),
  Who for (if others). Need type and Bought for are dropdowns in the table, so existing entries can be tagged
  without opening the form.
- Shop suggests common shops plus every shop already used on the page.
- Filters: Month (newest first), Need type, Bought for, Shop, Paid card.
- Summary (follows filters and search): Total spent, Basic need / Can be avoided / Unwanted with % of total,
  Need type not set, Bought for others, Pending returns.
- By month panel: each month's total, its top shop and its avoidable spend (Can be avoided + Unwanted).
  Click a month to filter the page to it (click again to clear).
- Top shops panel: top 8 shops with bars and % for what's shown, so with a month selected it answers
  "where am I spending most this month". Shop names are grouped ignoring case/extra spaces.
- Existing entries are unchanged; Need type and Bought for start blank.

Code-only: firebase deploy --only hosting  (rules unchanged)

---

# Update 13 — Calendar on every field labelled "Date"
- Besides real date fields, any plain text field whose label contains the word "Date" now gets 📅 + Today,
  in the Add / Edit form and in the table editor: e.g. Travel Plans → Dates, and any column you added or
  renamed with "Date" in its name ("Due Date", "Date of birth", "Wedding date"). Words like "Updated" don't count.
  Dropdown and long-text (notes) fields are never changed.
- Typing is unchanged: free text such as "Dec 20 - Jan 5" or "TBD" is saved exactly as typed. Only a single
  typed date (10/2/26, 2026-10-02) is tidied to the page's format.
- Sorting these columns puts real dates in date order first, then other text, then blanks.
- If you rename a column to include "Date" later, it picks this up automatically - no code change needed.

Code-only: firebase deploy --only hosting  (rules unchanged)

---

# Update 14 — Save / Cancel at the top of every Add / Edit form (all pages)
- The form's title bar now has ✕ Cancel and ✓ Save, and it stays pinned at the top while you scroll the form.
  The Save / Cancel buttons at the bottom are unchanged.
- Small phones (under 400px wide) show just the ✕ / ✓ icons.
- Ctrl+Enter (Cmd+Enter on Mac) saves from any field.
- Both Save buttons are disabled while a save is in progress, so a double click can't create a duplicate entry.

Code-only: firebase deploy --only hosting  (rules unchanged)

---

# Update 15 — Rename "Raja Event History" to "Daily Events / Diary"
- Menu, page title and search box use the new name. The internal page key (eventHistory) is unchanged,
  so existing entries, Who has access settings, menu group and Calendar entries all carry over.

Code-only: firebase deploy --only hosting  (rules unchanged)

---

# Update 16 — Daily Events / Diary: filter by Year and Month
- New filters above the table: Year (newest first) and Month (January–December order). Combine them,
  e.g. 2026 + October; each option shows its count. Search still works on top.
- Both are worked out from Date (any stored format) and never saved, so existing entries need no changes.
- Group by month also uses Date on this page.

Code-only: firebase deploy --only hosting  (rules unchanged)

---

# Update 17 — Search all pages (global search)
- New menu item under Calendar: 🔎 Search all pages. Ctrl+K (Cmd+K on Mac) opens it from anywhere.
  Typing in the sidebar menu box also offers "Search entries for …" at the top of the list.
- Searches every field of every entry on every page you can open (including custom columns and calculated
  values like Repeats or Month). Case doesn't matter; several words must all appear in the same entry, in any
  order ("costco october", "chandler amway"). Needs at least 2 letters.
- Results are grouped by page (most matches first), newest first within a page, with the matching words
  highlighted and the field each match was found in. Up to 25 per page, then "open the page".
- Click a result to open that entry's Edit form on its page (view-only access: the page opens with that row
  expanded). "Open page" goes to the page; a one-word search is also put in that page's search box.
- Respects access: only pages you're allowed to open are searched. Vault pages are skipped while locked
  (the result list says which), and password fields are never searched.
- Data: all pages are read once when you open Search and reused for 10 minutes (↻ reloads now), so typing
  doesn't cost Firestore reads. Each load reads every entry you can see — the same as opening Home.

Code-only: firebase deploy --only hosting  (rules unchanged)

---

# Update 18 — Page search box clears when you switch pages
- Bug fix: the search box on a page used one shared value, so text typed on one page kept filtering the
  next page you opened. Each page now opens with an empty search box.
- The search stays while you stay on the page (including live updates and edits).
- Exception: opening a page from Search all pages still pre-fills a one-word search, on purpose.
- The sidebar menu filter box is separate and unchanged.

Code-only: firebase deploy --only hosting  (rules unchanged)
