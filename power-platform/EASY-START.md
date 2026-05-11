# Drum Verify on Microsoft — the plain-English start guide

This is the "I've never touched Power Apps" version. It assumes nothing. Follow it like
furniture instructions. Total time: ~1 hour the first time.

You'll do three things:
1. **Make 3 lists in SharePoint** (these store your drum photos and your records).
2. **Import the ready-made flow** (the thing that emails the supervisor) — that's the
   `.zip` file in this folder.
3. **Build the phone app** by following click-by-click steps.

> Words you'll see a lot:
> - **List** = like a spreadsheet/table that lives in SharePoint.
> - **Flow** = an automation ("when X happens, do Y") built in Power Automate.
> - **Canvas app** = the screen people use on a phone/tablet, built in Power Apps.
> - **Connector / connection** = Power Apps' link to a service (SharePoint, Outlook). The
>   first time you use one it asks you to sign in / click "Create".

---

## Before you start — what you need

- A Microsoft 365 **work or school account** (the kind your employer gave you), and the
  ability to sign in at **make.powerapps.com**. (You said you can — good.)
- A SharePoint site you can create lists on. If you have a Team in Microsoft Teams, it
  already has a SharePoint site. Or ask whoever runs SharePoint for a site called
  **Shipping** (or use any site you have access to).
- A phone with the free **Power Apps** app installed (App Store / Google Play) — that's how
  shipping will use the finished app.

Download these two files from this folder onto your computer first:
- `VerifyDrum_import_package.zip`  ← the flow you'll import
- `drums-seed.csv`  ← starter data for the Drums list

---

## STEP 1 — Make the 3 SharePoint lists

Go to your SharePoint site in a browser. Top-left **+ New → List** → **Blank list**.

### List A: `Drums`
This is your catalog of what each drum *should* look like.

1. Name it `Drums`. Create.
2. It comes with a column called **Title** — leave it. You'll type the drum's code here
   (like `DRUM-BLUE-CART`).
3. Add columns with the **+ Add column** button at the top of the list:
   - **DrumName** — type: *Single line of text*
   - **Description** — type: *Multiple lines of text* → choose **Plain text**
   - **ReferenceImage** — type: *Image*  (if you don't see "Image" as an option, pick
     *Hyperlink or Picture* instead and choose "Picture" — or just skip it and use the
     paperclip "attach" button on each row later)
4. Put in your data:
   - Top of the list → **Edit in grid view** (looks like a little grid icon).
   - Open `drums-seed.csv` in Excel, copy the rows (not the header), and paste them into
     the grid. That fills in 6 example drums. Click **Exit grid view** to save.
   - For each row, add the photo: click the row to open it → set **ReferenceImage** (or use
     the **Attach file** button) → upload the matching picture. (The 6 example pictures are
     in this repo's `../drums/` folder — e.g. `drum_blue_cart.jpg` goes with
     `DRUM-BLUE-CART`.) Use your own real photos when you set up real drums.

### List B: `Shipments`
This is the list of loads/packing slips that need a drum verified.

1. **+ New → List → Blank list**, name it `Shipments`. Create.
2. Keep **Title** — you'll type the slip/load number here (like `SLIP-1001`).
3. Add columns:
   - **DrumSKU** — type: *Lookup* → "Get information from" = **Drums**, "In this column" =
     **Title**. (This makes each shipment point at a drum from your catalog.)
   - **Destination** — *Single line of text*
   - **Status** — type: *Choice* → enter three choices: `Pending`, `Verified`, `Mismatch`
     → set the default value to `Pending`
   - **SupervisorEmail** — *Single line of text* (who gets the result email for this load)
   - **VerifiedBy** — type: *Person*
   - **VerifiedAt** — type: *Date and time* → turn ON "Include time"
4. Add 2–3 test rows: a Title like `SLIP-1001`, pick a **DrumSKU**, type a **Destination**,
   leave **Status** = `Pending`, put **your own email** in **SupervisorEmail**.

### List C: `Verifications`
This is the log — every check that gets done lands here (this replaces the old app's
"History" tab).

