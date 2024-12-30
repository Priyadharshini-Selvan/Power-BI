# Super Store Report

### Objective of the Project
The objective is to analyze this data and provide actionable, data-driven recommendations to guide Super Store.


### Steps Followed

**Step 1:** Load data into Power BI Desktop. The dataset is a CSV file.

**Step 2:** Open Power Query Editor. In the "View" tab under the "Data Preview" section, check "Column Distribution," "Column Quality," and "Column Profile" options.

**Step 3:** Since by default, profiling is only opened for 1,000 rows, select "Column Profiling Based on Entire Dataset."

**Step 4:** It was observed that no errors or empty values were present in the columns.

**Step 5:** Switch to the report view, select a background, and start adding visuals.

**Step 6:** Add three card visuals to represent:
- Products sold
- Products returned
- Best-selling products

**Step 7:** Add two line charts to represent total sales and total profit.

**Step 8:** Add visual filters (Slicers) for four fields: "State," "Year," "Month," and "Quarter."

**Step 9:** Add two donut charts representing:
- Sales amount by category
- Sales amount by segment

**Step 10:** Add a bar chart representing total sales by each sub-category.

**Step 11:** Add a ribbon chart representing region-wise sales.

**Step 12:** Add a shape map representing state-wise sales.

**Step 13:** Under the "Insert" tab, add a text box with the title "Super Store Sales Report."

**Step 14:** Use the "Shapes" option to insert a rectangle and add icons using the "Image" option.

**Step 15:** Create a new calculated table with the start and end date derived from the "Order Date" column in the orders table using the following DAX expression:

```DAX
DimDate =
    VAR s_date = YEAR(MIN(Orders[Order Date]))
    VAR e_date = YEAR(MAX(Orders[Order Date]))
    RETURN
    CALENDAR(DATE(s_date,1,1),DATE(e_date,12,31))
```
**Step 16:** Add new calculated columns to the "DimDate" table:

(a) Year:
```DAX
Year = YEAR(DimDate[Date])
```
(b) Month:
```DAX
Month = FORMAT(DimDate[Date],"mmm")
```
(c) Quarter:
```DAX
Quarter = "Qtr" & QUARTER(DimDate[Date])
```
(d) Fiscal Year:
```DAX
F_Year =
    VAR yr = YEAR(DimDate[Date])
    VAR mo = MONTH(DimDate[Date])
    RETURN
    IF(mo <= 9, yr - 1 & "-" & yr, yr & "-" & yr + 1)
```
(e) Month Number:
```DAX
MonthNo = MONTH(DimDate[Date])
```

**Step 17:** Create the following measures in a new "Metrics" table:

