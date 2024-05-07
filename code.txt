select * from Data1
select * from project.dbo.Data2

-- no of rows into our dataset
select count(*) from project..Data1
select count(*) from project..Data1

-- dataset for jharkhand and bihar
select * from Data1 where state in ('Jharkhand', 'bihar')

-- population of India
select sum(Population) as Population from Data2

-- avg growth
select avg(Growth)*100 as avg_growth from Data1

-- avg growth by state in asc orderby growth
select state, avg(Growth)*100 as avg_growth from Data1 group by state order by avg_growth

-- avg sex ratio / using round func to remove decimal no
select state, round(avg(Sex_Ratio),0) as avg_Sex_Ratio from Data1 group by state order by avg_Sex_Ratio desc

-- avg litreacy rate
select state, round(avg(Literacy),0) as avg_Literacy from Data1 group by state having round(avg(Literacy),0)>90  order by avg_Literacy desc

--top 3 states showing highest growth ratio
select top 3 state, avg(Growth)*100 as avg_growth from Data1 group by state order by avg_growth desc

--bottom 3 states showing lowest sex ratio
select top 3 state, round(avg(Sex_Ratio),0) as avg_Sex_Ratio from Data1 group by state order by avg_Sex_Ratio asc 

-- top & bottom 3 states in literacy rate
drop table if exists #topstates
create table #topstates
(State nvarchar(255),
topstates float(8))

insert into #topstates
select top 3 state, round(avg(Literacy),0) as avg_Literacy from Data1 group by state order by avg_Literacy desc

select * from #topstates order by #topstates.topstates desc;

drop table if exists #bottomstates
create table #bottomstates
(State nvarchar(255),
bottomstates float(8))

insert into #bottomstates
select top 3 state, round(avg(Literacy),0) as avg_Literacy from Data1 group by state order by avg_Literacy asc

select * from #bottomstates order by #bottomstates.bottomstates asc;

select * from (select top 3 * from #topstates order by #topstates.topstates desc) a
union
select * from(select top 3 * from #bottomstates order by #bottomstates.bottomstates asc) b ;

-- states starting with letter a
select distinct state from project..data1 where lower(state) like 'a%' or lower(state) like 'b%'
select distinct state from project..data1 where lower(state) like 'a%' or lower(state) like '%m'

--joining both table to calculate no of males and females for district level 

select c.district, c.state, round((c.population/(c.sex_ratio + 1)), 0)males, round(((c.population*c.sex_ratio)/(c.sex_ratio+1)),0) females from
(select a.district, a.state, a.sex_ratio/1000 sex_ratio, b.population from project..Data1 a inner join project..Data2 b on a.District=b.District) c


--joining both table to calculate no of males and females for state level, using sum function as GROUP BY statement is  used with aggregate functions

select  d.state, sum(d.males) Total_Males, sum(d.females) Total_Females from
(select c.district, c.state, round((c.population/(c.sex_ratio + 1)), 0)males, round(((c.population*c.sex_ratio)/(c.sex_ratio+1)),0) females from
(select a.district, a.state, a.sex_ratio/1000 sex_ratio, b.population from project..Data1 a inner join project..Data2 b on a.District=b.District) c) d 
group by d.State

--total litreacy rate
select a.district, a.state, a.Literacy/100 Litreacy_ratio, b.population from project..Data1 a inner join project..Data2 b on a.District=b.District

--no of literate and illiterate people for district level

select d.district, d.state, round((d.Literacy_ratio*d.population),0) literate_people , round(((1- d.Literacy_ratio)*d.population),0) illiterate_people from
(select a.district, a.state, a.Literacy/100 Literacy_ratio, b.population from project..Data1 a inner join project..Data2 b on a.District=b.District) d
order by District 

--no of literate and illiterate people for state level

select e.state, sum(literate_people) Total_literate_people, sum(illiterate_people) Total_illiterate_people from
(select d.district, d.state, round((d.Literacy_ratio*d.population),0) literate_people , round(((1- d.Literacy_ratio)*d.population),0) illiterate_people from
(select a.district, a.state, a.Literacy/100 Literacy_ratio, b.population from project..Data1 a inner join project..Data2 b on a.District=b.District) d) e
group by e.State

--population in previous census
select e.state,sum( e.previous_census_population) previous_census_population, sum(e.current_census_population) current_census_population from
(select d.district, d.state,round(( d.Population/(1+d.growth)),0) previous_census_population, population current_census_population from
(select a.district, a.state, a.Growth, b.population from project..Data1 a inner join project..Data2 b on a.District=b.District) d) e
group by e.State

--total population in India in previous census and current census
select sum(m.previous_census_population) previous_census_population, sum(m.current_census_population) current_census_population from
(select e.state,sum( e.previous_census_population) previous_census_population, sum(e.current_census_population) current_census_population from
(select d.district, d.state,round(( d.Population/(1+d.growth)),0) previous_census_population, population current_census_population from
(select a.district, a.state, a.Growth, b.population from project..Data1 a inner join project..Data2 b on a.District=b.District) d) e
group by e.State) m

--population vs area

select g.total_area/g.previous_census_population previous_census_population_vs_area, g.total_area/g.current_census_population current_census_population_vs_area from
(select q.*, r.total_area from (

select '1' as keyy, n.* from 
(select sum(m.previous_census_population) previous_census_population, sum(m.current_census_population) current_census_population from
(select e.state,sum( e.previous_census_population) previous_census_population, sum(e.current_census_population) current_census_population from
(select d.district, d.state,round(( d.Population/(1+d.growth)),0) previous_census_population, population current_census_population from
(select a.district, a.state, a.Growth, b.population from project..Data1 a inner join project..Data2 b on a.District=b.District) d) e
group by e.State) m) n) q inner join (

select '1' as keyy, z.* from  
(select sum(Area_km2) total_area from project..Data2) z) r on q.keyy=r.keyy) g

--window function, output top three districts from each state with highest litreacy rates

select a.* from
(select District, State, Literacy,
RANK() OVER (PARTITION BY state ORDER BY Literacy DESC ) rnk
from project..Data1) a

where rnk in (1,2,3) order by State