# Drum Verify on Power Platform — build guide

This rebuilds the Drum Verify app inside Microsoft 365 using **SharePoint** (data),
**Power Apps** (the canvas app shipping uses on a phone/tablet), and **Power Automate**
(the flow that logs the result and emails the supervisor).

Time: ~45–60 minutes. No premium licensing required — it only uses the **standard**
SharePoint and Office 365 Outlook connectors, which are covered by a normal M365 license.

> Conventions in this guide
> - `Code font` = type this literally, or a control/list/column name.
> - **Power Fx formulas** are written for the control property named in bold before them.
> - Where you see `yourtenant` / `REPLACE_…`, substitute your real values.

---

## Part 0 — What you'll create

| Thing | Name | Notes |
|---|---|---|
| SharePoint site | `Shipping` (or reuse one you have) | Holds the 3 lists |
| SharePoint list | `Drums` | The catalog — replaces `drums.json` |
| SharePoint list | `Shipments` | The packing slips / loads to verify |
| SharePoint list | `Verifications` | The audit log — replaces the browser history |
| Power Automate flow | `VerifyDrum` | Logs result, attaches photo, updates shipment, emails supervisor |
| Power Apps canvas app | `Drum Verify` | 3 screens: pick shipment → verify → confirm |

---

## Part 1 — SharePoint lists

Full column tables are in **`sharepoint-schema.md`** — create the three lists exactly as
described there, then come back.

Quick version:

1. Go to your SharePoint site → **New → List** → create `Drums`, `Shipments`,
   `Verifications` with the columns from `sharepoint-schema.md`.
2. Open `Drums` → **Edit in grid view** → paste the rows from **`drums-seed.csv`**
   (you can also just type your own real drums).
3. For each `Drums` row, add the reference photo: either set the **ReferenceImage**
   (Image) column, or attach the matching `.jpg` from this repo's `../drums/` folder to
   the list item. (The app reads whichever you use — see the note in Part 3, Screen 2.)
4. Add a few test rows to `Shipments`: a `Title` like `SLIP-1001`, pick a `DrumSKU`,
   set `Destination`, leave `Status` = `Pending`, put your own email in `SupervisorEmail`.

---

## Part 2 — The `VerifyDrum` flow

Build this **before** the app, so it's available to wire into the buttons.
(`flow-VerifyDrum.json` in this folder is the same flow as a Logic Apps definition, for
reference / backup.)

1. **make.powerautomate.com** → same environment as your app → **Create → Instant cloud
   flow** → name it `VerifyDrum` → trigger: **Power Apps (V2)** → Create.

2. On the trigger, click **+ Add an input** once per parameter below (pick the type from
   the little dropdown — *Text*, *Number*, or **File**):

   | Type | Name |
   |---|---|
   | Number | `ShipmentItemId` |
   | Text | `ShipmentNumber` |
   | Text | `DrumSKU` |
   | Text | `DrumName` |
   | Text | `Result` |
   | Text | `Notes` |
   | Text | `SupervisorEmail` |
   | Text | `VerifierEmail` |
   | **File** | `PhotoFile` |

3. **+ New step → SharePoint → Create item**
   - **Site Address**: your `Shipping` site · **List Name**: `Verifications`
   - **Title**: switch to expression →
     `concat(triggerBody()['text_2'], ' — ', triggerBody()['text_4'], ' — ', utcNow())`
     *(don't worry about the exact `text_N` names — just pick the dynamic-content tokens
     **DrumSKU**, then `' — '`, then **Result**, then `' — '`, then **utcNow()**)*
   - **ShipmentRef Id**: dynamic content **ShipmentItemId**
   - **DrumSKU**: **DrumSKU** · **DrumName**: **DrumName**
   - **Result Value**: pick **Enter custom value** → **Result**
   - **Notes**: **Notes**
   - **VerifiedBy Claims**: **VerifierEmail**
   - **VerifiedAt**: expression `utcNow()`

