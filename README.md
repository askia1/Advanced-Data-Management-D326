CSN1 Task 1: Data Analysis (Updated)
Author: Devin Clary
Institution: Western Governor’s University
Course: Advanced Data Management D326
Professor: Mindy Crowder
Date: January 16th, 2024

A. Business Report Summary
Business Question: Which actors' films should each DVD store market by location by which likely targeted audience indicators?

Fields to be included in the report:

Country
District
City
MostRentedActorFirstName
MostRentedActorLastName
MostRentedGenre
MostRentedLanguage
NumberofRentals
All fields are string types except Number of Rentals, which is an INT. The necessary data for the report will be drawn from specific tables in the dataset: Customer, Address, City, Actor, Category, Language, Rental, and Country.

Data Transformation: Transformation includes converting actors' first and last names into a single full name format.

Usage of Data: Detailed tables for input into predictive models and algorithms. Summary tables for dashboards and informing decision-makers.

Report Refresh Rate: The report should be refreshed every financial quarter to inform business decisions regarding marketing strategies.

B. Function for Data Transformation

CREATE OR REPLACE FUNCTION GetTopActorsByCountry(country_name VARCHAR(50))
RETURNS TABLE(ActorFullName TEXT, TotalRentals BIGINT) AS $$
BEGIN
    RETURN QUERY
    SELECT (FirstName || ' ' || LastName) AS ActorFullName, NumberOfRentals
    FROM Top_rentals
    WHERE Country = country_name
    AND NumberOfRentals = (
        SELECT MAX(NumberOfRentals)
        FROM Top_rentals
        WHERE Country = country_name
    );
END;
$$ LANGUAGE plpgsql;

CREATE OR REPLACE FUNCTION ConcatenateMaxNames()
RETURNS TEXT AS $$
DECLARE
    maxFirstName TEXT;
    maxLastName TEXT;
BEGIN
    SELECT MAX(First_Name) INTO maxFirstName FROM Actor;
    SELECT MAX(Last_Name) INTO maxLastName FROM Actor;
    
    RETURN maxFirstName || ' ' || maxLastName;
END;
$$ LANGUAGE plpgsql;

SELECT ConcatenateMaxNames() AS MaxActorFullName FROM Actor;
Data transformation in this function uses the || operator to concatenate FirstName and LastName into ActorFullName, thereby changing the data format from individual components to a unified form.

C. SQL Code for Detailed and Summary Tables

CREATE TABLE RankedRentalsTable (
    Country VARCHAR(50),
    District VARCHAR(20),
    City VARCHAR(50),
    MostRentedActorFullName TEXT,
    MostRentedFilm TEXT,
    MostRentedGenre TEXT,
    MostRentedLanguage VARCHAR(255),
    NumberOfRentals INT,
    rental_date Timestamp,
    Rank INT,
    last_query_timestamp Timestamp
);

-- Other SQL statements for inserting data and creating views...

D. SQL Query for Raw Data Extraction

-- SQL queries for extracting raw data...
E. SQL Trigger for Updating Summary Table

CREATE OR REPLACE FUNCTION update_top_rentals_if_needed()
RETURNS void AS $$
-- Function and trigger code here...
F. Stored Procedure for Refreshing Data

CREATE OR REPLACE PROCEDURE UpdateLocationPreferences()
LANGUAGE plpgsql
AS $$

-- Stored procedure code here...

$$;
Job Scheduling Tool: Microsoft Power Automate or any Power app/auto app that can generalize automations.

G. References
Include all references here...
H. Acknowledgments
Acknowledging the use of sources and third-party code is crucial. If no external sources were utilized, declare that no sources were used in the submission.

