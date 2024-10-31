# COVID-19 Data Exploration Project

This project explores global COVID-19 data using SQL queries to gain insights into cases, deaths, and vaccination rates. The analysis covers data from January 2020 to April 2021 and leverages SQL Server Management Studio (SSMS) to query, filter, and aggregate data.

# Table of Contents
- Project Overview
- Dataset Information
- Queries and Analysis
   - Global Data Overview
   - Country-Specific Analysis (Nigeria & United States)
   - Infection Rates by Country
   - Death Counts by Continent and Country
   - Global Death Rates Over Time
   - Vaccination Analysis
- Conclusions
- Indexing and Optimization
- Technologies Used
- License

## Project Overview
This project explores the COVID-19 pandemic's impact across the world using publicly available data on cases, deaths, and vaccinations. Through various SQL queries, the project analyzes infection and death rates by location, highlights periods of high mortality, and assesses vaccination progress. The analysis covers trends in specific countries like Nigeria and the United States and insights on countries with high infection or vaccination rates.

## Dataset Information
Two tables are used for this analysis:

- **CovidDeaths** - Contains information on total cases, new cases, total deaths, and population data.
- **CovidVaccinations** - Contains data on vaccinations, including new vaccinations per day.
- The datasets include global data from January 2020 to April 2021.

## Queries and Analysis
### Global Data Overview
- The initial queries inspect general COVID-19 trends worldwide:

   - Filtering non-null continents for relevance.
   - Selecting key columns: location, date, total cases, new cases, total deaths, and population.
   - Confirming that data spans 219 unique locations.
     
```sql
SELECT COUNT(DISTINCT location) AS unique_locations
FROM CovidDeaths
WHERE continent IS NOT NULL;
```


### Country-Specific Analysis (Nigeria & United States)

The analysis examines total cases and deaths in *Nigeria* and the *United States*, calculating the death rate percentages. Key insights:
- Nigeria had a death rate of 1.25% by April 2021.
- United States reached a death rate of 1.8% by April 2021.
  
```sql
SELECT location, date, total_cases, total_deaths,
    ROUND((CAST(total_deaths AS FLOAT) / CAST(total_cases AS FLOAT) * 100), 2) AS DeathPercentage
FROM CovidDeaths
WHERE location IN ('Nigeria', 'United States') AND continent IS NOT NULL
ORDER BY location, date;
```


### Infection Rates by Country

The analysis also compares infection rates to population sizes across countries to identify the highest infection rates:
- Andorra had the highest infection rate (17.13%), followed by Montenegro and Czechia.
  
```sql
SELECT location, population,
    MAX(total_cases) AS Infection_count,
    MAX(ROUND((CAST(total_cases AS FLOAT) / CAST(population AS FLOAT) * 100), 2)) AS Infection_rate
FROM CovidDeaths
WHERE continent IS NOT NULL
GROUP BY location, population
ORDER BY Infection_rate DESC;
```

### Death Counts by Continent and Country
To identify the continents and countries most affected by COVID-19, queries analyze total death counts:
- North America reported the highest death toll among continents.
  
```sql
SELECT continent, MAX(CAST(total_deaths AS INT)) AS Highest_death_count
FROM CovidDeaths
WHERE continent IS NOT NULL
GROUP BY continent
ORDER BY Highest_death_count DESC;
```

### Global Death Rates Over Time
Using a timeline of death rates, this section examines periods with elevated mortality. For instance, February 2020 recorded the highest death rate at **29.52%**.

```sql
SELECT date, SUM(new_cases) AS Total_cases,
    SUM(CAST(new_deaths AS INT)) AS Total_deaths,
    ROUND(SUM(CAST(new_deaths AS INT)) / SUM(new_cases) * 100, 2) AS Global_Death_Percentage
FROM CovidDeaths
WHERE new_cases <> 0 AND new_deaths <> 0 AND continent IS NOT NULL
GROUP BY date
ORDER BY Global_Death_Percentage DESC;
```

### Vaccination Analysis
This segment explores vaccination rates across countries by calculating the cumulative sum of vaccinations over time:
- Gibraltar and Israel showed vaccination rates exceeding 100%, likely due to input errors or re-vaccinations.
The cumulative vaccination rate required temporary tables to optimize calculations.

```sql
-- Temporary Table Creation and Cumulative Vaccination Calculation
DROP TABLE IF EXISTS #Percentage_vaccinated;
CREATE TABLE #Percentage_vaccinated (
    continent NVARCHAR(255),
    location NVARCHAR(255),
    date DATETIME,
    population NUMERIC,
    new_vaccination NUMERIC,
    cumulation_people_vaccinated NUMERIC
);

INSERT INTO #Percentage_vaccinated
SELECT d.continent, d.location, d.date, d.population,
    v.new_vaccinations,
    SUM(CAST(v.new_vaccinations AS BIGINT)) OVER (
        PARTITION BY d.location ORDER BY d.location, d.date) AS RollupPeopleVaccinated
FROM CovidDeaths AS d
JOIN CovidVaccinations AS v ON d.location = v.location AND d.date = v.date
WHERE d.continent IS NOT NULL;
```

## Conclusions
This analysis has provided a snapshot of the pandemic's impact globally and in specific countries. The data highlights:
- Higher death rates during early pandemic months.
- Countries with high infection rates relative to their populations.
- High vaccination rates, sometimes exceeding 100% in certain areas.
  
## Indexing and Optimization
Indexes were added on location and date columns in both tables to improve query performance.

```sql
CREATE INDEX idx_deaths ON CovidDeaths(location, date);
CREATE INDEX idx_vacs ON CovidVaccinations(location, date);
```

## Technologies Used
- SQL Server Management Studio (SSMS)
- SQL Window Functions
- Temporary Tables and Indexing for performance optimization
  
## License
This project is licensed under the MIT License. See [LICENSE](https://opensource.org/licenses/MIT) for details.
