Here's a breakdown of their key aspects:

- **Purpose**: They are designed to help database administrators and developers manage and monitor SQL Server instances, databases, and objects. They can retrieve system information, configure server settings, perform maintenance tasks, and more.
- **Naming Convention**: Most built-in stored procedures start with `sp_` (for system procedures) or `xp_` (for extended stored procedures).
- **Execution**: You can execute them just like any other stored procedure, often with parameters to specify the desired action or information.
- **Benefits**:
    - **Efficiency**: They are pre-compiled, which means they execute faster than ad-hoc queries for the same tasks.
    - **Security**: They can be granted specific permissions, allowing users to perform certain administrative tasks without direct access to underlying system tables.
    - **Consistency**: They provide a standardized way to interact with the SQL Server system.
    - **Simplification**: They abstract complex system operations into simple, callable routines.

**Common Examples of T-SQL Built-in Stored Procedures**:

-   `sp_help`: Provides information about a database object (table, view, stored procedure, etc.).
    -   Example: `EXEC sp_help 'YourTableName';`
-   `sp_who`: Lists information about current SQL Server users and processes.
    -   Example: `EXEC sp_who;`
-   `sp_spaceused`: Displays the number of rows, disk space reserved, and disk space used by a table or the entire database.
    -   Example: `EXEC sp_spaceused 'YourTableName';`
-   `sp_configure`: Displays or changes global configuration settings for the current server.
    -   Example: `EXEC sp_configure;` (to view all options)
    -   Example: `EXEC sp_configure 'show advanced options', 1; RECONFIGURE;` (to enable advanced options)
-   `sp_databases`: Lists the databases on the current instance of SQL Server.
    -   Example: `EXEC sp_databases;`
-   `sp_rename`: Changes the name of a user-created object in the current database.
    -   Example: `EXEC sp_rename 'OldTableName', 'NewTableName';`

These procedures are invaluable tools for anyone working with SQL Server, providing a powerful interface for system management and introspection.