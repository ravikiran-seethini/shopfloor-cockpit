# Shopfloor Cockpit

A small, installable web app that replaces the paper shift book and tally sheets at the machine. Operators report stops, scrap and handover notes on a tablet or phone; the app calculates OEE, shows a downtime Pareto, tracks improvement actions and exports clean tables for Excel and Power BI.

It comes with sample data for an aluminium extrusion plant (three presses, about a week of shifts). The interface switches between German and English.

No AI, no account, no server: everything runs in the browser and works offline.

---

## What it does

| Area | What you can do | Industry term |
|---|---|---|
| Orders | Import a production-order list from Excel or CSV (e.g. an SAP export) | ERP link |
| Machine screen | Scan the machine's QR label, choose the order, report a stop reason and scrap in under a minute | BDE (operator data capture) |
| Machine signal | Switch on a simulated machine that reports running/stopped and part counts by itself; unexplained stops are flagged until someone assigns a reason | MDE (machine data capture) |
| Dashboard | OEE split into availability, performance and quality; OEE per day or shift; downtime Pareto; scrap by cause; plan vs. actual per order | MES / shop-floor management |
| Shift report | Automatic summary per shift, stop list, handover note with takeover confirmation | Digital shift book |
| Actions | Create an action straight from the biggest loss, with owner, due date and status | Lean action tracking (Maßnahmenverfolgung) |
| Export | One Excel file with seven tables, single CSVs, and a JSON backup | Confirmation to ERP, Power BI data source |
| System links | A diagram of where SAP, an MES and real machines would connect | Architecture |

### How OEE is calculated

- **Planned time** = shift length minus planned stops (breaks, planned maintenance, no order).
- **Availability** = run time ÷ planned time, where run time = planned time minus unplanned stops (including unexplained ones).
- **Performance** = parts produced ÷ (run time × ideal rate), capped at 100 %.
- **Quality** = good parts ÷ parts produced.
- **OEE** = availability × performance × quality. Totals over several shifts or machines are weighted by time and quantity, not averaged.

---

## Put it online for free (GitHub Pages, about 10 minutes)

1. Create a free account at github.com.
2. Click **New repository**, name it `shopfloor-cockpit`, set it to **Public**, and create it.
3. Click **uploading an existing file** and drag in **everything in this folder** (keep the `lib` and `icons` folders). Click **Commit changes**.
4. Go to **Settings → Pages**. Under *Branch*, choose `main` and `/ (root)`, then **Save**.
5. After a minute or two the app is live at `https://<your-username>.github.io/shopfloor-cockpit/`.

To update later, upload the changed files again in the same way.

### Install it on a phone or tablet

- **Android (Chrome):** open the link → menu ⋮ → *Install app* or *Add to home screen*.
- **iPhone/iPad (Safari):** open the link → Share → *Add to Home Screen*.

It then opens full screen with its own icon and keeps working without internet.

### Print the machine labels

Open **Setup → QR labels**, press *Print / PDF*, and stick each label on its machine. Pointing a normal phone camera at a label opens the app directly on that machine. The labels contain the app's own address, so print them **after** the app is online.

---

## Two-minute interview demo

1. **Problem (15 s):** "Many mid-sized plants have SAP but no MES. In between is paper: shift books, tally sheets, Excel on Monday morning."
2. **Machine screen (45 s):** scan the label of Press 2 with the phone, show the flagged unexplained stop, assign *Puller fault* with one tap, book 12 pcs scrap as *weld seam defect*.
3. **Machine signal (20 s):** switch on the simulation; the shift is replayed in fast motion and the status changes live.
4. **Dashboard (30 s):** OEE against target, the downtime Pareto, then *+ Action* on the top loss.
5. **Data (10 s):** export the Excel file and show it in Power BI (see `POWER_BI_GUIDE.md`).

Say clearly that the data is sample data and the machine signal is simulated, then explain how a real plant would connect it (System links tab).

---

## Limits of version 1

- Data is stored on the device that entered it (browser storage). Two tablets do not see each other's entries; combine them via export. Clearing the browser data deletes it, so use **Export → Backup** regularly.
- The machine signal is simulated. In a real plant it would come from a clamp-on current sensor or the PLC through an IoT gateway (OPC UA or MQTT).
- Order import is file-based. A live SAP interface (RFC/OData) would be set up with the IT department.

**Planned next step (version 2):** sync to a shared Google Sheet so several devices share live data and Power BI can refresh from it automatically.

---

## Files

| File | Purpose |
|---|---|
| `index.html` | The whole app |
| `sw.js`, `manifest.webmanifest`, `icons/` | Make it installable and offline-capable |
| `lib/qrcode.js` | Creates the QR labels (MIT licence) |
| `lib/jsQR.js` | Reads QR codes with the camera (Apache 2.0) |
| `lib/xlsx.mini.min.js` | Reads and writes Excel files (SheetJS Community Edition, Apache 2.0) |
| `auftraege_beispiel.csv` | Sample order list in SAP-export style for testing the import |
| `POWER_BI_GUIDE.md` | Step-by-step Power BI dashboard |
