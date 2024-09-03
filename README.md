# Data Harmonization
Standardizing OpenMRS HIV Patient Data to OMOP CDM for Enhanced Analytics and Patient-Level Insights

* **Data source** : [OpenMRS Demo data]( https://openmrs.atlassian.net/wiki/spaces/RES/pages/26273323/Demo+Data "Chose large-demo-data-2-2-1.sql.gz")
* **Tools used** : SQL in MySQL and PostgreSQL Databases ,Python ,Pentaho , [OHDSI Tools](https://www.ohdsi.org/software-tools/) (White-rabbit, Rabbit-in-a-Hat, ATLAS, Achilles, WebAPI, Athena, Data Quality Dashboard)

## Introduction
**OpenMRS**, an open-source electronic medical record system designed for resource-constrained environments, was developed collaboratively by a global community of developers, healthcare professionals, and organizations including Partners In Health and the Regenstrief Institute. It aims to enhance healthcare delivery by providing a customizable platform for managing patient health information. Effective for local data management, but lacks standardization for broader research and interoperability.

This is where the Observational Health Data Sciences and Informatics (OHDSI) community comes in. OHDSI focuses on improving health outcomes through large-scale observational studies by utilizing the OMOP Common Data Model (CDM). The OMOP CDM provides a standardized format for harmonizing diverse healthcare data, enabling consistent and reproducible research.

Transforming OpenMRS data into the OMOP CDM format is crucial for achieving interoperability, allowing data to be integrated, analyzed, and visualized using OHDSI tools like ATLAS. This process not only enhances the utility of the data but also supports advanced analytics, leading to better patient care and more informed decision-making in healthcare.

## Study Questions
1. What is the structure and format of the source data being the OpenMRS and the residing HIV Patients data?
2. What is the structure of the data destination, that is, the OMOP Common Data Model?
3. How exactly will data be extracted, loaded and transformed from source to destination ?
4. How will the data be checked to ensure that the destination data matches the source?
5. Finally how will the results be prepared ready for different stakeholders ?

## Results and Outcomes
1. ETL Pipeline
   * Logical Mapping using OHDSI Tools White-Rabbit and Rabbit-in-a-Hat
   * ETL pipeline using Pentaho moving data from OpenMRS MySQL Database into OMOP CDM format in PostgreSQL Database.

![ETL Pipeline](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ETL%20pipeline.png "ETL Pipeline")

Explanation



https://github.com/user-attachments/assets/6975999c-6ca8-414b-b3f9-b2e7cba08306



2. ATLAS Dashboards

https://github.com/user-attachments/assets/9937a4e6-ccab-42bd-a1a5-8456b1e9efc7

## Activities done

![Roadmap](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/Activities.png "Roadmap")

1. Exploratory Data Analysis

```sql
-/* The SQL queres are used to understand Demographics data for Patients in OpenMRS to be used for the project
 * 
 * 
 */

-- Number of People in OpenMRS
-- Count : 5,286
select count(*) as 'Number of People' from person p where voided = 0

-- What about Number of Patients
-- Count : 5,284
-- Comment: Only 2 are not Patients
select count(*) as 'Number of Patients' from patient p where voided = 0

-- Finding People who are not Patients
-- The People are of `person_id` 1 and 9,347
select *
from person
where person_id NOT IN (
    select patient_id 
    from patient 
) and voided = 0;

-- Seeing Patient programs available
-- `HIV Program` with program_id 3 , `TB Program` with program_id 4
select distinct pp.program_id, p.name as 'program name'
from patient_program pp join program p on pp.program_id = p.program_id 
where pp.voided = 0 and p.retired = 0


-- What is the Patients Count per Program
-- HIV Program - 3,100
-- TB Program - 821
select distinct p2.name as program, count(distinct p.patient_id) as 'patient count'
from patient p 
join patient_program pp on p.patient_id = pp.patient_id 
join program p2 on pp.program_id = p2.program_id 
where p.voided = 0 and pp.voided = 0 and p2.retired = 0
group by p2.name

-- All Locations in OpenMRS
-- Have "Wishard Hospital" of location_id 2, "Mosoriot Hospital" of 3, "Chulaimbo" of 4, and "Unknown Location"(s)
select * from location l where retired = 0

-- Patients Count per Location where Patient_program is "HIV Program"
-- Wishard Hospital : 3,098
-- Unknown Location : 1
select distinct l.name, count(distinct p.person_id) as 'patient count'
from person p
join patient p2 on p.person_id = p2.patient_id
join patient_identifier pi on p.person_id = pi.patient_id 
join location l on pi.location_id = l.location_id
join patient_program pp on pp.patient_id = p.person_id 
join program p3 on p3.program_id = pp.program_id 
where p.voided = 0 and p3.name = 'HIV PROGRAM' and p2.voided = 0 
and pi.voided = 0 and l.retired is not null and pp.voided = 0 and p3.retired = 0
group by name


-- Patient Count within "HIV Program" from "Wishard Hospital" with Obs
-- 3, 097
select count(distinct p.person_id) as 'Patients with obs'
from person p
join patient p2 on p.person_id = p2.patient_id
join patient_identifier pi on p.person_id = pi.patient_id 
join location l on pi.location_id = l.location_id
join patient_program pp on pp.patient_id = p.person_id 
join program p3 on p3.program_id = pp.program_id 
join obs o on p.person_id = o.person_id 
where p.voided = 0 and p3.name = 'HIV PROGRAM' and p2.voided = 0 and l.name = 'Wishard Hospital'
and pi.voided = 0 and l.retired is not null and o.voided = 0 and pp.voided = 0 and p3.retired = 0

-- Patient Count within "HIV Program" from "Wishard Hospital" without Obs
-- 0
-- Explanation: The Obs without a Patient is voided
select  p.*
from person p
join patient p2 on p.person_id = p2.patient_id
join patient_identifier pi on p.person_id = pi.patient_id 
join location l on pi.location_id = l.location_id
join patient_program pp on pp.patient_id = p.person_id 
join program p3 on p3.program_id = pp.program_id 
left join obs o on p.person_id = o.person_id 
where p.voided = 0 
  and p3.name = 'HIV PROGRAM' 
  and p2.voided = 0 
  and l.name = 'Wishard Hospital'
  and pi.voided = 0 
  and l.retired is not null 
  and (o.person_id is null)
  and pp.voided = 0
  and p3.retired = 0
  and o.voided = 0
 ;
 
 
-- Obs datetime distribution: min, q1, median, q3, maximum
-- min		: 1899
-- q1  		: 2006
-- median	: 2006
-- q3		: 2006
-- max		: 5006
-- Comment	: The Datetime 5006 is wrong whereas datetime 1899 seems like a long time ago
-- This needs to be investigated further
 
with ordered_obs as (
    select 
        o.obs_datetime,
        row_number() over (order by o.obs_datetime) as rn,
        count(*) over () as total_count
    from person p
	join patient p2 on p.person_id = p2.patient_id
	join patient_identifier pi on p.person_id = pi.patient_id 
	join location l on pi.location_id = l.location_id
	join patient_program pp on pp.patient_id = p.person_id 
	join program p3 on p3.program_id = pp.program_id 
	join obs o on p.person_id = o.person_id
	where p.voided = 0 and p3.name = 'HIV PROGRAM' and p2.voided = 0 and l.name = 'Wishard Hospital'
	and pi.voided = 0 and l.retired is not null and o.voided = 0 and pp.voided = 0 and p3.retired = 0
)
select 
    min(obs_datetime) as min,
    max(case when rn = floor(0.25 * total_count) then obs_datetime end) as q1,
    max(case when rn = floor(0.5 * total_count) then obs_datetime end) as median,
    max(case when rn = floor(0.75 * total_count) then obs_datetime end) as q3,
    max(obs_datetime) as max
from ordered_obs;

-- Checking distinct years in obs_datetime of obs
-- The years are 1899, 2004, 2005, 2006, 5006
-- The right ones here look like 2004, 2005 and 2006 and are the only ones to be considered

select distinct year(obs_datetime) as year
from person p
join patient p2 on p.person_id = p2.patient_id
join patient_identifier pi on p.person_id = pi.patient_id 
join location l on pi.location_id = l.location_id
join patient_program pp on pp.patient_id = p.person_id 
join program p3 on p3.program_id = pp.program_id 
join obs o on p.person_id = o.person_id 
where p.voided = 0 and p3.name = 'HIV PROGRAM' and p2.voided = 0 and l.name = 'Wishard Hospital'
and pi.voided = 0 and l.retired is not null and o.voided = 0 and pp.voided = 0 and p3.retired = 0
order by year(obs_datetime);


-- Also checking Obs total count based on the Years extracted from obs_datetime
-- 'Obs Count for Year 1899' : 3
-- 'Obs Count for Year 2004' : 3
-- 'Obs Count for Year 2005' : 198
-- 'Obs Count for Year 2006' : 282,266
-- 'Obs Count for Year 5006' : 1
select
    count(case when year(o.obs_datetime) = 1899 then 1 end) as 'Obs Count for Year 1899',
    count(case when year(o.obs_datetime) = 2004 then 1 end) as 'Obs Count for Year 2004',
    count(case when year(o.obs_datetime) = 2005 then 1 end) as 'Obs Count for Year 2005',
    count(case when year(o.obs_datetime) = 2006 then 1 end) as 'Obs Count for Year 2006',
    count(case when year(o.obs_datetime) = 5006 then 1 end) as 'Obs Count for Year 5006'
from person p
join patient p2 on p.person_id = p2.patient_id
join patient_identifier pi on p.person_id = pi.patient_id 
join location l on pi.location_id = l.location_id
join patient_program pp on pp.patient_id = p.person_id 
join program p3 on p3.program_id = pp.program_id 
join obs o on p.person_id = o.person_id 
where p.voided = 0 and p3.name = 'HIV PROGRAM' and p2.voided = 0 and l.name = 'Wishard Hospital'
and pi.voided = 0 and l.retired is not null and o.voided = 0 and pp.voided = 0 and p3.retired = 0

-- NOTE: The wrong values cannot be deleted as this is clinical data. The best way of dealing with such values is by changing the entry to voided
/* The syntax for the change is
 * 
 * UPDATE obs set voided = 1 on obs_datetime < 2004 or obs_datetime > 2006;
 * 
 */


-- Now Patients birthdate has to be checked and compared to Obs dates if they match
-- Patients birthdate
-- min		: 1950
-- q1  		: 1965
-- median	: 1972
-- q3		: 1980
-- max		: 2003
-- Comment	: Meaning that all obs with datetime < 1950 are wrong because the Patients were not born yet

with ordered_obs as (
    select 
        p.birthdate,
        row_number() over (order by p.birthdate) as rn,
        count(*) over () as total_count
    from person p
	join patient p2 on p.person_id = p2.patient_id
	join patient_identifier pi on p.person_id = pi.patient_id 
	join location l on pi.location_id = l.location_id
	join patient_program pp on pp.patient_id = p.person_id 
	join program p3 on p3.program_id = pp.program_id 
	join obs o on p.person_id = o.person_id
	where p.voided = 0 and p3.name = 'HIV PROGRAM' and p2.voided = 0 and l.name = 'Wishard Hospital'
	and pi.voided = 0 and l.retired is not null and o.voided = 0 and pp.voided = 0 and p3.retired = 0
)
select 
    min(year(birthdate)) as min,
    max(case when rn = floor(0.25 * total_count) then year(birthdate) end) as q1,
    max(case when rn = floor(0.5 * total_count) then year(birthdate) end) as median,
    max(case when rn = floor(0.75 * total_count) then year(birthdate) end) as q3,
    max(year(birthdate)) as max
from ordered_obs;

-- Person's Birthdate total count based on the Years extracted from Birthdate
-- birth_year < 1960				: 241
-- birth_year between 1960 and 1969	: 1,072
-- birth_year between 1970 and 1979	: 1,000
-- birth_year between 1980 and 1989	: 281
-- birth_year > 1990				: 472


-- Subquery to get distinct persons and their birth years
with distinct_persons as (
    select distinct
        p.person_id,
        year(p.birthdate) as birth_year
    from person p
    join patient p2 on p.person_id = p2.patient_id
    join patient_identifier pi on p.person_id = pi.patient_id 
    join location l on pi.location_id = l.location_id
    join patient_program pp on pp.patient_id = p.person_id 
    join program p3 on p3.program_id = pp.program_id 
    join obs o on p.person_id = o.person_id 
   	where p.voided = 0 and p3.name = 'HIV PROGRAM' and p2.voided = 0 and l.name = 'Wishard Hospital'
	and pi.voided = 0 and l.retired is not null and o.voided = 0 and pp.voided = 0 and p3.retired = 0
)
-- Main query to count distinct persons by birth year range
select
    count(case when birth_year < 1960 then 1 end) as 'Birthdate for Year < 1960',
    count(case when birth_year between 1960 and 1969 then 1 end) as 'Birthdate for Year 1960-1969',
    count(case when birth_year between 1970 and 1979 then 1 end) as 'Birthdate for Year 1970-1979',
    count(case when birth_year between 1980 and 1989 then 1 end) as 'Birthdate for Year 1980-1989',
    count(case when birth_year > 1990 then 1 end) as 'Birthdate for Year > 1990'
from distinct_persons;

-- Patients Age distribution
select distinct p.gender, count(distinct p.person_id) 
from person p
join patient p2 on p.person_id = p2.patient_id
join patient_identifier pi on p.person_id = pi.patient_id 
join location l on pi.location_id = l.location_id
join patient_program pp on pp.patient_id = p.person_id 
join program p3 on p3.program_id = pp.program_id 
join obs o on p.person_id = o.person_id 
where p.voided = 0 and p3.name = 'HIV PROGRAM' and p2.voided = 0 and l.name = 'Wishard Hospital'
and pi.voided = 0 and l.retired is not null and o.voided = 0 and pp.voided = 0 and p3.retired = 0
group by p.gender 

-- Counting Dead Patients
 -- 0 Patients Dead
 select * from person p join patient p2 on p.person_id = p2.patient_id where p.dead = 1
```

2.  ETL pipeline




3. Data Quality Check
   
5. Visualization of Results
   Patient-level analysis evidence generated in a dashboard format using an OHDSI tool called ATLAS.



