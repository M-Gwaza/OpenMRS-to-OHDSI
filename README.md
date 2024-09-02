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



## Activities to be done
- Exploratory data analysis of the data to identify what exactly to focus on
- Development of an ETL pipeline to transform data into OMOP CDM
- Data Quality Checks
- Showcasing results


## Results
1.  ETL pipeline
   * Logical Mapping using OHDSI Tools White-Rabbit and Rabbit-in-a-Hat
   * ETL pipeline using Pentaho moving data from OpenMRS MySQL Database into OMOP CDM format in PostgreSQL Database.
3. OpenMRS HIV Patients Data transformed and loaded into a Postgres Database in a Common Data Model format called OMOP CDM.
4. Patient-level analysis evidence generated in a dashboard format using an OHDSI tool called ATLAS.

https://github.com/user-attachments/assets/9937a4e6-ccab-42bd-a1a5-8456b1e9efc7



