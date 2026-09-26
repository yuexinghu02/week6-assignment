# USCG Search and Rescue Analysis Skill

A Claude skill (`uscg-sar-analysis`) that downloads, ranks and visualizes U.S. Coast Guard search and rescue (SAR) data, and builds an interactive drill-down HTML dashboard that someone outside the SAR field can understand.

The skill is packaged as [`week6-assignment.skill`](week6-assignment.skill), a zip archive you can install in Claude.

## What it's about

Each year the DHS Office of Homeland Security Statistics (OHSS) publishes a workbook called [USCG Search and Rescue Responses](https://ohss.dhs.gov/khsm/uscg-search-and-rescue-responses), built from the Coast Guard's MISLE case system. It lists SAR case counts and outcomes (lives saved, lost or unaccounted for, and property saved or lost) by fiscal year, month, Area, District and **Captain of the Port (COTP) zone**. The first release covers FY2020–2024 and has 35 COTP zones in 9 Districts.

The skill turns that workbook into answers to plain questions, such as:

- Which Coast Guard zones handle the most rescues?
- Where are the most lives on the line?
- Which zones have the biggest summer surge?
- Where are calls for help increasing?
- Where is the share of people saved lowest?

Claude uses it whenever someone asks about Coast Guard SAR cases, lives saved or lost, busy seasons, or where SAR demand is concentrated, even if they don't name the DHS site or say "SAR".

## What it accomplishes

1. **Gets the latest data.** It finds the current `.xlsx` on the DHS page (the file name changes with each release). If that fails, it uses the December 2024 file.
2. **Builds a clean dataset.** It reads years and zones from the file, so a new fiscal year needs no code changes. It also fixes a known typo in the source ("Saulte Ste. Marie").
3. **Ranks zones, districts or areas** on five measures:

   | Measure | Meaning |
   |---|---|
   | Cases a year | Average SAR cases (coordinated responses to people or property in distress) |
   | Lives on the line a year | People saved + lost + unaccounted for |
   | Busy season vs normal month | Busiest month's average cases ÷ the zone's monthly average |
   | Change per year | Least-squares trend in annual cases, as a percent of the zone's average |
   | Share saved | Lives saved ÷ (saved + lost after notification or arrival + unaccounted) |

   You can filter by year, district or area, and export results to CSV.
4. **Builds a self-contained dashboard** in one HTML file with the data inlined:
   - Five plain questions, each with a top-10 ranking, the real number in words (for example, "Jul is 3.1× a normal month") and a one-sentence "why it matters"
   - A "zones that keep coming up" panel that counts how many top-10 lists each zone appears on
   - Region, District, COTP zone and Fiscal year filters that stay in sync with clickable bars and a breadcrumb
   - Light and dark themes and a responsive layout
   - An original life-ring logo; it deliberately doesn't use the official Coast Guard seal
5. **Explains the numbers honestly.** It points out the limits of the data (see below) so results aren't overstated.

### Sample findings (FY2020–2024)

- National SAR cases fell every year, from 16,830 to 14,280.
- The busiest zones were San Francisco (7,530 cases), Miami (4,850), Buffalo (4,560), Columbia River (4,390) and Lake Michigan (3,400).
- Great Lakes zones (Sault Ste. Marie, Lake Michigan, Buffalo) have the sharpest seasonal surge, about 3× a normal month in July.
- Los Angeles-Long Beach has the fastest rising case count, about +25% a year.

## Data caveats

- **The COTP zone is the smallest unit.** The data has no breakdown by station, cutter, aircraft or staffing.
- **DHS rounds every row to the nearest 10**, so totals summed from rows differ slightly from DHS's published totals.
- **Small zones are unreliable for rates.** Zones under 50 people a year are left out of "share saved", and zones under 50 cases a year are left out of "busy season".
- **It shows demand, not resourcing.** A high ranking doesn't mean a zone is understaffed.
- Fiscal years run October–September.

## What's inside the package

```
uscg-sar-analysis/
├── SKILL.md                      # Instructions Claude follows: workflow, definitions, design rules
├── references/data-notes.md      # Workbook sheets, zone hierarchy, caveats, reference results
├── assets/dashboard_template.html
└── scripts/
    ├── fetch_data.py             # Download the latest DHS workbook
    ├── build_data.py             # Workbook -> data.json (needs pandas)
    ├── rank_zones.py             # Rankings as a table or CSV
    └── build_dashboard.py        # data.json -> self-contained HTML dashboard
```

## Using it

**In Claude:** install `week6-assignment.skill` and ask something like "Which Coast Guard zones handle the most rescues?" or "Build me a dashboard of Coast Guard search and rescue data."

**From the command line** (Python 3; `build_data.py` needs `pandas` and `openpyxl`), after unzipping the `.skill` file:

```bash
python scripts/fetch_data.py --out sar.xlsx
python scripts/build_data.py sar.xlsx data.json
python scripts/rank_zones.py data.json --metric surge --area Atlantic
python scripts/build_dashboard.py data.json uscg-sar-dashboard.html
```

`rank_zones.py` options: `--by zone|district|area`, `--metric vol|lives|surge|trend|save`, `--year`, `--district`, `--area`, `--csv`.

## Data source

U.S. Department of Homeland Security, Office of Homeland Security Statistics, *Key Homeland Security Metrics: USCG Search and Rescue Responses*. This is an unofficial analysis and isn't affiliated with or endorsed by the U.S. Coast Guard or DHS.
