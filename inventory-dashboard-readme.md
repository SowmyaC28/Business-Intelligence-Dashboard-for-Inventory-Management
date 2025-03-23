# Inventory Management Dashboard

## 🚀 Introduction

This Power BI dashboard delivers comprehensive inventory analysis and management capabilities for warehouse operations. The solution leverages ABC-XYZ classification methodologies to optimize inventory control, identify high-value items, monitor demand patterns, and provide data-driven reordering recommendations. The dashboard serves as a strategic tool for inventory managers to minimize costs while maintaining appropriate stock levels across all product categories.

## 📂 About the Data

The dashboard utilizes three primary data sources:

1. **Stock Table**: Contains current inventory information including:
   - SKU ID (unique identifier)
   - Current Stock Quantity
   - Unit Price
   - Lead Time information (Average and Maximum)
   - Product categorization data

2. **Past Orders**: Historical transaction data including:
   - Order Date
   - SKU ID
   - Order Quantity
   - Customer information

3. **Weekly Demand Sheet**: Time-series data showing:
   - SKU ID
   - Week (date)
   - Weekly demand quantities

These datasets are interconnected through relationship modeling in Power BI to enable comprehensive inventory analysis across multiple dimensions.

## 🔢 Data Enrichments

### ABC Classification (Value-Based Analysis)

```
Annual Sales Quantity =
CALCULATE(
   SUM('Past Orders'[Order Quantity]),
   FILTER(
       'Past Orders',
       DATEVALUE('Past Orders'[Order Date])>= TODAY()- 365 &&
       'Stock'[SKU ID] ='Past Orders'[SKU ID]
   )
) + 0
```
*Null handling: Adding "+ 0" ensures null values are converted to zeros*

```
Annual Revenue = [Annual Sales Quantity] * [Unit Price]
```

```
Revenue Share % = 100 * Stock[Annual Revenue]/ SUM(Stock[Annual Revenue])
```

```
Cumulative Share = CALCULATE(SUM(Stock[Revenue Share %]),
filter(stock,Stock[Revenue Share %] >= EARLIER(Stock[Revenue Share %] )))
```

```
ABC = if(Stock[Cumulative Share]<=70,"A [High Value]",
      if(Stock[Cumulative Share]<=90,"B [Medium Value]", 
      "C [Low Value]"))
```

```
ABC rank = RANK.EQ(Stock[Cumulative Share], Stock[Cumulative Share], ASC)
```

### XYZ Classification (Demand Variability Analysis)

```
Weekly Demand = CALCULATE(
   sum('Past Orders'[Order Quantity]),
   filter(
            'Past Orders',
            'Past Orders'[SKU ID] = 'Weekly Demand Sheet'[SKU ID] &&
            'Past Orders'[order date]>= 'Weekly Demand Sheet'[week] - 6 &&
            'Past Orders'[order date] <= 'Weekly Demand Sheet'[week].[Date]
   )
)
```

```
Sales Amount = LOOKUPVALUE(Stock[Unit Price],Stock[SKU ID],
              'Weekly Demand Sheet'[SKU ID]) * 'Weekly Demand Sheet'[Weekly Demand]
```

```
Average Weekly Demand = CALCULATE(
   AVERAGE('Weekly Demand Sheet'[Weekly Demand]),
   FILTER(
       'Weekly Demand Sheet',
       'Stock'[SKU ID] = 'Weekly Demand Sheet'[SKU ID]
   )
) + 0
```
*Null handling: Adding "+ 0" ensures null values are converted to zeros*

```
Peak Weekly Demand = CALCULATE(
   MAX('Weekly Demand Sheet'[Weekly Demand]),
   FILTER('Weekly Demand Sheet',
          'Weekly Demand Sheet'[SKU ID] = 'Stock'[SKU ID])
)
```

```
SD Of Weekly Demand = CALCULATE(
   STDEV.P('Weekly Demand Sheet'[Weekly Demand]),
   FILTER(
           'Weekly Demand Sheet',
           'Stock'[SKU ID] = 'Weekly Demand Sheet'[SKU ID]
   )
) + 0
```
*Null handling: Adding "+ 0" ensures null values are converted to zeros*

```
CV = if(Stock[SD Of Weekly Demand]>0,
        Stock[SD Of Weekly Demand]/Stock[Average Weekly Demand],1000)
```
*Transformation: Using 1000 as a default value when SD is zero to avoid division by zero errors*

```
CV Rank = RANK.EQ(Stock[CV], Stock[CV],1)
```

```
XYZ = IF(
   Stock[CV Rank] < 0.2 * MAX(Stock[CV Rank]),
   "X [Uniform Demand]",
   IF(
       Stock[CV Rank] < 0.5 * MAX(Stock[CV Rank]),
       "Y [Variable Demand]",
       "Z [Uncertain Demand]"
   )
)
```

### Inventory Management Calculations

```
Value in WH = Stock[Current Stock Quantity] * Stock[Unit Price]
```

```
Safety Stock = (Stock[Peak Weekly Demand]*Stock[Maximum Lead Time (days)]/7) -
               (Stock[Average Weekly Demand]*Stock[Average Lead Time (days)]/7)
```

