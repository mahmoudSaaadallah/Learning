
| EmployeeID | FirstName | LastName | DepartmentID | Salary |                                |
| ---------- | --------- | -------- | ------------ | ------ | ------------------------------ |
| 101        | Alice     | Smith    | 1            | 70000  |                                |
| 102        | Bob       | Johnson  | 2            | 85000  |                                |
| 103        | Carol     | Davis    | 1            | 72000  |                                |
| 104        | David     | Brown    | 6            | 60000  | -- Department 6 does not exist |
| 105        | Eve       | Null     | NULL         | 65000  | -- No department assigned      |
| 106        | Frank     | Green    | 3            | 90000  |                                |

- In SQL Server when we retrieve data from a table and this table has a null value, like if we want to get the employees LastName from the employees table then we could use the following query.
```SQL
select LastName
from employee
```
- This query will return all the `LastName` column from the employee table even the null values.
- so the result will be:

| LastName |
| -------- |
| Smith    |
| Johnson  |
| Davis    |
| Brown    |
| Null     |
| Green    |

- Now using the `isnull()` function we could replace the `null` value with any value that we want.
- Syntax
```SQL
select isnull(<column>, <value>)
from <table>
```
- what `isnull()` function do is the check if the record has a data, then it will return this data, else if it is `null` then it will will replace the `null` with the default `value` that provided in the function.

```SQL
select isnull(LastName, 'N/A')
from employee
```
- Then the result will be like below:

| LastName |
| -------- |
| Smith    |
| Johnson  |
| Davis    |
| Brown    |
| N/A      |
| Green    |

- As we can see in the result the null value was replaced with `N/A` which is the default value that we provided to the `isnull()` function.
---
**May be in other scenario the requirement is to display the `LastName` for employees if exist else display the `FirstName`**
= 
```SQL
select isnull(LastName, FirstName)
from employee
```

| LastName |
| -------- |
| Smith    |
| Johnson  |
| Davis    |
| Brown    |
| Eve      |
| Green    |


- Now what if the `FirstName`, and the `LastName` have `Null`, then the function will return `Null`.
- But what if want to add another replacement value to be added if both the `FirstName` and `LastName` are `Null`?
- Then we have to use [[Coalesce ()]].