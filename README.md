# -Reducing-Packaging-Costs-and-Enhancing-Sustainability-in-Warehouse-Operations-
This project aims to reduce packaging costs in warehouse operations while aligning with sustainability goals. The initiative involves evaluating high-cost, environmentally unfriendly packaging materials currently in use, then transitioning to eco-friendly, cost-effective alternatives.

![image](https://github.com/user-attachments/assets/b374f00d-0815-4640-a439-affc9c6ca831)







Here’s a complete step-by-step guide for a project, from MySQL data extraction to Power BI visualization. This example assumes you have a dataset related to monthly consumption data for various articles and want to visualize month-over-month reduction.

Step 1: Define the Project Scope
Objective: Analyze six months of article-wise consumption data to showcase month-over-month reduction in consumption amount.

Tools Used:

MySQL for data extraction and manipulation
Power BI for visualization and reporting
Step 2: Data Extraction Using MySQL
Assume your database has a table consumption_data with the following structure:

Column Name	Data Type	Description
article_id	INT	Unique identifier for each article
article_name	VARCHAR	Name of the article
month	DATE	Month and year of consumption
consumption_units	INT	Number of units consumed
consumption_cost	DECIMAL	Total cost of consumption for the month
warehouse_id	INT	Warehouse location identifier (optional)

Sample Data

article_id	article_name	month	consumption_units	consumption_cost
1	Article A	2024-05-01	300	6000
2	Article B	2024-05-01	150	3000
...	...	...	...	...


-- Query: Shipment Data
-- Writer: Manuj Singhwal
-- Description: Fetch last mile data for curdate() - 3

WITH shipment_data_1 AS (
    SELECT
        o.ShipmentId,
        c.Carrier,
        cs.CarrierService,
        l1.City AS OriginCity,
        l1.State AS OriginState,
        l1.Country AS OriginCountry,
        l2.City AS DestinationCity,
        l2.State AS DestinationState,
        l2.Country AS DestinationCountry,
        o.ShipmentImportedTimestamp,
        o.ShipmentFulfilledSLATimestamp,
        o.ShipmentFulfilledTimestamp,
        o.ShipmentCarrierPickUpTimestamp,
        o.ShipmentDeliveredTimestamp,
        os.OrderStatus,
        CASE
            WHEN o.CarrierServiceId = 11 THEN date_add(ShipmentCarrierPickUpTimestamp, INTERVAL 1 DAY) -- Next Day Delivery
            WHEN o.CarrierServiceId = 13 THEN date_add(ShipmentCarrierPickUpTimestamp, INTERVAL 2 DAY) -- Two Day Delivery
            WHEN o.CarrierServiceId = 12 THEN date_add(ShipmentCarrierPickUpTimestamp, INTERVAL 3 DAY) -- Standard (Three) Day Delivery
        END AS ShipmentExpectedDeliveryTimestamp
    FROM fact_orders AS o
    JOIN dim_carriers AS c
        ON c.CarrierId = o.CarrierId
    JOIN dim_carrier_services AS cs
        ON cs.CarrierServiceId = o.CarrierServiceId
    JOIN dim_locations AS l1
        ON l1.LocationId = o.OriginLocationId
    JOIN dim_locations AS l2
        ON l2.LocationId = o.DestinationLocationId
    JOIN dim_order_status AS os
        ON os.StatusId = o.StatusId
    WHERE date(ShipmentImportedTimestamp) = '2024-10-01' -- Use date_sub(curdate(), interval 3 day) for general use case
),

shipment_data_2 AS (
    SELECT
        *,
        CASE
            WHEN ShipmentDeliveredTimestamp IS NULL THEN 'InTransit'
            ELSE 'Delivered'
        END AS DeliveryStatus,
        CASE
            WHEN ShipmentDeliveredTimestamp IS NULL THEN 'Pending'
            WHEN ShipmentDeliveredTimestamp > ShipmentExpectedDeliveryTimestamp THEN 'Delayed'
            WHEN ShipmentDeliveredTimestamp <= ShipmentExpectedDeliveryTimestamp THEN 'OnTime'
        END AS DeliverySubStatus
    FROM shipment_data_1
)

SELECT
    *,
    CASE
        WHEN DeliverySubStatus = 'Delayed' THEN ROUND(timestampdiff(MINUTE, ShipmentExpectedDeliveryTimestamp, ShipmentDeliveredTimestamp) / 60.0, 2)
        ELSE 0
    END AS DeliveryDelayHour
FROM shipment_data_2;


Step 2.1: SQL Query to Pull Data
This query fetches the last six months of data, organized by article and month.

sql
Copy code
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
Explanation: This query groups consumption data by article and month, summing up consumption_units and consumption_cost.
Result: The output will have each article's monthly data for the past six months.
Export Data to CSV
After running the query, export the results to a CSV file (or directly connect MySQL to Power BI if possible).
**
Step 3: Load Data into Power BI**
Open Power BI Desktop.
Import Data:
Select Get Data > Text/CSV (if you exported CSV) or MySQL Database to connect directly to MySQL.
Choose your dataset and load it into Power BI.
