# Healthcare-Analysis-Project-with-SQL-and-Tableau

Healthcare Analysis Project with SQL and Tableau


Introduction: In the evolving landscape of healthcare, data-driven insights are crucial for improving patient care, optimizing hospital operations, and reducing costs. This analysis provides a meticulous overview of critical healthcare metrics, offering stakeholders a powerful tool for decision-making. This project aims to deliver an intuitive and visually appealing interface. The analysis of this data was performed using PostgreSQL and Tableau. By utilizing both SQL and Tableau, the project provides flexibility to cater to diverse user preferences and requirements. This project used data from Kaggle.com.


The challenges of this project are:
1. What is the average cost per day for a patient? What was the average billing amount?
2. What is the average time the patients typically spend in the hospital?
3. Are there any patterns in billing amounts related to specific medical conditions?
4. What are the most common medical conditions for different age groups?
5. Which doctors have the highest number of patient admissions?
6. What is the distribution of admission type?
7. What medications are most commonly prescribed for specific medical conditions?
8. Is there a correlation between gender and the prevalence of certain medical conditions?
Process
The data for this analysis contains one table which was created and then imported to PostreSQL.

CREATE TABLE Patients (
    Name VARCHAR(100),
    Age INT,
    Gender VARCHAR(10),
    BloodType VARCHAR(3),
    MedicalCondition TEXT,
    DateOfAdmission DATE,
    Doctor VARCHAR(100),
    Hospital VARCHAR(100),
    InsuranceProvider VARCHAR(100),
    BillingAmount DECIMAL(15, 5),
    RoomNumber VARCHAR(10),
    AdmissionType VARCHAR(50),
    DischargeDate DATE,
    Medication TEXT,
    TestResults TEXT
);

Real-world data is often messy, and this can lead to flawed conclusions and wasted resources. It is crucial to ensure that your data is clean and well-prepared for analysis. This ensures that your insights are accurate, forming a reliable foundation for business decisions. The first project step is to clean the data.  
All names have been changed:

Update patients
SET name=UPPER(name);

I wanted to find out if there are duplicates. Duplicate data is inefficient to store and can cause bad results. 

WITH duplicate_CTE AS (
    SELECT*, row_number() OVER(PARTITION BY full_name, age, gender, bloodtype,
medicalcondition, dateofadmission, doctor, hospital, insuranceprovider, admissiontype,
BillingAmount, roomnumber,admissiontype, dischargedate, medication,testresults) AS row_num 
FROM patients)
SELECT *
FROM duplicate_CTE
WHERE row_num>1

I created another table named patienten and then I copied the DISTINCT data there. I deleted the old table patients since there was no used to me anymore. 

INSERT INTO Patienten SELECT DISTINCT * FROM patients;

Because the data contained negative numbers in the billingamount column, I converted the negative numbers to positive numbers.

UPDATE patienten 
SET BillingAmount = ABS(BillingAmount)
WHERE BillingAmount < 0 
1. What is the average cost per day for a patient? What was the average billing amount?
2. What is the average time the patients typically spend in the hospital?
To answer these questions I used the aggregation function AVG to calculate the average billing amount for all patients, to find out the the average length of stay in the hospital for patients and to determine the average cost per day for a patient by dividing the average billing amount by the average length of stay.

SELECT  
    AVG(BillingAmount) as avg_billing,
    AVG(DischargeDate - DateOfAdmission) as AVG_LengthOfStay,
    AVG (BillingAmount) / AVG(DischargeDate - DateOfAdmission) as bill_per_day
FROM 
    Patienten
 
3. Are there any patterns in billing amounts related to specific medical conditions?
To identify patterns in billing amounts related to specific medical conditions, I used the aggregation functions: COUNT, SUM, AVG, MIN and MAX. Also I used GROUP BY and ORDER BY functions to extract specific data. The SQL query aggregates billing data by medical condition, providing summary statistics such as total records, total billing amount, average billing amount, and the minimum and maximum billing amounts for each condition. 
SELECT 
    medicalcondition,
    COUNT(*) AS TotalRecords,
    SUM(BillingAmount) AS TotalBillingAmount,
    AVG(BillingAmount) AS AverageBillingAmount,
    MIN(BillingAmount) AS MinimumBillingAmount,
    MAX(BillingAmount) AS MaximumBillingAmount
FROM 
    Patienten
GROUP BY 
    medicalcondition
ORDER BY 
    medicalcondition;
 