1. **+ New → List → Blank list**, name it `Verifications`. Create.
2. Keep **Title** (the flow fills it in automatically).
3. Add columns:
   - **DrumSKU** — *Single line of text*
   - **DrumName** — *Single line of text*
   - **Result** — *Choice* → choices: `Match`, `Mismatch`
   - **Notes** — *Multiple lines of text* → **Plain text**
   - **ShipmentRef** — *Lookup* → "Get information from" = **Shipments**, "In this column"
     = **Title**
   - **VerifiedBy** — *Person*
   - **VerifiedAt** — *Date and time* → "Include time" ON
4. (No data to add — the flow writes to this one.)

**Write down your site URL** — it looks like
`https://yourcompany.sharepoint.com/sites/Shipping`. You'll need it twice below.

---

## STEP 2 — Import the flow

1. Go to **make.powerautomate.com** → sign in with the same account.
2. Left side → **My flows**.
3. Top bar → **Import** → **Import Package (Legacy)**.
4. **Upload** the file `VerifyDrum_import_package.zip`.
5. After it scans the file you'll see an **Import setup** screen:
   - The flow **VerifyDrum** — make sure it says **"Create as new"** (it should by default).
   - **SharePoint** — click it → choose **"Select during import"** → pick your existing
     SharePoint connection from the dropdown. (If there isn't one, there's a link to create
     one — click it, sign in, come back, and pick it.)
   - **Office 365 Outlook** — same thing: select (or create) the connection.
6. Click **Import**. Wait for "successfully imported".

   > **If import fails or complains:** don't sweat it — building the flow by hand is only
   > about 8 short steps. Open `BUILD-GUIDE.md` in this folder and do **Part 2 — The
   > VerifyDrum flow**. Then come back here. Everything else is the same.

