# Exercises (Without Answers)

## Exercise 1
1. Select the first few records from the `power_generation` and `power_plants` tables. Write down the columns you believe are shared between these tables.
```sql
SELECT *
FROM power_generation
LIMIT 10;

SELECT *
FROM power_plants
LIMIT 10;
```
Columns shared between tables:
- `gppd_indr`
- `country_id`

2. Explore the characteristics of the `world_governance_indicators` database by querying the `sqlite_master` schema.
```sql
SELECT *
FROM sqlite_master;
```

## Exercise 2
1. How many unique types of `primary_fuel` exist in the `power_plants` table?
```sql
SELECT DISTINCT primary_fuel
FROM power_plants
LIMIT 10;

-- or
SELECT primary_fuel,
       count(*)
FROM power_plants
GROUP BY primary_fuel
LIMIT 10;
```

2. On average, which type of `primary_fuel` has the greatest `capacity_mw`?
```sql
SELECT primary_fuel,
       avg(capacity_mw) as avg_capacity
FROM power_plants
GROUP BY primary_fuel
ORDER BY avg_capacity DESC
LIMIT 10;
```

3. Which `country_id` has the highest total `generation_gwh` in the `power_generation` table?
```sql
SELECT country_id,
       sum(generation_gwh) as total_generation
FROM power_generation
GROUP BY country_id
ORDER BY total_generation DESC
LIMIT 10;
```

## Exercise 3
1. What type of join do you need if you want the number of power plants per country, including countries with no records in the `power_plants` table?
- Left join or outer join

2. Select all information from the `countries` and `power_plants` tables using a left join. Limit your results to 100.
```sql
SELECT c.*,
       p.*
FROM countries AS c
LEFT JOIN power_plants AS p
ON c.id = p.country_id
LIMIT 10;
```

## Exercise 4
1. How many power plants are in each country? Join the `power_plants` table to the `countries` table to get the country name.
```sql
SELECT c.id,
       c.name,
       count(p.gppd_idnr) AS pplants
FROM countries AS c
LEFT JOIN power_plants AS p
ON c.id = p.country_id
GROUP BY c.id,
         c.name
LIMIT 10;
```

2. What are the top 3 countries with the most power plants in the database? You can use `DESC` to sort your results in descending order.
```sql
SELECT c.name,
       count(p.gppd_idnr) AS pplants
FROM countries AS c
LEFT JOIN power_plants AS p
ON c.id = p.country_id
GROUP BY c.id,
         c.name
ORDER BY pplants DESC
LIMIT 3;
```

3. Which country has the most unique sources in the `data_sources` table?
```sql
SELECT p.country_id,
       count(DISTINCT ds.source) AS sources
FROM power_plants p
LEFT JOIN data_sources ds
ON p.source_id = ds.id
GROUP BY p.country_id
ORDER BY sources DESC
LIMIT 10;
```

## Exercise 5
1. What is the oldest power plant in the `power_plants` table? Use the `commissioning_year` column to find out.
```sql
SELECT name,
       country_id,
       gppd_idnr,
       commissioning_year
FROM power_plants
ORDER BY commissioning_year DESC
LIMIT 10;
```

2. Create a new column called `fuels` that combines the `primary_fuel` and `other_fuel`.
```sql
SELECT name,
       country_id,
       gppd_idnr,
       primary_fuel || ', ' || other_fuel AS fuels
FROM power_plants
LIMIT 10;
```

