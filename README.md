# SQL United States Emissions Dashboard

SQL End-to-End Data Project In Databricks.

This project analyzes emissions data using SQL to uncover trends and insights across different states and counties. The results are organized to support a dashboard that helps visualize total emissions and identify key areas for environmental focus.

# Finished United States Emissions Dashboard 

This data was collected by the EPA (Environmental Protection Agency) in 2023

<img width="1363" height="831" alt="Screenshot 2025-12-09 at 5 06 36 PM" src="https://github.com/user-attachments/assets/2fa66dbe-6485-4f48-81cf-331833d6e959" />

[Emissions Dashboard.pdf](https://github.com/user-attachments/files/24064711/Emissions.Dashboard.2025-12-09.22_03.pdf)

# Project Skills

SQL querying & data manipulation │ Data analysis & aggregation │ Database design & schema understanding │ Data cleaning & preprocessing │ Reporting & dashboard insights

# Business Task

Develop a dashboard that visualizes U.S. emissions data, highlighting where emissions originate, where the most emissions are coming from, which states and counties contribute the most, and how much emissions we have in general on a per-person, per-area basis. The goal is to support better environmental decision-making by clearly showing emission patterns across regions and populations.


# 1. Ask

• Where are the highest emissions concentrated?

• Which states and counties produce the most emissions?

• How many emissions do we have on a per-person, per-area basis?


# 2. Prepare

Starting Dataset: [Emissions data link](https://github.com/AlexTheAnalyst/DatabricksSeries/blob/main/Emissions_Data_2023.csv) raw (CSV file) dataset containing detailed emissions data.

To begin, I uploaded the CSV file to Databricks by creating a new catalog named emissions. Within the catalog’s default schema, I created a new table and imported the Emissions 2023 CSV, saving it as emissions_data for analysis.

<img width="1325" height="878" alt="Screenshot 2025-12-09 at 12 59 24 PM" src="https://github.com/user-attachments/assets/87d4d132-2e75-44f4-a01a-7f93992f9693" />

# 3. Process 

I then opened the SQL Editor and ran a simple query to view the table:

SELECT * 
FROM emissions.default.emissions_data;

From this output, I identified the column names I needed and wrote a more targeted query:

# 4. Analyze

# Building The First Visual, Emissions Per Location Map

In the SQL Editor

SELECT 
    latitude,
    longitude,
    `GHG emissions mtons CO2e` AS Emissions
FROM emissions.default.emissions_data;


Next, I created my dashboard dataset by navigating to Dashboards → Datasets → Create from SQL and pasting the refined query to generate the dataset for visualization. I named the dataset Location Data.

<img width="499" height="731" alt="Screenshot 2025-12-09 at 1 38 26 PM" src="https://github.com/user-attachments/assets/f974d644-5fbb-48b8-bc9b-7c3be27151e1" />


Next, I added a visualization in the Dashboard by selecting Add Visualization and choosing the Point Map option. I assigned the longitude field to the Longitude setting and the latitude field to the Latitude setting. To improve clarity and accuracy on the map, I reduced the point size so the data markers were easier to interpret.

<img width="1121" height="718" alt="Screenshot 2025-12-09 at 1 47 40 PM" src="https://github.com/user-attachments/assets/f892b1ed-9e46-4b5f-ab1f-b4d30a6d0a11" />

Looking at the map, it’s immediately clear that the majority of emissions are concentrated on the eastern side of the United States.

Next, I added a text box at the top of the dashboard to create a title. I named it United States Emissions Breakdown, centered the text, applied bold formatting, and increased the font size for clear visibility. Below the title, I added a short subtext noting that the data was collected by the EPA (Environmental Protection Agency) in 2023.

# Second Visual, Emissions vs Population Scatter Plot

Back to SQL Editor, I created a new query to look at county names and the population, specifically to look at the emissions per person.

SELECT  county_state_name,
       population, 
       CAST(`GHG emissions mtons CO2e` AS DOUBLE) / CAST(population AS DOUBLE) AS Emissions_per_person
FROM emissions.default.emissions_data

That returned an error because the data is showing as a string value 
I need to remove the thousands separator (comma) from the string before casting to DOUBLE. I Use the replace function to do this.

'''sql 

SELECT 
    county_state_name,
    population,
    CAST(
        REPLACE(`GHG emissions mtons CO2e`, ',', '') AS DOUBLE
    ) / NULLIF(
        CAST(
            REPLACE(population, ',', '') AS DOUBLE
        ), 0
    ) AS Emissions_per_person
FROM emissions_data;


<img width="655" height="703" alt="Screenshot 2025-12-09 at 2 20 33 PM" src="https://github.com/user-attachments/assets/8d5d9bf9-5058-4e44-a067-0fa0a8ab13df" />

To analyze emissions per person, I ordered the results by the calculated Emissions_per_person value and limited the output to the top 10. This allowed me to quickly identify the counties with the highest emissions per capita.


SELECT 
    county_state_name,
    population,
    CAST(
        REPLACE(`GHG emissions mtons CO2e`, ',', '') 
        AS DOUBLE
    ) 
    / NULLIF(
        CAST(
            REPLACE(population, ',', '') 
            AS DOUBLE
        ), 
        0
    ) AS Emissions_per_person
FROM emissions_data
ORDER BY Emissions_per_person DESC
LIMIT 10

<img width="607" height="783" alt="Screenshot 2025-12-09 at 2 24 53 PM" src="https://github.com/user-attachments/assets/0fdf051e-9a86-4948-a5e2-08e18209f96f" />

Now that I had this query, I returned to the dashboard and created a new dataset by going to Data → Create from SQL and pasting the query. I removed the LIMIT 10 clause so the scatter plot would include all counties, allowing me to analyze whether emissions per person increase or decrease as the population grows. I then saved the dataset and named it Emissions Per Person.

Next, I returned to the dashboard and added a new visualization. I selected the Emissions per Person dataset and set up a scatter plot, assigning Emissions_per_person to the X-axis and population to the Y-axis. I switched the data settings to use raw values with no aggregation and adjusted the point size for better visibility.

To make the chart more informative, I added county_state_name to the tooltip. This allows me to hover over any point and immediately see which county and state it represents.

From the scatter plot, I can clearly see an outlier in the upper left corner Los Angeles, which shows approximately 0.5 megatons of emissions per person and a population of over 10 million.

I then added a title to the new visual, naming it Emissions vs. Population, and included a brief description noting that areas with larger populations tend to have lower emissions per person.

# Visual 3 Total Emissions Per State Pie Chart

Back in the SQL Editor, I opened a new tab to create a query that analyzes total emissions by state. My goal was to identify the top 10 highest-emitting states in the U.S. I began by copying my previous query and then modifying it. Since I only needed state-level totals, I replaced the selected columns with just the state abbreviation column. I removed the per-person calculation and renamed the emissions field as Total_Emissions.

Because each state appears multiple times in the dataset, I added a GROUP BY on the state abbreviation. I also updated the query to use SUM() before the CAST function to aggregate emissions across all counties in each state. Finally, I adjusted the ORDER BY clause to sort by the total emissions.

My query ended up looking like this, 

SELECT 
    state_abbr,
    SUM(CAST(
        REPLACE(`GHG emissions mtons CO2e`, ',', '') 
        AS DOUBLE
     ) ) AS Total_Emissions
FROM emissions_data
GROUP BY state_abbr
ORDER BY Total_Emissions DESC
LIMIT 10

<img width="475" height="743" alt="Screenshot 2025-12-09 at 3 02 22 PM" src="https://github.com/user-attachments/assets/e452c8f7-1480-4e4e-8639-6da73dffb251" />

I then took this query back to the dashboard, navigated to Data → Create from SQL, and pasted it to generate a new dataset. Then named the dataset Total Emissions Per State.

Back in the dashboard, I added a new visual and selected the Total Emissions Per State dataset. I changed the visualization type to a pie chart, set the angle field to Total Emissions, and used the state abbreviation field to define the color segments. I enabled labels for clarity and renamed Total Emissions to Emissions Percentage to make the chart easier to interpret. Finally, I added a description noting that the top 10 states account for X amount of emissions for all of the US.

I added X for now because I don't know what the percentage is yet and will have to query that next.

<img width="1148" height="781" alt="Screenshot 2025-12-09 at 4 12 19 PM" src="https://github.com/user-attachments/assets/5601947e-1f92-4890-8c7c-46286cdfa16c" />

Now I’m going to return to the SQL Editor, open a new query, and enter the following CTE.

This query uses a CTE to identify the top 10 highest-emitting states by summing their total emissions. It then calculates what percentage these top 10 states contribute to the overall U.S. emissions.

WITH top10 AS (
    SELECT 
        state_abbr,
        SUM(
            CAST(
                REPLACE(`GHG emissions mtons CO2e`, ',', '') 
                AS DOUBLE
            )
        ) AS Total_Emissions
    FROM emissions_data
    GROUP BY state_abbr
    ORDER BY Total_Emissions DESC
    LIMIT 10
)
SELECT 
    SUM(Total_Emissions) AS Top10_Emissions,
    (
        SUM(Total_Emissions) /
        (
            SELECT 
                SUM(
                    CAST(
                        REPLACE(`GHG emissions mtons CO2e`, ',', '') 
                        AS DOUBLE
                    )
                )
            FROM emissions_data
        )
    ) * 100 AS Top10_Percentage
FROM top10;

<img width="742" height="623" alt="Screenshot 2025-12-09 at 4 22 12 PM" src="https://github.com/user-attachments/assets/51e6811a-6b10-46ce-9e92-92caa2c8c27b" />

The query showed that the top 10 emitting states account for over 50% of total U.S. emissions.
With this new information, I can return to my pie chart and replace the placeholder ‘x’ with the correct value of 50.8% in the description.
I changed the description to read These 10 States account for 50.8%  of all emissions in the US.

<img width="405" height="346" alt="Screenshot 2025-12-09 at 4 28 40 PM" src="https://github.com/user-attachments/assets/bae959b5-623c-400c-a5a7-6e0ee22e8c92" />

# Total Emissions by mTon of CO2e Bar Chart

Next, I want to create a bar chart showing the top 10 counties in the U.S. by total emissions. To do this, I open a new query in the SQL Editor and use a previous query as a starting point. Since I’m only interested in total emissions, I run the following query:

SELECT 
    county_state_name,
    population,
    CAST(
        REPLACE(`GHG emissions mtons CO2e`, ',', '') 
        AS DOUBLE) as Total_Emissions
FROM emissions_data
ORDER BY Total_Emissions DESC
LIMIT 10

<img width="594" height="748" alt="Screenshot 2025-12-09 at 4 35 44 PM" src="https://github.com/user-attachments/assets/632e2e8b-59bd-4f6e-b3bd-97ae47b65a85" />

I then take that query into the Dashboards section by selecting Data → Create from SQL, and paste the query to generate the dataset for the visualization. I then named that dataset County Emissions.

Next, I return to the dashboard, add a new visualization, and select the county emissions dataset. I chose a bar chart, set Total Emissions as the X-axis, County_State_Name as the Y-axis, sorted the bars from highest to lowest, and made sure to add labels.
i then added the title Total Emissions by mTon of CO2e

<img width="420" height="361" alt="Screenshot 2025-12-09 at 4 51 37 PM" src="https://github.com/user-attachments/assets/a7b9a35e-bb90-4477-8c2e-c12a2d8987bf" />

Afterwards, I spent some time cleaning up the labels and titles throughout the dashboard for clarity and readability.

# 5. Share 

# Emissions Dashboard 

<img width="1363" height="831" alt="Screenshot 2025-12-09 at 5 06 36 PM" src="https://github.com/user-attachments/assets/2fa66dbe-6485-4f48-81cf-331833d6e959" />

[Emissions Dashboard.pdf](https://github.com/user-attachments/files/24064711/Emissions.Dashboard.2025-12-09.22_03.pdf)

# 6. Act



