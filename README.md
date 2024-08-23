# COVID-19 Data Analysis Project

## Project Overview
This project involves analyzing COVID-19 data using SQL to gain insights into the pandemic's impact across different countries and continents. The analysis focuses on critical metrics such as total cases, total deaths, death rates, infection rates, and vaccination progress. The data used in this project comes from publicly available sources and is intended to showcase my SQL skills and ability to derive meaningful insights from complex datasets.

## Project Structure
### SQL Queries:
- Analyze cases, deaths, and vaccination rates.
- Calculate infection and death rates.
- Identify high-impact countries and continents.
  
### Temporary Tables & Views:
Create temporary tables and views for more complex analyses and visualization preparation.

### How to Run the Queries
To run the queries, clone this repository and execute the SQL scripts in your PostgreSQL environment. Ensure that the public.coviddeaths and public.covidvaccinations tables are properly populated with the relevant data.
```bash
git clone https://github.com/your-username/covid-data-analysis.git
cd covid-data-analysis
psql -f covid-analysis.sql
```

## Key Analyses and Insights

### 1. Total Cases and Deaths
**Purpose**: Understand the overall impact of the pandemic over time.

**Query:** Retrieve data on total COVID-19 cases and deaths across various locations and dates.
```sql
SELECT location, date, total_cases, new_cases, total_deaths, population
FROM public.coviddeaths
ORDER BY 1,2;
```
### 2. Death Rate Analysis

**Purpose**: Compare death rates between different countries to assess the severity of the pandemic.
```sql
SELECT location, date, total_cases, total_deaths, (total_deaths::float / total_cases::float) * 100 AS death_rate
FROM public.coviddeaths
WHERE location ILIKE '%states%'
ORDER BY 1,2;
```
**United States**: As of June 2024, the death rate from reported COVID-19 cases was approximately 1.14%.

```sql
SELECT location, date, total_cases, total_deaths, (total_deaths::float / total_cases::float) * 100 AS death_rate
FROM public.coviddeaths
WHERE location ILIKE '%kenya%'
ORDER BY 1,2;
```
**Kenya**: As of June 2024, the death rate from reported COVID-19 cases was approximately 1.65%.

### 3. Infection Rate vs. Population
**Purpose**: Evaluate how widespread the infection was relative to the population.
```sql
SELECT location, date, total_cases, population, (total_cases::float / population) * 100 AS case_rate
FROM public.coviddeaths
WHERE location ILIKE '%states%'
ORDER BY 1,2;
```
**United States**: Approximately 30.5% of the population had been infected by June 2024.
```sql
SELECT location, date, total_cases, population, (total_cases::float / population) * 100 AS percent_population_infected
FROM public.coviddeaths
WHERE location ILIKE '%kenya%'
ORDER BY 1,2;
```
**Kenya**: Approximately 0.6% of the population had been infected by June 2024.

### 4. Highest Infection Rate by Country
**Purpose**: Highlight the countries most affected by the pandemic.
**Query**: Identify countries with the highest infection rates compared to their populations.
```sql
SELECT continent, location, population, 
       MAX(total_cases) AS highest_infection_count,
       MAX((total_cases::float / population)) * 100 AS percent_population_infected
FROM public.coviddeaths
GROUP BY continent, location, population
ORDER BY percent_population_infected DESC;
```
### 5. Highest Death Count by Continent
**Purpose**: Provide a continental overview of the pandemic’s deadliest regions.

**Query**: Determine the continent with the highest death count relative to the population.
```sql
SELECT continent, MAX(total_deaths) AS total_death_count
FROM public.coviddeaths
WHERE continent IS NOT NULL
GROUP BY continent
ORDER BY total_death_count DESC;
```
### 6. Global Case and Death Totals
**Purpose**: Provide a global perspective on the pandemic.

**Query**: Aggregate global numbers to calculate the overall death percentage.
```sql
SELECT 
    SUM(new_cases) AS total_cases,
    SUM(new_deaths) AS total_deaths, 
    CASE 
        WHEN SUM(new_cases) = 0 THEN 0
        ELSE SUM(new_deaths)::float / SUM(new_cases)::float * 100 
    END AS death_percentage
FROM public.coviddeaths
ORDER BY 1,2;
```
### 7. Vaccination Analysis
**Purpose**: Assess the progress of vaccination efforts globally.

**Query**: Analyze vaccination progress and calculate the percentage of vaccinated populations across different locations.
```sql
WITH population_vsvaccination AS (
    SELECT 
        death.date,
        death.continent,
        death.location,
        death.population,
        vac.new_vaccinations,
        SUM(vac.new_vaccinations) OVER (PARTITION BY death.location ORDER BY death.date) AS cumulative_vaccinations
    FROM 
        public.coviddeaths death
    JOIN 
        public.covidvaccinations vac
    ON 
        death.location = vac.location
    AND 
        death.date = vac.date
    WHERE 
        death.continent IS NOT NULL
    ORDER BY 
        death.continent, death.location
)
SELECT 
    continent,
    location,
    date,
    population,
    new_vaccinations,
    cumulative_vaccinations,
    (cumulative_vaccinations::float / population::float) * 100 AS vaccination_percentage
FROM 
    population_vsvaccination;

```
## Future Enhancements
- Add time-series analysis to track the progression of the pandemic.
- Integrate additional data sources for deeper insights.
- Create interactive dashboards to visualize the results.

## Conclusion
This project demonstrates my ability to work with real-world datasets and perform detailed analysis using SQL. It showcases skills in data manipulation, query optimization, and insight generation, which are essential for data-driven decision-making.

## Data Source
https://ourworldindata.org/explorers/coronavirus-data-explorer?zoomToSelection=true&time=2020-03-01..latest&facet=none&pickerSort=asc&pickerMetric=location&hideControls=false&Metric=Confirmed+deaths&Interval=7-day+rolling+average&Relative+to+Population=true&Color+by+test+positivity=false&country=IND~USA~GBR~CAN~DEU~FRA