4. What are the most common medical conditions for different age groups?
First, I wanted to see the minimum and maximum age for the patients, therefore I used the aggregation functions: MIN and MAX.
SELECT 
MIN(age),
MAX(age)
FROM patienten
Based on this information, I divided the patients into 8 different age groups.  To add the condition and return specific values, I used the CASE WHEN functions.The group by function groups the results by each medical condition so that the counts are calculated for each condition separately. The SQL query provided is designed to identify the most common medical conditions for different age groups by counting the occurrences of each condition within specified age ranges.
SELECT medicalcondition, 
count (CASE WHEN age<21 THEN 1 end) AS "age<21",
count(CASE WHEN age between 21 and 30 then 1 end) as "21<age<30",
count(CASE WHEN age between 31 and 40 then 1 end) as "31<age<40",
count(CASE WHEN age between 41 and 50 then 1 end) as "41<age<50",
count(CASE WHEN age between 51 and 60 then 1 end) as "51<age<60",
count(CASE WHEN age between 61 and 70 then 1 end) as "61<age<70",
count(CASE WHEN age between 71 and 80 then 1 end) as "71<age<80",
count(CASE WHEN age between 81 and 89 then 1 end) as "81<age<89"
FROM patienten
GROUP BY medicalcondition

5. Which doctors have the highest number of patient admissions?
This part of the query counts the number of doctors. The SQL query effectively identifies which doctors have the highest number of patient admissions by counting the doctor and sorting the results in descending order.

SELECT doctor, count(doctor) as count_doctor
FROM patienten
GROUP BY doctor
order BY count_doctor desc




6. What is the distribution of admission type?
In this query I counted the number of records for each admission type and assign this count to a new column. I used the aggregation function COUNT.
SELECT admissiontype,    
    count(*)
    FROM 
    Patienten
    GROUP BY admissiontype
    ORDER BY admissiontype

7. What medications are most commonly prescribed for specific medical conditions?
I used the same query as above to effectively identify the most commonly prescribed medications for specific medical conditions by counting and organizing the data by condition and medication. 

SELECT 
    medication, medicalcondition,
    COUNT(*) AS TotalRecords
    FROM Patienten
GROUP BY 
     medicalcondition,medication
ORDER BY medicalcondition, totalRecords desc

8. Is there a correlation between gender and the prevalence of certain medical conditions?
The SQL query aims to explore the correlation between gender and the prevalence of certain medical conditions by counting the occurrences of each medical condition for each gender.

SELECT gender, medicalcondition, count(gender)
FROM patienten
GROUP BY medicalcondition, gender
order by medicalcondition

Findings
1. What is the average cost per day for a patient? What was the average billing amount?
Average cost per day for a patient: 1648 EUR
Average billing amount: 25546 EUR
2. What is the average time the patients typically spend in the hospital?
Average time spent in the hospital: 15 days
3. Are there any patterns in billing amounts related to specific medical conditions?
There is a wide range between the minimum and maximum billing amounts for each condition. For instance, cancer has the lowest minimum billing amount at 9 EUR, while hypertension has the highest maximum billing amount at 52 764 EUR. This significant range suggests variability in treatment costs based on the severity and specific requirements of individual cases.
The average billing amounts across different medical conditions are relatively close, ranging from approximately 25 154 EUR (cancer) to 25 806 EUR (obesity). This indicates a certain level of consistency in the overall cost structure across these conditions.
The minimum billing amounts are relatively low across all conditions, indicating that there are cases with minimal intervention or treatment required. 
4. What are the most common medical conditions for different age groups?
The data reveals distinct patterns in the prevalence of medical conditions across different age groups. Arthritis is prevalent in the youngest and oldest groups, while obesity and diabetes are significant concerns in middle age. Hypertension becomes more common in the elderly, indicating changing health priorities as people age.

5. Which doctors have the highest number of patient admissions?
This information can be useful for hospital management to recognize doctors with high patient volumes and potentially adjust their schedules or support staff to ensure optimal patient care.

6. What is the distribution of admission type?

The distribution of admission types shows that there is a relatively even spread among Elective, Emergency, and Urgent admissions. Elective admissions are the most common, followed closely by Urgent and then Emergency admissions. This distribution suggests a balanced approach to different types of patient admissions.


7. What medications are most commonly prescribed for specific medical conditions?

This analysis shows a pattern of specific medications being favored for certain conditions, which can help in understanding prescription trends and managing medical treatment plans.
Aspirin is most commonly prescribed for Arthritis.
Paracetamol is most commonly prescribed for Asthma.
Lipitor is most commonly prescribed for Cancer and Diabetes.
Ibuprofen is most commonly prescribed for Hypertension.
Penicillin is most commonly prescribed for Obesity.


8. Is there a correlation between gender and the prevalence of certain medical conditions?

The data suggests that there is no strong correlation between gender and the prevalence of these specific medical conditions. Any observed differences are minimal and likely not statistically significant.

Conclusions
Using SQL and Tableau for a project is a powerful combination as SQL efficiently handles data extraction, transformation, and complex querying from databases, while Tableau excels at visualizing and presenting data insights. SQL prepares and structures the data, ensuring accuracy and relevance, and Tableau transforms this data into interactive and intuitive visualizations, facilitating data-driven decision-making and storytelling. Together, they enhance the analytical process, allowing for comprehensive data analysis and effective communication of results.

