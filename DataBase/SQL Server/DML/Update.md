## Update Query
- To change a record in the data base we could use `update` 
- Syntax

```SQL
UPDATE [table_name]  
SET [column1] = [value1], [column2] = [value2], ...
```

- Here using the previous Syntax we have to know that we run such a query like this, it will affect all the records in the database for the selected column.
- For example:

```SQL
update employee 
set FName = 'Mohamed';
```

- This Query will change all the records in the `FName`  column to be `Mohamed`, so we have to be careful when dealing with update, or we could override all the records in the database.
- To Avoid this scenario we could use `Where` clause to select specific record using specific condition, then we will override this selected record only.

```SQL
update employee
set FName = "Mohamed"
where Id = 3;
```

- Here in the previous query the only record that will be override(change) is the record with `Id = 3`, so here we avoided overriding all the records in the database.
- **_Note: Be careful when updating records in a table! Notice the_ `WHERE` _clause in the_ `UPDATE` _statement. The_ `WHERE` _clause specifies which record(s) that should be updated. If you omit the_ `WHERE` _clause, all records in the table will be updated!_**

- We have to know if the condition after the `Where` clause match multiple records, then all those records will be updated to the new value.