4. **+ New step → SharePoint → Add attachment**
   - **Site Address** / **List Name** `Verifications`
   - **Id**: dynamic content **ID** (from *Create item*)
   - **File Name**: expression
     `coalesce(triggerBody()?['file']?['name'], concat(triggerBody()['text_2'], '-', triggerBody()['text_4'], '.jpg'))`
     *(again, easier: just type `loaded-` then insert **DrumSKU** then `.jpg`)*
   - **File Content**: dynamic content **PhotoFile** *(the file token — Power Automate
     passes the bytes through)*

5. **+ New step → SharePoint → Update item**
   - **Site Address** / **List Name** `Shipments`
   - **Id**: **ShipmentItemId**
   - **Status Value**: **Enter custom value** → expression
     `if(equals(toLower(triggerBody()['text_4']), 'match'), 'Verified', 'Mismatch')`
     *(`text_4` = the **Result** input)*
   - **VerifiedBy Claims**: **VerifierEmail** · **VerifiedAt**: `utcNow()`

6. **+ New step → Office 365 Outlook → Send an email (V2)**
   - **To**: **SupervisorEmail** *(optionally wrap: `coalesce(triggerBody()?['…SupervisorEmail'], 'REPLACE_default_supervisor@yourtenant.com')`)*
   - **Subject**: `Drum Verify — ` + **Result** + ` — ` + **DrumSKU** + ` (` + **ShipmentNumber** + `)`
   - **Body** (click **</>** to switch to HTML, or just type plain text):
     ```
     Result: <Result>
     Shipment: <ShipmentNumber>
     Drum: <DrumName> (<DrumSKU>)
     Verified by: <VerifierEmail>
     When: <utcNow()>
     Notes: <Notes>
     Photo of the loaded drum is attached.
     ```
   - **Show advanced options → Attachments**: **Add new item** →
     **Attachments Name** = expression `coalesce(triggerBody()?['file']?['name'], 'loaded-drum.jpg')`,
     **Attachments Content** = dynamic content **PhotoFile**.
   > The email is sent *from the flow author's mailbox*. If you want it from a shared
   > mailbox, use **Send an email from a shared mailbox (V2)** instead and put the shared
   > address in the *Original Mailbox Address* field.

7. **+ New step → Respond to a PowerApp or flow**
   - **+ Add an output → Text** named `ResultMessage` = `concat(triggerBody()['text_4'], ' recorded for ', triggerBody()['text_2'])` *(Result recorded for DrumSKU)*
   - **+ Add an output → Number** named `VerificationId` = dynamic content **ID** (from *Create item*)

8. **Save**. Test it once with the **Test** button (supply dummy values + any small image)
   to confirm the connections work.

---

## Part 3 — The `Drum Verify` canvas app

**make.powerapps.com → + Create → Blank app → Blank canvas app** → name `Drum Verify`,
format **Phone** → Create.

### Connect data
Left rail → **Data → + Add data** → add the three SharePoint lists: `Drums`,
`Shipments`, `Verifications`. Then **Power Automate → + Add flow → VerifyDrum**.

### App settings
Settings → **General** → make sure *Modern controls* are on (so you get the Barcode
reader). Settings → **Display** → **Lock orientation** → Portrait (optional).

---

### Screen 1 — `scrShipments` (pick the load to verify)

Rename `Screen1` to `scrShipments`.

Add controls:

- **Label** `lblTitle1` — **Text** = `"Drum Verify"` ; big, top of screen.
- **Text input** `txtSearch` — **HintText** = `"Search slip #…"`.
- **(optional) Barcode reader** `bcShipment` — Insert → *Barcode reader*.
  **OnScan**:
  ```powerapps
  With(
      { s: LookUp(Shipments, Title = bcShipment.Value) },
      If(
          IsBlank(s),
          Notify("No shipment found for " & bcShipment.Value, NotificationType.Error),
          Set(varShipment, s);
          Navigate(scrVerify, ScreenTransition.Cover)
      )
  )
  ```
  *(If your printed codes look like `sku:DRUM-…` from the old app, use
  `Trim(Last(Split(bcShipment.Value, ":")).Result)` in place of `bcShipment.Value`.)*

