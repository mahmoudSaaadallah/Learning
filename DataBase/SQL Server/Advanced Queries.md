let's consider that we have the following employee table:


| EmployeeID | FirstName | LastName | DepartmentID | Salary | HireDate   |                                                                               |
| ---------- | --------- | -------- | ------------ | ------ | ---------- | ----------------------------------------------------------------------------- |
| 101        | Alice     | Smith    | 1            | 70000  | 2020-01-15 |                                                                               |
| 102        | Bob       | Johnson  | 2            | 85000  | 2019-03-10 |                                                                               |
| 103        | Carol     | Davis    | 1            | 72000  | 2020-02-20 |                                                                               |
| 104        | David     | Brown    | 6            | 60000  | 2021-07-01 |                                                                               |
| 105        | Eve       | White    | NULL         | 65000  | 2020-05-12 |                                                                               |
| 106        | Frank     | Green    | 3            | 90000  | 2018-11-01 |                                                                               |
| 107        | Grace     | Hopper   | 5            | 95000  | 2017-09-20 |                                                                               |
| 108        | Charlie   | Chaplin  | 2            | 78000  | 2019-05-01 |                                                                               |
| 109        | Anna      | Anderson | 1            | 71000  | 2020-01-15 | -- Alice and Anna hired on same date, different salaries                      |
| 110        | John      | Doe      | 2            | 85000  | 2019-03-10 | -- Bob and John have same salary and hire date (tie for salary and hire date) |

#### Scenario number one
- Get the **Highest Salary in each department **

```sql
select *
from (
	select *, row_number() over(partition by DepartmentId order by Salary Desc) as Ranking
	from employee
) as RankedTable
where Ranking = 1;


or 

with RankedTable as(
	select *, row_number()  over(partition by DepartmentID order by salary) as ranking
	from employee
)select *
from RankedTable
where ranking = 1;
```
[[Rank()]][[Row_Number()]]
- In both of the previous statement we used the `row_number()` function with `partition by` to divide the result of the select statement into groups as the `partition by` will do.
- then we used the output from the inner statement in the first query and `with` statement in the second query to get only the records that has a `Ranking = 1`, which is the highest salary in each department.
---

#### Scenario number two
- Get the highest 2 salaries from each department, with ties in the salary value, so even if there are more than 2 employees have the highest salary return all of them

```SQL
with SalaryDense as(
	select *, Denes_rank() over(partition by DepartmentID order by salary desc) as DR
	from employee
)select *
from SalaryDense
where DR <= 2;


or 


select * 
from (
	select *, Dense_Rank() over(partition by DepartmentId order by salary Desc) as DR
	from employee
) as SalaryDense
where DR <= 2;
```
[[Dense_Rank()]]

---
#### Scenario number three

You want to count how many employees are in each department, but also get a count of "Senior" employees (e.g., those with more than 10 years of experience) within each department, all in one query.

```sql
SELECT
    DepartmentName,
    COUNT(EmployeeID) AS TotalEmployees,
    SUM(CASE WHEN YearsOfExperience > 10 THEN 1 ELSE 0 END) AS SeniorEmployees
FROM
    Employees
GROUP BY
    DepartmentName;
```
[[Case]]

---
#### Scenario 4: Ordering Results with Custom Logic

You want to sort a list of tasks, prioritizing 'High' priority tasks first, then 'Medium', then 'Low', and finally any 'Unknown' priority tasks.

```sql
SELECT
    TaskID,
    TaskName,
    Priority
FROM
    Tasks
ORDER BY
    CASE Priority
        WHEN 'High' THEN 1
        WHEN 'Medium' THEN 2
        WHEN 'Low' THEN 3
        ELSE 4 -- Assign a numerical order to each priority level
    END,
    TaskName; -- Secondary sort for tasks within the same priority
```
**Example Output (conceptual):**

| TaskID | TaskName          | Priority |
|--------|-------------------|----------|
| 201    | Fix critical bug  | High     |
| 205    | Review code       | High     |
| 202    | Update documentation | Medium   |
| 204    | Plan next sprint  | Medium   |
| 203    | Clean up old data | Low      |
| 206    | Research new tech | Unknown  |


---
#### Scenario 5
Consider the following table
![[Pasted image 20260117172253.png]]

