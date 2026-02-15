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

----
#### Scenario 11

| Hackers                                                                                                           | Difficulty                                                                                                        | Challenges                                                                                                        | Submissions                                                                                                       |
| ----------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| ![](https://s3.amazonaws.com/hr-challenge-images/19504/1458526776-67667350b4-ScreenShot2016-03-21at7.45.59AM.png) | ![](https://s3.amazonaws.com/hr-challenge-images/19504/1458526915-57eb75d9a2-ScreenShot2016-03-21at7.46.09AM.png) | ![](https://s3.amazonaws.com/hr-challenge-images/19504/1458527032-f9ca650442-ScreenShot2016-03-21at7.46.17AM.png) | ![](https://s3.amazonaws.com/hr-challenge-images/19504/1458527077-298f8e922a-ScreenShot2016-03-21at7.46.29AM.png) |

Julia just finished conducting a coding contest, and she needs your help assembling the leaderboard! Write a query to print the respective _hacker_id_ and _name_ of hackers who achieved full scores for _more than one_ challenge. Order your output in descending order by the total number of challenges in which the hacker earned a full score. If more than one hacker received full scores in same number of challenges, then sort them by ascending _hacker_id_.


_**Solution**_

##### Step 1: The "Paper Trail" (Joining the Tables)

Imagine four filing cabinets. We need to pull a folder from one and find its matching folder in the next.
1. **Submissions (s):** This is our starting point. It lists every attempt made by every student.
2. **Challenges (c):** We look up the submission's `challenge_id` here to see _which_ challenge they were doing.
3. **Difficulty (d):** We look at the challenge's `difficulty_level` to find out the **maximum possible score** for that specific task.
4. **Hackers (h):** Finally, we look up the `hacker_id` from the **submission** to get the actual name of the student.

**The Golden Rule of Joins:** Always connect the student who _did_ the work (`s.hacker_id`), not the one who _created_ the task (`c.hacker_id`).

---

### Step 2: The "Quality Control" (The WHERE Clause)

Now we have a giant pile of data, but most of it is "noise." We only care about **Perfect Scores**.
We compare the student's actual score (`s.score`) to the maximum allowed for that difficulty level (`d.score`).

```sql
WHERE s.score = d.score
```

_If a student scored 10, but the max was 100, this line throws that record away._

---

### Step 3: The "Tally" (GROUP BY & COUNT)

Now we have a pile of only "Perfect" folders. But the same student might appear multiple times if they were perfect on different challenges. We need to stack all folders belonging to "Alice" in one pile and all folders for "Bob" in another.

```sql
GROUP BY h.hacker_id, h.name
```

Once they are in piles, we count how many folders are in each stack using `COUNT(s.submission_id)`.

---

### Step 4: The "Elite Filter" (The HAVING Clause)

Julia only wants students who were perfect **more than once**.

In SQL, you cannot use a `WHERE` clause on a count (an aggregate). You must use `HAVING`. It’s like a second filter that happens _after_ the piles are made.

```sql
HAVING COUNT(s.submission_id) > 1
```

_If Alice has 3 perfect scores, she stays. If Bob only has 1, his pile is removed._

---

### Step 5: The "Presentation" (ORDER BY)

Finally, we arrange the survivors.

- **Primary Sort:** The "Big Winners" (highest count) at the top (`DESC`).
- **Tie-Breaker:** If two people have the same number of wins, sort by their ID number (`ASC`).

---

### The Final Blueprint

When you put it all together, it looks like this:

```SQL

SELECT 
    h.hacker_id, 
    h.name
FROM Submissions AS s 
INNER JOIN Challenges AS c ON s.challenge_id = c.challenge_id
INNER JOIN Difficulty AS d ON c.difficulty_level = d.difficulty_level
INNER JOIN Hackers AS h ON s.hacker_id = h.hacker_id 
WHERE s.score = d.score
GROUP BY h.hacker_id, h.name
HAVING COUNT(s.submission_id) > 1
ORDER BY COUNT(s.submission_id) DESC, h.hacker_id ASC;
```

---
#### Scenario 12
You are given three tables: _Students_, _Friends_ and _Packages._ _Students_ contains two columns: _ID_ and _Name_. _Friends_ contains two columns: _ID_ and _Friend_ID_ (_ID_ of the ONLY best friend). _Packages_ contains two columns: _ID_ and _Salary_ (offered salary in $ thousands per month).

![](https://s3.amazonaws.com/hr-challenge-images/12895/1443820186-2a9b4939a8-1.png)

Write a query to output the names of those students whose best friends got offered a higher salary than them. Names must be ordered by the salary amount offered to the best friends. It is guaranteed that no two students got same salary offer.

**Sol**
##### The Step-by-Step Logic

1. **Start with the Student:** Join `Students` to `Friends` to find out who their best friend is.
2. **Get the Student's Salary:** Join the `Packages` table to the student.
3. **Get the Friend's Salary:** Join the `Packages` table _again_, but this time link it to the `Friend_ID`.
4. **Compare and Filter:** Use the `WHERE` clause to find cases where the friend's salary is higher than the student's.

```SQL
Select Name
from Students as s
Inner Join Friends as f on s.ID = f.ID
Inner Join Packages as p1 on s.ID = p1.ID
Inner Join Packages as p2 on f.Friend_ID = p2.ID
where p2.Salary > P1.Salary
order by p2.Salary;
```
