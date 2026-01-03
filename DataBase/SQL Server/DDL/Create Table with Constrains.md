### The Essence of `CREATE TABLE`

At its core, the `CREATE TABLE` statement is a Data Definition Language (DDL) command used to define a new table in a database. It specifies the table's name, its columns, and the data type for each column. However, a table without constraints is like a house without a foundation – it might stand for a bit, but it's prone to collapse under pressure.

### The Power of Constraints

Constraints are rules enforced on data columns in a table. They are used to limit the type of data that can go into a table, ensuring the accuracy and reliability of the data. This data integrity is paramount for any system, from a small departmental application to a large-scale enterprise solution.

Let's explore the primary types of constraints with practical examples:

---

#### 1. `NOT NULL` Constraint

**Purpose**: Ensures that a column cannot have a `NULL` value. This means that every row inserted into the table must have a value for that column.

**Explanation**: Imagine a scenario where you're tracking student enrollments. A student's name is absolutely essential. If it's `NULL`, the record is practically useless. `NOT NULL` prevents such omissions.

**Example**:

```sql
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    DateOfBirth DATE,
    Email VARCHAR(100) UNIQUE
);
```

In this example, `FirstName` and `LastName` *must* have a value.

---

#### 2. `UNIQUE` Constraint

**Purpose**: Ensures that all values in a column (or a group of columns) are distinct. While `PRIMARY KEY` implicitly has `UNIQUE`, you can have multiple `UNIQUE` constraints per table.

**Explanation**: Consider an email address for a user. Each user should ideally have a unique email. The `UNIQUE` constraint ensures that no two rows will have the same email address.

**Example**:

```sql
CREATE TABLE Users (
    UserID INT PRIMARY KEY,
    Username VARCHAR(50) UNIQUE NOT NULL,
    Email VARCHAR(100) UNIQUE,
    PasswordHash VARCHAR(255) NOT NULL
);
```

Here, both `Username` and `Email` columns must contain unique values across all rows.

---

#### 3. `PRIMARY KEY` Constraint

**Purpose**: Uniquely identifies each record in a table. A table can have only one `PRIMARY KEY`, which can consist of one or more columns. It implicitly enforces `NOT NULL` and `UNIQUE`.

**Explanation**: This is the cornerstone of relational database design. It provides a stable, unchanging identifier for each row, crucial for linking tables together. Think of it as the social security number for each record.

**Example**:

```sql
CREATE TABLE Products (
    ProductID INT PRIMARY KEY, -- Single column primary key
    ProductName VARCHAR(100) NOT NULL,
    UnitPrice DECIMAL(10, 2) NOT NULL
);

-- Composite Primary Key Example:
CREATE TABLE OrderDetails (
    OrderID INT NOT NULL,
    ProductID INT NOT NULL,
    Quantity INT NOT NULL,
    UnitPrice DECIMAL(10, 2) NOT NULL,
    PRIMARY KEY (OrderID, ProductID) -- Composite primary key
);
```

In `Products`, `ProductID` uniquely identifies each product. In `OrderDetails`, the combination of `OrderID` and `ProductID` uniquely identifies each line item within an order.

---

#### 4. `FOREIGN KEY` Constraint

**Purpose**: Establishes a link between two tables. It ensures referential integrity, meaning that a value in the foreign key column(s) must exist in the primary key (or unique key) of the referenced table.

**Explanation**: This is how relationships are built in a relational database. If you have an `Orders` table and a `Customers` table, a `CustomerID` in `Orders` should always refer to an existing `CustomerID` in `Customers`. This prevents "orphan" records.

**Example**:

```sql
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    CustomerName VARCHAR(100) NOT NULL,
    City VARCHAR(50)
);

CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    OrderDate DATE NOT NULL,
    CustomerID INT NOT NULL,
    TotalAmount DECIMAL(10, 2),
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);
```

Here, `CustomerID` in the `Orders` table is a foreign key referencing `CustomerID` in the `Customers` table. This ensures that an order can only be placed by an existing customer.

---

#### 5. `CHECK` Constraint

**Purpose**: Enforces a domain integrity rule by limiting the range of values that can be placed in a column.

**Explanation**: Sometimes, data needs to fall within a specific range or meet certain criteria. For instance, a product's price cannot be negative, or an employee's age must be above a certain threshold.

**Example**:

```sql
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    Age INT CHECK (Age >= 18), -- Age must be 18 or older
    Salary DECIMAL(10, 2) CHECK (Salary > 0) -- Salary must be positive
);

-- Another example with a list of allowed values:
CREATE TABLE Orders_Status (
    OrderStatusID INT PRIMARY KEY,
    StatusName VARCHAR(20) CHECK (StatusName IN ('Pending', 'Processing', 'Shipped', 'Delivered', 'Cancelled'))
);
```

The `CHECK` constraint on `Age` ensures no employee younger than 18 can be added. The `CHECK` on `Salary` ensures it's always positive.

---

#### 6. `DEFAULT` Constraint

**Purpose**: Provides a default value for a column when no value is explicitly specified during an `INSERT` operation.

**Explanation**: This is useful for columns that often have a common initial value. For example, a `CreationDate` column might default to the current date and time, or an `IsActive` flag might default to `TRUE`.

**Example**:

```sql
CREATE TABLE Tasks (
    TaskID INT PRIMARY KEY,
    TaskName VARCHAR(255) NOT NULL,
    Description TEXT,
    IsCompleted BIT DEFAULT 0, -- Defaults to 0 (false)
    CreatedDate DATETIME DEFAULT GETDATE() -- Defaults to current date/time
);
```

If you insert a new task without specifying `IsCompleted` or `CreatedDate`, they will automatically take their default values.

---

### Naming Constraints (A Best Practice)

While SQL Server automatically assigns names to constraints if you don't provide them, it's a strong best practice to explicitly name your constraints. This makes it much easier to manage, alter, or drop constraints later, and significantly improves the readability of your schema.

**Syntax for Naming Constraints**:

```sql
CREATE TABLE Products_Named (
    ProductID INT CONSTRAINT PK_Products_ProductID PRIMARY KEY,
    ProductName VARCHAR(100) NOT NULL,
    SKU VARCHAR(20) CONSTRAINT UQ_Products_SKU UNIQUE,
    UnitPrice DECIMAL(10, 2) NOT NULL CONSTRAINT CK_Products_UnitPrice_Positive CHECK (UnitPrice > 0),
    CategoryID INT,
    CONSTRAINT FK_Products_CategoryID FOREIGN KEY (CategoryID) REFERENCES Categories(CategoryID)
);
```

In this example, `PK_Products_ProductID`, `UQ_Products_SKU`, `CK_Products_UnitPrice_Positive`, and `FK_Products_CategoryID` are user-defined names for the constraints.