3. Trim down the url column in the `data_sources` table using the `REPLACE()` function to remove `http://`. You can read about how it works [here](https://www.sqlitetutorial.net/sqlite-replace-function/). 
```sql
SELECT id,
       source,
       geolocation_source,
       REPLACE(url, 'http://', '') AS url
FROM data_sources
LIMIT 10;
```

## Exercise 6
1. Calculate the standardized average power generation across each year. Complete this by first calculating the average in each country per year using a subquery. _Save this query for a later exercise!_
```sql
SELECT year,
       avg(avg_gwh) as yrly_avg_gwh
FROM (
  SELECT country_id,
         year,
         avg(generation_gwh) as avg_gwh
  FROM power_generation
  GROUP BY country_id,
           year
) avg_power_gen
GROUP BY year;
```

2. What year in the United States has the highest total power generation?
```sql
SELECT c.name,
       pg.year,
       sum(pg.generation_gwh) AS power_gen
FROM countries c
INNER JOIN power_plants p
ON c.id = p.country_id
INNER  JOIN power_generation pg
ON p.country_id = pg.country_id
AND p.gppd_idnr = pg.gppd_idnr
WHERE c.name = 'USA'
GROUP BY c.name,
         pg.year
ORDER BY power_gen DESC;
```

## Exercise 7
1. Find the average power generation per year for ONLY countries that have geothermal power plants. Complete this using a subquery.
```sql
SELECT c.country,
       pg.year,
       avg(pg.generation_gwh) AS power_gen
FROM countries c
INNER JOIN power_plants p
ON c.id = p.country_id
INNER JOIN power_generation pg
ON p.country_id = pg.country_id
AND p.gppd_idnr = pg.gppd_idnr
WHERE c.id IN (
    SELECT DISTINCT country_id
    FROM power_plants
    WHERE primary_fuel = 'Geothermal')
GROUP BY c.country,
         pg.year;
```

2. Return average power generation by country with a value higher than the overall average.
```sql
SELECT c.country,
       avg(pg.generation_gwh) AS power_gen
FROM countries c
INNER JOIN power_plants p
ON c.id = p.country_id
INNER JOIN power_generation pg
ON p.country_id = pg.country_id
AND p.gppd_idnr = pg.gppd_idnr
GROUP BY c.country
HAVING avg(pg.generation_gwh) > (
    SELECT avg(generation_gwh)
    FROM power_generation);
```

## Exercise 8
1. Rewrite your first query from Exercise 6 using a Common Table Expression (CTE).
```sql
WITH avg_power_gen AS (
  SELECT country_id,
         year,
         avg(generation_gwh) as avg_gwh
  FROM power_generation
  GROUP BY country_id,
           year)
SELECT year,
       avg(avg_gwh) as yrly_avg_gwh
FROM avg_power_gen
GROUP BY year;
```

2. Add a second CTE calculating the overall average power generation per year (without country). Subtract the country’s value from this overall value to get the difference from the average.
```sql
WITH avg_power_gen AS (
  SELECT country_id,
         year,
         avg(generation_gwh) as avg_gwh
  FROM power_generation
  GROUP BY country_id,
           year),
overall_avg_power_gen AS (
  SELECT year,
         avg(generation_gwh) as ov_avg_gwh
  FROM power_generation
  GROUP BY year  
)
SELECT av.year,
       oav.ov_avg_gwh,
       avg(av.avg_gwh) as yrly_avg_gwh,
       oav.ov_avg_gwh - avg(av.avg_gwh) AS avg_diff
FROM avg_power_gen av
INNER JOIN overall_avg_power_gen oav
ON av.year = oav.year
GROUP BY av.year,
         oav.ov_avg_gwh;
```

## Exercise 9
1. How many power plants in the database have a primary fuel that is a form of renewable energy -- `IN ('Hydro', 'Wind', 'Solar', 'Geothermal', 'Wave and Tidal')`?
```sql
SELECT
    CASE WHEN primary_fuel IN ('Hydro', 'Wind', 'Solar', 'Geothermal', 'Wave and Tidal')
         THEN 'Renewable' ELSE 'Non-Renewable' END AS fuel_type,
    count(*) as pplants
FROM power_plants
GROUP BY 1;
```

2. There are a lot of missing values in the `year_of_capacity_data` column. Replace all missing values with the year `'2018'` using a `CASE` statement.
```sql
SELECT *,
       CASE WHEN year_of_capacity_data IS NULL 
            THEN 2018 ELSE year_of_capacity_data end AS year_of_capacity_data
FROM power_plants;
```

## Exercise 10
1. Prepare a “pivot table” of countries’ average `capacity_mw` by whether or not the power plant’s primary fuel is a renewable `('Hydro', 'Wind', 'Solar', 'Geothermal', 'Wave and Tidal')`. Each renewable type should be a separate column.
```sql
SELECT c.country,
       avg(CASE WHEN p.primary_fuel IN ('Hydro', 'Wind', 'Solar', 'Geothermal', 'Wave and Tidal')
            THEN p.capacity_mw END) AS renewable_capacity,
       avg(CASE WHEN p.primary_fuel NOT IN ('Hydro', 'Wind', 'Solar', 'Geothermal', 'Wave and Tidal')
            THEN p.capacity_mw END) AS non_renewable_capacity
FROM countries c
INNER JOIN power_plants p
ON c.id = p.country_id
GROUP BY 1
LIMIT 100;
```

2. Determine the average `capacity_mw` across all power plants. Then, calculate the average `capacity_mw` by primary_fuel. What % higher or lower is each fuel type capacity compared to your benchmark average?
```sql
```
