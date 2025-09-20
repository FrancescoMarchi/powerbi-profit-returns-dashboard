![Live demo – 20s walkthrough](docs/gif/walkthrough.gif)

# Profit & Returns Performance Dashboard (Power BI)

**Purpose:** pinpoint where profit is created and where margin is eroded by returns—so actions are targeted.

- **Model:** clean star schema (FactOrders, Returns, DimDate ✓ marked, DimProduct, DimState, DimSegment)
- **Measures:** Adjusted Profit/Sales, Returned Profit/Sales, **Adjusted Margin %**, **Return Impact %** (Sales Returned % & Profit Erosion %)
- **Slicers:** Year → Quarter, Region
- **Drill path:** Segment → Category → **Sub-Category (Top-5 / Bottom-3)**
- **UX:** dynamic titles, **combo charts (Profit bars + Margin % line)**, and focused sub-category tooltips (Profit, Sales, Margin %, Return Impact %)

---

## Model (Star Schema)
![Star Schema](docs/model/star-schema.png)

---

## Screenshots
**Segment overview**  
![Segment overview](docs/screenshots/segment.png)

**Category breakdown**  
![Category breakdown](docs/screenshots/category.png)

**Sub-Category drilldown (Top-5 / Bottom-3 + Margin % line + tooltip)**  
![Sub-Category drilldown](docs/screenshots/subcategory.png)

**Tooltip example (hover on a Sub-Category)**  
![Tooltip example](docs/screenshots/tooltip.png)

---

## Highlights & Findings (example with default filters)
- **Winners (Top-5 Sub-Categories):** concentrated share of **Adjusted Profit** with healthy **Margin %**.
- **Laggards (Bottom-3):** thin or negative profit; **Return Impact %** and **Margin %** make the erosion drivers obvious.
- **Regional & seasonal context:** Year/Quarter and Region slicers localize actions.

**How to read it**
1) Set **Year/Quarter** and **Region**.  
2) Scan Segment → Category → Sub-Category.  
3) On Sub-Category, focus on **Bottom-3**; hover bars for the tooltip drivers.  
4) Use the **Margin % line** + **Return Impact %** to separate pricing/mix from returns pressure.

---

## Getting started
- Open `report/Profit-Returns-Dashboard.pbix` in Power BI Desktop  
  _or_ open `report/Profit-Returns-Dashboard.pbit` and point to your (demo) data.

---

## Repository
- `report/` – PBIX (report) and/or PBIT (template)  
- `docs/gif/` – walkthrough GIF  
- `docs/screenshots/` – page & tooltip screenshots  
- `docs/model/` – star-schema image  
- `dax/Measures.md` – key DAX

*Demo/sanitized data for portfolio purposes. License: MIT.*