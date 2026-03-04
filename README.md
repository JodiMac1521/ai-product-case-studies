# AI Product Case Studies

I’ve made my product thinking public.

This repository documents applied product strategy, systems thinking, architecture judgment, instrumentation design, and real-world tradeoff decisions in scaling AI systems without breaking trust.

Each case study includes:
- The business constraint
- The architectural decision
- The tradeoffs considered
- The execution model
- The measurable outcomes

---

## Case Study: Building Capacity Through Better Scoping & AI Workflow

**Problem:** Revenue growth was constrained by a 3–5 day manual scoping bottleneck tied directly to headcount.

**Decision:** Build a human-in-the-loop AI Scoping Agent instead of hiring 3 additional operations staff.

**Outcome:**
- 83% reduction in intake time  
- 89% scope accuracy (up from 70%)  
- 40% increase in average project value  
- 100% operations team retention  

The system balanced automation efficiency with trust preservation — infrastructure over labor.

→ [Read the full case study](./case-studies/building-trust-through-better-scoping.md)

---

## System Architecture & Workflows

These visuals support the system design behind the case study:

- [Compounding Knowledge Loop](./diagrams/compounding-knowledge-loop.mmd)  
- Student Onboarding Workflow  
- AI Scoping System Architecture

---

# Case Study: AI-Assisted Market Sizing for a National EdTech Company

**Client:** U.S.-based EdTech company serving trade and technical education
**Scope:** National market sizing across electrical, HVAC, and manufacturing programs
**Deliverable:** Clean, structured database of 2,552 institutions with enrollment estimates
**Tools:** Claude (AI), Python, IPEDS, NCES CCD, Perkins V federal data

---

## The Challenge

An EdTech company serving career and technical education (CTE) needed to understand the full addressable market for their platform across three trades: **Electrical**, **HVAC**, and **Manufacturing/Mechatronics**.

Their existing customer list included 1,193 institutions — but they had no reliable way to size the total universe of potential customers or estimate enrollment at institutions they hadn't yet reached.

Key obstacles:
- Enrollment data is not publicly available in a single source
- High schools, colleges, and trade schools each require entirely different data pipelines
- 327 institutions lacked any enrollment data at all — flagged manually for verification
- 164 high schools were not found in standard federal databases (BOCES, regional consortiums, private programs)
- Data existed across multiple federal systems with no common key

---

## The Solution: A Multi-Source Data Pipeline

WorkSimplr built an automated matching and enrichment pipeline using public federal datasets, combining four data sources into one unified database.

---

## Workflow

**Step 1 — Define the Universe**
Pull all U.S. institutions offering electrical, HVAC, or manufacturing programs from IPEDS (Integrated Postsecondary Education Data System). Result: 2,552 institutions across colleges, trade schools, and high schools.

**Step 2 — Structure the Database**
Organize institutions into a 6-tab Excel workbook: Master (all trades), Customer List, Colleges, High Schools, Trade Schools, and an Overview narrative for leadership. Apply color-coded confidence tiers: Green = Exact match, Yellow = Medium confidence, Purple = District/Targeted, Orange = No match.

**Step 3 — Populate Colleges and Trade Schools via IPEDS EF2024A**
Join IPEDS Fall Enrollment file (EF2024A) to 327 "Verify" rows using UNITID as the primary key. Apply institution-type-specific program share percentages (e.g., vocational < 2-year = 35%, 2-year public = 5%, 4-year = 1–2%) to estimate program-level enrollment. Result: 326 of 327 resolved in a single automated pass.

**Step 4 — Build High School Pipeline via NCES CCD**
Download NCES Common Core of Data (CCD) 2024–25: school directory, characteristics, and membership enrollment files. Filter 2.35 GB membership file to 99,420 schools. Run 4-pass matching against 415 high schools.

**Step 5 — 4-Pass + Phase 2 Matching**
- Pass 1: Exact school name match
- Pass 2: Medium-confidence fuzzy match (≥0.85 SequenceMatcher score)
- Pass 3: District/LEA name matching with noise stripping
- Pass 4: Targeted lookups for BOCES, ISDs, and regional vocational consortiums
- Phase 2: Direct substring and state-filtered searches for remaining no-matches

**Step 6 — Apply Perkins V Enrollment Rates**
Apply federal Perkins V CTE concentrator rates to total school/district enrollment to estimate program-specific enrollment: Electrical ~2.4%, HVAC ~2.0%, Manufacturing ~1.6%.