(a) **BEST SELLING PRODUCT:**
```DAX
BEST SELLING PRODUCT =
    FIRSTNONBLANK(
        TOPN(1, VALUES(Orders[Product Name]), CALCULATE(SUM(Orders[Quantity]))),
        1
    )
```
(b) **Total Sales:**
```DAX
Total Sales = SUM(Orders[Sales])
```
(c) **Total Profit:**
```DAX
Total Profit = SUM(Orders[Profit])
```
(d) **LY Sales:**
```DAX
Ly Sales =
    CALCULATE(
        [Total Sales],
        DATEADD(DimDate[Date], -1, YEAR)
    )
```
(e) **LY Profit:**
```DAX
Ly Profit =
    CALCULATE(
        [Total Profit],
        DATEADD(DimDate[Date], -1, YEAR)
    )
```
(f) **Products Sold:**
```DAX
Products Sold = SUM(Orders[Quantity])
```
(g) **Products Returned:**
```DAX
Products Returned =
    CALCULATE(COUNT('Return'[Order ID]), 'Return'[Returned] = "yes")
```
(h) **YOY Growth:**
```DAX
YOY Growth = DIVIDE([Total Sales] - [Ly Sales], [Ly Sales])
```
(i) **YOY Growth_P:**
```DAX
YOY Growth_P = DIVIDE([Total Profit] - [Ly Profit], [Ly Profit], 0)
```
(j) **Sales Main Title:**
```DAX
Sales Main Title =
    "TOTAL SALES" & REPT(UNICHAR(10), 2) & REPT(" ", 30) &
    FORMAT([YOY Growth], "+0.0%;-0.0%") & "   ▲ LY  " &
    IF([YOY Growth] > 0, "🟢", "🔴")
```
(k) **Sales Sub Title:**
```DAX
Sales Sub Title = FORMAT([Total Sales], "#,##0,.0 K")
```
(l) **Profit Main Title:**
```DAX
Profit Main Title =
    "TOTAL PROFIT" & REPT(UNICHAR(10), 2) & REPT(" ", 25) &
    FORMAT([YOY Growth_P], "+0.0%;-0.0%") & "   ▲ LY  " &
    IF([YOY Growth_P] >= 0, "🟩", "🔵")
```
(m) **Profit Sub Title:**
```DAX
Profit Sub Title = FORMAT([Total Profit], "#,##0,.0 K")
```
(n) **Sub-Cat Rank CY:**
```DAX
Sub-Cat Rank Cy = RANKX(ALL(Orders[Sub-Category]), [Total Sales])
```
(o) **Sub-Cat Rank PY:**
```DAX
Sub-Cat Rank Py = RANKX(ALL(Orders[Sub-Category]), [Ly Sales])
```
(p) **Ranking Label:**
```DAX
Ranking Label = "#" & [Sub-Cat Rank Cy]
```
(q) **Ranking Development Label:**
```DAX
Ranking Development Label =
    VAR _SubCatRankChange = [Sub-Cat Rank Cy] - [Sub-Cat Rank Py]
    VAR Label =
        SWITCH(
            TRUE,
            _SubCatRankChange = 0, "#" & [Sub-Cat Rank Cy] & " —",
            _SubCatRankChange > 0, "#" & [Sub-Cat Rank Cy] & " ▼" & _SubCatRankChange,
            _SubCatRankChange < 0, "#" & [Sub-Cat Rank Cy] & " ▲" & ABS(_SubCatRankChange)
        )
    RETURN
    Label
```
(r) **Ranking Development Color:**
```DAX
Ranking Development Color =
    VAR _SubCatRankChange = [Sub-Cat Rank Cy] - [Sub-Cat Rank Py]
    VAR Color =
        SWITCH(
            TRUE,
            _SubCatRankChange = 0, "black",
            _SubCatRankChange > 0, "red",
            _SubCatRankChange < 0, "green"
        )
    RETURN
    Color
```

---

### Report Publishing
The report was then published to Power BI Service.

![Alt Text](C:\Users\PriyaSelvan\Pictures\Screenshots\superstore)


### Key Points

- **Total Number of Products Sold:** 38k
- **Number of Products Returned:** 296
- **Total Sales:** 2,297.2k
- **Total Profit:** 286.4k

#### Recommendations

1. **Focus on High-Performing Products:**
   - Insight: Phones and Chairs are top-performing sub-categories.
   - Recommendation: Explore opportunities to expand these categories, enhance marketing, and consider bundling.

2. **Address Underperforming Categories:**
   - Insight: Appliances and Bookcases are lagging in sales.
   - Recommendation: Conduct market research to improve offerings or reallocate resources.

3. **Regional Strategy Optimization:**
   - Insight: Sales vary significantly across states and regions.
   - Recommendation: Tailor marketing campaigns and optimize inventory distribution.

4. **Enhance Customer Segmentation:**
   - Insight: Sales contributions reveal opportunities to realign marketing efforts.
   - Recommendation: Implement targeted campaigns and loyalty programs.

5. **Improve Profit Margins:**
   - Insight: Profit margins are under pressure.
   - Recommendation: Adjust pricing strategies and reduce unnecessary costs.

6. **Reduce Product Returns:**
   - Insight: 296 products were returned.
   - Recommendation: Address quality issues, improve product descriptions, and refine return policies.

7. **Leverage Best-Selling Products for Cross-Selling:**
   - Insight: Staples are the best-selling product.
   - Recommendation: Develop cross-selling strategies to boost profitability.


