# Data Harmonization
Standardizing OpenMRS HIV Patient Data to OMOP CDM for Enhanced Analytics and Patient-Level Insights

* **Data source** : [OpenMRS Demo data]( https://openmrs.atlassian.net/wiki/spaces/RES/pages/26273323/Demo+Data "Chose large-demo-data-2-2-1.sql.gz")
* **Tools used** : SQL in MySQL and PostgreSQL Databases, Pentaho, [OHDSI Tools](https://www.ohdsi.org/software-tools/) (White-rabbit, Rabbit-in-a-Hat, ATLAS, Achilles, WebAPI, Athena, Data Quality Dashboard)

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

## Activities done

![Roadmap](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/Activities.png "Roadmap")


## Results and Outcomes
1. ETL Pipeline
   * Logical Mapping using OHDSI Tools White-Rabbit and Rabbit-in-a-Hat
   * ETL pipeline using Pentaho moving data from OpenMRS MySQL Database into OMOP CDM format in PostgreSQL Database.

![ETL Pipeline](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ETL%20pipeline.png "ETL Pipeline")

Explanation of the diagram

   i. Logical Mapping of Tables

![Logical Mappings](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/Logical%20Mapping.png "Logical Mapping")

   ii. Pentaho in action

https://github.com/user-attachments/assets/6975999c-6ca8-414b-b3f9-b2e7cba08306



2. ATLAS Dashboards

https://github.com/user-attachments/assets/2b2980da-6f3e-440f-8368-a0bcdbce909b

## All the steps done in detail

### 1. Setting Up

#### Prerequisites
* [MySQL Workbench](https://dev.mysql.com/downloads/workbench/)
* [Pentaho PDI](https://pentaho.com/pentaho-community-edition/)
* The data schema structure and values as an [SQL dump](https://openmrs.atlassian.net/wiki/spaces/RES/pages/26273323/Demo+Data)

#### Steps to set up the Data schema model
* Visit the data schema structure and values link to download the preferred SQL dump which is 2.2.1 as it has a lot of data about 5,000 patients and 500,000 observations
[OpenMRS Demo data](https://openmrs.atlassian.net/wiki/download/attachments/26273323/large-demo-data-2-2-1.sql.gz?api=v2) 

Note: Malawi uses OpenMRS version 1.7.0 but the demo data has a small dataset

Create a schema and use query editor to input the SQL Demo dump data
	For instance:
```sql 
CREATE SCHEMA demo_data_2_2_1_openmrs ; 
```

  
### 2. Exploratory Data Analysis

#### The SQL queres are used to understand Demographics data for Patients in OpenMRS to be used for the project

> Number of People in OpenMRS

```sql
select count(*) as 'Number of People' from person p where voided = 0
```

| Number of People | 
| --- |
| 5286 |


> What about Number of Patients

```sql
-- Comment: Only 2 are not Patients
select count(*) as 'Number of Patients' from patient p where voided = 0
```

| Number of People | 
| --- |
| 5284 |



> Finding People who are not Patients

```sql
select person_id 
from person
where person_id NOT IN (
    select patient_id 
    from patient 
) and voided = 0;
```

| person_id | 
| --- |
| 1 |
| 9347 |


> Seeing Patient programs available

```sql
select distinct pp.program_id, p.name as 'program name'
from patient_program pp join program p on pp.program_id = p.program_id 
where pp.voided = 0 and p.retired = 0
```

| program_id | program name |
| --- | --- |
| 3 | HIV Program |
| 4 | TB Program |


> What is the Patients Count per Program

```sql
select distinct p2.name as program, count(distinct p.patient_id) as 'patient count'
from patient p 
join patient_program pp on p.patient_id = pp.patient_id 
join program p2 on pp.program_id = p2.program_id 
where p.voided = 0 and pp.voided = 0 and p2.retired = 0
group by p2.name
```

| program | patient count |
| --- | --- |
| HIV Program | 3100 |
| TB Program | 821 |



> All Locations in OpenMRS

```sql
select location_id, name from location l where retired = 0 limit 4
```

| location_id | name |
| --- | --- |
| 1 | Unknown Location |
| 2 | Wishard Hospital |
| 3 | Mosoriot Hospital |
| 4 | Chulaimbo |

Comment : location_id 5-19 is "Unknown Location (with a number)"


> Patients Count per Location where Patient_program is "HIV Program"

```sql
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
```

| name | patient count |
| --- | --- |
| Unknown Location | 1 |
| Wishard Hospital | 3098 |



> Patient Count within "HIV Program" from "Wishard Hospital" with Obs

```sql
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
```

| Patients with obs | 
| --- |
| 3097 |



> Patient Count within "HIV Program" from "Wishard Hospital" without Obs

```sql
select  count(p.person_id) as 'Patients without obs'
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
```

| Patients with obs | 
| --- |
| 3097 | 

Explanation: The Obs without a Patient is voided


> Obs datetime distribution: min, q1, median, q3, maximum

```sql
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
    min(DATE(obs_datetime)) as min,
    max(case when rn = floor(0.25 * total_count) then DATE(obs_datetime) end) as q1,
    max(case when rn = floor(0.5 * total_count) then DATE(obs_datetime) end) as median,
    max(case when rn = floor(0.75 * total_count) then DATE(obs_datetime) end) as q3,
    max(DATE(obs_datetime)) as max
from ordered_obs;
```

| min | q1 | median | q3 | max |
| --- | --- | --- | -- | --- |
| 1899-12-30 | 2006-03-30 | 2006-04-27 | 2006-05-23 | 5006-03-17 |

Comment	: The Datetime 5006 is wrong whereas datetime 1899 seems like a long time ago. This needs to be investigated further.


> Checking distinct years in obs_datetime of obs

```sql
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
```

| Year | 
| --- |
| 1899 |
| 2004 |
| 2005 |
| 5006 |

Comment: The right ones here look like 2004, 2005 and 2006 and are the only ones to be considered

> Also checking Obs total count based on the Years extracted from obs_datetime

```sql
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
```

| Obs Count for Year 1899 | Obs Count for Year 2004 | Obs Count for Year 2005 | Obs Count for Year 2006 | Obs Count for Year 5006 |
| --- | --- | --- | -- | --- |
| 3 | 3 | 198 | 282266| 1 |


> NOTE: The wrong values cannot be deleted as this is clinical data. The best way of dealing with such values is by changing the entry to voided

```sql
/* The syntax for the change is
 * 
 * UPDATE obs set voided = 1 on obs_datetime < 2004 or obs_datetime > 2006;
 * 
 */
```

> Now Patients birthdate has to be checked and compared to Obs dates if they match

```sql
-- Patients birthdate
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
```

| min | q1 | median | q3 | max |
| --- | --- | --- | -- | --- |
| 1950 | 1965| 1972 | 1980 | 2003|

Comment	: Meaning that all obs with datetime < 1950 are wrong because the Patients were not born yet. Also, the birth years for the Patients look a bit evenly distributed

> Person's Birthdate total count based on the Years extracted from Birthdate


```sql
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
```

| Birthdate for Year < 1960 | Birthdate for Year 1960-1969| Birthdate for year 1970-1979 | Birthdate for year 1980-1989 | Birthdate for year > 1990 |
| --- | --- | --- | -- | --- |
| 241 | 1072| 1000 | 281 | 472|


> Patients Age distribution

```sql
select distinct p.gender, count(distinct p.person_id)  as 'Patients Count'
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
```

| gender | Patients Count |
| --- | --- |
| F | 2048 |
| M | 1049 |


> Counting Dead Patients

```sql
 select count(*) as 'Number of Patients that died' from person p join patient p2 on p.person_id = p2.patient_id where p.dead = 1
``` 

| Number of Patients that died | 
| --- |
| 0 |
 
 


### 3.  ETL pipeline

i. Logical Mapping using Rabbit-In-A-Hat
Explanation

#### Source Data Mapping Approach to CDMV5.4

![All tables](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/RabbitInAHat/all_tables.png "All tables")

#### Table name: person
Reading from person

![person](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/RabbitInAHat/person.png "Person")

Reading from location

![location](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/RabbitInAHat/Reading%20from%20location.png "Location")

Reading from person_address

 ![person_address](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/RabbitInAHat/Reading%20from%20person_address.png "Person Address")

#### Table name: location
Reading from person_address

![location](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/RabbitInAHat/Reading%20from%20person_address.png "Location")

#### Table name: care_site
Reading from location

![care_site](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/RabbitInAHat/Reading%20from%20location%20-%20care_site.png "care_site")

#### Table name: provider

#### Table name: observation_period
Reading from obs

![obs](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/RabbitInAHat/Reading%20from%20obs.png "obs")

Reading from patient

![obs](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/RabbitInAHat/Reading%20from%20patient.png "patient")

ii. ETL Pipeline in Pentaho PDI

How to run the ETL

Prerequisities
1. Pentaho PDI
2. MySQL Workbench and PostgreSQL PgAdmin or just DBeaver
3. An already setup Database in MySQL with the Tables and Records from the SQL dump with a name called “demo_data_2_2_1_openmrs”


What to do for the ETL to run

* Create an empty database in Postgres with name called “omop_cdm”
(this is a requirement otherwise the database connection in every transformation needs to be edited)
* Download data 
* Create a file repository in Pentaho for the files.
* Open the file repository in Postgres.
* Open a 00 Master transformation and go to Data Connection edit to change the database connections. 
	* mysql-source: For the MySQL tables and records input
	* postgres-output: For the results to be written in Postgres
* Then finally run the 00 Master transformation.

The transformation explanation

00 MASTER (job)

![master](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ETL%20pipeline%20Pentaho/master.png "master")

Table

01 Demographics (job)

![demographics](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ETL%20pipeline%20Pentaho/demographics.png "demographics")

Table

01 Care Site (transformation)

![care_site](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ETL%20pipeline%20Pentaho/care_site.png "care_site")

Table

02 Provider (transformation)

![provider](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ETL%20pipeline%20Pentaho/provider.png "provider")

Table

03 Location (transformation)

![location](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ETL%20pipeline%20Pentaho/location.png "location")

Table

04 Person (transformation)

![person](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ETL%20pipeline%20Pentaho/person.png "person")

Table

05 Observation Period (transformation)

![observation_period](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ETL%20pipeline%20Pentaho/observation_period.png "observation_period")

Table

### 4. Data Quality Check

Explanation

Scripts
   
### 5. Visualization of Results
   Patient-level analysis evidence generated in a dashboard format using an OHDSI tool called ATLAS.


Explanation about ATLAS

Focus on Data Sources - Dashboard

Select a Report

Dashboard

CDM Summary 

| Number of persons | 3097 |
| --- | --- |
| cdm | omop_cdm5_4 |

Population by Gender

![Population by Gender](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ATLAS%20Images%20results/Dashboard_PopulationbyGender.png "Population by Gender")

![Age at First Observation](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ATLAS%20Images%20results/Dashboard_AgeatFirstObservation.png "Age at First Observation")

![Cumulative Observation](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ATLAS%20Images%20results/Dashboard_CumulativeObservation.png "Cumulative Observation")

Data Destiny

![Data Density](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ATLAS%20Images%20results/DataDensity_TotalRows.png "Data Density")

Person

Cohort definition

* Explain

![Cohort definition](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ATLAS%20Images%20results/cohort_definition.png "Cohort Definition")

![Person Year of Birth](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ATLAS%20Images%20results/Person_YearofBirth.png "Person Year of Birth")

![Person Gender](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ATLAS%20Images%20results/Person_Gender.png "Person Gender")

Observation Period

![Observation Period Age by Gender](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ATLAS%20Images%20results/ObservationPeriod_AgebyGender.png "Observation Period Age by Gender")

![Observation Period Observation Length](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ATLAS%20Images%20results/ObservationPeriod_ObservationLength.png "Observation Period Observation Length")

Other Reports Explanation

* Explanation




