# Covid19-data-analysis-with-SQL
## Table of content
- [Project Overview](#project-overview)
- [Data Source](#data-source)
- [Tools used](#tools-used)
- [Analysis](#analysis)
  - [Import data](#import-data)
  - [Data analysis](#data-analysis)
- [Result and Findings](#result-and-findings)
  
### Project Overview
This project is to show case my SQL knowledge by analyze Covid19 data using SQL tools such as creating database, import data, creating view, query using aggregated functions and joining tables.

### Data Source
https://ourworldindata.org/covid-deaths

### Tools used
- Excel
- SQL Server management studio

### Analysis
#### Import data

1.	Download data from https://ourworldindata.org/covid-deaths
2.	Open download file and save as excel workbook files (to import into database)
3.	Create database named "Covid19 data cleaning project"
4.	Import 2 tables to database.
- ![image](https://github.com/nghphucthinh/Covid19-data-cleaning-and-exploration-with-SQL/assets/89053686/193205ad-a81c-4fb2-9105-7e0339e62ddf)
5. View table design to check columns data type
- ![image](https://github.com/nghphucthinh/Covid19-data-cleaning-and-exploration-with-SQL/assets/89053686/4e8afce3-658e-4de9-bcb3-840589ea407f)
6. Remove the rows with total_cases is null and total_death is higher than total_cases
```sql
DELETE FROM [Covid19 data cleaning project]..CovidDeaths$ WHERE total_cases IS NULL; --Delete rows where total_cases are null
DELETE FROM [Covid19 data cleaning project]..CovidDeaths$ WHERE cast(total_cases as int) < cast(total_deaths as int); --Delete rows where total_cases are less than total_deaths
``` 

#### Data analysis

1. Check the fatality of infected cases

```sql
select distinct(location), cast(total_cases as float), total_deaths, round((cast(total_deaths as float)/cast(total_cases as float))*100,4) as Death_rate
from [Covid19 data cleaning project]..CovidDeaths$
where continent is not null
order by 1,2
```
![image](https://github.com/nghphucthinh/Covid19-data-cleaning-and-exploration-with-SQL/assets/89053686/863dd817-09d7-4505-9c0e-731ec20b8da6)

2. Check the infection rate
```sql
select distinct(location), population, cast(total_cases as float) as total_cases,total_deaths, round((cast(total_cases as float)/population)*100,4) as Infection_rate
from [Covid19 data cleaning project]..CovidDeaths$
where continent is not null
order by 1,3
```
![image](https://github.com/nghphucthinh/Covid19-data-cleaning-and-exploration-with-SQL/assets/89053686/bcb1fe70-03d1-4999-b1bd-2dc6dc75a6f4)

3.  Country with highest infection rate 
```sql
select location, population, max(cast(total_cases as float)) as Highest_Infection_count, max(round((cast(total_cases as float)/population)*100,4)) as Infection_rate
from [Covid19 data cleaning project]..CovidDeaths$
where continent is not null
Group by location, population
order by Infection_rate desc
```
![image](https://github.com/nghphucthinh/Covid19-data-cleaning-and-exploration-with-SQL/assets/89053686/317f67eb-886d-4359-b14b-ecaf2952cad0)

4. Countries with highest death rate
```sql
With Highest_cases_and_death(location, Highest_case_count, Highest_Death_count)
 as(
select location, max(cast(total_cases as float)) as Highest_case_count, max(cast(total_deaths as float)) as Highest_Death_count
from [Covid19 data cleaning project]..CovidDeaths$
where continent is not null
group by location
)
select *, round((Highest_Death_count/Highest_case_count)*100, 4) as Death_rate
from Highest_cases_and_death
order by Death_rate desc
```
![image](https://github.com/nghphucthinh/Covid19-data-cleaning-and-exploration-with-SQL/assets/89053686/4696d389-82e8-42a2-bdfe-51749bb03ea4)

5. Infection rate by level of income
```sql
select location, population, max(cast(total_cases as float)) as Highest_Cases_count ,max(round((cast(total_cases as float)/population)*100,4)) as Infection_rate
from [Covid19 data cleaning project]..CovidDeaths$
where location like '%income%'
Group by location, population
order by Infection_rate desc
```
![image](https://github.com/nghphucthinh/Covid19-data-cleaning-and-exploration-with-SQL/assets/89053686/54e65caa-5463-4dab-815a-f4c3a1c72026)

6. Death rate by level of income
```sql
with Highest_cases_and_death(location, Highest_case_count, Highest_Death_count)
 as(
select location, max(cast(total_cases as float)) as Highest_case_count, max(cast(total_deaths as float)) as Highest_Death_count
from [Covid19 data cleaning project]..CovidDeaths$
where location like '%income%'
group by location
)
select *, round((Highest_Death_count/Highest_case_count)*100, 4) as Death_rate
from Highest_cases_and_death
order by Death_rate desc
```
![image](https://github.com/nghphucthinh/Covid19-data-cleaning-and-exploration-with-SQL/assets/89053686/f3d0893d-860b-4f92-94d9-69d279e9945f)

7. Infection rate by continent
```sql
select location, population, max(cast(total_cases as float)) as Highest_Cases_count ,max(round((cast(total_cases as float)/population)*100,4)) as Infection_rate
from [Covid19 data cleaning project]..CovidDeaths$
where location not like '%income%' and location <> 'World' and location not like '%Union%' and continent is null
Group by location, population
order by Infection_rate desc
```
![image](https://github.com/nghphucthinh/Covid19-data-cleaning-and-exploration-with-SQL/assets/89053686/9e1242b0-9a35-4cde-953e-f18e09d5ae37)

8. Death rate by continent
```sql
with Highest_cases_and_death(location, Highest_case_count, Highest_Death_count)
 as(
select location, max(cast(total_cases as float)) as Highest_case_count, max(cast(total_deaths as float)) as Highest_Death_count
from [Covid19 data cleaning project]..CovidDeaths$
where location not like '%income%' and location <> 'World' and location not like '%Union%' and continent is null
group by location
)
select *, round((Highest_Death_count/Highest_case_count)*100, 4) as Death_rate
from Highest_cases_and_death
order by Death_rate desc
```
![image](https://github.com/nghphucthinh/Covid19-data-cleaning-and-exploration-with-SQL/assets/89053686/367adc5d-a43f-405f-b3e8-f5bb1dfbf25c)

7. Check vaccinated percentage by date in each country
```sql
With PopulationVsVaccinated (Continent,Location, Date, Population,New_vaccinated, VaccinatedByDate)
as (
select death.continent, death.location, death.date,death.population, vaccine.new_vaccinations,
	sum(cast(vaccine.new_vaccinations as float)) over (partition by death.location order by death.location, death.date) as VaccinatedByDate	
from [Covid19 data cleaning project]..CovidDeaths$ death join [Covid19 data cleaning project]..CovidVaccination$ vaccine
	on death.location = vaccine.location and death.date = vaccine.date 
where death.continent is not null
)
select *, round((VaccinatedByDate/population)*100, 4) as VaccinatedPercentage
from PopulationVsVaccinated
where New_vaccinated is not null and  VaccinatedByDate is not null
```
![image](https://github.com/nghphucthinh/Covid19-data-cleaning-and-exploration-with-SQL/assets/89053686/f7779c6a-7354-477e-881d-082375cdcce2)

### Result and Findings

- The country with highest infection rate (total cases over population) was Brunei with 76.5%, follow by Cyprus and San Marino with 76% and 75%
- The higher income countries tend to have more infection rate compared to lower income countries. This might be due to the high population density of those countries or many lower income countries don’t have the accurate record of those who are infected. However, this ranking is reverse when it comes to death rate, as the lower income the countries have the higher their death rate gets. 
- Out of the 6 continent Europe and Oceania has the most infection rate (33.87% and 32.84%), significantly more than the 3rd continent with highest infection rate which is North America with 20.74%. Same behavior with income level, when it comes to death rate, the ranking is reverse compared to infection rate, where countries with most infected rate have the least death rate. 

