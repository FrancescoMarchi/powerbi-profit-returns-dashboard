# Measures (Power BI – Profit & Returns Dashboard)

> Placement: put all measures in your **Measures** table (disconnected).  
> Formatting: set Currency (0 decimals) and Percent (1 decimal) in the model—avoid `FORMAT()` in business measures.

---

## 1) Base totals

```dax
Total Sales =
SUM ( FactOrders[Sales] )

Total Profit =
SUM ( FactOrders[Profit] )

Orders =
DISTINCTCOUNT ( FactOrders[Order ID] )

2) Returns
-- If Returns is related to FactOrders on Order ID (1:many), this TREATAS pattern still works safely.
Returned Sales =
CALCULATE ( [Total Sales], TREATAS ( VALUES ( Returns[Order ID] ), FactOrders[Order ID] ) )

Returned Profit =
CALCULATE ( [Total Profit], TREATAS ( VALUES ( Returns[Order ID] ), FactOrders[Order ID] ) )

Returned Orders =
DISTINCTCOUNT ( Returns[Order ID] )

If you have an active relationship Returns[Order ID] -> FactOrders[Order ID], you could also use:
Returned Sales = CALCULATE([Total Sales], KEEPFILTERS( VALUES(Returns[Order ID]) ))

3) Adjusted metrics (returns subtracted)
Adjusted Sales =
COALESCE ( [Total Sales], 0 ) - COALESCE ( [Returned Sales], 0 )

Adjusted Profit =
COALESCE ( [Total Profit], 0 ) - COALESCE ( [Returned Profit], 0 )

) Percent KPIs (viz-safe: never BLANK)
Adjusted Margin % (viz) =
DIVIDE ( [Adjusted Profit], [Adjusted Sales], 0 )

-- Return impact in sales: share of sales returned (0–100%)
Sales Returned % (viz) =
VAR d = [Total Sales]
RETURN IF ( d = 0, 0, MIN ( DIVIDE ( [Returned Sales], d ), 1 ) )

-- Return impact in profit: magnitude of profit erosion (0–100%)
Profit Erosion % (viz) =
VAR d = ABS ( [Total Profit] )
VAR n = ABS ( [Returned Profit] )
RETURN IF ( d = 0, 0, MIN ( DIVIDE ( n, d ), 1 ) )


Model formats

Adjusted Margin % (viz), Sales Returned % (viz), Profit Erosion % (viz) → Percent, 1 decimal

Adjusted Sales, Adjusted Profit, Total/Returned Sales/Profit → Currency, 0 decimals

5) Ranking (Sub-Category Top-5 / Bottom-3)
Rank SubCat by AdjProfit =
RANKX (
    ALLSELECTED ( DimProduct[Sub-Category] ),
    [Adjusted Profit],
    ,
    DESC,
    DENSE
)

Is Top5 SubCat =
IF ( [Rank SubCat by AdjProfit] <= 5, 1 )

Rank SubCat by AdjProfit (Asc) =
RANKX (
    ALLSELECTED ( DimProduct[Sub-Category] ),
    [Adjusted Profit],
    ,
    ASC,
    DENSE
)

Is Bottom3 SubCat =
IF ( [Rank SubCat by AdjProfit (Asc)] <= 3, 1 )


Alternative within-Category ranking:
replace ALLSELECTED(DimProduct[Sub-Category]) with
CALCULATETABLE( ALL ( DimProduct[Sub-Category] ), ALLEXCEPT ( DimProduct, DimProduct[Category] ) ).

6) Dynamic title (single/multi Year–Quarter, Region)

Requires a tiny Titles table with one column: Titles[Label]. Filter each visual to the correct row.

Title (Universal) =
VAR Base = SELECTEDVALUE ( Titles[Label], "Title" )

-- Year: single year, range, or All
VAR YrMin = CALCULATE ( MIN ( DimDate[Year] ) )
VAR YrMax = CALCULATE ( MAX ( DimDate[Year] ) )
VAR YearPart =
    IF (
        ISBLANK ( YrMin ),
        "All Years",
        IF ( YrMin = YrMax, FORMAT ( YrMin, "0" ), FORMAT ( YrMin, "0" ) & "-" & FORMAT ( YrMax, "0" ) )
    )

-- Quarter: works with QuarterNo or Quarter text
VAR QNoCount  = COUNTROWS ( VALUES ( DimDate[QuarterNo] ) )
VAR QTxtCount = COUNTROWS ( VALUES ( DimDate[Quarter] ) )
VAR QuarterPart =
    IF (
        QNoCount > 0,
            IF ( QNoCount = 1, "Q" & SELECTEDVALUE ( DimDate[QuarterNo] ), "Multiple Quarters" ),
        IF (
            QTxtCount > 0,
                IF ( QTxtCount = 1, SELECTEDVALUE ( DimDate[Quarter] ), "Multiple Quarters" ),
            "All Quarters"
        )
    )

-- Region: one, multiple, or all
VAR RCount = COUNTROWS ( VALUES ( DimState[Region] ) )
VAR RegionPart =
    IF ( RCount = 0, "All Regions", IF ( RCount = 1, SELECTEDVALUE ( DimState[Region] ), "Multiple Regions" ) )

RETURN
    Base & " — " & YearPart & " " & QuarterPart & " — " & RegionPart


Hook-up: Visual → Title → fx → Format by = Field value → Title (Universal).

7) Tooltip header (Sub-Category page)
Tooltip Label (SubCat) =
VAR sc = SELECTEDVALUE ( DimProduct[Sub-Category], "Sub-Category" )
VAR yr = SELECTEDVALUE ( DimDate[Year] )
VAR q  = SELECTEDVALUE ( DimDate[QuarterNo] )
VAR r  = SELECTEDVALUE ( DimState[Region] )
RETURN
    sc
    & IF ( NOT ISBLANK ( yr ), " - " & FORMAT ( yr, "0" ), "" )
    & IF ( NOT ISBLANK ( q ),  " Q" & q, "" )
    & IF ( NOT ISBLANK ( r ),  " - " & r, "" )


Tooltip page: add a Card (classic) bound to this measure + a Multi-row card with:
[Adjusted Profit], [Adjusted Sales], [Adjusted Margin % (viz)], [Sales Returned % (viz)], [Profit Erosion % (viz)], Returned Sales, Orders.
Attach to visuals: Visual → Tooltip → Report page → choose the tooltip.

8) Clarity subtitles (optional)
9a) When everything is 0% in the slice
No Returns Subtitle (Universal) =
VAR AnySeg =
    MAXX (
        VALUES ( DimSegment[Segment] ),
        COALESCE ( [Sales Returned % (viz)], 0 ) + COALESCE ( [Profit Erosion % (viz)], 0 )
    ) > 0
VAR AnyCat =
    MAXX (
        VALUES ( DimProduct[Category] ),
        COALESCE ( [Sales Returned % (viz)], 0 ) + COALESCE ( [Profit Erosion % (viz)], 0 )
    ) > 0
VAR AnySub =
    MAXX (
        VALUES ( DimProduct[Sub-Category] ),
        COALESCE ( [Sales Returned % (viz)], 0 ) + COALESCE ( [Profit Erosion % (viz)], 0 )
    ) > 0
RETURN
    IF ( AnySeg || AnyCat || AnySub, BLANK(), "No returns in this selection (0% across all items)" )

9b) Segment page — list which items are 0% (bars won’t draw)
Missing Bars Subtitle (Segment) =
VAR Sales0  =
    CONCATENATEX (
        FILTER ( VALUES ( DimSegment[Segment] ), COALESCE ( [Sales Returned % (viz)], 0 ) = 0 ),
        DimSegment[Segment], ", "
    )
VAR Profit0 =
    CONCATENATEX (
        FILTER ( VALUES ( DimSegment[Segment] ), COALESCE ( [Profit Erosion % (viz)], 0 ) = 0 ),
        DimSegment[Segment], ", "
    )
VAR TotalSeg =
    COUNTROWS ( VALUES ( DimSegment[Segment] ) )
VAR AllZero =
    COUNTROWS (
        FILTER (
            VALUES ( DimSegment[Segment] ),
            COALESCE ( [Sales Returned % (viz)], 0 ) = 0
                && COALESCE ( [Profit Erosion % (viz)], 0 ) = 0
        )
    ) = TotalSeg
VAR PartSales  = IF ( Sales0  <> "", "Sales Returned % = 0: "  & Sales0,  BLANK() )
VAR PartProfit = IF ( Profit0 <> "", "Profit Erosion % = 0: " & Profit0, BLANK() )
VAR Msg =
    IF (
        PartSales <> "" || PartProfit <> "",
        PartSales & IF ( PartSales <> "" && PartProfit <> "", "  |  ", "" ) & PartProfit,
        BLANK()
    )
RETURN IF ( AllZero, "No returns in this selection (0% across all segments)", Msg )


Hook-up: Visual → Subtitle → On → fx → Field value → pick one of the above.