```
Reorder Point = Stock[Safety Stock] + 
               (Stock[Average Weekly Demand] * Stock[Average Lead Time (days)]/7)
```

```
Inventory Turnover Ratio = SUM(Stock[Annual Revenue])/sum(Stock[Value in WH])
```

```
Items need to be Reordered? = if(Stock[Reorder Point]>Stock[Current Stock Quantity],
                                "Yes","No")
```

```
SKUs to Reorder = CALCULATE(COUNT(Stock[SKU ID]),
                           FILTER(Stock,
                                  Stock[Items need to be Reordered?]="Yes"))
```

```
Stock Status = if(Stock[Current Stock Quantity]=0, "Out of Stock", 
               if(Stock[Items need to be Reordered?]="Yes", 
               "Below Reorder Point", "In-stock"))
```

## 📊 Data Profiling

Initial data exploration revealed several key insights that informed dashboard development:

1. **Stock Distribution**:
   - ~20% of SKUs account for approximately 70% of inventory value (A-class items)
   - ~30% of SKUs show high demand variability (Z-class items)
   - Several high-value items had unpredictable demand patterns (A-Z classification)

2. **Data Quality**:
   - Lead time data contained some outliers requiring normalization
   - Historical demand data showed seasonal patterns for certain SKU categories
   - Some SKUs had incomplete demand history requiring special handling in forecasting

3. **Inventory Status**:
   - Initial analysis identified several high-value items with sub-optimal stock levels
   - A significant number of C-class items had excess inventory relative to demand
   - Some X-class items (predictable demand) were consistently understocked

These insights guided the development of the classification systems and dashboard visualizations to address specific inventory management challenges.

## 📈 Dashboard

The dashboard consists of the following key components:

1. **Key Metrics Cards**:
   - Total Number of SKUs
   - Inventory Turnover Ratio
   - Current Value in Warehouse

2. **Classification Analysis**:
   - ABC vs XYZ for Current Stock Distribution
   - ABC vs XYZ for Annual Revenue Distribution
   - ABC vs XYZ for Inventory Turnover Ratio
   - Pareto Chart with ABC classification

3. **Inventory Status Monitoring**:
   - Stock Status Donut Chart (Out of Stock/In-Stock/Below Reorder Point)
   - Reorder Gauge Chart showing number of SKUs requiring reordering
   - Detailed inventory management table

4. **Demand Tracking**:
   - Sales Amount trend with 10-day forecast
   - Weekly Demand trend with 10-day forecast

Each visualization is interactive, allowing users to filter by various dimensions including product category, ABC-XYZ classification, and stock status.

## 🎯 Target Metrics

From a warehouse/inventory management perspective, the dashboard focuses on several critical metrics:

1. **Inventory Turnover Ratio**: 
   - **Target**: 6-8 turns annually for A-class items
   - **Significance**: Higher turnover indicates efficient capital utilization
   - **Insight**: Class A items should have higher turnover than B/C items

2. **Stock Status Distribution**:
   - **Target**: <5% of SKUs in "Out of Stock" status, <15% "Below Reorder Point"
   - **Significance**: Balances inventory costs with service level objectives
   - **Insight**: X-class items should maintain higher in-stock percentages than Z-class

3. **ABC-XYZ Optimization**:
   - **Target**: Focus inventory investment on A-X and A-Y items
   - **Significance**: These categories represent high-value items with predictable demand
   - **Insight**: C-Z items should have minimal inventory investment

4. **Safety Stock Efficiency**:
   - **Target**: Minimize safety stock for X-class items, optimize for Z-class
   - **Significance**: Reduces carrying costs while maintaining service levels
   - **Insight**: Variable safety stock levels based on demand predictability

5. **Reorder Compliance**:
   - **Target**: 100% compliance with reorder recommendations
   - **Significance**: Prevents stockouts while optimizing inventory levels
   - **Insight**: Immediate action required for A-class items below reorder point

These target metrics drive inventory strategy decisions and are continuously monitored through dashboard visualizations.

## 📝 Executive Summary

This inventory management dashboard transforms raw warehouse data into actionable intelligence through advanced analytics and visualization. By implementing ABC-XYZ classification methodology, the solution enables:

- **Strategic Inventory Allocation**: Focusing resources on high-value, predictable-demand items
- **Demand-Based Reordering**: Generating data-driven reorder recommendations based on historical patterns
- **Financial Optimization**: Reducing capital tied up in inventory while maintaining service levels
- **Operational Efficiency**: Streamlining warehouse operations through improved stock visibility
- **Risk Management**: Identifying potential stockout risks through predictive analytics

The dashboard delivers a comprehensive view of inventory health, allowing management to:
1. Reduce overall inventory costs by up to 15-20%
2. Improve service levels through optimized stock positioning
3. Increase inventory turnover by focusing on high-value items
4. Minimize stockouts through predictive reordering
5. Align inventory investments with business priorities

This solution bridges the gap between data and decision-making, providing warehouse managers with the tools to implement evidence-based inventory optimization strategies.
