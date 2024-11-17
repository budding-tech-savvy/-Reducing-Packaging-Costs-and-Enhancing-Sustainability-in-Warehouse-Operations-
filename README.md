# -Reducing-Packaging-Costs-and-Enhancing-Sustainability-in-Warehouse-Operations-

**This project aims to reduce packaging costs in warehouse operations while aligning with sustainability goals. The initiative involves evaluating high-cost, environmentally unfriendly packaging materials currently in use, then transitioning to eco-friendly, cost-effective alternatives**.

**Note:- The projects on my GitHub are structured as proof of concepts to demonstrate the workflows and methodologies I would apply in a professional setting.**

![image](https://github.com/user-attachments/assets/b374f00d-0815-4640-a439-affc9c6ca831)
![image](https://github.com/user-attachments/assets/3122bae8-b7cf-4c15-bb42-5f66e7623256)

[Download the Power BI Report (.pbix)](https://github.com/budding-tech-savvy/-Reducing-Packaging-Costs-and-Enhancing-Sustainability-in-Warehouse-Operations-/blob/main/Mom_reduction_in_cost.pbix)


Here’s a complete step-by-step guide for a project, from MySQL data extraction to Power BI visualization. This example assumes you have a dataset related to monthly consumption data for various articles and want to visualize month-over-month reduction.

# Step 1: Define the Project Scope
**Objective: Analyze six months of article-wise consumption data to showcase month-over-month reduction in consumption amount**.

**Tools Used**:

MySQL for data extraction and manipulation <br>
Power BI for visualization and reporting

# Step 2: Data Extraction Using MySQL

Assume your database has a table consumption_data with the following structure:

## Database Schema

| Column Name        | Data Type | Description                                  |
|--------------------|-----------|----------------------------------------------|
| `article_id`       | INT       | Unique identifier for each article           |
| `article_name`     | VARCHAR   | Name of the article                          |
| `month`            | DATE      | Month and year of consumption                |
| `consumption_units`| INT       | Number of units consumed                     |
| `consumption_cost` | DECIMAL   | Total cost of consumption for the month      |



## Sample Data

| article_id | article_name | month      | consumption_units | consumption_cost |
|------------|--------------|------------|-------------------|------------------|
| 1          | Article A    | 2024-05-01 | 300               | 6000            |
| 2          | Article B    | 2024-05-01 | 150               | 3000            |
| ...        | ...          | ...        | ...               | ...             |





# Step 2.1: SQL Query to Pull Data
This query fetches the last six months of data, organized by article and month.

```sql
SELECT 
    article_id,
    article_name,
    DATE_FORMAT(month, '%Y-%m') AS month,
    SUM(consumption_units) AS total_units,
    SUM(consumption_cost) AS total_cost
FROM 
    consumption_data
WHERE 
    month BETWEEN DATE_SUB(CURDATE(), INTERVAL 6 MONTH) AND CURDATE()
GROUP BY 
    article_id, month
ORDER BY 
    article_id, month;
```

 
**Explanation**: This query groups consumption data by article and month, summing up consumption_units and consumption_cost.
Result: The output will have each article's monthly data for the past six months.

**Export Data to CSV**
After running the query, export the results to a CSV file (or directly connect MySQL to Power BI if possible).

# Step 3: Load Data into Power BI
Open Power BI Desktop.
Import Data:
Select Get Data > Text/CSV (if you exported CSV) or MySQL Database to connect directly to MySQL
Choose your dataset and load it into Power BI.

**In Power BI, I have used DAX to make the dashboard more interactive and dynamic, calculating various metrics as shown below**.
```sql
<!-- creation of table -->
ranktable = DATATABLE(
                   "sort", INTEGER,
                   "type",string ,"no",INTEGER,
                   {
                      {0, "All", 0},
                      {1,"Top 5", 5},
                      {2,"Top 10", 10},
                      {3,"Top 20", 20},
                      {4,"Top 50", 50},
                      {5,"Top 100", 100}
                   }
)
```

```sql
<!--To arrange bar chart in more dyanamics way-->
rank_wise_sales = var rankvalue=rankx(all(consumption_data[article_name]),[total_cost],,DESC)
                 var selectedrank=SELECTEDVALUE(ranktable[no])

return 
if(selectedrank=0,[total_cost],
if(rankvalue<=selectedrank,[total_cost],blank()))
```
```sql
<!--To make chart's title more dynamic-->
dynamic_title = var selectedrank=SELECTEDVALUE(ranktable[type])
                
 return 
 selectedrank&" Articles "
```

```sql
<!--Creation of New table to calculate the Month-over-Month_%-->
total_cost_%_change = SUMMARIZE(consumption_data,consumption_data[Month1],"total_amount",sum(consumption_data[consumption_cost]))
```

```sql

<!--Calculation for Month-over-Month_% for total Cost-->
MoM_Percentage_Change_Column = 
    VAR CurrentMonthAmount = 'total_cost_%_change'[total_amount]
    VAR PreviousMonthAmount = 
        CALCULATE(
            SUM('total_cost_%_change'[total_amount]),
            FILTER(
                'total_cost_%_change',
                MONTH('total_cost_%_change'[MonthAsDate]) = MONTH(EARLIER('total_cost_%_change'[MonthAsDate])) - 1 &&
                YEAR('total_cost_%_change'[MonthAsDate]) = YEAR(EARLIER('total_cost_%_change'[MonthAsDate]))
            )
        )
    RETURN 
        IF(ISBLANK(PreviousMonthAmount), 0, (CurrentMonthAmount - PreviousMonthAmount) / PreviousMonthAmount )

```

```sql
<!--Dax used for inserting arrows into table-->
MoM_Arrow = 
    IF('total_cost_%_change'[MoM_Percentage_Change_Column] > 0, "↑", 
       IF('total_cost_%_change'[MoM_Percentage_Change_Column] <0, "↓", "→"))
```

```sql
<!--Month-over-Month % change based on total average -->
MoM_Percentage_Change_Column = 
    VAR CurrentMonthAmount = avg_total_amount_table[avg_amount]
    VAR PreviousMonthAmount = 
        CALCULATE(
            SUM(avg_total_amount_table[avg_amount]),
            FILTER(
                avg_total_amount_table,
                MONTH(avg_total_amount_table[MonthAsDate]) = MONTH(EARLIER(avg_total_amount_table[MonthAsDate])) - 1 &&
                YEAR(avg_total_amount_table[MonthAsDate]) = YEAR(EARLIER(avg_total_amount_table[MonthAsDate]))
            )
        )
    RETURN 
        IF(ISBLANK(PreviousMonthAmount), 0, (CurrentMonthAmount - PreviousMonthAmount) / PreviousMonthAmount )
```
