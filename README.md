# Advanced-Data-Management-D326
Needed to meet various standards for tasks to answer a business question.

 
A. Summarize one real-world written business report that can be created from the DVD Dataset from the “Labs on Demand Assessment Environment and DVD Database” attachment. 
•	Business question:  Which actors' films should each DVD store market by location by which likely targeted audience indicators?
1.	Specific Fields that will be included in the detailed table and summary table of the report:
o	Detailed and Summarized fields: (Country, District, City, MostRentedActorFirstName, MostRentedActorLN, MostRentedGenre, MostRentedLanguage, NumberofRentals)
2.	Describe the types of Data Fields Used for the Report
o	All of the fields will be string fields,  Text and VarChar variations with exception of the Number of Rentals, which would be an INT.
3.	 Identify at least two specific tables from the given dataset that will provide the data necessary for the detailed table section and the summary table section of the report.
o	The Tables: Customer,  Address, City, Actor, Category, Language, Rental, Country.
4.	Identify at least one field in the detailed table section that will require a custom transformation with a user-defined function and explain why it should be transformed (e.g., you might translate a field with a value of N to No and Y to Yes).
1.	I transformed the fields that designated the actors' first and last name, so that the query would be more concise. Also, I made a function so that if you were to enter a country’s name, you would receive the country’s preferred actors in this format.
5.	Explain the different business uses of the detailed table section and the summary table section of the report. 
1.	The detailed table section could be used to input data into algorithms to support learning models' predictive capabilities, as it is raw data.
2.	The summary table section would be best in for use in data visuals, dashboards, and informing decision makers.
6.	Explain how frequently your report should be refreshed to remain relevant to stakeholders.
1.	It is unlikely that these statistics would change drastically, and some time would have to pass in order for the data to meaningfully inform a business decision, so the report should be refreshed every financial quarter, in line for new possible deliberations in marketing strategy.
B. Provide original code for function(s) in text format that perform the transformation(s) you identified in part A4.

CREATE OR REPLACE FUNCTION GetTopActorsByCountry(country_name VARCHAR(50))
RETURNS TABLE(ActorFullName TEXT, TotalRentals BIGINT) AS $$
BEGIN
    RETURN QUERY
    SELECT (FirstName || ' ' || LastName) AS ActorFullName, --data transformation
,NumberOfRentals
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


Data transformation, as defined by Informatica, is "the process of taking data that exists in one format or state and converting it into a different format or state" (Informatica, n.d.).
In this function, the ‘| |’ operator is used to concatenate the ‘FirstName’ and ‘LastName’ into a single ‘ActorFullName’, thereby transforming the data format. This transformation is a clear example of changing the state of data from individual components into a unified form, aligning with the definition provided by Informatica.
References:
Microsoft. (n.d.). Create User-defined Functions (Database Engine) - SQL Server. Microsoft Learn. Retrieved December 21, 2023, from Microsoft Learn.
Informatica. (n.d.). What is Data Transformation? [Web page]. Retrieved from https://www.informatica.com/resources/articles/what-is-data-transformation.html







C. Provide original SQL code in a text format that creates the detailed and summary tables to hold your report table sections. AND D. Provide an original SQL query in a text format that will extract the raw data needed for the detailed section of your report from the source database.

Create the base table first:
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

INSERT INTO RankedRentalsTable (Country, District, City, MostRentedActorFullName, MostRentedFilm, MostRentedGenre, MostRentedLanguage, NumberOfRentals, Rank)
SELECT
    Country.Country AS Country,
    ADD.District AS District,
    City.City AS City,
    ConcatenateActorNames() AS MostRentedActorFullName,
    MAX(F.Title) AS MostRentedFilm,
    Max(CAT.Name) AS MostRentedGenre,
    MAX(L.Name) AS MostRentedLanguage,
    Count(R.Rental_ID) AS NumberOfRentals,
    Row_Number() OVER (Partition By Country.Country, ADD.District, City.City Order BY Count(R.Rental_ID) DESC) AS Rank
FROM
    Customer C
    INNER JOIN Rental R ON R.Customer_ID = C.Customer_ID
    INNER JOIN Address ADD ON ADD.Address_ID = C.Address_ID
    INNER JOIN Store S ON S.Store_ID = C.Store_ID
    INNER JOIN City ON City.City_ID = ADD.City_ID
    INNER JOIN Country ON Country.Country_ID = City.Country_ID
    INNER JOIN Inventory I ON I.Inventory_ID = R.Inventory_ID
    INNER JOIN Film F ON F.Film_ID = I.Film_ID
    INNER JOIN Language L ON L.Language_ID = F.Language_ID
    INNER JOIN Film_Actor FA ON FA.Film_ID = F.Film_ID
    INNER JOIN Film_Category FC ON FC.Film_ID = F.Film_ID
    INNER JOIN Actor A ON A.Actor_ID = FA.Actor_ID
    INNER JOIN Category CAT ON CAT.Category_ID = FC.Category_ID
GROUP BY
    Country.Country, ADD.District, City.City
ORDER BY
    Country, District, City, Rank;