A _median_ is defined as a number separating the higher half of a data set from the lower half. Query the _median_ of the _Northern Latitudes_ (_LAT_N_) from **STATION** and round your answer to 4 decimal places.
```SQL
with NumberedTable as(
	select LAT_N,
		row_number() over(order by LAT_N) as RN,
		count(*) over() as counted
	from station
)select LAT_N
from NumberedTable
where RN in ((counted + 1) / 2, (counted + 2) / 2);
```



---
#### Scenario 6
The total score of a hacker is the sum of their maximum scores for all of the challenges. Write a query to print the _hacker_id_, _name_, and total score of the hackers ordered by the descending score. If more than one hacker achieved the same total score, then sort the result by ascending _hacker_id_. Exclude all hackers with a total score of `0` from your result.

**Input Format**

The following tables contain contest data:

- _Hackers:_ The _hacker_id_ is the id of the hacker, and _name_ is the name of the hacker.

![](https://s3.amazonaws.com/hr-challenge-images/19503/1458522826-a9ddd28469-ScreenShot2016-03-21at6.40.27AM.png)

- _Submissions:_ The _submission_id_ is the id of the submission, _hacker_id_ is the id of the hacker who made the submission, _challenge_id_ is the id of the challenge for which the submission belongs to, and _score_ is the score of the submission.

![](https://s3.amazonaws.com/hr-challenge-images/19503/1458523022-771511df90-ScreenShot2016-03-21at6.40.37AM.png)

```SQL
with NewTable as(
	select hacker_id, challenge_id, max(score) as mx
	from Submissions
	group by hacker_id, challenge_id
)select new.hacker_id, h.name, sum(mx) as sm
from Hackers as h inner join NewTable as new
on h.hacker_id = new.hacker_id 
group by new.hacker_id, h.name
having sum(mx) > 0
order by sm desc, new.hacker_id asc;	
```


----
#### Scenario 7
You are given a table, _Functions_, containing two columns: _X_ and _Y_. 

![](https://s3.amazonaws.com/hr-challenge-images/12892/1443818798-51909e977d-1.png)

Two pairs _(X1, Y1)_ and _(X2, Y2)_ are said to be _symmetric_ _pairs_ if _X1 = Y2_ and _X2 = Y1_.
Write a query to output all such _symmetric_ _pairs_ in ascending order by the value of _X_. List the rows such that _X1 ≤ Y1_.

```SQL
select f1.x, f1.y
from functions f1 inner join functions f2
on f1.x = f2.y and f1.y = f2.x
group by f1.x, f1.y
having f1.x < f1.y or (f1.x = f1.y and count(*) > 1)
order by f1.x
```

1. **The Self-Join**
We join the table to itself (`f1` and `f2`) using the symmetric condition: `f1.X = f2.Y` and `f1.Y = f2.X`. This pairs every row with its potential mirror image.

 **2. The `GROUP BY` and `HAVING` Clauses**
This is where we filter for valid pairs and the specific requirements:
- **`f1.X < f1.Y`**: This handles the unique pairs. By requiring $X < Y$, we satisfy the requirement $X_1 \leq Y_1$ and ensure we don't list the same symmetric pair twice (e.g., we show $(20, 21)$ but not $(21, 20)$).
- **`f1.X = f1.Y AND COUNT(*) > 1`**: This handles the "equal" case. If $X=Y$, a self-join will always match a row to itself. However, for it to be a _symmetric pair_, there must be a _different_ row with the same values. Counting the occurrences ensures there are at least two.

---

#### Scenario 7
Consider the following table 

| Region  | ProductCategory | ProductSubCategory | SalesAmount |
| ------- | --------------- | ------------------ | ----------- |
| East    | Electronics     | Laptops            | 1500.00     |
| East    | Electronics     | Laptops            | 1200.00     |
| East    | Electronics     | Smartphones        | 800.00      |
| East    | Clothing        | Shirts             | 300.00      |
| West    | Electronics     | Laptops            | 1800.00     |
| West    | Electronics     | Smartphones        | 950.00      |
| West    | Clothing        | Pants              | 450.00      |
| Central | Clothing        | Shirts             | 200.00      |
| Central | Electronics     | Laptops            | 1000.00     |

We want to get the sales totals for each `Region` and for each `ProductCategory`, which means sales Amount for `electronics` in each `Region` and `clothing` for each `Region` and so on.
Then get the total `SalesAmount` for all `Regions`

```SQL
-- 1. Detailed Sales (Region and Category)
select region, ProductCategory, sum(SalesAmount) as SalesAmount
from Sales
group by region, ProductCategory
union 
select region, null,sum(sales.SalesAmount)
from Sales
group by region
union
select 'ztotal', null, sum(salesAmount)
from sales
order by region asc, SalesAmount;
```
[[Union Family]]

| Region  | ProductCategory | TotalSales | --                             |
| :------ | :-------------- | :--------- | ------------------------------ |
| Central | Clothing        | 200.00     |                                |
| Central | Electronics     | 1000.00    |                                |
| Central | NULL            | 1200.00    | -- Subtotal for Central Region |
| East    | Clothing        | 300.00     |                                |
| East    | Electronics     | 3500.00    |                                |
| East    | NULL            | 3800.00    | -- Subtotal for East Region    |
| West    | Clothing        | 450.00     |                                |
| West    | Electronics     | 2750.00    |                                |
| West    | NULL            | 3200.00    | -- Subtotal for West Region    |
| ztotal  | NULL            | 8200.00    | -- Grand Total                 |

> using `ROLLUP` [[T-SQL Rollup]]

Also we could use `Rollup` to solve this problem with more simple code.

```SQL
select Region, ProductCategory, SUM(SalesAmount) AS TotalSales
from Sales
group by rollup (Region, ProductCategory);
```

| Region  | ProductCategory | TotalSales | --                             |
| :------ | :-------------- | :--------- | ------------------------------ |
| Central | Clothing        | 200.00     |                                |
| Central | Electronics     | 1000.00    |                                |
| Central | NULL            | 1200.00    | -- Subtotal for Central Region |
| East    | Clothing        | 300.00     |                                |
| East    | Electronics     | 3500.00    |                                |
| East    | NULL            | 3800.00    | -- Subtotal for East Region    |
| West    | Clothing        | 450.00     |                                |
| West    | Electronics     | 2750.00    |                                |
| West    | NULL            | 3200.00    | -- Subtotal for West Region    |
| NULL    | NULL            | 8200.00    | -- Grand Total                 |


> Using `GOUPING SETS` [[T-SQL Grouping sets]]

```SQL
SELECT Region, ProductCategory, SUM(SalesAmount) AS TotalSales
from Sales
group by grouping sets((Region, ProductCategory), (Region), ())
```

| Region  | ProductCategory | TotalSales | --                             |
| :------ | :-------------- | :--------- | ------------------------------ |
| Central | Clothing        | 200.00     |                                |
| Central | Electronics     | 1000.00    |                                |
| Central | NULL            | 1200.00    | -- Subtotal for Central Region |
| East    | Clothing        | 300.00     |                                |
| East    | Electronics     | 3500.00    |                                |
| East    | NULL            | 3800.00    | -- Subtotal for East Region    |
| West    | Clothing        | 450.00     |                                |
| West    | Electronics     | 2750.00    |                                |
| West    | NULL            | 3200.00    | -- Subtotal for West Region    |
| NULL    | NULL            | 8200.00    | -- Grand Total                 |

---

#### Scenario 8

| SaleID | SaleYear | Product  | Amount  | Region |
| :----- | :------- | :------- | :------ | :----- |
| 1      | 2023     | Laptop   | 1000.00 | North  |
| 2      | 2023     | Mouse    | 50.00   | North  |
| 3      | 2023     | Keyboard | 75.00   | South  |
| 4      | 2024     | Laptop   | 1200.00 | South  |
| 5      | 2024     | Mouse    | 60.00   | South  |
| 6      | 2024     | Monitor  | 200.00  | East   |
| 7      | 2023     | Laptop   | 1100.00 | North  |
| 8      | 2024     | Keyboard | 80.00   | East   |

**Goal**: Show total sales amount for each product, broken down by `SaleYear` AND `Region`, with products as columns.

```sql
SELECT
    SaleYear,
    Region,
    [Laptop],
    [Mouse],
    [Keyboard],
    [Monitor]
FROM
    (SELECT SaleYear, Region, Product, Amount FROM Sales) AS SourceTable
PIVOT
(
    SUM(Amount)
    FOR Product IN ([Laptop], [Mouse], [Keyboard], [Monitor])
) AS PivotTable
ORDER BY
    SaleYear, Region;
```
[[T-SQL PIVOT]]
**Result of Example 2:**

| SaleYear | Region | Laptop  | Mouse | Keyboard | Monitor |
| :------- | :----- | :------ | :---- | :------- | :------ |
| 2023     | North  | 2100.00 | 50.00 | NULL     | NULL    |
| 2023     | South  | NULL    | NULL  | 75.00    | NULL    |
| 2024     | East   | NULL    | NULL  | 80.00    | 200.00  |
| 2024     | South  | 1200.00 | 60.00 | NULL     | NULL    |


----
#### Scenario 9
Julia asked her students to create some coding challenges. Write a query to print the _hacker_id_, _name_, and the total number of challenges created by each student. Sort your results by the total number of challenges in descending order. If more than one student created the same number of challenges, then sort the result by _hacker_id_. If more than one student created the same number of challenges and the count is less than the maximum number of challenges created, then exclude those students from the result.

Hackers
![](https://s3.amazonaws.com/hr-challenge-images/19506/1458521004-cb4c077dd3-ScreenShot2016-03-21at6.06.54AM.png)

Challenges

![](https://s3.amazonaws.com/hr-challenge-images/19506/1458521079-549341d9ec-ScreenShot2016-03-21at6.07.03AM.png)

To solve this in SQL Server, we need to:

1. Count challenges per hacker.
2. Determine the **Maximum** number of challenges created by anyone.
3. Determine which challenge counts are **Unique** (only one hacker has that count).
4. Apply the filter: Keep hackers if they hit the maximum OR if their count is unique.
```SQL
WITH ChallengeCounts AS (
    -- Step 1: Get the count for every hacker
    SELECT 
        h.hacker_id, 
        h.name, 
        COUNT(c.challenge_id) AS total_challenges
    FROM Hackers h
    JOIN Challenges c ON h.hacker_id = c.hacker_id
    GROUP BY h.hacker_id, h.name
),
CounterSummary AS (
    -- Step 2: Analyze the counts to find the Max and the frequency of each count
    SELECT 
        total_challenges,
        COUNT(*) AS count_frequency,
        MAX(total_challenges) OVER() AS max_challenges
    FROM ChallengeCounts
    GROUP BY total_challenges
)
-- Step 3: Join and filter based on Julia's rules
SELECT 
    cc.hacker_id, 
    cc.name, 
    cc.total_challenges
FROM ChallengeCounts cc
JOIN CounterSummary cs ON cc.total_challenges = cs.total_challenges
WHERE 
    cc.total_challenges = cs.max_challenges -- Rule: Keep all if they hit the max
    OR cs.count_frequency = 1               -- Rule: Keep if the count is unique
ORDER BY cc.total_challenges DESC, cc.hacker_id;
```

---
#### Scenario 10
Students
![](https://s3.amazonaws.com/hr-challenge-images/12891/1443818166-a5c852caa0-1.png)

Grades
![](https://s3.amazonaws.com/hr-challenge-images/12891/1443818137-69b76d805c-2.png)

_Ketty_ gives _Eve_ a task to generate a report containing three columns: _Name_, _Grade_ and _Mark_. _Ketty_ doesn't want the NAMES of those students who received a grade lower than _8_. The report must be in descending order by grade -- i.e. higher grades are entered first. If there is more than one student with the same grade (8-10) assigned to them, order those particular students by their name alphabetically. Finally, if the grade is lower than 8, use "NULL" as their name and list them by their grades in descending order. If there is more than one student with the same grade (1-7) assigned to them, order those particular students by their marks in ascending order.

```SQL
select 
	case
	when g.grade >= 8 then s.name
	else null 
	end as name,
	g.grade,
	s.marks
from Students as s inner join Grades as g
on s.marks between g.Min_Marks and g.Max_Marks
order by 
	g.grade desc,
	name asc,
	marks asc
```
[[Case]]