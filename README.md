# Power BI Project

### **Overview**
This is my first portfolio project, aimed at showcasing my proficiency in Power BI.

The project is inspired by a portfolio showcased by Mo Chen and his wife on YouTube. You can view [the original video](https://www.youtube.com/watch?v=BLxW9ZSuuVI&t=975s). 

**Dataset:** [Plant_DTS.xls](https://github.com/mochen862/power-bi-portfolio-project/blob/main/Plant_DTS.xls)

---

### **Project Steps**

#### **1. Importing Data into Power BI**
- Downloaded the dataset from the provided link and used the *Get Data* option in Power BI to import the Excel Workbook.

#### **2. Data Transformation in Power Query**
- Loaded necessary tables for analysis: `Accounts`, `plant_FACTS`, and `plant_HIERARCHY`.
- Applied the following transformations:
  - **Renamed tables** for clarity and consistency:
    - `Accounts` to `dim_accounts`
    - `plant_HIERARCHY` to `dim_product`
    - `plant_FACTS` to `fact_sales`
  - **Removed duplicates** in `product_name_id` (primary key in `dim_product`) and `Account_id` (identifier in `dim_accounts`).
  - Cleaned `latitude` and `country_name` fields by removing unnecessary characters.

After transformations, the clean data was loaded into the Power BI workspace.

---

#### **3. Creating Virtual Tables**
1. **Date Table**  
   - Created using the following DAX expression:
     ```dax
     dim_Date = CALENDAR(DATE(2022, 1, 1), DATE(2024, 12, 31))
     ```
   - The Date Table, combined with Power BI’s time intelligence, generated a date hierarchy with Year, Quarter, Month, and Day levels for easy time-based filtering.

2. **Flag Column in Date Table**  
   - Created a calculated column in `dim_date` to filter data to only the last 12 months. This DAX expression creates a Boolean flag:
     ```dax
     inpast = 
     VAR lastsalesdate = MAX(fact_Sales[Date_Time])
     VAR lastsalesdatePY = EDATE(lastsalesdate, -12)
     RETURN dim_Date[Date] <= lastsalesdatePY
     ```

3. **Slicer Table**  
   - Created a slicer table for switch values by using *Enter Data*. The table, `slc_Values`, contains a single column, `Values`, populated with `Sales`, `Gross Profit`, and `Quantity`.

---

#### **4. Building Measures**
- **Base Measures**  
   Created the base measures for Sales, Quantity, and Gross Profit:
   ```dax
   Sales = SUM(fact_Sales[Sales_USD])
   Quantity = SUM(fact_Sales[Quantity])
   COGS = SUM(fact_Sales[Cost_of_goods_sold])
   Gross_Profit = [Sales] - [COGS]
   - **Year-to-Date (YTD) and Prior Year-to-Date (PYTD) Measures**  
   Created YTD and PYTD measures for tracking growth:
   - **PYTD**:
     ```dax
     PYTD_Sales = CALCULATE([Sales], SAMEPERIODLASTYEAR(dim_Date[Date]), dim_Date[inpast] = TRUE())
     ```
   - Repeated the same process for Gross Profit and Quantity using similar logic.

   - **YTD**:
     ```dax
     YTD_GrossProfit = TOTALYTD([Gross Profit], dim_Date[Date])
     ```
   - Similarly, created YTD measures for Sales and Quantity.

- **Switch Measures for Slicer Options**  
   Created switch measures for easy selection of YTD and PYTD metrics:
   - **PYTD Switch**:
     ```dax
     S_PYTD = 
     VAR selected_value = SELECTEDVALUE(slc_Values[Values])
     VAR result = SWITCH(selected_value, "Sales", [PYTD_Sales], "Quantity", [PYTD_Quantity], "Gross Profit", [PYTD_GrossProfit], BLANK())
     RETURN result
     ```
   - **YTD Switch**:
     ```dax
     S_YTD = 
     VAR selected_value = SELECTEDVALUE(slc_Values[Values])
     VAR result = SWITCH(selected_value, "Sales", [YTD_Sales], "Quantity", [YTD_Quantity], "Gross Profit", [YTD_GrossProfit], BLANK())
     RETURN result
     ```

- **Comparative Measure (PYTD vs YTD)**  
   Calculated the difference to analyze year-over-year changes:
   ```dax
   PYTD_vs_YTD = [S_YTD] - [S_PYTD]

### Dashboard Layout and Visuals

First, I adjusted the layout by installing a simple background image, following guidance from the YouTube video mentioned earlier. You can use any background of your choice to suit the project style.

Creating the **measures** first made it straightforward to configure the visuals needed for the dashboard. Here’s how each visual element was organized:

- **Sales, Quantity, and Gross Profit Metrics**: 
  Each metric is dynamically displayed based on the **Switch Measures** (YTD and PYTD) for easy, on-the-go selections. This allows users to toggle between different metrics and track values over different periods.

- **Top 5 Products by Sales**:
  A clustered column chart displays the top 5 products based on sales. This visual was filtered using a Top N filter to highlight only the highest-performing products.

- **Sales by Region**:
  A map visual shows total sales by region, providing a geographic breakdown for a quick overview of where most sales are generated.

- **Monthly Sales Trend**:
  A line chart depicts monthly sales trends to illustrate seasonal changes or growth patterns over time. This visual is set to drill down by months to allow for an easy comparison of month-over-month growth.

The dashboard structure and visualizations aim to communicate key performance insights with simplicity and clarity.
