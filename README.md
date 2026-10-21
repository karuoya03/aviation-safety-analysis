# Aviation Safety Analysis — Small vs. Large Aircraft

## Table of Contents
- [Business Problem](#business-problem)
- [Dataset](#dataset)
- [Methodology](#methodology)
- [Key Findings](#key-findings)
- [Recommendations](#recommendations)
- [How to Run](#how-to-run)
- [Repo Structure](#repo-structure)
- [Author](#author)

## Business Problem

An airline/airplane insurer wants to know which aircraft makes and models exhibit:
1. **Low rates of total destruction** in an accident, and
2. **Low likelihood of fatal or serious passenger injuries** in an accident.

They also want to understand what other incident conditions — weather, phase of flight,
etc. — correlate with severe outcomes, and they want **separate recommendations for
small vs. large passenger aircraft**.

## Dataset

- **Source:** NTSB Aviation Accident Database (1948–2023), available on
  [Kaggle](https://www.kaggle.com/datasets/khsamaha/aviation-accident-database-synopses).
- **Raw size:** 88,889 rows × 31 columns.
- **Filtered population:**
  - Airplanes only (`Aircraft.Category == "Airplane"`)
  - Professional builds only (`Amateur.Built == "No"`)
  - Events from **1983-01-01 onward** (40-year retirement assumption)
  - Makes with ≥ 50 events
  - Models present (not null)
- **Final cleaned size:** 17,917 rows.

### Derived variables
| Column | Definition |
|---|---|
| `Total.Aboard` | Fatal + Serious + Minor + Uninjured |
| `Fatal.Serious.Injuries` | Fatal + Serious |
| `Fatal.Serious.Ratio` | `Fatal.Serious.Injuries / Total.Aboard` |
| `Is.Destroyed.num` | 1 if `Aircraft.damage == "Destroyed"` else 0 |

## Methodology

1. **Cleaning** (`Aviation_Accidents_Cleaning.ipynb`)
   - Enforce airplane-only, professional-build-only, ≥1983 event dates.
   - Coerce injury columns to numeric, fill `NaN` with 0, clip at 0.
   - Standardize `Make` (dedupe aliases like `Beech Aircraft Corp → BEECH`).
   - Build `Plane.Type = Make + " " + Model`.
   - Drop columns with > 20,000 nulls unless they are analysis-critical.

2. **Analysis** (`Aviation_Accidents_Data_Analysis.ipynb`)
   - Split at **20 passengers aboard** → *small* (17,038 events) vs *large* (879 events).
   - Per-make and per-plane-type aggregations of
     mean fatal/serious injury fraction and destruction rate.
   - **Statistical robustness floor:** ≥30 events for makes, ≥10 events for plane types.
   - Distributional plots (violin / strip) to check that means weren't driven by outliers.

## Key Findings

### Small aircraft (under 20 aboard) — safest makes by injury fraction
| Make | n | Mean fatal/serious ratio | Destroyed rate |
|---|---:|---:|---:|
| McDonnell Douglas | 56 | 0.091 | 0.105 |
| Boeing | 714 | 0.142 | 0.067 |
| Aviat Aircraft Inc | 76 | 0.164 | 0.040 |
| Maule | 215 | 0.165 | 0.042 |
| Grumman (Schweizer) | 58 | 0.207 | 0.017 |

### Large aircraft (20+ aboard) — safest makes by injury fraction
| Make | n | Mean fatal/serious ratio | Destroyed rate |
|---|---:|---:|---:|
| McDonnell Douglas | 54 | 0.008 | 0.108 |
| Bombardier | 44 | 0.028 | 0.037 |
| Boeing | 547 | 0.057 | 0.097 |
| Embraer | 78 | 0.065 | 0.089 |
| Airbus | 118 | 0.086 | 0.170 |

### Safest large plane types (injury fraction)
`Boeing 777` (0.001), `Boeing 757` (0.001), `Boeing 787` (0.003),
`Boeing 737-7H4` (0.003), `Bombardier CL-600-2B19` (0.004).

### Factor 1 — Weather condition
- **IMC (instrument conditions):** 0.629 mean injury fraction, **36.9% destroyed** (n=907)
- **VMC (visual conditions):** 0.233 mean injury fraction, **7.3% destroyed** (n=14,317)
- **Conclusion:** accidents in IMC are roughly **2.7× deadlier** and **5× more likely
  to destroy the aircraft** than in VMC.

### Factor 2 — Broad phase of flight
| Phase | n | Mean injury ratio | Destroyed rate |
|---|---:|---:|---:|
| Maneuvering | 127 | 0.359 | 0.291 |
| Approach | 210 | 0.213 | 0.111 |
| Cruise | 239 | 0.186 | 0.137 |
| Takeoff | 426 | 0.111 | 0.108 |
| Landing | 1111 | 0.008 | 0.013 |
- **Conclusion:** **maneuvering** is the most dangerous phase (highest injury ratio *and*
  highest destruction rate). **Landing** is by far the safest, likely because most
  landing accidents are low-speed and low-altitude.

## Recommendations

**For small aircraft**, recommend makes with both a low injury fraction *and* a low
destruction rate: **Aviat Aircraft Inc, Maule, Grumman (Schweizer)** and, at the
plane-type level, the **Maule M-5-210C** and **Diamond DA 20 C1** (zero destroyed,
zero fatal/serious injuries across their samples).

**For large passenger aircraft**, recommend **Boeing** (especially the 777, 757, 787,
737-7H4), **Bombardier CL-600-2B19**, and **Embraer EMB-145**. These combine a
very low mean injury fraction (< 0.01) with a below-average destruction rate.

**Underwriting guidance:** price up premiums sharply for IMC operations and
maneuvering-phase exposure — these are the two variables with the strongest
association with total-loss outcomes in the dataset.

## How to Run

```bash
git clone https://github.com/<your-user>/<repo>.git
cd <repo>
pip install pandas numpy matplotlib seaborn jupyter

# Download AviationData.csv from Kaggle (link above) and place it at:
#   data/AviationData.csv

jupyter notebook
# Run notebooks/Aviation_Accidents_Cleaning.ipynb first
# Then notebooks/Aviation_Accidents_Data_Analysis.ipynb