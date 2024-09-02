# OpenMRS-to-OHDSI

## Introduction
OpenMRS, an open-source electronic medical record system designed for resource-constrained environments, was developed collaboratively by a global community of developers, healthcare professionals, and organizations including Partners In Health and the Regenstrief Institute. It aims to enhance healthcare delivery by providing a customizable platform for managing patient health information.

## Why OMOP CDM
* **Standardization:** Provides a consistent structure for data integration and analysis across different sources.
* **Interoperability:** Facilitates collaboration and data sharing by harmonizing data from various healthcare systems.
* **Efficient Analysis:** Enables the use of pre-built tools and methodologies, saving time and effort.
* **Reproducibility:** Enhances the reproducibility and comparability of research findings.
* **Comprehensive Research:** Supports diverse research activities, including epidemiology and clinical trials.
* **Regulatory Compliance:** Aids in meeting privacy and security standards for data sharing and use.


## Activities to be done
1. Exploratory data analysis of the data using 
  * ERD and documentation
  * Python and SQL scripting to understand the data
> Outcome: Identifying what exactly to focus on

2.Development of an ETL pipeline to transform Demographics data into OMOP CDM
  * WhiteRabbit, Rabbit-in-a-Hat for Logical Mapping
* Pentaho and SQL for actual Mapping
>	Outcome: OpenMRS data into OMOP CDM

3.Data Quality Checks
> Outcome: Data Quality Dashboard or just scripts

4. OMOP CDM descriptive stats using WebAPI and Achilles
  * ATLAS - Cohort definition and Visualization of results
> Outcome: Dashboard for OpenMRS patients
