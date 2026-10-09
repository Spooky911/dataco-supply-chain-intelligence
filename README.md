# DataCo Supply Chain Intelligence

This project started with a CSV and a practical question: what can the data tell us about sales, profitability and the reliability of order delivery?

We built a seven-page Power BI report to explore those questions, then created a separate 27-page PDF for readers who need something they can print or share. Both use the same source data, with layouts suited to how people will use them.

## Where the project stands

[Open the Power BI Service report](https://app.powerbi.com/groups/me/reports/174da804-c5fc-4a33-9e25-aa6b3cb25593/Executive) · [GitHub repository](https://github.com/Spooky911/dataco-supply-chain-intelligence)

The owner has chosen to make the report, repository and dataset public. Readers will be able to access the data through the CSV and PBIX. Power BI **Publish to web** can also expose underlying model data, even when a visual does not display it. See [Microsoft's publication guidance](https://learn.microsoft.com/en-us/power-bi/collaborate-share/service-publish-to-web).

## What we wanted to understand

1. How much business was booked, and how much recorded benefit did it generate?
2. Which markets, categories and products drive value and losses?
3. How reliably do shipping modes meet their scheduled duration?
4. Where are orders sent, and how does destination performance compare?
5. Which customer segments and individual orders merit attention?

## The data we worked with

The supplied file is `DataCoSupplyChainDataset.csv`. Its original local location was `E:\Analysis project supply chain\raw data`. The analysis covers 1 January 2015 through 31 January 2018, with 180,519 order-item records, 65,752 distinct orders, 20,652 customers and 384,079 units.

We worked from the supplied file. Its upstream attribution and redistribution license have not been independently verified, so this project does not claim a license over the original data.

The source does not document a currency, so amounts are shown in source units. Sales include all order statuses unless filtered, and recorded profit reflects only the costs captured in the dataset. These figures should not be read as audited accounting results.

## How we built it

### 1. Start with the data

We read the complete CSV with pandas and inspected dimensions, dates, statuses, missing values, identifiers and numeric fields. The first important finding was that each record represents an order item, not an entire order. We checked unique item identifiers and consistency of shipping, destination and status fields within each order.

### 2. Sort out the duplicate columns

We found that `Sales per customer` duplicates `Order Item Total`, and `Order Profit Per Order` duplicates `Benefit per order`. We kept one field for each amount so totals do not double-count duplicate columns. We summed item amounts for financial totals, but counted distinct orders for order KPIs.

### 3. Define the KPIs

We used `Sales` for gross sales, `Order Item Total` for net booked sales, `Benefit per order` for recorded profit and `Order Item Discount` for discount value. We calculated margin as total profit divided by total net sales and weighted discount as total discount divided by gross sales. This gives each transaction the right weight; averaging row-level percentages would produce a different answer.

We counted late delivery once per order and excluded `Shipping canceled` from the eligible delivery denominator. We added up all item profit within an order before classifying that order as loss-making; an unprofitable item does not necessarily make its whole basket unprofitable.

### 4. Check dates and destinations

We parsed source dates using US conventions and the CSV using Windows-1252 encoding. We retained order and shipping timestamps. We did not create an arrival timestamp: the supplied shipping timestamp is not verified arrival information. We compared timestamp elapsed duration with recorded actual shipping days; 9,737 items, approximately 5.4%, differ. We used the recorded duration fields for delivery comparisons and documented this discrepancy.

We used order region, country, state and city for destination analysis. The store/customer coordinates did not establish delivery locations, so we excluded them from destination mapping. We grouped cities with country and state to avoid merging identically named locations.

### 5. Prepare the data in Power Query

We selected analytical fields, gave them consistent names, assigned explicit types and removed personal names, street addresses, coordinates and duplicate financial fields from the analytical model. We added order date, shipping date, positive delay days, destination label, order month and order year. We kept the original CSV separately as the source.

The import definition is supplied in `report/Power Query.m`. The existing source path must be updated before refreshing on another computer.

### 6. Build the model and measures

We created an imported fact table and date table, and defined 37 explicit DAX measures. Measures cover sales, profit, margin, discounts, order volume, customer activity, losses, delivery rates, durations and time comparisons. We used distinct-order calculations and order-level iterators where an item-weighted average would distort interpretation. The full definitions are supplied in `report/Measures.dax`.

### 7. Design the interactive report

| Page | Purpose |
|---|---|
| Executive overview | Headline business performance and time trends |
| Commercial performance | Sales, discounts, recorded profitability and loss exposure |
| Delivery performance | Shipping modes, late delivery and scheduled versus actual duration |
| Markets and destinations | Market, region, country and city performance |
| Customers and products | Segment behavior and product/category portfolios |
| Order explorer | Detailed order-level investigation |
| Business insights | Management interpretation, priorities and limitations |

We used native interactive charts, matrices, slicers, scatterplots and a treemap, with consistent colors, labels and financial formatting. Monthly charts follow chronological order. Chart selections cross-filter or highlight related visuals; matrices support deeper geographic exploration.

### 8. Check the report in Desktop

We validated all 116 PBIR definition files against their referenced Microsoft JSON schemas. We opened the source project in Power BI Desktop and successfully refreshed it from the CSV. We compared the displayed headline metrics with independently calculated pandas totals. We inspected every final page and selected the Consumer segment to check that the related KPIs and visuals responded. Clearing the selection restored the full totals.

An initial project save stalled after a model-format prompt. We successfully saved a fresh final copy as `DataCo.pbix`, checked its ZIP integrity and loaded model, then closed and reopened it from disk to confirm its seven pages and KPI values persist without requiring refresh. The editable PBIP source is included alongside the PBIX.

### 9. Explain what the findings mean

We wrote a separate business insight brief connecting the results to actions management could take. We kept the distinction between evidence and explanation: the data can show associations, but does not prove that discounts or delays caused losses. We also separated complete-period comparisons from the partial 2018 coverage.

### 10. Make the PDF readable

The interactive report's ordinary PDF export clipped scrolling visuals and could not preserve slicer interaction. We built a separate fixed-layout report from the complete dataset using ReportLab vector elements. We made the charts fit on the page, stated the fixed data scope and split long tables across pages with repeated headings.

The 27-page PDF contains business analysis and complete appendices for 164 countries, 50 categories, 118 products and all 37 months. Its order review is explicitly a ten-order loss sample. The companion CSVs contain all 65,752 orders and 3,772 destination city groups for readers who need the complete detail.

### 11. Check every printed page

We rendered and visually inspected all 27 pages for clipping, overlap and legibility. We checked the appendix row ranges and confirmed that the final month was included. We verified that the full order CSV contains 65,752 unique order identifiers and that its sales and profit reconcile to source totals. We packaged the PDF and companion tables together.

### 12. Publish the project

We signed in to Power BI Service and uploaded the verified PBIX to My workspace. The service created both the report and its semantic model. Opening the executive page confirmed 33.05M net booked sales, 3.97M recorded profit, 12.0% margin and a 57.3% late-delivery rate, matching the Desktop results. All seven pages are listed in the service.

We then tried File → Embed report → Publish to web (public). Power BI returned: “Contact your admin to enable embed code creation.” Public access remains blocked until the university tenant administrator permits new public embed codes. The service link above is therefore an authenticated report link, not an anonymous public link.

We created the public GitHub repository under Spooky911. A command-line push failed because the local Git credential helper could not run, so we used the signed-in browser to upload the deliverables. The 92.3 MB source CSV exceeds GitHub's browser upload limit; we compressed it into an 18.2 MB ZIP. The editable project and supporting exports are in a separate archive. The PBIX, PDF and README are directly accessible at the repository root.

The owner chose public access to the dataset as well as the report. Enabling anonymous Power BI viewing is the remaining publication step and requires the tenant administrator; it cannot be completed from the report owner's current controls.

## What the results show

| Metric | Full-dataset result |
|---|---:|
| Gross sales | 36,784,735.01 |
| Net booked sales | 33,054,402.38 |
| Recorded profit | 3,966,902.97 |
| Recorded margin | 12.0% |
| Eligible delivery orders | 62,897 |
| Late orders | 36,048 |
| Late-delivery rate | 57.3% |
| Loss-making whole orders | 13,909 |

Europe and LATAM together contribute approximately 57.5% of net booked sales. First Class records a 100% late rate under the source definitions and warrants investigation of service promises and data definitions. January 2018 has more orders than January 2017 but substantially lower booked sales and average order value; compare equivalent months rather than treating January 2018 as a complete year.

## Files

- `DataCo.pbix`: the loaded seven-page Desktop report.
- `DataCo Print Report.pdf`: the 27-page print edition.
- `DataCo-Source-Data.zip`: the original CSV, under `data/raw/` after extraction.
- `DataCo-Editable-Project.zip`: the PBIP source, DAX, Power Query, verification records, insight brief, scripts and complete analytical exports.
- `README.md`: the project walkthrough you are reading.

Extract the project archive to preserve its `report/`, `docs/`, `scripts/` and `data/derived/` folders. The scripts document how the project was built and retain local paths and dependencies that need adjustment on another machine.

## Open and refresh

Open `DataCo.pbix` in Power BI Desktop. It contains the loaded model. For a new refresh, update the CSV path in the Fact query's Advanced Editor to your local source location, preserving its encoding and date locale. Keep `DataCo.pbip`, `DataCo.Report` and `DataCo.SemanticModel` adjacent when editing the source project.

The print PDF and CSV exports are historical snapshots. Refreshing Desktop does not automatically regenerate them. Scheduled cloud refresh has not been configured; a local file source requires an appropriate service refresh arrangement before a schedule can work.

## Things to keep in mind

- No verified arrival timestamp or documented currency.
- Recorded benefit includes only costs represented by the source.
- Financial totals include all statuses unless filters explicitly restrict them.
- Source geography labels are retained; store/customer coordinates are not destination coordinates.
- Source durations and elapsed timestamps do not always agree.
- 2018 coverage contains January only.
- Public report, PBIX and CSV publication permits data access; it is not a private reporting solution.

## References

- [Power BI project report format](https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-report)
- [PDF export behavior](https://learn.microsoft.com/en-us/power-bi/collaborate-share/end-user-pdf)
- [Publish to web](https://learn.microsoft.com/en-us/power-bi/collaborate-share/service-publish-to-web)
- [Paginated reports for future print automation](https://learn.microsoft.com/en-us/power-bi/paginated-reports/paginated-reports-report-builder-power-bi)

