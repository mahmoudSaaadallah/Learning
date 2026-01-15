### 1. Database Integrity: The Foundation

What we mean by "Database Integrity." It refers to the accuracy, consistency, and reliability of data stored in a database. It ensures that data remains valid and meaningful over its entire lifecycle. There are several types of integrity, but the ones we'll focus on today are primarily related to **Entity Integrity** (via Primary Keys) and **Referential Integrity** (via Foreign Keys).

### 2. Primary Key (PK)

The Primary Key is arguably the most fundamental constraint for ensuring data integrity.

*   **Definition:** A Primary Key is a column or a set of columns in a table that uniquely identifies each row (or record) in that table. No two rows can have the same Primary Key value.
*   **Purpose:**
    *   **Entity Integrity:** It guarantees that each entity (row) in the table is unique and can be distinctly identified.
    *   **Data Retrieval:** It provides a fast and efficient way to locate specific records. SQL Server automatically creates a clustered index on the Primary Key by default, which physically sorts the data in the table based on the PK values, significantly speeding up lookups.
    *   **Basis for Relationships:** Primary Keys are the "parent" side of relationships with other tables, allowing us to link related data across different tables using Foreign Keys.
*   **Characteristics:**
    *   **Uniqueness:** Every value in the Primary Key column(s) must be unique.
    *   **Non-NULL:** A Primary Key column cannot contain `NULL` values. This is often referred to as the "Entity Integrity Rule."
    *   **Stability:** Ideally, Primary Key values should not change over time. Using natural keys (like Social Security Numbers) can be problematic if those values can change or be reassigned. Surrogate keys (like auto-incrementing integers) are often preferred for their stability.

**Example:**

Consider a `Students` table:

```sql
CREATE TABLE Students (
    StudentID INT PRIMARY KEY IDENTITY(1,1), -- StudentID is the Primary Key, auto-incrementing
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    DateOfBirth DATE,
    Email VARCHAR(100) UNIQUE
);
```
In this example, `StudentID` is the Primary Key. Each student will have a unique `StudentID`, and it cannot be `NULL`.
What make the `PK` auto incremented is `Identity(1, 1)` which means the `PK` is starting by `1` and each row will be incremented by `1`.

### 3. Composite Primary Key

Sometimes, a single column isn't enough to uniquely identify a row. In such cases, we use a Composite Primary Key.

*   **Definition:** A Composite Primary Key is a Primary Key that consists of two or more columns whose combined values uniquely identify each row in a table.
*   **Purpose:** It's used when the uniqueness of a record can only be guaranteed by the combination of multiple attributes. This is common in "junction" or "associative" tables that resolve many-to-many relationships.
*   **Characteristics:** The combination of all columns in the composite key must be unique, and none of the individual columns participating in the composite key can be `NULL`.

**Example:**

Consider a `CourseEnrollments` table, where a student can enroll in multiple courses, and a course can have multiple students. To uniquely identify an enrollment, we need both the `StudentID` and the `CourseID`.

