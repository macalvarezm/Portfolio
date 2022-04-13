Select *
From PortfolioProject..[Covid Deaths]
Where continent is not null
Order by 3,4

--Selección de información que se va a usar--
Select location, date, total_cases, new_cases, total_deaths, population
From PortfolioProject..[Covid Deaths]
Where continent is not null
Order by 1,2

-- Casos totales vs Muertes totales--
-- Probabilidad de morir de covid si se contrae covid en Argentina
Select location, date, total_cases, new_cases, total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
From PortfolioProject..[Covid Deaths]
Where location = 'Argentina'
and continent is not null
Order by 1,2 desc

-- Probabilidad de morir de covid si se contrae covid en Uruguay
Select location, date, total_cases, new_cases, total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
From PortfolioProject..[Covid Deaths]
Where location = 'Uruguay'
and continent is not null
Order by 1,2 desc

--Casos totales vs población

--Muestra el porcentaje de la población infectada en Argentina
 
Select location, date, total_cases, new_cases, population, (total_cases/population)*100 as PercentPopulationInfected
From PortfolioProject..[Covid Deaths]
Where location = 'Argentina'
and continent is not null
Order by 1,2

--Muestra el porcentaje de la población infectada en Uruguay
 
Select location, date, total_cases, new_cases, population, (total_cases/population)*100 as PercentPopulationInfected
From PortfolioProject..[Covid Deaths]
Where location = 'Uruguay'
and continent is not null
Order by 1,2

--Países con mayor tasa de infección comparado con la población--
Select location, population , MAX(total_cases) as HighestInfectionCount, max((total_cases/population))*100 as PercentPopulationInfected
From PortfolioProject..[Covid Deaths]
--Where location = 'Argentina'
Where continent is not null
Group by location, population
Order by PercentPopulationInfected desc

--Países con mayor cantidad de muertes por población
Select location, MAX(cast(total_deaths as int)) as TotalDeathsCount
From PortfolioProject..[Covid Deaths]
--Where location like '%Argentina%'--
Where continent is not null
Group by location
Order by TotalDeathsCount desc

--Análisis por continente--

--Continente con la mayor cantidad de muertos por población
Select continent, MAX(cast(total_deaths as int)) as TotalDeathsCount
From PortfolioProject..[Covid Deaths]
--Where location like '%Argentina%'--
Where continent is not null
Group by continent
Order by TotalDeathsCount desc

--País de América del Sur con la mayor cantidad de muertos 
Select location, Max(cast(total_deaths as int)) as TotalDeathsCount
From PortfolioProject..[Covid Deaths]
Where continent ='South America' 
Group by location
Order by TotalDeathsCount desc


-- En el mundo--
Select SUM(new_cases) as total_cases, SUM(cast(new_deaths as int)) as total_deaths, 
SUM(cast(new_deaths as int))/SUM(New_Cases)*100 as DeathPercentage
From PortfolioProject..[Covid Deaths]
--Where location like '%Argentina%'
where continent is not null 
--Group By date
order by 1,2

-- Cantidad de vacunas aplicadas en el mundo
Select location, sum(cast(total_vaccinations as bigint) ) as total_vaccinationscount
From PortfolioProject..[Covid Vaccinations]
Where location= 'World'
Group by location

-- Cantidad de vacunas aplicadas por país
Select continent, location, sum(cast(total_vaccinations as bigint) ) as total_vaccinationscount , sum(cast(new_vaccinations as bigint)) as total_newvac
From PortfolioProject..[Covid Vaccinations]
Where continent is not null
Group by location, continent
Order by total_vaccinationscount desc

--Población total vs Vacunación
--Porcentaje de la población que recibió al menos una dosis
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(bigint,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as Rolling_PeopleVaccinated
From PortfolioProject..[Covid Deaths] dea
Join PortfolioProject..[Covid Vaccinations] vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
order by 2,3

--CTE 
With popvsvac (Continent, location, Date, Population, New_vaccinations, Rolling_PeopleVaccinated) as
(
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(bigint,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as Rolling_PeopleVaccinated
From PortfolioProject..[Covid Deaths] dea
Join PortfolioProject..[Covid Vaccinations] vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
)
Select*, (Rolling_PeopleVaccinated/Population)*100 as PopvsVac
From popvsvac
Where location='Argentina'
Order by Date desc

--Tabla temporal--
 DROP TABLE IF EXISTS #percentpopulationvaccinated

 CREATE TABLE #percentpopulationvaccinated (
 Continent nvarchar(255),
 Location nvarchar(255),
 Date datetime,
 Population numeric,
 New_vaccinations numeric,
 RollingPeopleVaccinated numeric
 )

 INSERT INTO #percentpopulationvaccinated 
 SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM(CONVERT(bigint,new_vaccinations)) OVER (PARTITION BY dea.location ORDER BY dea.date)
AS RollingPeopleVaccinated
FROM PortfolioProject..[Covid Deaths] dea
JOIN PortfolioProject..[Covid Vaccinations] vac
ON dea.location = vac.location
AND dea.date = vac.date
WHERE dea.continent is not null

SELECT *, (RollingPeopleVaccinated/Population)*100 FROM #percentpopulationvaccinated 
WHERE Location = 'Argentina'
Order by Date desc


--Creando vistas
Create View 
PorcentajePoblaciónVacunada as
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(bigint,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From PortfolioProject..[Covid Deaths] dea
Join PortfolioProject..[Covid Vaccinations] vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 

/*
Select *
From PorcentajePoblaciónVacunada 
/*
