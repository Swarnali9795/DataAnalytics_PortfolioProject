# COVID-19 Data Exploration

Dive into the COVID-19 pandemic through data!
This project analyzes global COVID-19 trends including cases, deaths, vaccinations, and population impact. Using SQL, I explored infection rates, death percentages, vaccination progress, and country/continent-level comparisons.

SQL queries? Check them out here: [Covid_Project folder](/Covid_Project/).

---

# Background

The COVID-19 pandemic reshaped the world, and data became critical in tracking its spread and management. This project was created to analyze COVID-19 data and uncover insights such as infection rates, mortality risks, and vaccination coverage.

The dataset contains information on cases, deaths, and vaccinations across different countries and continents.
By leveraging SQL queries, I aimed to transform raw COVID-19 data into meaningful insights that highlight the global and regional impact of the pandemic.

### Key Questions I Wanted to Answer:

1. What percentage of people contracted COVID-19 across countries?
2. Which countries experienced the highest infection and death counts?
3. How did continents compare in terms of COVID-19 deaths?
4. What do global case and death numbers reveal about the pandemic’s severity?
5. What percentage of populations received COVID-19 vaccinations?

---

# Tools I Used

For this deep dive into COVID-19 data, I used several tools and techniques:

* **SQL:** The core of my analysis, including joins, CTEs, temp tables, and aggregate functions.
* **SQL Server Management Studio (SSMS):** The database management system for querying and managing large COVID-19 datasets.
* **Git and GitHub:** For version control and sharing queries in a structured project format.

---

# The Analysis

Each query was designed to answer one of the research questions above.

---

### 1. Total Cases vs. Total Deaths

To evaluate mortality risk, I calculated the percentage of deaths out of total cases for each country.

```sql
Select Location, date, total_cases, total_deaths, 
       (total_deaths/total_cases)*100 as DeathPercentage
From Portfolio_Project_SQL..CovidDeaths
Where location like '%india%'
  and continent is not null 
order by 1,2
```

**Insight:** In India, the mortality rate from COVID-19 steadily declined from around 1.44% in mid-January 2021 to about 1.10% by the end of April 2021, indicating improved survival outcomes despite rapidly increasing case counts.

---

### 2. Total Cases vs. Population

This analysis shows what percentage of a country’s population was infected.

```sql
Select Location, date, Population, total_cases,  
       (total_cases/population)*100 as PercentPopulationInfected
From Portfolio_Project_SQL..CovidDeaths
order by 1,2
```

**Insight:** The percentage of population infected varied widely across countries and time periods. Some countries experienced sharp early spikes, while others showed a slower, gradual increase. By looking at infection rates as a share of the population, we can better compare the relative impact of COVID-19 across different regions, beyond just raw case numbers.

---

### 3. Countries with Highest Infection Rates

To identify the hardest-hit countries relative to population:

```sql
Select Location, Population, 
       MAX(total_cases) as HighestInfectionCount,  
       Max((total_cases/population))*100 as PercentPopulationInfected
From Portfolio_Project_SQL..CovidDeaths
Group by Location, Population
order by PercentPopulationInfected desc
```

**Insight:** Small nations like Andorra, Montenegro, and San Marino recorded the highest infection percentages, with more than 14–17% of their populations infected at peak. Larger countries such as the United States and Israel also crossed ~10% infection rates, showing that both small and large nations were heavily impacted.

---

### 4. Countries with Highest Death Counts

```sql
Select Location, MAX(cast(Total_deaths as int)) as TotalDeathCount
From Portfolio_Project_SQL..CovidDeaths
Where continent is not null 
Group by Location
order by TotalDeathCount desc
```

**Insight:** The highest absolute death counts were concentrated in large, populous nations - the United States, Brazil, Mexico, and India leading the list. This highlights how raw death numbers often reflect population size and scale of outbreaks, rather than the relative severity seen in smaller countries.

---

### 5. Continent-Level Breakdown

```sql
Select continent, MAX(cast(Total_deaths as int)) as TotalDeathCount
From Portfolio_Project_SQL..CovidDeaths
Where continent is not null 
Group by continent
order by TotalDeathCount desc
```

