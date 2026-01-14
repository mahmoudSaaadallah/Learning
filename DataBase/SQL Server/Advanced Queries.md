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