- **Vertical gallery** `galShipments` — **Items**:
  ```powerapps
  SortByColumns(
      Filter(
          Shipments,
          Status.Value = "Pending",
          IsBlank(txtSearch.Text) || StartsWith(Title, txtSearch.Text)
      ),
      "Title",
      SortOrder.Ascending
  )
  ```
  In the gallery template:
  - Title label **Text** = `ThisItem.Title`
  - Subtitle label **Text** = `ThisItem.DrumSKU.Value & "  →  " & ThisItem.Destination`
  - **OnSelect** (on the template / the chevron):
    ```powerapps
    Set(varShipment, ThisItem);
    Navigate(scrVerify, ScreenTransition.Cover)
    ```

- **Screen `scrShipments` → OnVisible**:
  ```powerapps
  Refresh(Shipments); Refresh(Drums)
  ```

---

### Screen 2 — `scrVerify` (compare & confirm)

Insert → New screen → blank, rename `scrVerify`.

**Screen `scrVerify` → OnVisible**:
```powerapps
Set(varRefDrum, LookUp(Drums, Title = varShipment.DrumSKU.Value));
Set(varPhoto, Blank());
Reset(camDrum);
Reset(txtNotes)
```

Controls:

- **Label** `lblShipNo` — **Text** = `"Shipment: " & varShipment.Title`
- **Label** `lblDest` — **Text** = `"Dest: " & varShipment.Destination`
- **Label** `lblDrumName` — **Text** = `varRefDrum.DrumName` ; bold.
- **Label** `lblDrumDesc` — **Text** = `varRefDrum.Description`

- **"Should look like" image** `imgReference` — **Image**:
  ```powerapps
  // If you used the ReferenceImage (Image) column:
  varRefDrum.ReferenceImage
  // --- OR, if you used a list-item attachment instead, use this instead: ---
  // First(varRefDrum.Attachments).AbsoluteUri
  ```
  Keep only one of those lines; delete the comments.

- **Camera control** `camDrum` — Insert → Media → *Camera*.
  **OnSelect** = `Set(varPhoto, Self.Photo)`  *(tapping the camera captures the still)*
  Optionally set **Camera** = `0` and add a "switch camera" button later; the rear camera
  is usually `1` on phones — leave default for now.

- **"Your drum" image** `imgCaptured` — **Image** = `varPhoto` ; **Visible** = `!IsBlank(varPhoto)`.

- **Button** `btnRetake` — **Text** = `"Retake"` ; **Visible** = `!IsBlank(varPhoto)` ;
  **OnSelect** = `Set(varPhoto, Blank()); Reset(camDrum)`

- **Text input** `txtNotes` — **HintText** = `"Note (required if mismatch)"` ; multiline.

- **Button** `btnMatch` — **Text** = `"✅ MATCH"` ; green.
  **DisplayMode** = `If(IsBlank(varPhoto) || varBusy, DisplayMode.Disabled, DisplayMode.Edit)`
  **OnSelect**:
  ```powerapps
  Set(varBusy, true);
  Set(
      varFlowResult,
      VerifyDrum.Run(
          varShipment.ID,
          varShipment.Title,
          varShipment.DrumSKU.Value,
          varRefDrum.DrumName,
          "Match",
          txtNotes.Text,
          Coalesce(varShipment.SupervisorEmail, "REPLACE_default_supervisor@yourtenant.com"),
          User().Email,
          { file: varPhoto, filename: "loaded-" & varShipment.Title & ".jpg" }
      )
  );
  Set(varBusy, false);
  Set(varResultMsg, varFlowResult.resultmessage);
  Navigate(scrConfirm, ScreenTransition.Cover)
  ```
  > The last argument is the **File** input. When you actually type `VerifyDrum.Run(`
  > Power Apps IntelliSense shows you the exact shape it expects for that parameter — if it
  > differs from `{ file:…, filename:… }`, follow IntelliSense. (Alternative if the file
  > param gives you trouble: change the flow input `PhotoFile` to a **Text** input and pass
  > `JSON(varPhoto, JSONFormat.IncludeBinaryData)`, then in the flow `base64ToBinary(...)`
  > the string before attaching/emailing.)

- **Button** `btnMismatch` — **Text** = `"❌ MISMATCH"` ; red.
  **DisplayMode** = `If(IsBlank(varPhoto) || IsBlank(txtNotes.Text) || varBusy, DisplayMode.Disabled, DisplayMode.Edit)`
  **OnSelect**: same as `btnMatch` but with `"Mismatch"` instead of `"Match"`.

