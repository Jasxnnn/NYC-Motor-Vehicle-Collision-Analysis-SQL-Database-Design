# NYC Motor Vehicle Collisions Database

*By Jason Guan*

## Overview

This project designs and implements a **MySQL relational database** using New York City motor vehicle collision data to examine patterns involving vehicles, drivers, collision characteristics, contributing factors, and vehicle damage.

The goal is to transform a public motor vehicle collision dataset into a structured relational database and use SQL to analyze:

* Vehicle damage associated with pre-crash direction
* Driver gender distribution across vehicle types
* Collision contributing factors
* Vehicle characteristics associated with New York-licensed drivers
* Vehicle distribution by model year
* Relationships between drivers, vehicles, collisions, and contributing factors

**Technical Tools Used:**

* MySQL
* MySQL Workbench
* SQL
* Relational Database Design
* Entity-Relationship Modeling
* Database Normalization
* Data Modeling
* Exploratory Data Analysis (EDA)

**Database Pipeline**

`NYC Motor Vehicle Collision Dataset → Data Selection → ERD & Relational Schema → MySQL Database → SQL Views → Analytical Analysis`

## Executive Summary

The project focuses on building a structured relational database from publicly available New York City motor vehicle collision data.

The original dataset contains a large number of records and multiple categories of collision-related information. To make the dataset manageable for implementation, our team selected collision records from **August 1, 2016, between approximately 12:00 PM and 2:07 PM**.

The database was designed around **nine interconnected tables**, separating information about drivers, vehicles, collisions, contributing factors, vehicle damage, and public property damage. Primary and foreign keys were used to establish relationships between entities, while relationship tables were used to connect drivers to vehicles, vehicles to collisions, and collisions to contributing factors.

After implementing the database, five SQL views were created to answer analytical questions involving vehicle damage, driver demographics, contributing factors, license jurisdictions, and vehicle model years.

The project demonstrates practical experience with **relational database design, SQL querying, data modeling, foreign key management, database troubleshooting, and analytical data exploration**.

---

## The Data

The project uses publicly available **New York City Motor Vehicle Collision** data.

Because the original dataset was large, the project focused on a smaller time period:

**August 1, 2016 — 12:00 PM to approximately 2:07 PM**

The selected data contains information related to vehicles, drivers, collisions, contributing factors, and damage.

### Key Data Categories

* Vehicle type
* Vehicle make/model
* Vehicle model year
* Driver gender
* Driver license jurisdiction
* Pre-crash direction
* Collision information
* Contributing factors
* Vehicle damage
* Public property damage

The dataset was structured into multiple relational tables to reduce redundancy and make relationships between different types of collision information easier to analyze.

---

## The Database

The database consists of **nine interconnected tables** designed to organize the collision data into logical entities.

<img width="1000" alt="NYC Motor Vehicle Collisions Entity Relationship Diagram" src="images/ERD.png"/>

### Database Tables

* **`vehicle_info`** — Stores vehicle characteristics including vehicle type, model year, vehicle make/trim, registration information, and damage identifiers.
* **`drivers`** — Stores driver information including gender, license status, and license jurisdiction.
* **`crash_info`** — Stores collision information including pre-crash direction and occupant-related information.
* **`contributing_factors`** — Contains reported factors associated with collisions.
* **`crash_factor`** — Connects collision records with contributing factors.
* **`vehicle_crash`** — Connects vehicles with collision events.
* **`driver_vehicle`** — Connects drivers with the vehicles they were operating.
* **`vehicle_damage`** — Stores information about damage sustained by vehicles.
* **`public_property`** — Stores information related to public property damage.

The relational structure allows information to be connected through primary and foreign keys rather than storing all collision information in one large table.

Relationship tables such as `driver_vehicle`, `vehicle_crash`, and `crash_factor` allow the database to represent relationships between entities while minimizing unnecessary data duplication.

---

## Database Design

The database was developed using an **Entity-Relationship Diagram (ERD)** to define the structure and relationships between tables before implementation.

During implementation, several changes were made to the original design.

### Relationship Adjustments

Some identifying one-to-many relationships were changed to **non-identifying one-to-many relationships** to better represent how the entities were related.

### Foreign Key Adjustments

Foreign key relationships were also modified during implementation.

For example:

* `damage_id` was incorporated into the vehicle-related structure to connect vehicles with damage information.
* `public_property_id` was incorporated into the collision-related structure to associate collision records with public property damage.

