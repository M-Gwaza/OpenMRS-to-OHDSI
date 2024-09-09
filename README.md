# Data Harmonization

Standardizing OpenMRS HIV Patient Data to OMOP CDM for Enhanced Analytics and Patient-Level Insights

![Project Roadmap](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/OpenMRS-ETL.png "project roadmap")

* **Data source** : [OpenMRS Demo data]( https://openmrs.atlassian.net/wiki/spaces/RES/pages/26273323/Demo+Data "Chose large-demo-data-2-2-1.sql.gz")
* **Tools used** :
	* SQL in MySQL and PostgreSQL Databases scripting
 	* Pentaho PDI for data engineering - ETL Pipeline development
 	* [OHDSI Tools](https://www.ohdsi.org/software-tools/) (Rabbit-in-a-Hat - For Logical mapping presentation of OpenMRS tables to OMOP CDM Format tables on how the ETL pipeline will be created, ATLAS - For Visualizing patient-level analysis conducted on OpenMRS data in form of OMOP CDM)

## Introduction
[OpenMRS](https://openmrs.org/), an open-source electronic medical record system designed for resource-constrained environments, was developed collaboratively by a global community of developers, healthcare professionals, and organizations including Partners In Health and the Regenstrief Institute. It aims to enhance healthcare delivery by providing a customizable platform for managing patient health information. Effective for local data management, but lacks standardization for broader research and interoperability.

This is where the Observational Health Data Sciences and Informatics (OHDSI) community comes in. OHDSI focuses on improving health outcomes through large-scale observational studies by utilizing The Observational Medical Outcomes Partnership (OMOP) Common Data Model (CDM). The OMOP CDM provides a standardized format for harmonizing diverse healthcare data, enabling consistent and reproducible research.

Transforming OpenMRS data into the OMOP CDM format is crucial for achieving interoperability, allowing data to be integrated, analyzed, and visualized using OHDSI tools like ATLAS. This process not only enhances the utility of the data but also supports advanced analytics, leading to better patient care and more informed decision-making in healthcare.

**Keywords** : OpenMRS, OMOP CDM, OHDSI

## Understanding OHDSI

The Observational Health Data Sciences and Informatics (or OHDSI, pronounced "Odyssey") program is a multi-stakeholder, interdisciplinary collaborative to bring out the value of health data through large-scale analytics. All our solutions are open-source. [OHDSI](https://ohdsi.org/)

OHDSI tools help turn health data into a common format, making it easier for different organizations to work together and study health trends. They also provide useful tools to check data quality, make predictions, and create easy-to-understand reports.

This common format is called the [OMOP CDM](https://ohdsi.github.io/TheBookOfOhdsi/CommonDataModel.html).  The [Observational Medical Outcomes Partnership (OMOP) Common Data Model (CDM)](https://www.ohdsi.org/data-standardization/) is a way of organizing health data in a standard format so that different organizations can easily share and analyze it. It helps make sure that data from different sources—like hospitals, clinics, or research studies—can be understood and used in the same way, no matter where it comes from. 

Some of the key OHDSI tools include:

1. **ATLAS**: A web-based platform for analyzing and visualizing patient data. It allows you to create *cohorts* (groups of patients with shared characteristics), run epidemiological studies, and generate insights through data visualization.
  
2. **ACHILLES**: A tool that provides automated data quality checks and summarizes patient data. It’s written in R and runs SQL scripts to generate high-level summaries of your dataset across various domains, ensuring data completeness and consistency.

3. **Data Quality Dashboard (DQD)**: A more detailed tool for assessing data quality, the DQD also uses R and SQL queries to check for issues like missing or inconsistent data. It provides a comprehensive report on the health of your data.

4. **ATHENA**: A vocabulary browser that gives access to standardized medical terminologies, allowing you to search, browse, and download vocabularies needed for data standardization in OMOP CDM.

5. **WebAPI**: This acts as the backend infrastructure for OHDSI tools, enabling them to interact with your standardized data. WebAPI supports ATLAS by facilitating the sharing of cohorts, studies, and results across institutions.

6. **WhiteRabbit**: A tool that scans your source data, providing an overview of its structure and contents, which helps in planning the Extract, Transform, Load (ETL) process to convert your data into OMOP format.

7. **Rabbit-in-a-Hat**: A visual tool that assists in designing the ETL process. It lets users visually map source data fields to the OMOP CDM structure, making data transformation easier.

## Study Questions
1. What is the structure and format of the source data being the OpenMRS and the residing HIV Patients data?
2. What is the structure of the data destination, that is, the OMOP Common Data Model?
3. How exactly will data be extracted, loaded and transformed from source to destination ?
4. How will the data be checked to ensure that the destination data matches the source?
5. Finally how will the results be prepared ready for different stakeholders ?

## Activities done

![Roadmap](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/Activities.png "Roadmap")


## Results and Outcomes

### 1. ETL Pipeline

[The repository](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/tree/main/Data%20Harmonization%20Project/OpenMRS%20Demo%20to%20OMOP%20CDM)

https://github.com/user-attachments/assets/6975999c-6ca8-414b-b3f9-b2e7cba08306

### 2. ATLAS Dashboards

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

For the OMOP CDM, firstly understand the data model:
* [OMOP CDM](https://ohdsi.github.io/CommonDataModel/cdm54.html)
* NOTE: The project uses OMOP CDM version 5.4 are it is the second to the latest 6.0 and supports all the OHDSI tools unlike  version 6.0.


And for OpenMRS model:
* [OpenMRS Information Model](https://guide.openmrs.org/getting-started/openmrs-information-model/)
* [OpenMRS ERD](https://openmrs.atlassian.net/wiki/spaces/docs/pages/25477157/Data+Model)

![OpenMRS Data Model](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/openmrs_data_model_1.9.0.png "OpenMRS ERD")

As a summary, the Tables will be divided into:

1. Transaction Tables

   1. encounter: Records interactions between patients and healthcare providers.
   2. observation: Stores data collected during encounters, such as test results and clinical notes.
   3. order: Manages orders for tests, medications, and other interventions made by healthcare providers.
   4. visit: Tracks patient visits, including details about the timing and nature of the visit.

2. Reference Data Tables

   1. concept: Contains medical concepts like diagnoses, symptoms, and procedures.
   2. person: Stores demographic information about individuals, including both patients and healthcare workers.
   3. patient: Links to the person table and includes additional details specific to patients.
   4. location: Represents physical or organizational locations where healthcare services are provided.
   5. provider: Contains information about healthcare providers, such as doctors and nurses.
   6. user: Stores information about system users, including their usernames and roles.

* NOTE: The OpenMRS has alot of tables from patients demographics information to medication prescriptions, conditions, symptoms and results. But for this project, the focus will only be about Patients' demographics.

_______

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


![ETL Pipeline](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ETL%20pipeline.png "ETL Pipeline")

The Extract Transform Load Pipeline has 3 parts;
   * i. Extract
     
     	The source data is OpenMRS in MySQL. The data gets uploaded in an OHDSI tool called WhiteRabbit to produce a report with summaries about the database.
     
   * ii. Transform
     
     	The summaries document gets to OHDSI tool Rabbit-in-a-Hat where a Logical Mapping is done to showcase which OpenMRS tables and column get loaded to the OMOP CDM format. A Mapping document then gets created. Athena is another OHDSI tool that gets used to look for Concepts for the column field values. *Note: A "concept" represents health information obtained from electronic health records (EHR) or participant provided information (PPI) sources, which are derived from surveys.*

        Pentaho - ETL Implementation :   At this point a pipeline gets generated to extract and transform data from OpenMRS to finally load the data to OMOP CDM.
      
   * iii. Load
     
     	The summaries document gets to OHDSI tool Rabbit-in-a-Hat where a Logical Mapping is done to showcase which OpenMRS tables and column get loaded to the OMOP CDM format in PostgreSQL database.   


#### Key Points of the diagram

   i. Logical Mapping of Tables - using Rabbit-in-a-Hat

* Start with [WhiteRabbit](https://www.ohdsi.org/analytic-tools/whiterabbit-for-etl-design/) to create an OpenMRS tables summary stats
* Then [Rabbit-In-A-Hat](https://ohdsi.github.io/TheBookOfOhdsi/ExtractTransformLoad.html#rabbit-in-a-hat) for matching the OpenMRS tables to OMOP CDM

![Logical Mappings](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/Logical%20Mapping.png "Logical Mapping")

##### Destination Table name: person of OMOP CDM
Reading from person (OpenMRS)

![person](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/RabbitInAHat/person.png "Person")

Reading from location (OpenMRS)

![location](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/RabbitInAHat/Reading%20from%20location.png "Location")

Reading from person_address (OpenMRS)

 ![person_address](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/RabbitInAHat/Reading%20from%20person_address.png "Person Address")

##### Destination Table name: location of OMOP CDM
Reading from person_address (OpenMRS)

![location](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/RabbitInAHat/Reading%20from%20person_address.png "Location")

##### Destination  Table name: care_site of OMOP CDM
Reading from location (OpenMRS)

![care_site](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/RabbitInAHat/Reading%20from%20location%20-%20care_site.png "care_site")

##### Destination Table name: provider of OMOP CDM

##### Destination Table name: observation_period of OMOP CDM
Reading from obs (OpenMRS)

![obs](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/RabbitInAHat/Reading%20from%20obs.png "obs")

Reading from patient (OpenMRS)

![obs](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/RabbitInAHat/Reading%20from%20patient.png "patient")

#### ii. ETL Pipeline in Pentaho PDI

[The ETL repository](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/tree/main/Data%20Harmonization%20Project/OpenMRS%20Demo%20to%20OMOP%20CDM)

##### How to run the ETL

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

##### The transformation explanation

00 MASTER (job)

![master](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ETL%20pipeline%20Pentaho/master.png "master")

| Step Name | What happens |
| --- | --- |
| Start | Begins the ETL execution |
| Create OMOP CDM Tables | Creates OMOP CDM tables in a schema using the DDL queries from https://github.com/OHDSI/CommonDataModel/blob/main/ddl/5.4/postgresql/OMOPCDM_postgresql_5.4_ddl.sql |
| Create Staging Tables | Creates Temporary tables to be used in the transformations for populating other tables|
| Demographics| Goes to the Demographics repository/ folder and runs the Demographics job for the other transformations to run|
| Success | The final step of the ETL after a successful run|

01 Demographics (job)

![demographics](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ETL%20pipeline%20Pentaho/demographics.png "demographics")

| Step Name | What happens |
| --- | --- |
| Start | Begins the ETL execution |
| *The table names* | Goes to a specific transformation of that table to execute and move to the next table |
| Success | The final step of the job after a successful run|

01 Care Site (transformation)

![care_site](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ETL%20pipeline%20Pentaho/care_site.png "care_site")

| Step Name | What happens |
| --- | --- |
| Location input | This connects to MySQL OpenMRS database and gets the location for "Wishard Hospital" |
| Add constants | 'care_site_id' as 1 since we only getting from "Wishard Hospital", "location_id" as 1 for the location table and 'place_of_service_concept_id' of 9202 as “Outpatient Visit” assuming all the patients are outpatients|
| Modified Javascript value | name value goes to location_source_value and care_site_name|
| Select Values| Selects necessary columns|
| Care_site output | Loads final data to Postgres OMOP CDM Care_site table|
| Select Values 3 | Selects necessary columns |
| Care_site [staging] output |  Loads final data to Postgres Staging Care_site table |
| Select Values 2 | Selects necessary columns|
| Location output | Loads final data to Postgres OMOP CDM Location table|

02 Provider (transformation)

![provider](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ETL%20pipeline%20Pentaho/provider.png "provider")

| Step Name | What happens |
| --- | --- |
| Generate rows | provider_name = Provider, specialty_concept_id and specialty_source_concept_id = 33003 "Meaning Provider", care_site_source_value = 2 "For Wishard Hospital" |
| Sort rows | Sorts the values from generate rows using care_site_source_value before merging (it is important to be done in Pentaho) |
| Care_site [staging] input | This connects to Postgres Staging Care_site database and gets the details |
| Sort rows | Also sorts values from care_site [staging] input using care_site_source_value|
| Merge join | Joins the Tables based on the id for "Wishard Hospital" in care_site_source_value|
| Add sequence | Auto-increments the provider_id value starting from 1|
| Select values 2 | Selects necessary columns |
| Provider [staging] output | Loads final data to Postgres Staging Provider table|
| Select values | Selects necessary columns |
| Provider output | Loads final data to Postgres OMOP CDM Provider table |

03 Location (transformation)

![location](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ETL%20pipeline%20Pentaho/location.png "location")

| Step Name | What happens |
| --- | --- |
| person_address_input | This connects to MySQL OpenMRS database and gets the person_address for "Wishard Hospital" Patients with an observation and part of "HIV Program" |
| Modified Javascript value | Creates OMOP CDM tables in a schema using t |
| Add sequence | Auto-increments the locaationid value starting from 1 |
| Add constants | latitude and longitude as 0 as they cannot be left empty|
| Select values |  Selects necessary columns |
| Location output | Loads final data to Postgres OMOP CDM Location table |
| Select values 2 | Selects necessary columns |
| Location [staging] output | Loads final data to Postgres Staging Location table |

04 Person (transformation)

![person](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ETL%20pipeline%20Pentaho/person.png "person")

| Step Name | What happens |
| --- | --- |
| Person input | This connects to MySQL OpenMRS database and gets "Wishard Hospital" Patients with an observation part of "HIV Program"  |
| Sort rows | Sorts the values from Person input using person_address_id before merging |
| Location [staging] input | This connects to Postgres Staging Care_site database and gets the details |
| Sort rows 2 | Sorts the values from Person input using person_address_id before merging |
| Merge join | Joins the Tables using person_address_id for Person input and Location [staging] input |
| Sort rows 3 | Sorts the values using location_id |
| Care_site [staging] input | This connects to Postgres Staging Care_site database and gets the details |
| Sort rows 4 | Sorts the values using location_id |
| Merge join | Joins the Tables using location_id |
| Modified Javascript value | Birthdate creates year_of_birth, month_of_birth and day_of_birth, gender creates gender_source_value, gender_source_value and gender_concept_id|
| Add constants | race_source_concept_id, ethnicity_source_concept_id, ethnicity_concept_id and race_concept_id get created with value 0 and provider_id with 1|
| Add sequence |  Auto-increments the person_id value starting from 1 for deidentification purposes|
| Select values |  Selects necessary columns |
| Location output | Loads final data to Postgres OMOP CDM Person table |
| Select values 2 | Selects necessary columns |
| Location [staging] output | Loads final data to Postgres Person Location table |

05 Observation Period (transformation)

![observation_period](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ETL%20pipeline%20Pentaho/observation_period.png "observation_period")

| Step Name | What happens |
| --- | --- |
| Observation_query input | This connects to MySQL OpenMRS database and gets Observations for "Wishard Hospital" Patients part of "HIV Program" |
| Sort rows | Sorts the values from Person input using person_id before merging |
| Location [staging] input | This connects to Postgres Staging Care_site database and gets the details |
| Sort rows 2 | Sorts the values from Person input using person_id before merging |
| Merge join | Joins the Tables using person_id |
| Add constants | period_type_concept_id with 32817 |
| Add sequence | Auto-increments the observation_period_id value starting from 1 |
| Select values |  Selects necessary columns |
| Observation period output | Loads final data to Postgres OMOP CDM Observation_period table |

### 4. Data Quality Check

#### The SQL queres are used to check the Quality of the data in comparison with the Source data

> Number of People in OMOP CDM

```sql
select count(*) as 'Number of People'
from person p 
```

| Number of People | 
| --- |
| 3097 |

> Number of People where year_of_birth < 1950 or > 2003

```sql
select count(*) as 'Number of People'
from person p
where year_of_birth < 1950 or year_of_birth > 2003
```

| Number of People where year_of_birth < 1950 or > 2003 | 
| --- |
| 0 |

> Patients without location

```sql
select count(person.*) as 'Patients without location'
from person
left join "location" 
on person.location_id = location.location_id 
where location.location_id is null
```

| Patients without location | 
| --- |
| 0 |

> Patients without care site

```sql
select count(person.*) as 'Patients without care site'
from person
left join care_site 
on person.care_site_id = care_site.care_site_id 
where care_site.care_site_id is null
```

| Patients without care site| 
| --- |
| 0 |

> Number of Patients without observation period

```sql
select count(person.*) as 'Number of Patients without observation_period'
from person
left join observation_period 
on person.person_id = observation_period.person_id
where observation_period.person_id is null
```

| Number of Patients without observation period | 
| --- |
| 0 |

> Number of Patients where observation period dates < 2004 or > 2006

```sql
select count(person.*) as 'Number of Patients where observation period dates < 2004 or > 2006'
from person
join observation_period 
on person.person_id = observation_period.person_id
where observation_period.observation_period_start_date < '2004-01-01' or observation_period.observation_period_start_date > '2006-12-31'
or observation_period.observation_period_end_date < '2004-01-01' or observation_period.observation_period_end_date > '2006-12-31'

```

| Number of Patients where observation period dates < 2004 or > 2006 | 
| --- |
| 0 |
   
### 5. Visualization of Results

ATLAS is an open-source web-based platform developed by the Observational Health Data Sciences and Informatics (OHDSI) community. It provides tools for conducting observational healthcare research using standardized healthcare data in the OMOP Common Data Model (CDM). ATLAS allows users to define and analyze cohorts of patients, perform population-level estimation, prediction modeling, and conduct patient-level data analysis using intuitive, interactive tools.

For ATLAS to work, firstly OHDSI Achilles and Webapi has to be run. **ACHILLES** is a tool that performs automated data quality checks and generates summary statistics for OMOP CDM databases, providing insights into data content and integrity. **WebAPI** is the backend service that powers ATLAS by handling database queries, cohort execution, and facilitating communication between the OMOP CDM and the front-end application. The WebAPI for the project used Apache Tomcat as a local server environment for managing and processing requests between the ATLAS interface and the OMOP CDM database.

The reports ATLAS produces are below:

#### Dashboard

Overview: The dashboard provides a summary of key metrics from the data source, giving users a quick overview of the most important characteristics of the patient population and data quality.

CDM Summary 

| Number of persons | 3097 |
| --- | --- |
| cdm | omop_cdm5_4 |

![Population by Gender](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ATLAS%20Images%20results/Dashboard_PopulationbyGender.png "Population by Gender")

![Age at First Observation](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ATLAS%20Images%20results/Dashboard_AgeatFirstObservation.png "Age at First Observation")

![Cumulative Observation](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ATLAS%20Images%20results/Dashboard_CumulativeObservation.png "Cumulative Observation")

#### Data Destiny

Overview: The Data Density report visualizes the availability of data across time for various healthcare domains (e.g., conditions, drug exposures, procedures, measurements). Some reports not generated since few data was loaded in OMOP CDM - for demographics only.

![Data Density](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ATLAS%20Images%20results/DataDensity_TotalRows.png "Data Density")

#### Person

Overview: Displays demographics of patients in the data source, including age, gender, race, and ethnicity. And a person report in ATLAS usually has Cohorts defined. 

##### Cohort definition

   * A cohort in ATLAS refers to a group of patients who meet a set of criteria defined by the user. The Cohort Definition tool in ATLAS allows users to create and define these groups based on specific characteristics, conditions, treatments, or other healthcare data. This is a critical feature for performing patient-level analysis, enabling researchers and healthcare professionals to study specific populations within the overall dataset. 

![Person Year of Birth](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ATLAS%20Images%20results/Person_YearofBirth.png "Person Year of Birth")

![Person Gender](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ATLAS%20Images%20results/Person_Gender.png "Person Gender")

#### Observation Period

Overview: The Observation Period report focuses on the time span during which data is collected for each patient in the dataset. It represents the duration when the healthcare system "observes" a patient, meaning the period when the healthcare system can capture relevant healthcare information for that person. Each patient can have one or more observation periods, depending on their interactions with the healthcare system.

![Observation Period Age by Gender](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ATLAS%20Images%20results/ObservationPeriod_AgebyGender.png "Observation Period Age by Gender")

![Observation Period Observation Length](https://github.com/M-Gwaza/OpenMRS-to-OHDSI/blob/main/Data%20Harmonization%20Project/Images/ATLAS%20Images%20results/ObservationPeriod_ObservationLength.png "Observation Period Observation Length")

#### Other Reports Explanation

* Other reports like Death or Visit have no data since the OMOP CDM tables have no data.

### Conclusion

The OpenMRS HIV Patients demographics data was successfully transformed to OMOP CDM and patient-level analysis was conducted. 

### Challenges faced

* Undestanding the OpenMRS is difficult since there alot of tables
* Undestand the OMOP CDM is also difficult and needs to be looked at properly to make data from different sources gets transformed and loaded to it's appropriate tables
* The usage of OHDSI tools is also difficult as the tools are Open source and require the exact packages and supporting libraries to be used