- **Button** `btnBack` — **Text** = `"Cancel"` ; **OnSelect** = `Navigate(scrShipments, ScreenTransition.UnCover)`

- **Label** `lblBusy` — **Text** = `"Saving…"` ; **Visible** = `varBusy`. (Or use a `Spinner` modern control.)

---

### Screen 3 — `scrConfirm`

Insert → New screen → blank, rename `scrConfirm`.

- **Icon** (Check) — big green check.
- **Label** `lblConfirm` — **Text** = `varResultMsg` *(e.g. "Match recorded for DRUM-BLUE-CART")*
- **Button** `btnNext` — **Text** = `"Next shipment"` ;
  **OnSelect** = `Navigate(scrShipments, ScreenTransition.UnCover)`

Set the app's start screen: **App → StartScreen** = `scrShipments` (or just make sure
`scrShipments` is the first screen in the tree).

---

## Part 4 — Run it

1. **Save** → **Publish** the app.
2. Open it on the **Power Apps mobile app** (iOS/Android) signed in as a warehouse user,
   or share the app + the three lists with the shipping team.
3. Worker: opens the app → taps (or scans) a shipment → sees the reference drum photo →
   taps the **Camera** to snap the drum on the truck → taps **MATCH** or **MISMATCH**
   (mismatch requires a note) → sees the confirmation. Behind the scenes the `Verifications`
   list gets a new row with the photo attached, the `Shipments` row flips to
   `Verified`/`Mismatch`, and the supervisor gets an email with both the details and the photo.

---

## Part 5 — Equivalents of the old "supervisor" features

| Old web app feature | Power Platform equivalent |
|---|---|
| Add product + photo (supervisor tab) | Add a row to the **`Drums`** list in SharePoint (or generate the auto list app from SharePoint). Optionally build a small `scrAdmin` screen with a SharePoint **EditForm** on `Drums` — `New`/`EditForm` controls support the Add picture control for the reference image. |
| Generate / print QR codes | Add a **Hyperlink** column to `Drums` (or `Shipments`) with a calculated value `="https://api.qrserver.com/v1/create-qr-code/?size=200x200&data=" & [Title]`, then print the list view; or use a Power Automate flow with the *Encodian / Plumsail* QR action; or just print barcodes of the slip numbers. |
| History tab | The **`Verifications`** list. Make views: *All*, *Mismatches only* (`Result = Mismatch`), *Today*. |
| Email report / Export CSV | Built into SharePoint: list view → **Export to Excel/CSV**. Or add a scheduled "weekly digest" flow that emails a CSV of the week's `Verifications`. |
| localStorage | Replaced by SharePoint (real, shared, durable storage). |

---

## Troubleshooting notes

- **Delegation warning on the gallery `Items`**: Choice-column equality (`Status.Value = "Pending"`)
  isn't delegable in SharePoint, so only the first 500/2000 items are scanned. Fine for a
  live "pending loads" list. If it ever isn't, add a plain **Text** `StatusText` column to
  `Shipments` (the flow already could maintain it) and filter on that, or switch the lists
  to Dataverse.
- **`varRefDrum` is blank** on `scrVerify`: the `Shipments.DrumSKU` lookup must point at the
  `Drums` list's **Title** column, and the `Drums` `Title` values must match exactly
  (`DRUM-BLUE-CART`, etc.).
- **Camera shows but `varPhoto` stays blank**: make sure `camDrum.OnSelect` is
  `Set(varPhoto, Self.Photo)` (not `.Stream`), and that you're tapping the camera control
  itself to capture.
- **Flow "file" parameter**: if `VerifyDrum.Run(...)` complains about the last argument,
  trust IntelliSense for the exact shape, or fall back to the Text-input + `JSON(...)`
  approach noted on `btnMatch`.
- **Emails not arriving**: the flow's Outlook connection = the flow author. Check the flow
  run history; re-authorize the Office 365 Outlook connection if it shows an auth error.
- **Sharing**: users need access to the app *and* to all three SharePoint lists.
