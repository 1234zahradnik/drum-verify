# Drum Verify → Power App — where we are

**Goal:** rebuild the existing Drum Verify web app as a Microsoft **Power App** (the boss
specifically wants a Power App). The shipping crew likes how the current app works; we're
recreating the same flow in Power Apps + Power Automate + SharePoint.

**Status:** plan + materials are ready in this `power-platform/` folder. Next step is a
**guided, step-by-step build** — the user (new to Power Apps) will share their screen
state / screenshots, and we go one small step at a time.

## What the user is gathering before we start the walkthrough

1. **Can you sign in at `make.powerapps.com`** with your work Microsoft account, and does it
   let you in (not blocked by IT)? — said yes; we'll confirm on the day.
2. **Power Automate too** — can you get into `make.powerautomate.com`?
3. **A SharePoint site** you can create lists on (or a Microsoft **Teams** team — those come
   with a SharePoint site). Name/URL if you have one.
4. **What phones** does the shipping crew use — iPhone, Android, or a mix?
5. **Supervisor email** — who should get the "MATCH / MISMATCH" result emails?
6. **Real drum list + photos** — can come later; not needed to start.
7. If anything says *"you don't have access"* or *"contact your admin"* when you try to
   create an app — note the exact message; that's an IT/licensing thing to sort first.

## The build, in plain terms (what we'll actually do together)

1. Make 3 SharePoint "lists" (think: simple spreadsheets):
   - **Drums** — the catalog: each drum's code, name, description, reference photo.
   - **Shipments** — the loads to check: slip number, which drum, destination, status.
   - **Verifications** — the log: every check, with the photo taken, who, when, match or not.
   (Column-by-column details: `sharepoint-schema.md`.)
2. Import the ready-made automation (`VerifyDrum_import_package.zip`) — the bit that writes
   the log entry, attaches the photo, updates the shipment, and emails the supervisor. If the
   import won't take, we build it by hand (~8 clicks; `BUILD-GUIDE.md` Part 2).
3. Build the phone app — 3 screens:
   - **Screen 1:** list of loads waiting to be checked (tap one, or scan the slip barcode).
   - **Screen 2:** shows the reference photo + description of the drum that *should* go on
     that load → worker taps the camera, snaps the drum that's actually there → taps
     **MATCH** or **MISMATCH** (mismatch makes them type a note).
   - **Screen 3:** "Done — recorded" → back to the list.
4. Publish it; install the free **Power Apps** app on the crew's phones; share the app +
   the 3 lists with them.

## Reference files in this folder

- `EASY-START.md` — the plain-English version of all the steps above (we'll follow this).
- `BUILD-GUIDE.md` — the detailed version with every formula.
- `sharepoint-schema.md` — exact columns for the 3 lists.
- `VerifyDrum_import_package.zip` — the importable automation; `flow-VerifyDrum.json` /
  `_flow-package-src/` are its readable insides.
- `drums-seed.csv` — 6 example drums to start with.

> When the user comes back with the info above, start the walkthrough at **STEP 1** of
> `EASY-START.md`, one sub-step at a time, waiting for them to confirm each click. Keep
> jargon out of it.