7. Now **fix the placeholders** in the imported flow (it was built without knowing your
   site name):
   - **My flows** → click **VerifyDrum** → **Edit**.
   - You'll see steps with a warning triangle. Open each SharePoint step
     (*Create item*, *Add attachment*, *Update item*):
     - **Site Address**: it currently says `https://yourtenant.sharepoint.com/sites/Shipping`
       — click it and pick **your** site from the list (or paste your real URL).
     - **List Name**: pick `Verifications` (for the create + attach steps) or `Shipments`
       (for the update step) from the dropdown.
     - If any field shows red text like `item/Result/Value`, just re-pick the matching
       field name from that step's options.
   - Open the **Send an email (V2)** step → in **To**, replace `changeme@yourtenant.com`
     (it's only a fallback) — or leave it; the app will pass the real supervisor address.
   - **Save** (top right). If it won't save, it'll tell you which field it doesn't like —
     usually just re-picking the Site/List from the dropdowns fixes it.
8. Quick test: top right **Test → Manually → Test**. It'll ask for the inputs — type
   anything for the text ones, pick a tiny image file for **PhotoFile**, Run. If it goes
   green and you get an email, the flow works. (If a step is red, open it and re-pick its
   Site/List.)

---

## STEP 3 — Build the phone app

Go to **make.powerapps.com** → **+ Create** → **Blank app** → **Blank canvas app** →
name it `Drum Verify`, format **Phone** → **Create**. You're now in the app editor.

### 3.1 — Hook up your data and the flow
- Left rail → the **Data** icon (looks like a cylinder) → **+ Add data** → search and add
  **SharePoint** → pick your site → tick **Drums**, **Shipments**, **Verifications** →
  **Connect**.
- Left rail → **Power Automate** icon → **+ Add flow** → pick **VerifyDrum**.

### 3.2 — How the editor works (30-second orientation)
- Left rail **Tree view** = list of your screens and the controls on them.
- **+ Insert** (top) = add controls (labels, buttons, images, camera…).
- When you click a control, the **Properties** panel is on the right, and there's a
  **formula bar** at the top. To set a property: pick the property name from the little
  dropdown on the left of the formula bar, then type the formula in the bar.
- **File → Save** often. **Publish** when you want phones to get the new version.

### 3.3 — Screen 1: the list of loads to check
You start with one screen called `Screen1`.

1. In the Tree view, right-click `Screen1` → **Rename** → `scrShipments`.
2. Click the screen name `scrShipments` in the tree. In the formula bar, pick the property
   **OnVisible** and type:  `Refresh(Shipments); Refresh(Drums)`
3. **+ Insert → Label**. Drag it to the top. Click it, set its **Text** property to:
   `"Drum Verify"`  (keep the quotes). Make the font big in the right-hand panel.
4. **+ Insert → Text input** (a typing box). Set its **HintText** to: `"Search slip #"`
   In the tree, rename this control to `txtSearch`.
5. *(Optional but nice — barcode scanning):* **+ Insert** → search **Barcode reader** →
   add it. Rename it `bcShipment`. Set its **OnScan** property to:
   ```
   With(
       { s: LookUp(Shipments, Title = bcShipment.Value) },
       If(IsBlank(s),
          Notify("No shipment found for " & bcShipment.Value, NotificationType.Error),
          Set(varShipment, s); Navigate(scrVerify))
   )
   ```
6. **+ Insert → Vertical gallery** (a scrolling list). When it asks, choose your
   **Shipments** data source. Rename it `galShipments`. Set its **Items** property to:
   ```
   SortByColumns(
       Filter(Shipments,
              Status.Value = "Pending",
              IsBlank(txtSearch.Text) || StartsWith(Title, txtSearch.Text)),
       "Title", SortOrder.Ascending)
   ```
   (You may see a blue "delegation" squiggle — it's just a warning; ignore it for now.)
   - In the gallery, there are sample labels. Click the first label → set **Text** to
     `ThisItem.Title`. Click the second → set **Text** to
     `ThisItem.DrumSKU.Value & "  ->  " & ThisItem.Destination`.
   - Click the small **arrow ( > )** in the gallery row (or the row template) → set its
     **OnSelect** to:  `Set(varShipment, ThisItem); Navigate(scrVerify)`

### 3.4 — Screen 2: see the reference photo, snap yours, confirm
1. Top of the tree → **+ New screen → Blank**. Rename it `scrVerify`.
2. Click `scrVerify` in the tree → set **OnVisible** to:
   ```
   Set(varRefDrum, LookUp(Drums, Title = varShipment.DrumSKU.Value));
   Set(varPhoto, Blank()); Reset(camDrum); Reset(txtNotes)
   ```
3. Add these **Labels** (Insert → Label), one per line, set each one's **Text**:
   - `"Shipment: " & varShipment.Title`
   - `"To: " & varShipment.Destination`
   - `varRefDrum.DrumName`  (make this one bold/big)
   - `varRefDrum.Description`
4. Add an **Image** (Insert → Media → Image). Rename it `imgReference`. Set its **Image**
   property to:  `varRefDrum.ReferenceImage`
   - *If you used the paperclip/attach method instead of the Image column*, use this
     instead:  `First(varRefDrum.Attachments).AbsoluteUri`
5. Add a **Camera** control (Insert → Media → Camera). Rename it `camDrum`. Set its
   **OnSelect** property to:  `Set(varPhoto, Self.Photo)`
   (When the worker taps this camera, it takes the picture.)
6. Add another **Image**. Rename it `imgCaptured`. Set **Image** = `varPhoto` and set
   **Visible** = `!IsBlank(varPhoto)`  (so it only appears after they take the photo).
7. Add a **Button** (Insert → Button). Set its **Text** = `"Retake"`,
   **Visible** = `!IsBlank(varPhoto)`, **OnSelect** = `Set(varPhoto, Blank()); Reset(camDrum)`
8. Add a **Text input**. Rename it `txtNotes`. Set **HintText** = `"Note (needed if mismatch)"`.
9. Add a **Button**, set **Text** = `"MATCH"` (color it green in the right panel).
   Rename it `btnMatch`. Set:
   - **DisplayMode** = `If(IsBlank(varPhoto) || varBusy, DisplayMode.Disabled, DisplayMode.Edit)`
   - **OnSelect** =
     ```
     Set(varBusy, true);
     Set(varFlowResult,
         VerifyDrum.Run(
             varShipment.ID,
             varShipment.Title,
             varShipment.DrumSKU.Value,
             varRefDrum.DrumName,
             "Match",
             txtNotes.Text,
             Coalesce(varShipment.SupervisorEmail, "changeme@yourtenant.com"),
             User().Email,
             { file: varPhoto, filename: "loaded-" & varShipment.Title & ".jpg" }
         )
     );
     Set(varBusy, false);
     Set(varResultMsg, varFlowResult.resultmessage);
     Navigate(scrConfirm)
     ```
     > When you type `VerifyDrum.Run(` the editor pops up a hint showing the exact inputs
     > it wants and in what order — follow that hint. The last input is the photo file; if
     > the editor shows a different shape than `{ file: ..., filename: ... }`, type what it
     > shows.
10. Add another **Button**, **Text** = `"MISMATCH"` (color it red). Rename it `btnMismatch`.
    Set the same two properties as `btnMatch`, but:
    - **DisplayMode** = `If(IsBlank(varPhoto) || IsBlank(txtNotes.Text) || varBusy, DisplayMode.Disabled, DisplayMode.Edit)`
    - in the **OnSelect** formula, change `"Match"` to `"Mismatch"`.
11. Add a **Button** **Text** = `"Cancel"`, **OnSelect** = `Navigate(scrShipments)`.
12. Add a **Label** **Text** = `"Saving..."`, **Visible** = `varBusy`.

### 3.5 — Screen 3: the "done" screen
1. **+ New screen → Blank**. Rename it `scrConfirm`.
2. Add an **Icon** (Insert → Icons → Check) — make it big and green.
3. Add a **Label**, **Text** = `varResultMsg`.
4. Add a **Button**, **Text** = `"Next shipment"`, **OnSelect** = `Navigate(scrShipments)`.

### 3.6 — Make `scrShipments` the first screen
In the tree, drag `scrShipments` to the top of the screen list (or: click **App** at the
top of the tree → set **StartScreen** = `scrShipments`).

### 3.7 — Save, publish, use it
- **File → Save**, then **Publish** → **Publish this version**.
- On your phone, open the **Power Apps** app, sign in, and **Drum Verify** will be there.
  (To let the shipping team use it: in make.powerapps.com → Apps → "…" next to Drum Verify
  → **Share** → add them; also make sure they have access to the three SharePoint lists.)

---

## What "done" looks like

Worker on the phone:
1. Opens **Drum Verify** → sees the list of pending loads → taps one (or scans the slip).
2. Sees the photo + description of the drum that *should* go on that load.
3. Taps the camera → snaps the drum that's actually there.
4. Taps **MATCH** or **MISMATCH** (mismatch makes them type a note).
5. Sees "Match recorded for DRUM-…" and goes back to the list.

Behind the scenes the flow added a row to **Verifications** (with the photo attached),
flipped the **Shipments** row to `Verified`/`Mismatch`, and emailed the supervisor the
result plus the photo.

To see history / make reports: open the **Verifications** list in SharePoint → **Export to
Excel**, or make filtered views (e.g. only `Result = Mismatch`).

---

## If you get stuck

- **The app says `varRefDrum` is blank on screen 2** — the `Shipments` → `DrumSKU` lookup
  must point at the `Drums` list's **Title** column, and the codes have to match exactly.
- **Camera shows but `MATCH` stays greyed out** — that button is disabled until a photo is
  taken; make sure `camDrum.OnSelect` is `Set(varPhoto, Self.Photo)` and that you're
  tapping the camera control itself.
- **`VerifyDrum.Run(...)` shows a red error on the last input** — trust the editor's popup
  hint for the photo input's exact shape.
- **No email arrives** — open the flow's **Run history** (in make.powerautomate.com →
  VerifyDrum) to see which step failed; usually it's a step that still needs you to pick
  the Site/List, or the Outlook connection needs re-authorizing.
- **Stuck for real?** Ask the assistant that generated this — it can walk you through any
  step live, or look at a screenshot of the error.