Create View as Top_Rentals
SELECT RRT.country,
    RRT.district,
   RRT.city,
    RRT.MostRentedActorFullName,
    RRT.mostrentedfilm,
    RRT.mostrentedgenre,
    RRT.mostrentedlanguage,
RRT.numberofrentals,
    RRT.rental_date,
    RRT.last_query_timestamp
   FROM rankedrentalstable as RRT
  WHERE RRT.rank <= 5
  ORDER BY RRT.country, RRT.district, RRT.city, RRT.rank;
Select * From Top_rentals;

References:
1.	Gupta, R. (2019, July 3). Overview of SQL RANK functions. SQLShack. Retrieved December 15, 2023, from https://www.sqlshack.com/overview-of-sql-rank-functions/
2.	SQLTutorial.org. (2009 - Present). SQL ROW_NUMBER(). Retrieved December 15, 2023, from https://www.sqltutorial.org/sql-window-functions/sql-row_number/
3.	Gupta, R. (2019, April 9). SQL PARTITION BY Clause overview. SQLShack. Retrieved December 15, 2023, from https://www.sqlshack.com/sql-partition-by-clause-overview/
4.	W3Schools. (n.d.). SQL String Functions. Retrieved December 15, 2023, from https://www.w3schools.com/sql/sql_ref_string_functions.asp



C/D. (This is for the Summary)
Create Materialized View location_preferences AS
WITH rankedpreferences AS (
         SELECT TR.country,
            TR.mostrentedgenre,
            TR.MostRentedActorFullName
            TR.mostrentedfilm,
            row_number() OVER (PARTITION BY TR.country ORDER BY TR.numberofrentals DESC) AS rank
           FROM top_rentals as TR
        )
 SELECT RP.country,
    RP.mostrentedgenre,
    TR.FirstName,
    TR.LastName,
    RP.mostrentedfilm
   FROM rankedpreferences as RP
  WHERE RP.rank = 1;
Select * from location_preferences;

OR (Using the function)
Select * from GetTopActorsByCountry (‘Canada’)
References:
W3Schools. (n.d.). SQL CREATE VIEW, REPLACE VIEW, DROP VIEW Statements. Retrieved December 15, 2023, from https://www.w3schools.com/sql/sql_view.asp


E. Provide original SQL code in a text format that creates a trigger on the detailed table of the report that will continually update the summary table as data is added to the detailed table.
This creates a trigger updates after an interval, or defines n as number of rentals worth to update.the data.
CREATE OR REPLACE FUNCTION update_top_rentals_if_needed()
RETURNS void AS $$
DECLARE
    max_last_query_timestamp TIMESTAMP;
    current_timestamp TIMESTAMP;
    rental_count INTEGER;
    current_quarter INTEGER;
    last_query_quarter INTEGER;
BEGIN
    current_timestamp := NOW();
    SELECT MAX(last_query_timestamp) INTO 
	 max_last_query_timestamp  FROM RankedRentalsTable;

    IF  max_last_query_timestamp  IS NULL THEN
        max_last_query_timestamp  := current_timestamp - INTERVAL '6 months';
    END IF;

    -- calculate quarters
    current_quarter := EXTRACT(QUARTER FROM current_timestamp);
    last_query_quarter := EXTRACT(QUARTER FROM  max_last_query_timestamp );

    -- check if a new quarter started
    IF last_query_quarter <> current_quarter THEN
        SELECT COUNT(*) INTO rental_count
        FROM top_rentals
        WHERE rental_date >= current_timestamp - INTERVAL '3 months';

        IF rental_count >= 3 THEN
            UPDATE top_rentals SET last_query_timestamp = current_timestamp;
        END IF;
    END IF;
	
	Return New;
END;
$$ LANGUAGE plpgsql;

Create Trigger Update_rental_stat
After Insert or Update
On RankedRentalsTable
For Each Row
Execute function update_top_rentals_if_needed();



References:
1.	MySQL. (n.d.). MySQL 8.0 Reference Manual: 11.2.2 The DATE, DATETIME, and TIMESTAMP Types. Retrieved December 15, 2023, from https://dev.mysql.com/doc/refman/8.0/en/datetime.html
2.	The PostgreSQL Global Development Group. (1996-2023). PL/pgSQL — SQL Procedural Language. In PostgreSQL Documentation, 16, Chapter 43. Retrieved December 15, 2023, from https://www.postgresql.org/docs/current/plpgsql.html
3.	PostgreSQL Tutorial. (2011 - Present). PostgreSQL Triggers. Retrieved December 15, 2023, from https://www.postgresqltutorial.com/postgresql-triggers/
4.	The PostgreSQL Global Development Group. (1996-2023). Materialized Views. In PostgreSQL Documentation, 16, Chapter 41.3. Retrieved December 15, 2023, from https://www.postgresql.org/docs/current/rules-materializedviews.html
5.	SQLBook. (n.d.). SQL Date Functions. Retrieved December 15, 2023, from https://www.sqlbook.com/sql-date-functions/



