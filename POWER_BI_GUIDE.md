# Power BI dashboard from Shopfloor Cockpit

Build this yourself once, so you can explain every step in an interview. It takes about 30–45 minutes the first time. You need **Power BI Desktop** (free, Windows; TU Dortmund students can also use the web version through Microsoft 365 if available).

## 1. Get the data

1. In the app, open **Export** and click **Excel file (all tables)**.
2. Save it in a fixed folder, for example `OneDrive/Cockpit/shopfloor_cockpit.xlsx`. Rename it to that fixed name each time you export, so Power BI always finds it.

The file has seven sheets:

| Sheet | One row is … | Key columns |
|---|---|---|
| `Maschinen` | a machine | Maschine, Name, Soll_Stk_h |
| `Auftraege` | a production order | Auftrag, Material, Menge, Maschine, Termin |
| `Stillstaende` | a stop | Maschine, Auftrag, Start, Ende, Dauer_min, Grund, Geplant, Quelle, Schicht, Datum |
| `Mengen` | a quantity booking | Maschine, Auftrag, Zeit, Menge, Ausschuss, Gutmenge, Ausschussgrund, Quelle, Schicht, Datum |
| `OEE_Schicht` | a machine in a shift | Datum, Schicht, Maschine, Planzeit_min, Laufzeit_min, Menge, Gutmenge, Sollmenge, Verfuegbarkeit, Leistung, Qualitaet, OEE |
| `Rueckmeldung` | an order confirmation | Auftrag, Sollmenge, Gutmenge, Ausschuss, Stillstand_min, Status |
| `Massnahmen` | an action | Titel, Maschine, Verantwortlich, Termin, Status |

## 2. Load it

1. Power BI Desktop → **Get data → Excel workbook** → choose the file.
2. Tick all seven sheets → **Transform data**.
3. In Power Query, set the column **Datum** to type *Date* in `OEE_Schicht`, `Stillstaende` and `Mengen`. Check that the number columns are *Whole number* or *Decimal*.
4. **Close & Apply**.

## 3. Model (relationships)

In the model view, drag:

- `Maschinen[Maschine]` → `OEE_Schicht[Maschine]`, `Stillstaende[Maschine]`, `Mengen[Maschine]` (one to many)
- `Auftraege[Auftrag]` → `Mengen[Auftrag]`, `Rueckmeldung[Auftrag]`

Optional but good practice: create a date table (`Modeling → New table`):

```
Kalender = CALENDAR ( MIN ( OEE_Schicht[Datum] ), MAX ( OEE_Schicht[Datum] ) )
```

and link `Kalender[Date]` to the `Datum` columns.

## 4. Measures

`Home → New measure`, one at a time. OEE must be **weighted**, not averaged, so build it from sums:

```
Planzeit h      = SUM ( OEE_Schicht[Planzeit_min] ) / 60
Laufzeit h      = SUM ( OEE_Schicht[Laufzeit_min] ) / 60
Verfügbarkeit   = DIVIDE ( SUM ( OEE_Schicht[Laufzeit_min] ), SUM ( OEE_Schicht[Planzeit_min] ) )
Leistung        = MIN ( 1, DIVIDE ( SUM ( OEE_Schicht[Menge] ), SUM ( OEE_Schicht[Sollmenge] ) ) )
Qualität        = DIVIDE ( SUM ( OEE_Schicht[Gutmenge] ), SUM ( OEE_Schicht[Menge] ) )
OEE             = [Verfügbarkeit] * [Leistung] * [Qualität]
Ausschussquote  = DIVIDE ( SUM ( Mengen[Ausschuss] ), SUM ( Mengen[Menge] ) )
Stillstand h    = CALCULATE ( SUM ( Stillstaende[Dauer_min] ), Stillstaende[Geplant] = "nein" ) / 60
OEE Ziel        = 0.65
```

Format OEE, Verfügbarkeit, Leistung, Qualität and Ausschussquote as **Percentage**.

## 5. Visuals (one page)

1. **Cards** across the top: OEE, Verfügbarkeit, Leistung, Qualität, Stillstand h.
2. **Clustered column chart:** X axis `OEE_Schicht[Datum]`, Y axis `[OEE]`, Legend `OEE_Schicht[Maschine]`. Add a constant line at 65 % (Analytics pane) as the target.
3. **Bar chart (Pareto):** Y axis `Stillstaende[Grund]`, X axis `Stillstaende[Dauer_min]` (Sum). Add a visual-level filter `Geplant = nein`, sort descending.
4. **Bar chart:** `Mengen[Ausschussgrund]` against `Mengen[Ausschuss]` (Sum), sorted descending.
5. **Table:** `Rueckmeldung` with Auftrag, Material, Sollmenge, Gutmenge, Ausschuss, Status. Add conditional formatting (data bars) on Gutmenge.
6. **Slicers:** `Maschinen[Maschine]` and the date.

## 6. Refresh

After a new export, overwrite the same file and click **Refresh** in Power BI. With the file on OneDrive and the report published to the Power BI service, refresh can also be scheduled.

## How to explain it in an interview

"The app captures the data at the source, at the machine. Power BI only reads clean tables: one row per stop, one per quantity booking, one per machine and shift. OEE is calculated from sums, so a short shift doesn't weigh as much as a full one. In a real plant the Excel file would be replaced by a database or the MES, and the dashboard would stay the same."