**Insight:** Europe followed by North America showed the highest death counts at a continental level.

---

### 6. Global Numbers

```sql
Select SUM(new_cases) as total_cases, 
       SUM(cast(new_deaths as int)) as total_deaths, 
       SUM(cast(new_deaths as int))/SUM(New_Cases)*100 as DeathPercentage
From Portfolio_Project_SQL..CovidDeaths
where continent is not null 
```

**Insight:** Globally, there were about 150 million cases and over 3.18 million deaths, putting the global death percentage around 2.1%. This measure gives a sense of the worldwide mortality risk, while masking the huge variation across countries and regions.

---

### 7. Vaccination Progress

By joining vaccination data with deaths data, I tracked cumulative vaccinations using window functions.

```sql
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
       SUM(CONVERT(int,vac.new_vaccinations)) 
         OVER (Partition by dea.Location Order by dea.location, dea.Date) 
         as RollingPeopleVaccinated
From Portfolio_Project_SQL..CovidDeaths dea
Join Portfolio_Project_SQL..CovidVaccinations vac
     On dea.location = vac.location
    and dea.date = vac.date
where dea.continent is not null 
order by 2,3
```

**Insight:** Vaccination rollouts showed striking contrasts. Some countries like Uruguay ramped up quickly, with hundreds of thousands vaccinated within weeks, while others like Zimbabwe saw a more gradual rise. Tracking cumulative vaccinations reveals how differently countries managed to scale immunization efforts during the same period.

---

### 8. Using CTEs and Temp Tables for Vaccination Analysis

I used **CTEs** and **Temp Tables** to calculate vaccination percentages more cleanly and reuse the logic in later steps.

```sql
With PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
as
(
   ...
)
Select *, (RollingPeopleVaccinated/Population)*100
From PopvsVac
```

```sql
DROP Table if exists #PercentPopulationVaccinated
Create Table #PercentPopulationVaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_vaccinations numeric,
RollingPeopleVaccinated numeric
)

Insert into #PercentPopulationVaccinated
    ...
Select *, (RollingPeopleVaccinated/Population)*100
From #PercentPopulationVaccinated
```

---

### 9. Creating a View

Finally, I created a **View** for persistent storage, making it easier to build dashboards and visualizations later.

```sql
Create View PercentPopulationVaccinated as
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
       SUM(CONVERT(int,vac.new_vaccinations)) 
         OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
From Portfolio_Project_SQL..CovidDeaths dea
Join Portfolio_Project_SQL..CovidVaccinations vac
     On dea.location = vac.location
    and dea.date = vac.date
where dea.continent is not null
```

---

# What I Learned

Through this project, I:

* **Strengthened my SQL skills** by applying joins, window functions, aggregates, and filtering.
* **Learned to use CTEs and Temp Tables** for cleaner, reusable calculations.
* **Gained practice in creating Views** for storing results for future visualization.
* Improved my ability to translate **real-world data into insights** using structured queries.

---

# Conclusions

### Insights

1. **Infection & Mortality Patterns:** Smaller countries like Andorra and San Marino experienced the highest infection rates relative to population, whereas larger countries such as the United States, Brazil, and India accounted for the largest absolute death counts, highlighting how population size shapes the impact.
2. **Continent-Level Patterns:** Europe and North America consistently reported higher death totals, reflecting both outbreak scale and population density differences.
3. **Global Overview:** While global death percentages remained relatively low (~2%), the absolute number of lives lost underscores the serious worldwide impact of the pandemic.
4. **Vaccination Rollouts:** Vaccination progress was uneven across countries; some nations achieved rapid coverage, whereas others showed slower, incremental uptake, emphasizing disparities in vaccine access and distribution strategies.

### Closing Thoughts

This project gave me a chance to work with **real-world health data**, explore the power of SQL for analysis, and understand how global events unfold through numbers.
It also reinforced the importance of **data storytelling**, where queries transform into insights that reflect humanity’s collective experience.

---
