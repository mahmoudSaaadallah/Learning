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
