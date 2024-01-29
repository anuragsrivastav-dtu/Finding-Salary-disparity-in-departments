# BUSINESS CASE 
  
As a data analyst working in a firm. The manager is seeking insights into salary disparities and wants you to identify the departments with Salaries above and below the average within each department.  
B. Objective: Create a query that identifies a high amount of variation in departments salaries.  
C. Deliverable: List from a SQL database with Average Salary, Department, and a way to score variation  
D. Hypothesis: PWD department has been flagged a department that has a high amount of salary spread.

# METHODOLOGY
Simply put, a z-score (also called a standard score) gives you an idea of how far from the mean a data point is. But more technically it’s a measure of how many standard deviations below or above the population mean a raw score is.  
Here's a step-by-step process how i implemented the project :

1 Calculating the mean  
firstly the inbuilt function of mean in sql is used to find the mean of salaries in every department, using a group by clause, the output provided the mean salary of every departmetn in the company.  
2 calculating the standard deviation
again the inbuit standard deviation function is used to calculate st. deviation of salaries for every department.This is used as reference point for standardization.
3 determining z score   
the formula for z score is given below :
![image](https://github.com/anuragsrivastav-dtu/SQL-project-Finding-Salary-disparities-within-departments/assets/140643875/fdeee732-7d25-4853-afb0-db038ab6ce05)  
 the calculated values of mean and standard deviation is used in the above formula to get z scores for each department
Z-scores help identify outliers—individuals whose salaries deviate significantly from the mean. A Z-score greater than a certain threshold (e.g., ±2 or ±3) may indicate a salary that is notably higher or lower than the average.  
4 comparing z scores of different department to gain neccessary insights
The final z scores was compared , and the derived conclusions are discussed in the next section  

# CONCLUSION
Based on these metrics (coefficient_of_Variation, outliers, Standard_deviation) , here are the departments which shows the most variance and discrepancy in salary according to the metrics:  
1.	CMD: High CV(Coefficient_of_Variation) ,high standard deviation and some outliers
2.	AGR: High CV,fairly high Standard deviation, and some outliers
3.	CCC:  High CV,fair number of outliers
4.	PWD: High number of outliers, and fair CV
5.	CWA: Fairly high standard deviation ,high CV, a fair number of outliers.




Z-scores help identify outliers—individuals whose salaries deviate significantly from the mean. A Z-score greater than a certain threshold (e.g., ±2 or ±3) may indicate a salary that is notably higher or lower than the average.
Compare Z-scores Across Departments:

If you are analyzing salary disparities between departments, compare the Z-scores of employees in different departments. This allows you to identify departments where salaries deviate more or less from the overall company average.
Analyze Patterns and Trends:

Examine the patterns and trends revealed by the Z-scores. Are certain departments consistently above or below the company average? Are there specific individuals with Z-scores that stand out? This analysis can provide insights into potential salary disparities.

# SQL QUERY
--query the dataset

SELECT * FROM [dbo].[Salaries_now]  

--group by the dept and obtain the standard deviation and average  

WITH Departmentstats AS (  
    SELECT   
    Department  
	,STDEV(Salary) AS Standard_deviation  
	,AVG(Salary) AS Average  
FROM [dbo].[Salaries_now]  
WHERE Salary >= 10000  
GROUP BY Department  
)  


--create dept outliers   
SELECT  
    emp.Department  
	,emp.salary  
	,dt.Standard_Deviation  
	,dt.Average  
	,(emp.Salary-dt.Average)/dt.Standard_Deviation AS zscore  
FROM [dbo].[Salaries_now] AS emp  
JOIN Departmentstats AS dt  
    ON emp.Department=dt.Department  
WHERE emp.Salary >= 10000 ;  

-- join the two tables together using CTEs  
GO  

WITH Departmentstats AS (  
    SELECT    
    Department  
	,STDEV(Salary) AS Standard_deviation  
	,AVG(Salary) AS Average  
FROM [dbo].[Salaries_now]  
WHERE Salary >= 10000  
GROUP BY Department  
),  
DepartmentOutliers AS  (  
    SELECT emp.Department  
	,emp.salary  
	,dt.Standard_Deviation  
	,dt.Average  
	,(emp.Salary-dt.Average)/dt.Standard_Deviation AS zscore  
FROM [dbo].[Salaries_now] AS emp  
JOIN Departmentstats AS dt  
    ON emp.Department=dt.Department  
    WHERE emp.Salary >= 10000 )  

-- get the coefficient of variation and outlier count using a CASE statement on your zscore   
SELECT dt.Department,  
       dt.Standard_Deviation,  
	   dt.Average,  
	   dt.Standard_Deviation/dt.Average AS Coefficient_of_Variation,  
	   SUM(CASE WHEN (do.zscore > 1.96 OR do.zscore < -1.96 ) THEN 1 ELSE 0 END) AS Outlier_Count  
FROM Departmentstats AS dt  
LEFT JOIN DepartmentOutliers AS do  
       ON dt.Department=do.Department  
GROUP BY  
dt.Department, dt.Standard_deviation, dt.Average,  
dt.Standard_deviation/dt.Average  
ORDER BY dt.Department ;  

--use the round function to get rid of the trailing figures and get cleaner values  


WITH Departmentstats AS (  
    SELECT   
    Department  
	,STDEV(Salary) AS Standard_deviation  
	,AVG(Salary) AS Average  
FROM [dbo].[Salaries_now]  
WHERE Salary >= 10000  
GROUP BY Department  
),   
DepartmentOutliers AS  (   
    SELECT emp.Department  
	,emp.salary   
	,dt.Standard_Deviation  
	,dt.Average  
	,(emp.Salary-dt.Average)/dt.Standard_Deviation AS zscore  
FROM [dbo].[Salaries_now] AS emp  
JOIN Departmentstats AS dt  
    ON emp.Department=dt.Department  
    WHERE emp.Salary >= 10000 )  

SELECT dt.Department,  
       ROUND(dt.Standard_Deviation,2) AS Standard_Deviation,  
	   ROUND(dt.Average,2) AS Salary_Average,  
	   ROUND((dt.Standard_Deviation/dt.Average),2) *100 AS Coefficient_of_Variation,  
	   SUM(CASE WHEN (do.zscore > 1.96 OR do.zscore < -1.96 ) THEN 1 ELSE 0 END) AS Outlier_Count  
FROM Departmentstats AS dt  
LEFT JOIN DepartmentOutliers AS do  
       ON dt.Department=do.Department  
GROUP BY  
dt.Department, dt.Standard_deviation, dt.Average,  
dt.Standard_deviation/dt.Average  
ORDER BY Outlier_Count DESC ;  