```sql
CREATE TABLE CourseEnrollments (
    StudentID INT NOT NULL,
    CourseID INT NOT NULL,
    EnrollmentDate DATE DEFAULT GETDATE(),
    Grade CHAR(1),
    PRIMARY KEY (StudentID, CourseID) -- Composite Primary Key
);
```
Here, `(StudentID, CourseID)` forms the Composite Primary Key. A student can enroll in 'Course A', and another student can enroll in 'Course A', but the *same* student cannot enroll in 'Course A' twice (at least not without a different `EnrollmentDate` if that were also part of the PK, but for simplicity, we're assuming one enrollment per student per course).

### 4. Foreign Key (FK)

Foreign Keys are the mechanism for enforcing **Referential Integrity**, which ensures that relationships between tables remain consistent.

*   **Definition:** A Foreign Key is a column or a set of columns in one table (the "child" table) that refers to the Primary Key (or a Unique Key) in another table (the "parent" table). It establishes a link between the two tables.
*   **Purpose:**
    *   **Referential Integrity:** It ensures that a value in the Foreign Key column of the child table must either match an existing value in the Primary Key of the parent table or be `NULL` (if the column allows `NULL`s). This prevents "orphan" records, where a child record refers to a non-existent parent.
    *   **Relationship Enforcement:** It formally defines and enforces the relationships between entities in your database schema.
*   **Characteristics:**
    *   A Foreign Key column can contain `NULL` values, unless it is also part of the child table's Primary Key or has a `NOT NULL` constraint.
    *   The data type of the Foreign Key column(s) must match the data type of the referenced Primary Key column(s).

**Example:**

Building on our `Students` and `CourseEnrollments` tables:

```sql
CREATE TABLE Courses (
    CourseID INT PRIMARY KEY IDENTITY(1,1),
    CourseName VARCHAR(100) NOT NULL,
    Credits INT
);

CREATE TABLE CourseEnrollments (
    StudentID INT NOT NULL,
    CourseID INT NOT NULL,
    EnrollmentDate DATE DEFAULT GETDATE(),
    Grade CHAR(1),
    PRIMARY KEY (StudentID, CourseID),
    FOREIGN KEY (StudentID) REFERENCES Students(StudentID), -- FK referencing Students table
    FOREIGN KEY (CourseID) REFERENCES Courses(CourseID)     -- FK referencing Courses table
);
```
Now, `StudentID` in `CourseEnrollments` must exist in the `Students` table's `StudentID` column, and `CourseID` in `CourseEnrollments` must exist in the `Courses` table's `CourseID` column. This prevents enrolling a non-existent student or in a non-existent course.

### 5. Foreign Key `ON DELETE` and `ON UPDATE` Actions

These clauses define how SQL Server should behave when a row in the parent table (the one with the Primary Key) is deleted or its Primary Key value is updated, and there are related rows in the child table (the one with the Foreign Key). These are crucial for maintaining referential integrity automatically.

The common actions are:

*   **`NO ACTION` (Default):** This is the default behavior. If you try to delete or update a parent row that has matching child rows, the operation will fail with an error. It prevents any changes to the parent key if dependent rows exist.
    *   *Use Case:* When you absolutely cannot allow a parent record to be removed or changed if it has children.
*   **`CASCADE`:** If a parent row is deleted or its Primary Key is updated, all corresponding child rows are also deleted or updated.
    *   *Use Case:* When child records are entirely dependent on the parent and should not exist without it like week entity (e.g., order items for an order, details for a master record).
*   **`SET NULL`:** If a parent row is deleted or its Primary Key is updated, the Foreign Key values in all corresponding child rows are set to `NULL`. This requires the Foreign Key column(s) in the child table to be nullable.
    *   *Use Case:* When child records can exist independently of the parent, but their association with the parent is lost (e.g., an employee's department might be set to `NULL` if the department is dissolved).
*   **`SET DEFAULT`:** If a parent row is deleted or its Primary Key is updated, the Foreign Key values in all corresponding child rows are set to their default value. This requires a default constraint to be defined on the Foreign Key column(s).
    *   *Use Case:* Less common, but useful if you want to reassign orphaned children to a predefined "default" parent.

**Examples of `ON DELETE` and `ON UPDATE`:**

Let's modify our `CourseEnrollments` table:

```sql
CREATE TABLE CourseEnrollments (
    StudentID INT NOT NULL,
    CourseID INT NOT NULL,
    EnrollmentDate DATE DEFAULT GETDATE(),
    Grade CHAR(1),
    PRIMARY KEY (StudentID, CourseID),
    FOREIGN KEY (StudentID) REFERENCES Students(StudentID)
        ON DELETE CASCADE    -- If a student is deleted, all their enrollments are deleted
        ON UPDATE CASCADE,   -- If a StudentID changes (rare, but possible), enrollments update
    FOREIGN KEY (CourseID) REFERENCES Courses(CourseID)
        ON DELETE SET NULL   -- If a course is deleted, enrollments remain but CourseID becomes NULL
        ON UPDATE NO ACTION  -- If a CourseID changes, prevent it if enrollments exist
);
```
*   If `StudentID` 101 is deleted from `Students`, all rows in `CourseEnrollments` where `StudentID` is 101 will also be deleted.
*   If `CourseID` 500 is deleted from `Courses`, all rows in `CourseEnrollments` where `CourseID` is 500 will have their `CourseID` set to `NULL`.

### 6. Derived Attributes and `PERSISTED` Computed Columns

This topic bridges data modeling with performance optimization.

*   **Derived Attribute:**
    *   **Definition:** A derived attribute is an attribute whose value is not stored directly in the database but can be computed or derived from other attributes already stored in the database.
    *   **Pros:**
        *   **Reduces Redundancy:** Avoids storing the same information twice.
        *   **Ensures Consistency:** The derived value is always up-to-date with its source data.
    *   **Cons:**
        *   **Performance Overhead:** Calculating the value every time it's queried can be slow, especially for complex calculations or large datasets.
        *   **Complexity:** The calculation logic needs to be maintained.
    *   **Example:** `Age` derived from `DateOfBirth`, `FullName` derived from `FirstName` and `LastName`, `TotalOrderAmount` derived from `Quantity * Price` for all order items.

*   **`PERSISTED` Computed Columns:**
    *   **Definition:** In SQL Server, you can define a column whose value is computed from an expression involving other columns in the same table. By default, these are *virtual* columns (not stored). However, you can mark them as `PERSISTED`. A `PERSISTED` computed column means that its values are physically stored in the table and updated automatically when any of its source columns change.
    *   **Purpose:** To improve query performance on frequently accessed derived attributes. Since the value is pre-calculated and stored, SQL Server doesn't need to compute it every time it's read.
    *   **Characteristics:**
        *   **Storage:** Values are stored on disk, consuming space.
        *   **Indexing:** `PERSISTED` computed columns can be indexed, which can dramatically speed up queries that filter or sort by that derived value.
        *   **Determinism:** The expression for a `PERSISTED` computed column must be *deterministic*. This means it must always return the same result for the same set of input values, regardless of when or how it's evaluated (e.g., `GETDATE()` is non-deterministic, `FirstName + ' ' + LastName` is deterministic).
        *   **Update Overhead:** There's a slight overhead on `INSERT` and `UPDATE` operations on the source columns, as the computed column's value must also be calculated and stored.

**Example:**

Let's add a `FullName` derived attribute to our `Students` table:

```sql
ALTER TABLE Students
ADD FullName AS (FirstName + ' ' + LastName) PERSISTED;
```
Now, the `FullName` column will be physically stored in the `Students` table. If `FirstName` or `LastName` for a student changes, `FullName` will automatically update. Crucially, because it's `PERSISTED`, we can now create an index on `FullName` to speed up searches:

```sql
CREATE INDEX IX_Students_FullName ON Students (FullName);
```
This is a classic trade-off: you consume more storage space and incur a small write overhead, but you gain significant read performance for queries involving that derived attribute.

---

Mastering these concepts is fundamental to designing and maintaining high-quality, performant, and reliable database systems. They are the building blocks for ensuring that your data is not just stored, but stored *correctly* and *efficiently*.