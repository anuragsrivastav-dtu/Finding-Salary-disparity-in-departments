# salary disparities within departments
A. Business Case:  
As a data analyst working in a firm. The manager is seeking insights into salary disparities and wants you to identify the departments with Salaries above and below the average within each department.  
B. Objective: Create a query that identifies a high amount of variation in departments salaries.  
C. Deliverable: List from a SQL database with Average Salary, Department, and a way to score variation  
D. Hypothesis: PWD department has been flagged a department that has a high amount of salary spread.

# queries 
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


