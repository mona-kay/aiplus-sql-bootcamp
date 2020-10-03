# Exercises (With Answers)

## Exercise 1
1. Select the first few records from the `power_generation` and `power_plants` tables. Write down the columns you believe are shared between these tables.

2. Explore the characteristics of the `world_governance_indicators` database by querying the `sqlite_master` schema.


## Exercise 2
1. How many unique types of `primary_fuel` exist in the `power_plants` table?

2. On average, which type of `primary_fuel` has the greatest `capacity_mw`?

3. Which `country_id` has the highest total `generation_gwh` in the `power_generation` table?


## Exercise 3
1. What type of join do you need if you want the number of power plants per country, including countries with no records in the `power_plants` table?

2. Select all information from the `countries` and `power_plants` tables using a left join. Limit your results to 100.


## Exercise 4
1. How many power plants are in each country? Join the `power_plants` table to the `countries` table to get the country name.

2. What are the top 3 countries with the most power plants in the database? You can use `DESC` to sort your results in descending order.

3. Which country has the most unique sources in the `data_sources` table?


## Exercise 5
1. What is the oldest power plant in the `power_plants` table? Use the `commissioning_year` column to find out.

2. Create a new column called `fuels` that combines the `primary_fuel` and `other_fuel`.

3. Trim down the url column in the `data_sources` table using the `REPLACE()` function to remove `http://`. You can read about how it works [here](https://www.sqlitetutorial.net/sqlite-replace-function/). 


## Exercise 6
1. Calculate the standardized average power generation across each year. Complete this by first calculating the average in each country per year using a subquery. _Save this query for a later exercise!_

2. What year in the United States has the highest total power generation?


## Exercise 7
1. Find the average power generation per year for ONLY countries that have geothermal power plants. Complete this using a subquery.

2. Return average power generation by country with a value higher than the overall average.


## Exercise 8
1. Rewrite your first query from Exercise 6 using a Common Table Expression (CTE).

2. Add a second CTE calculating the overall average power generation per year (without country). Subtract the country’s value from this overall value to get the difference from the average.


## Exercise 9
1. How many power plants in the database have a primary fuel that is a form of renewable energy -- `IN ('Hydro', 'Wind', 'Solar', 'Geothermal', 'Wave and Tidal')`?

2. There are a lot of missing values in the `year_of_capacity_data` column. Replace all missing values with the year `'2018'` using a `CASE` statement.


## Exercise 10
1. Prepare a “pivot table” of countries’ average `capacity_mw` by whether or not the power plant’s primary fuel is a renewable `('Hydro', 'Wind', 'Solar', 'Geothermal', 'Wave and Tidal')`. Each renewable type should be a separate column.

2. Determine the average `capacity_mw` across all power plants. Then, calculate the average `capacity_mw` by primary_fuel. What % higher or lower is each fuel type capacity compared to your benchmark average?

