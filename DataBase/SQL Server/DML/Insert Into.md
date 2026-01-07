## Insert Into 
- To insert data into a table in database, we use `insert into` command.

```SQL
insert into [table name] values (val1, val2, val3, ...)
```
- This is the stander of the syntax for the `insert` command, but we have to know the data that will be inserted to that table which provided in after `values(...)` must me in the same order as the table columns as 
		`val1` => `col1`
		`val2` => `col2`
		`val3` => `col2`
- So We have to provide a value for each column and must be ordered in the same order of the columns.
- Even if there is a column that allows `null`, then we have to write null in the value for it.

- Example
```SQL
create table employee (
	Id int primary key,
	FName varchar(25) not null,
	LName varchar(25) not null,
	Salary decimal(7, 2) check (Salary >= 7000 or Salary is null),
	age int,
	email varchar(250) unique,
)

insert into employee values
(1, 'Mahmoud', 'Saadallah', 17000.00, null,'mahmoudsaadallah@gmail.com')
```

- So here we added a value for each column, but the question is _could I insert data for specific columns or even insert data without restrict to the order of the columns?_
- The answer is **Yes**.
- We could do that by specifying the columns name after the table name, so the syntax will be:

```SQL
insert into employee(Id, email, FName, LName) values (2, 'omar@gmail.com', 'Omar', 'Ibrahim')
```

- As we can see here we  specify some columns that we are going to insert data inside it, but we have to know that the unmentioned columns must allow null, or this query will return error.

### Insert multiple records in single query
- We don't have to repeat the query to insert multiple records to the table, instead we could add multiple records by specifying the values for each record

```SQL
insert into employee (Id, email, FName, LName)
values
	(3, 'saad@gamil.com', 'Saad', 'Osman'),
	(4, 'gaiper@gmail.com', 'Gaiper', 'Mostafa'),
	(5, 'Ra7eem@gmail.com', 'Raheem', 'Baker');
```

- This Query will insert three records to the employee table in the database.