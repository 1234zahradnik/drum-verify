# SharePoint lists — schema

Create these three lists on a SharePoint site (e.g. a Team site called **Shipping**).
"List" type, not a document library. Internal names are auto-generated from the display
name; if you create a column called `Drum SKU` SharePoint stores it as `Drum_x0020_SKU`
— the build guide uses the **display names** below; Power Apps handles the mapping.

> Tip: when you add a column, SharePoint may add it to the default view but you can hide
> ones you don't want users editing by hand. The app drives everything; the lists just store data.

---

## 1. `Drums` — the catalog (replaces `drums.json`)

| Column (display name) | Type | Notes |
|---|---|---|
| `Title` | Single line of text | **Use this as the SKU**, e.g. `DRUM-BLUE-CART`. (Rename the built-in Title column's display to "SKU" if you like — keep internal name `Title`.) |
| `DrumName` | Single line of text | Friendly name, e.g. "Blue Drum on Cart" |
| `Description` | Multiple lines of text (plain) | What it should look like |
| `ReferenceImage` | **Image** column (a.k.a. "Image") | The reference photo. (If your tenant doesn't offer the Image column type, use **Hyperlink or Picture → Picture** instead, or just rely on the list-item **Attachment**.) |

Seed it with `drums-seed.csv` (open the list → **Edit in grid view** → paste). Then add
the reference photo to each row — either by setting the `ReferenceImage` column, or by
attaching the matching file from this repo's `../drums/` folder to the list item.

---

## 2. `Shipments` — the packing slips / loads

| Column (display name) | Type | Notes |
|---|---|---|
| `Title` | Single line of text | Packing slip / load number, e.g. `SLIP-10432` |
| `DrumSKU` | **Lookup** → `Drums` list, showing the `Title` (SKU) column | The drum that belongs on this load |
| `Destination` | Single line of text | Customer / dock / truck — optional |
| `Status` | **Choice**: `Pending`, `Verified`, `Mismatch` | Default `Pending` |
| `SupervisorEmail` | Single line of text | Who gets the result email for this load. (Or hard-code one address in the flow if it's always the same person.) |
| `VerifiedBy` | Person or Group | Set by the flow |
| `VerifiedAt` | Date and Time (include time) | Set by the flow |

Add a couple of test rows pointing at SKUs from the `Drums` list.

---

## 3. `Verifications` — the audit log (replaces the localStorage history)

| Column (display name) | Type | Notes |
|---|---|---|
| `Title` | Single line of text | The flow sets this to `<SKU> — <Result> — <timestamp>` |
| `ShipmentRef` | **Lookup** → `Shipments`, showing `Title` | Which load |
| `DrumSKU` | Single line of text | Copied in by the flow (handy for filtering/exports) |
| `DrumName` | Single line of text | Copied in by the flow |
| `Result` | **Choice**: `Match`, `Mismatch` | |
| `Notes` | Multiple lines of text (plain) | Optional note the worker typed on a mismatch |
| `VerifiedBy` | Person or Group | The signed-in worker |
| `VerifiedAt` | Date and Time (include time) | When |
| *(attachment)* | — | The captured truck photo is added as a **list item attachment** by the flow. |

Exports: SharePoint list views → **Export to Excel/CSV** replaces the old CSV button. A
saved view grouped by `Result` or filtered to `Result = Mismatch` is your "report".