F.  Provide an original stored procedure in a text format that can be used to refresh the data in both the detailed table and summary table. The procedure should clear the contents of the detailed table and summary table and perform the raw data extraction from part D.
•	Identify a relevant job scheduling tool that can be used to automate the stored procedure.
o	Microsoft Power Automate template, generally any power app/auto app that can generalize automations.
CREATE OR REPLACE PROCEDURE UpdateLocationPreferences()
LANGUAGE plpgsql
AS $$

BEGIN
    
    UPDATE
        top_rentals AS TR
    SET
        Country = GU.Country,
        District = GU.District,
        City = GU.City,
        FirstName = GU.FirstName,
	 LastName = GU.LastName,
        MostRentedGenre = GU.MostRentedGenre,
        MostRentedLanguage = GU.MostRentedLanguage
    FROM
        ( Select *
		From RankedRentalsTable) AS GU
    WHERE 
        TR.Country = GU.Country AND TR.District = GU.District AND 
		TR.City = GU.City;
--refresh location preferences
  Refresh Materialized view location_preferences;
END;
$$;


References:
1.	The PostgreSQL Global Development Group. (1996-2023). CREATE PROCEDURE. In PostgreSQL Documentation, 16. Retrieved December 15, 2023, from https://www.postgresql.org/docs/current/sql-createprocedure.html 
2.	W3Schools. (n.d.). SQL Procedures. Retrieved December 15, 2023, from https://www.w3schools.com/sql/sql_procedures.asp
3.	SQLTutorial.org. (n.d.). SQL UPDATE. Retrieved December 15, 2023, from https://www.sqltutorial.org/sql-update/
4.	EssentialSQL. (n.d.). What Are SQL Window Functions? Retrieved December 15, 2023, from https://www.essentialsql.com/what-are-sql-window-functions/


H. Acknowledge all utilized sources, including any sources of third-party code, using in-text citations and references. If no sources are used, clearly declare that no sources were used to support your submission.
1.	Gupta, R. (2019, July 3). Overview of SQL RANK functions. SQLShack. Retrieved December 15, 2023, from https://www.sqlshack.com/overview-of-sql-rank-functions/
2.	SQLTutorial.org. (2009 - Present). SQL ROW_NUMBER(). Retrieved December 15, 2023, from https://www.sqltutorial.org/sql-window-functions/sql-row_number/
3.	Gupta, R. (2019, April 9). SQL PARTITION BY Clause overview. SQLShack. Retrieved December 15, 2023, from https://www.sqlshack.com/sql-partition-by-clause-overview/
4.	W3Schools. (n.d.). SQL String Functions. Retrieved December 15, 2023, from https://www.w3schools.com/sql/sql_ref_string_functions.asp
5.	W3Schools. (n.d.). SQL CREATE VIEW, REPLACE VIEW, DROP VIEW Statements. Retrieved December 15, 2023, from https://www.w3schools.com/sql/sql_view.asp
6.	MySQL. (n.d.). MySQL 8.0 Reference Manual: 11.2.2 The DATE, DATETIME, and TIMESTAMP Types. Retrieved December 15, 2023, from https://dev.mysql.com/doc/refman/8.0/en/datetime.html
7.	The PostgreSQL Global Development Group. (1996-2023). PL/pgSQL — SQL Procedural Language. In PostgreSQL Documentation, 16, Chapter 43. Retrieved December 15, 2023, from https://www.postgresql.org/docs/current/plpgsql.html
8.	PostgreSQL Tutorial. (2011 - Present). PostgreSQL Triggers. Retrieved December 15, 2023, from https://www.postgresqltutorial.com/postgresql-triggers/
9.	The PostgreSQL Global Development Group. (1996-2023). Materialized Views. In PostgreSQL Documentation, 16, Chapter 41.3. Retrieved December 15, 2023, from https://www.postgresql.org/docs/current/rules-materializedviews.html
10.	SQLBook. (n.d.). SQL Date Functions. Retrieved December 15, 2023, from https://www.sqlbook.com/sql-date-functions/
11.	The PostgreSQL Global Development Group. (1996-2023). CREATE PROCEDURE. In PostgreSQL Documentation, 16. Retrieved December 15, 2023, from https://www.postgresql.org/docs/current/sql-createprocedure.html 
12.	W3Schools. (n.d.). SQL Procedures. Retrieved December 15, 2023, from https://www.w3schools.com/sql/sql_procedures.asp 
13.	SQLTutorial.org. (n.d.). SQL UPDATE. Retrieved December 15, 2023, from https://www.sqltutorial.org/sql-update/ 
14.	EssentialSQL. (n.d.). What Are SQL Window Functions? Retrieved December 15, 2023, from https://www.essentialsql.com/what-are-sql-window-functions/ 
15.	Microsoft. (n.d.). Create User-defined Functions (Database Engine) - SQL Server. Microsoft Learn. Retrieved December 21, 2023, from Microsoft Learn.
16.	Informatica. (n.d.). What is Data Transformation? [Web page]. Retrieved from https://www.informatica.com/resources/articles/what-is-data-transformation.html 

	
