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
| `EASY-START.md` | **Start here if Power Apps is new to you.** Plain-English, click-by-click: make the lists, import the flow `.zip`, build the app. ~1 hr. |
| `VerifyDrum_import_package.zip` | The Power Automate flow, ready to **import** (Power Automate → My flows → Import → *Import Package (Legacy)*). You still re-pick your SharePoint site/lists afterwards — `EASY-START.md` walks through it. If the import is grumpy, the by-hand build is only ~8 steps (`BUILD-GUIDE.md` Part 2). |
| `BUILD-GUIDE.md` | The fuller reference: SharePoint lists, the canvas app screen-by-screen (every control + Power Fx formula), and building the flow by hand. |
| `sharepoint-schema.md` | Column-by-column definition of the three SharePoint lists, in one place. |
| `flow-VerifyDrum.json` | The flow as a readable Logic Apps definition (what's inside the `.zip`) — handy for review or rebuilding. |
| `drums-seed.csv` | The 6 demo drums from the original `drums.json`, ready to paste into the **Drums** list (Edit in grid view). |
| `_flow-package-src/` | The unzipped source of `VerifyDrum_import_package.zip` (so the package contents are reviewable / editable in git). |

## Why only the *flow* is a downloadable file (and not the whole app)

- The **Power Automate flow** *can* ship as an importable `.zip` — `VerifyDrum_import_package.zip`
  here. It's hand-built, so the import screen may need you to pick connections, and after
  import you re-point it at your SharePoint site/lists. If it won't import, rebuilding it by
  hand is ~8 short steps (`BUILD-GUIDE.md` Part 2).
- The **canvas app** (`.msapp`) and a full **solution `.zip`** are binary-ish artifacts
  tightly bound to a specific environment, connection references, and publisher prefix — a
  hand-authored one almost never opens cleanly in Power Apps Studio. So the app is a
  click-by-click guide instead. Once it's built in your tenant you *can* export your own
  solution package for backup / moving between environments.

If you want, the assistant that generated this can walk you through any step
interactively, or look at a screenshot of an error — just ask.
