
-- Average Age
SELECT AVG(age) AS Average_Age FROM insurance_data;



-- Age Distribution
SELECT 
    age,
    COUNT(*) AS Frequency
FROM insurance_data
GROUP BY age
ORDER BY age;




-- Average Charges by Gender
SELECT 
    sex,
    AVG(charges) AS Average_Charge
FROM insurance_data
GROUP BY sex;




-- BMI Distribution by Gender
SELECT 
    sex,
    ROUND(AVG(bmi), 2) AS Average_BMI
FROM insurance_data
GROUP BY sex;



-- Proportion of Smokers
SELECT 
    smoker,
    COUNT(*) AS Total,
    COUNT(*) / (SELECT COUNT(*) FROM insurance_data) * 100 AS Percentage
FROM insurance_data
GROUP BY smoker;


-- Average Charges by Smoking Status
SELECT 
    smoker,
    AVG(charges) AS Average_Charge
FROM insurance_data
GROUP BY smoker;


-- Average Charges by Region

SELECT 
    region,
    AVG(charges) AS Average_Charge
FROM insurance_data
GROUP BY region;


-- Smoker Distribution by Region
SELECT 
    region,
    SUM(CASE WHEN smoker = 'yes' THEN 1 ELSE 0 END) AS Total_Smokers
FROM insurance_data
GROUP BY region;



-- BMI vs. Charges Scatter Plot (Data extraction for visualization)
SELECT bmi, charges FROM insurance_data;



-- Relationship between Children and Charges
SELECT 
    children,
    AVG(charges) AS Average_Charge
FROM insurance_data
GROUP BY children;



~~~ SQL
/*Age Categories*/

SELECT 
  age, region, charges,

  CASE 
  WHEN age <= 19 THEN 'Teen'
  WHEN age BETWEEN 20 AND 39 THEN 'Adult'
  WHEN age BETWEEN 40 AND 59 THEN 'Middle Age'
  WHEN age >= 60 THEN 'Senior'
  END AS AgeCategory
FROM insurance_data


~~~

~~~ SQL 
/*Age Category Averages*/
SELECT 
  age,
  ROUND(AVG(charges), 2) AS AvgCharge
FROM insurance_data
GROUP BY age
ORDER BY age;

~~~



~~~ SQL 
/*Correlation Between Age and Charges*/
WITH Calculations AS (
    SELECT 
        COUNT(*) AS n,
        SUM(age) AS sum_age,
        SUM(charges) AS sum_charges,
        SUM(age * charges) AS sum_age_charges,
        SUM(age * age) AS sum_age_squared,
        SUM(charges * charges) AS sum_charges_squared
    FROM insurance_data
)

SELECT 
    (n * sum_age_charges - sum_age * sum_charges) / 
    SQRT((n * sum_age_squared - sum_age * sum_age) * (n * sum_charges_squared - sum_charges * sum_charges)) AS correlation_coefficient
FROM Calculations;

-- Low correlation between age and charges 
~~~

**Result**

|CORR_Age_Charges|
|---|
|0.30|

~~~ SQL
SELECT  
  DISTINCT sex,
  FLOOR(charges / 10000) * 10000 AS charge_bins,
  COUNT(*) AS Count
FROM insurance_data
WHERE Sex = 'male'
GROUP BY sex, charge_bins
ORDER BY charge_bins
LIMIT 0, 25;

--Right skew
~~~

**Result**

|sex|charge_bins|count|
|---|---|---|
|male|0.0|357|
|male|10000.0|161|
|male|20000.0|55|
|male|30000.0|56|
|male|40000.0|43|
|male|50000.0|2|
|male|60000.0|2|

~~~ SQL 
SELECT  
  DISTINCT sex,
  FLOOR(charges / 10000) * 10000 AS charge_bins,
  COUNT(*) AS Count
FROM insurance_data
WHERE Sex = 'female'
GROUP BY sex, charge_bins
ORDER BY charge_bins
LIMIT 0, 25;

~~~

**Result**
|sex|charge_bins|Count|
|---|---|---|
|female|0.0|355|
|female|10000.0|192|
|female|20000.0|56|
|female|30000.0	|27|
|female|40000.0|29|
|female|50000.0|2|
|female|60000.0|1|

~~~ SQL 
/*Male and Female Average Charges*/
SELECT 
  DISTINCT Sex, 
  ROUND(AVG(charges) OVER (PARTITION BY sex),2) AS Avg_Charges_Sex
FROM insurance_data;
~~~

**Result**

|Sex|Avg_Charges_Sex|
|---|---|
|male|13956.75|
|female|12569.58|