### Data Scope

The original dataset contained significantly more records than what was practical for the project environment. We therefore narrowed the imported dataset to a specific time period.

This allowed us to focus on database architecture, data relationships, and SQL analysis while keeping the implementation manageable.

---

# SQL Analysis

Using MySQL, I created **five analytical SQL views** to examine different aspects of the collision dataset.

The queries demonstrate:

* Multi-table joins
* Correlated subqueries
* Aggregate functions
* `COUNT()`
* `GROUP BY`
* `ORDER BY`
* `WHERE`
* SQL views
* Relational data analysis

---

## Query 1 — Vehicle Damage for Vehicles Traveling Straight Ahead

This query identifies vehicles that were traveling **straight ahead before the collision** and displays their corresponding post-crash vehicle damage.

```sql
CREATE VIEW query_1 AS
SELECT 
    vehicle_id,
    pre_crash AS 'Pre-Crash Direction',
    vehicle_damage AS 'Damage to the Vehicle Post-Crash'
FROM crash_info
JOIN vehicle_info USING (vehicle_id)
JOIN vehicle_damage USING (damage_id)
WHERE pre_crash = 'Going Straight Ahead';
```

**Question answered:**

> What types of vehicle damage were reported for vehicles traveling straight ahead before a collision?

This query combines information from `crash_info`, `vehicle_info`, and `vehicle_damage` using relational joins.

---

## Query 2 — Driver Gender by Vehicle Type

This query calculates the number of male and female drivers associated with each vehicle type.

```sql
CREATE VIEW query_2 AS
SELECT 
    vehicle_type AS 'Vehicle Type',

    (
        SELECT COUNT(unique_id)
        FROM drivers 
        JOIN driver_vehicle USING (unique_id)
        JOIN vehicle_info USING (vehicle_id)
        WHERE driver_sex = 'M' 
        AND vehicle_type = v.vehicle_type
    ) AS 'Number of Male Drivers',

    (
        SELECT COUNT(unique_id)
        FROM drivers 
        JOIN driver_vehicle USING (unique_id)
        JOIN vehicle_info USING (vehicle_id)
        WHERE driver_sex = 'F' 
        AND vehicle_type = v.vehicle_type
    ) AS 'Number of Female Drivers'

FROM vehicle_info v
JOIN driver_vehicle USING (vehicle_id)
JOIN drivers USING (unique_id)
GROUP BY vehicle_type;
```

**Question answered:**

> How does driver gender distribution vary across different vehicle types?

This query demonstrates the use of correlated subqueries, aggregation, joins, and grouping.

---

## Query 3 — Vehicles by Contributing Factor

This query counts the number of vehicles associated with each reported contributing factor.

```sql
CREATE VIEW query_3 AS
SELECT 
    contributing_factor AS 'Contributing Factor',
    COUNT(vehicle_id) AS 'Number of Vehicles per Contributing Factor'
FROM contributing_factors
JOIN crash_factor USING (factor_id)
JOIN vehicle_crash USING (collision_id)
GROUP BY contributing_factor;
```

**Question answered:**

> Which contributing factors are associated with the greatest number of vehicles involved in collisions?

This analysis demonstrates how relationship tables can be used to connect multiple entities and perform aggregate analysis.

---

## Query 4 — Vehicles Associated with New York-Licensed Drivers

This query identifies vehicle years and vehicle makes/trims associated with drivers whose license jurisdiction is New York.

```sql
CREATE VIEW query_4 AS
SELECT 
    drivers_license_jurisdiction AS 'License Jurisdiction',
    vehicle_year AS 'Vehicle Year',
    vehicle_model_trim AS 'Vehicle Make'
FROM drivers
JOIN driver_vehicle USING (unique_id)
JOIN vehicle_info USING (vehicle_id)
WHERE drivers_license_jurisdiction = 'NY'
ORDER BY vehicle_year;
```

**Question answered:**

> What vehicle years and makes/trims are represented among drivers with New York licenses?

This demonstrates filtering and sorting information across multiple related tables.

---

## Query 5 — Vehicle Distribution by Model Year

This query counts the number of vehicles associated with each vehicle model year.