**Step 7 — Clean, Verify, and Deliver**
Standardize terminology (no ambiguous labels), fix false-positive cross-state matches, verify all key statistics, update the Overview narrative, and confirm zero data inconsistencies across all tabs.

---

## Iterations

**v3.1 → v3.2**
Problem: Name matching was too strict, missing obvious variations.
Fix: Lowered fuzzy threshold from 0.92 to 0.85. Added name cleaning to strip institutional noise ("bookstore," "ISD," "school district," "unified," "parish"). Saved as multi-tab CSV and Excel.

**v3.2 → v4**
Problem: Spreadsheet had no context — data without narrative is hard to act on.
Fix: Added an Overview tab with a point-by-point narrative for leadership explaining the data, methodology, what's missing, and what comes next.

**v4 → v5 (Round 1)**
Problem: High school enrollment was missing for 415 institutions.
Fix: Built the full CCD 4-pass matching pipeline. Populated 367 of 415 (88%) high schools. Resolved 326 of 327 "Verify" rows using IPEDS EF2024A direct UNITID join.

**v5 → v5 Final (Round 2 — Phase 2)**
Problem: 29 high schools still unmatched; 19 matched schools had zero/suppressed enrollment; several Phase 2 matches had cross-state false positives.
Fix: Ran targeted direct-substring searches by state. Fixed false positives (e.g., "Robeson County NC" had matched to "Mason County WV"). Added matches for major districts: Tracy Joint Unified CA (13,770 enrolled), SD U-46 IL (34,036), Bend-LaPine OR (16,838), Providence Career Technical RI (759), Greater Lawrence Regional Vocational MA (1,837), Greater Lowell Regional Vocational MA (2,314), Provo City District UT (13,897), Rich Twp HSD 227 IL (2,433). Final result: 380 of 415 (92%) populated.

---

## Results

| Metric | Result |
|---|---|
| Total institutions sized | 2,552 |
| Master database populated | 2,551 / 2,552 (99.96%) |
| High schools populated | 380 / 415 (92%) |
| Colleges + trade schools | ~100% via IPEDS EF2024A |
| Remaining manual items | 1 (Poplar Bluff Technical Career Center) |
| True no-match high schools | 16 (categorized: private, admin entries, college-hosted programs) |
| Matching passes used | 5 (Exact, Medium, District, Targeted, Phase 2 Direct) |
| Data sources combined | IPEDS EF2024A, NCES CCD Directory, CCD Membership, Perkins V rates |
| Enrollment estimates generated | Program-level estimates for all populated rows |

---

## Key Technical Decisions

**Avoiding O(n²) timeout:** Initial fuzzy matching against 30,000 CCD schools × 415 high schools timed out. Solution: Used Python's `get_close_matches()` as a pre-filter (returning 3–5 candidates), then applied full SequenceMatcher only to those candidates — reducing runtime from hours to seconds.

**Bridging mismatched keys:** IPEDS UNITID was stored as float64 in one file and int64 in another. A three-step conversion (`str(int(float(x)))`) resolved all join failures.

**False positive prevention:** Broad name matching without state validation caused cross-state mismatches. Added a validation layer requiring state consistency between the source entry and the CCD match.

**Suppressed enrollment handling:** Career/technical centers often report enrollment rolled into their parent LEA rather than as standalone schools. These were escalated to LEA-level matching to retrieve district-wide enrollment for Perkins V rate application.

**Chunked file processing:** The 2.35 GB CCD membership file was read in 500,000-row chunks, filtered to `TOTAL_INDICATOR == 'Education Unit Total'`, and reduced to a 1.8 MB working file — making it manageable without exceeding memory limits.

---

## What This Enables

The finished database gives the EdTech company:

- **A complete view of the addressable market** — every accredited institution in the U.S. offering their target programs, with enrollment estimates
- **Prioritization by program size** — ranks institutions by estimated program enrollment for sales and outreach
- **Customer vs. prospect segmentation** — cross-referenced against their existing 1,193-customer list
- **A repeatable data foundation** — the pipeline can be re-run annually as IPEDS and CCD update

---

## About WorkSimplr

WorkSimplr builds AI-powered workflows for non-technical teams. We specialize in turning messy, multi-source data challenges into clean, actionable deliverables — no data engineering team required.

[worksimplr.com](https://worksimplr.com) · jodi@worksimplr.com

More case studies and workflow patterns will be added.
