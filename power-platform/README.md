# Drum Verify — Microsoft Power Platform version

This folder is a port of the Drum Verify web app to **Power Apps + Power Automate +
SharePoint**, so the whole thing lives inside your Microsoft 365 tenant.

The flow it implements:

```
Shipping gets a packing slip / load
        │
        ▼
Worker opens the "Drum Verify" Power App on a phone/tablet
        │   (scans the slip barcode OR picks the shipment from a list)
        ▼
App shows the REFERENCE PHOTO of the drum that belongs on this load
        │
        ▼
Worker taps "Take Photo" → snaps the drum actually being loaded
        │
        ▼
Worker taps  ✅ MATCH  or  ❌ MISMATCH
        │
        ▼
Power Automate flow "VerifyDrum":
   • writes a row to the Verifications list (with the captured photo attached)
   • updates the Shipment status to Verified / Mismatch
   • emails the supervisor the result + both photos
        │
        ▼
App shows a confirmation and returns to the shipment list
```

## What's in here

| File | Purpose |
|---|---|
| `BUILD-GUIDE.md` | **Start here.** Step-by-step: create the SharePoint lists, build the canvas app screen-by-screen (every control + Power Fx formula), and build the flow. ~45–60 min. |
| `sharepoint-schema.md` | Column-by-column definition of the three SharePoint lists, in one place. |
| `flow-VerifyDrum.json` | The Power Automate flow as a Logic Apps workflow definition — reference for the flow steps in the guide, and usable if you import it via a solution package (see notes in the guide). |
| `drums-seed.csv` | The 6 demo drums from the original `drums.json`, ready to paste into the **Drums** list (Edit in grid view). |

## Why a build guide and not a one-click import?

Power Apps `.msapp` files and Power Platform solution `.zip` packages are binary-ish
artifacts that are tightly bound to a specific environment, connection references, and
publisher prefix. A hand-authored one almost never imports cleanly. A written guide with
exact formulas is the reliable path — and once it's built in your tenant you *can* export
your own solution package for backup / moving between environments.

If you want, the assistant that generated this can walk you through the build
interactively — just ask.