```sql
CREATE VIEW query_5 AS
SELECT 
    vehicle_year AS 'Vehicle Model Year',
    COUNT(vehicle_id) AS 'Number of Vehicles per Year'
FROM vehicle_info
GROUP BY vehicle_year
ORDER BY vehicle_year;
```

**Question answered:**

> How are vehicles distributed across different model years?

This query demonstrates aggregation using `COUNT()`, `GROUP BY`, and `ORDER BY`.

---

# Key Takeaways

**Vehicle Damage**

* The database can be used to examine relationships between pre-crash vehicle direction and reported post-crash damage.
* Joining collision, vehicle, and damage information allows analysts to explore vehicle-level collision characteristics.

**Driver & Vehicle Demographics**

* Driver information can be combined with vehicle characteristics to compare driver demographics across different vehicle types.
* The relational structure makes it possible to analyze driver and vehicle attributes without storing the same information repeatedly.

**Contributing Factors**

* Aggregating vehicles by contributing factor provides a way to identify which reported factors appear most frequently within the selected collision records.
* The `crash_factor` and `vehicle_crash` relationship tables allow contributing factors to be connected to individual vehicles through collision events.

**License Jurisdiction**

* Driver license information can be connected with vehicle characteristics to examine the vehicles associated with drivers from a particular jurisdiction.
* SQL filtering makes it possible to isolate specific jurisdictions such as New York.

**Vehicle Model Years**

* Grouping vehicles by model year provides a view of the vehicle age distribution represented within the selected collision records.
* This could serve as a foundation for future analysis examining relationships between vehicle age and collision outcomes.

---

# Technical Challenges

### Foreign Key Constraints

One of the main challenges was resolving foreign key and relationship issues during database implementation.

The team had to review the ERD and make adjustments to relationship types and foreign key placement before the database could be successfully forward-engineered.

### Data Import Order

Because multiple tables were connected through foreign keys, the tables could not always be populated in an arbitrary order.

Understanding the dependency between parent and child tables was necessary to determine the correct sequence for importing the data.

### Database Design Iteration

The final database structure differed from the initial design.

As implementation progressed, we identified issues with relationships and constraints that required us to revisit the ERD and make structural changes.

This reinforced the importance of validating database architecture before loading and querying large amounts of data.

---

# Data Ethics & Limitations

The dataset does not contain direct personally identifiable information such as names, Social Security numbers, or personal addresses, which reduces privacy concerns associated with the analysis.

However, several limitations should be considered:

* The analysis covers only a short time period.
* Collision reporting may vary across geographic areas.
* The dataset does not contain all relevant road infrastructure information.
* Certain demographic variables are unavailable.
* The selected records should not be considered representative of all New York City motor vehicle collisions.

The SQL queries therefore provide **descriptive analysis of the selected dataset rather than causal conclusions about why collisions occur**.

---

# Lessons Learned

This project provided hands-on experience across the database development lifecycle.

**Relational Database Design**

* Learned how to translate a real-world dataset into logical entities and relationships.
* Gained experience using primary keys, foreign keys, and relationship tables.
* Learned how database normalization can reduce redundant information.

**SQL**

* Strengthened SQL skills through joins, subqueries, aggregation, filtering, grouping, and views.
* Learned how to combine information across multiple related tables to answer analytical questions.

**Database Troubleshooting**

* Learned how foreign key constraints affect database implementation.
* Gained experience troubleshooting ERD and data import issues.
* Learned the importance of understanding table dependencies before importing data.

**Team Collaboration**

* Developed experience working with a team on database design and implementation.
* Learned the importance of assigning responsibilities, establishing internal deadlines, and communicating technical issues early.

---

## Files

* **`SQL/`** — Contains SQL scripts used to create, populate, and analyze the database.
* **`ERD/`** — Contains the Entity-Relationship Diagram representing the database schema.
* **`Data/`** — Contains sample or permitted source data used for the project.
* **`Documentation/`** — Contains supporting project documentation and the original final report.

---

## Project Limitations

This project is based on a limited subset of publicly available New York City motor vehicle collision records. Because the database focuses on **August 1, 2016, from approximately 12:00 PM to 2:07 PM**, the findings should not be generalized to all NYC collisions.

Additionally, the dataset does not include every factor that may influence collision outcomes, including comprehensive information about road infrastructure, traffic volume, weather conditions, vehicle utilization, or driver experience.

The analysis should therefore be interpreted as **descriptive analysis of the selected records rather than evidence of causal relationships**.
