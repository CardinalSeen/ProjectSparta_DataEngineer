[github_readme.md](https://github.com/user-attachments/files/34260589/github_readme.md)
# Prevalence of Undernourishment of K-6 Learners in SDO Sto. Tomas City
> **Project SPARTA — Data Engineering Pathway Capstone Project**  
> **Author:** Marc Alvin O. Sandrino  
> **Date:** December 4, 2023  

---

## 📌 Executive Summary

This capstone project proposes a robust **Data Engineering & Relational Database Approach** to modernize the tracking, monitoring, and evaluation of the **School-Based Feeding Program (SBFP)** for Kindergarten to Grade 6 (K-6) learners under the newly created **Schools Division Office (SDO) of Sto. Tomas City** (est. 2022 via RA 11086).

Transitioning from traditional Friday photo-collages and fragmented spreadsheet monitoring, this system designs a scalable relational database structure (OLTP/OLAP integration, ERDs, and 1NF–3NF normalization) to systematically track learner growth (BMI, height, weight), monitor feeding events, and support compliance with **Republic Act 11037** (*Masustansiyang Pagkain para sa Batang Pilipino Act*) and **DepEd Order No. 38, s. 2022**.

---

## 🎯 Problem Statement & Objectives

### Key Research Questions
1. How can SDO Sto. Tomas City move beyond basic spreadsheet logging to track learner health progress and weighting assessments effectively?
2. How can the Division identify high-priority schools requiring urgent SBFP intervention while maintaining compliance with RA 11037?
3. What data best practices and database structures enable long-term monitoring to prevent child malnutrition across the division?
4. How can data-driven reporting integrate with local government initiatives such as Mayor Arth Jhun Marasigan's *"Aksyon Bilis"* program?

### Project Scope & Limitations
* **Scope:** 
  * Relational database modeling, Entity-Relationship Diagrams (ERD), and 3NF Data Normalization.
  * Architectural design distinguishing Online Transaction Processing (OLTP) and Online Analytical Processing (OLAP).
  * Data framework for SBFP meals (milk, nutribun, hot meals), facilitators, assessors, and learners.
* **Limitations:**
  * Excludes booster vaccination, deworming tracking, literacy programs, and physical fitness tests.
  * Excludes live UI dashboard execution (focused strictly on Data Engineering architecture; visualization/data science modeling targeted for future pathways).
  * Datasets used in code/schema examples are anonymized/synthetic to comply with the **Data Privacy Act of 2012 (RA 10173)**.

---

## 🏗 System Architecture & Methodology

The proposed pipeline integrates a dual approach balancing operational efficiency with analytical reporting:

```
[ Field Facilitators / Assessors ] ──(Transactional Inputs)──►  OLTP Database (Data Collection)
                                                                       │
                                                                 (ETL Clean & Sync)
                                                                       ▼
[ SDO Analytics & Leadership ]   ◄──(Visual Reporting)──────  OLAP Data Warehouse (Analysis & RA 11037 Compliance)
```

1. **OLTP (Online Transaction Processing):** Enables real-time updates by school-level feeding facilitators (e.g., meal distribution logs, event dates, quantity).
2. **OLAP (Online Analytical Processing):** Aggregates cleaned data for the SDO Analytics Team to produce quarterly nutrition progress reports and evaluate school-level health outcomes.

---

## 📐 Data Modeling & Database Schema

### Entity Relationship Diagram (ERD) Architecture
The relational database design encompasses six primary entities:
* **`School`**: Basic administrative info of participating schools.
* **`Learner`**: Demographic data, body metrics (height, weight, BMI), and nutritional status (`Healthy`, `Severely Wasted`, `Wasted`, etc.).
* **`Facilitator`**: School-based coordinators (e.g., TLE teachers) managing meal distributions.
* **`Assessor`**: Independent SDO representatives conducting objective weight/height assessments.
* **`Meals`**: Nutritional components (e.g., Fortified Milk, Nutribun, Hot Meals).
* **`FeedingProgram`**: Transaction logs linking schools, facilitators, meals, and distribution counts.

---

### Database Normalization Standard

#### 🔹 First Normal Form (1NF)
Eliminates repeating groups and ensures atomic attributes.

```sql
CREATE TABLE Facilitator (
    faci_id INT PRIMARY KEY,
    faci_lname VARCHAR(50) NOT NULL,
    faci_fname VARCHAR(50) NOT NULL,
    faci_position VARCHAR(50)
);

CREATE TABLE Assessor (
    assess_id INT PRIMARY KEY,
    assess_lname VARCHAR(50) NOT NULL,
    assess_fname VARCHAR(50) NOT NULL,
    assess_position VARCHAR(50)
);

CREATE TABLE Meals (
    meal_id INT PRIMARY KEY,
    meal_name VARCHAR(50) NOT NULL,
    meal_details VARCHAR(100)
);

CREATE TABLE School (
    school_id INT PRIMARY KEY,
    school_name VARCHAR(100) NOT NULL,
    school_address VARCHAR(150)
);
```

#### 🔹 Second Normal Form (2NF)
Removes partial dependencies by introducing dedicated transactional tables for **Feeding Events** and **Assessments**.

```sql
CREATE TABLE FeedingProgram (
    feeding_id INT PRIMARY KEY,
    feeding_name VARCHAR(100) NOT NULL,
    feeding_date DATE NOT NULL,
    school_id INT FOREIGN KEY REFERENCES School(school_id),
    meal_id INT FOREIGN KEY REFERENCES Meals(meal_id),
    faci_id INT FOREIGN KEY REFERENCES Facilitator(faci_id),
    quantity NUMERIC(10, 2)
);

CREATE TABLE LearnerAssessment (
    assessment_id INT PRIMARY KEY,
    learner_id INT NOT NULL,
    assessment_date DATE NOT NULL,
    weight_kg DECIMAL(5, 2),
    height_cm DECIMAL(5, 2),
    bmi DECIMAL(4, 2),
    health_status VARCHAR(40)
);

CREATE TABLE SchoolAssessment (
    assessment_id INT PRIMARY KEY,
    school_id INT FOREIGN KEY REFERENCES School(school_id),
    number_of_learners INT,
    school_health_status VARCHAR(50)
);
```

#### 🔹 Third Normal Form (3NF)
Removes transitive dependencies, decoupling learner profiles from school assignments and mapping junction tables for facilitators/assessors.

```sql
CREATE TABLE Learner (
    learner_id INT PRIMARY KEY,
    learner_lname VARCHAR(50) NOT NULL,
    learner_fname VARCHAR(50) NOT NULL,
    grade VARCHAR(10),
    age NUMERIC(3, 0),
    sex VARCHAR(10),
    school_id INT FOREIGN KEY REFERENCES School(school_id)
);

CREATE TABLE FacilitatorSchools (
    faci_id INT FOREIGN KEY REFERENCES Facilitator(faci_id),
    school_id INT FOREIGN KEY REFERENCES School(school_id),
    PRIMARY KEY (faci_id, school_id)
);

CREATE TABLE AssessorSchools (
    assess_id INT FOREIGN KEY REFERENCES Assessor(assess_id),
    school_id INT FOREIGN KEY REFERENCES School(school_id),
    assessment_id INT FOREIGN KEY REFERENCES LearnerAssessment(assessment_id),
    PRIMARY KEY (assess_id, school_id, assessment_id)
);
```

---

## ⚖️ Gap Analysis: Traditional vs. Data Engineering Approach

| Dimension | Online Photo Monitoring (Status Quo) | Data Engineering Approach (Proposed) |
| :--- | :--- | :--- |
| **Methodology** | Weekly collage photo submissions via messaging apps. | Normalized SQL Relational Database + OLAP pipeline. |
| **Granularity** | Macro-level event proof (compliance tracking only). | Micro-level learner progress (BMI, weight changes over time). |
| **Data Actionability** | Reactive (verifies if feeding took place). | Proactive (identifies high-risk schools & tracks health recovery). |
| **Privacy Compliance** | Risk of exposed student images. | Secure database storage adhering to **Data Privacy Act (RA 10173)**. |

---

## 📜 Regulatory & Policy Alignment

* **Republic Act 11086:** Charter creating the City Schools Division of Sto. Tomas (2022).
* **Republic Act 11037:** *Masustansiyang Pagkain para sa Batang Pilipino Act*, mandating national school feeding programs.
* **DepEd Order No. 38, s. 2022:** Operational guidelines for SBFP implementation.
* **Republic Act 10173:** *Data Privacy Act of 2012*, securing learner health records and personally identifiable information (PII).

---

## 🚀 Future Enhancements

* **Data Analytics & Visualization:** Build interactive BI dashboards (Power BI / Tableau / Metabase) to showcase live school rankings by malnutrition prevalence.
* **Advanced Analytics Pathway:** Implement predictive machine learning models to forecast feeding budget allocations and nutritional recovery rates.

---

## 📚 Key References

1. **Wang, D., & Fawzi, W. (2020).** *Impacts of school feeding on educational and health outcomes of school-age children in LMICs.* Systematic Reviews.
2. **Pollitt, E., et al. (1978).** *Educational benefits of the United States school feeding program: a critical review.* AJPH.
3. **Mojica-Sevilla, F. (2023).** *Philippine Department of Education's School-Based Feeding Program Report.* USDA Foreign Agricultural Service.
4. **Dimaano, G., et al. (2019).** *Impact of School-Based Feeding Program in San Luis Elementary School, Sto. Tomas South District, S. Y. 2017–2018.*

---

*Developed as part of the **Project SPARTA (Smarter Philippines through Data Analytics, Research and Development, Training and Adoption)** Data Engineering Pathway.*