~~~ SQL 
/*Smoker Region Totals*/
SELECT 
    region,
    SUM(CASE WHEN smoker = 'yes' THEN 1 ELSE 0 END) AS Smokers_Count
FROM insurance_data
GROUP BY region
ORDER BY region;

//////////////////////////////////

SELECT
  DISTINCT region,smoker, COUNT(smoker) OVER (PARTITION BY region, 
  smoker) AS SmokerRegionCount,COUNT(smoker) OVER () AS TotalSmokerCount,
  ROUND((COUNT(smoker) OVER (PARTITION BY region, smoker)/COUNT(smoker) OVER ()*100),0) AS SmokerPercentage
FROM insurance_data
WHERE smoker = true
ORDER BY SmokerRegionCount;
--Southwest has the least amount of smokers
~~~

**Result**

|region|smoker|SmokerRegionCount|TotalSmokerCount|SmokerPercentage|
|---|---|---|---|---|
|northwest|true|58|274|21.0|
|southwest|true|58|274|21.0|
|northeast|true|67|274|24.0|
|southeast|true|91|274|33.0|

~~~ SQL
-- AVERAGE SMOKER CHARGES
SELECT
  Smoker, 
  ROUND(AVG(charges),2) AS Avg_Smoking_Charges
FROM `single-being-353600.Medical_Insurance_Data.Insurance_Data`
GROUP BY Smoker;
~~~
**Result**

|Smoker|Avg_Smoking_Charges|
|---|---|
|false|8434.27|
|true|32050.23|

~~~ SQL
/*Comparing charges for smokers to non-smokers*/

SELECT 
    smoker AS Smoking_Status,
    COUNT(*) AS Total_Individuals,
    ROUND(AVG(charges), 2) AS Average_Charge,
    MIN(charges) AS Minimum_Charge,
    MAX(charges) AS Maximum_Charge,
    ROUND(STDDEV(charges), 2) AS Standard_Deviation_Charge
FROM insurance_data
GROUP BY smoker
ORDER BY Smoking_Status;

--smoking accounts for an average cost that is about four times greater than a non-smoker
~~~

**Result**
|Smoker_Non_Smoker_Comparsion|
|---|
|4.0|

~~~ SQL

/*Smokers by Region - Sex*/
SELECT
  DISTINCT sex,region,smoker, COUNT(smoker) OVER (PARTITION BY region, sex, smoker) AS SmokerRegionCount,
  COUNT(smoker) OVER () AS SmokerTotal, 
  ROUND((COUNT(smoker) OVER (PARTITION BY region, sex, smoker)/COUNT(smoker) OVER () * 100),0) AS RegionFemaleSmokerPercentage
FROM insurance_data
WHERE smoker = true
ORDER BY SmokerRegionCount;


SELECT 
    region,
    sex,
    COUNT(*) AS Total_Smokers
FROM insurance_data
WHERE smoker = 'yes'
GROUP BY region, sex
ORDER BY region, sex;

--Southwest has lowest number of female smokers
~~~

**Result**
|sex|region|smoker|SmokerRegionCount|SmokerTotal|RegionSexSmokerPercentage|
|---|---|---|---|---|---|
|female|southwest|true|21|274|8.0|
|female|northwest|true|29|274|11.0|
|female|northeast|true|29|274|11.0|
|male|northwest|true|29|274|11.0|
|female|southeast|true|36|274|13.0|
|male|southwest|true|37|274|14.0|
|male|northeast|true|38|274|14.0|
male|southeast|true|55|274|20.0|

~~~ SQL 
/*Children and Charge Correlation*/
SELECT 
    (
        COUNT(*) * SUM(children * charges) - SUM(children) * SUM(charges)
    ) / 
    SQRT(
        (COUNT(*) * SUM(children * children) - SUM(children) * SUM(children))
        *
        (COUNT(*) * SUM(charges * charges) - SUM(charges) * SUM(charges))
    ) AS Correlation_Coefficient
FROM insurance_data;

~~~

**Result**
|CORR_CO_Charges|
|---|
|0.07|


~~~ SQL 
/*BMI Categories*/
SELECT 
    CASE 
        WHEN bmi < 18.5 THEN 'Underweight'
        WHEN bmi >= 18.5 AND bmi < 25 THEN 'Normal weight'
        WHEN bmi >= 25 AND bmi < 30 THEN 'Overweight'
        ELSE 'Obese'
    END AS BMI_Category,
    COUNT(*) AS Count
FROM insurance_data
GROUP BY BMI_Category
ORDER BY FIELD(BMI_Category, 'Underweight', 'Normal weight', 'Overweight', 'Obese');
~~~
