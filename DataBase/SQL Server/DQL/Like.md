### The `LIKE` Operator and Its Wildcards in SQL Server

The `LIKE` operator in SQL is used in a `WHERE` clause to search for a specified pattern in a column. It's incredibly versatile when you don't know the exact value you're looking for, or when you need to find values that conform to a certain structure.

**Basic Syntax:**
```sql
SELECT column_list
FROM TableName
WHERE ColumnName LIKE pattern;
```

The `pattern` is where the magic happens, utilizing special wildcard characters to define the search criteria.

Let's use our familiar `Employees` table for demonstration:

**`Employees` Table:**

| EmployeeID | FirstName | LastName | DepartmentID | Salary |
|------------|-----------|----------|--------------|--------|
| 101        | Alice     | Smith    | 1            | 70000  |
| 102        | Bob       | Johnson  | 2            | 85000  |
| 103        | Carol     | Davis    | 1            | 72000  |
| 104        | David     | Brown    | 6            | 60000  |
| 105        | Eve       | White    | NULL         | 65000  |
| 106        | Frank     | Green    | 3            | 90000  |
| 107        | Grace     | Hopper   | 5            | 95000  |
| 108        | Charlie   | Chaplin  | 2            | 78000  |
| 109        | Anna      | Anderson | 1            | 71000  |

Now, let's explore the wildcards.

#### 1. The Percent Sign (`%`)

The percent sign wildcard represents _**zero or more characters**_ of any type. It's the most commonly used wildcard for broad pattern matching.

**Purpose:** To match any sequence of characters, or no characters at all.

**Examples:**

*   **Find all employees whose `FirstName` starts with 'A':**
```sql
SELECT FirstName, LastName
FROM Employees
WHERE FirstName LIKE 'A%';
```
**Result:**

| FirstName | LastName |
|-----------|----------|
| Alice     | Smith    |
| Anna      | Anderson |

*   **Find all employees whose `LastName` ends with 'son':**
```sql
SELECT FirstName, LastName
FROM Employees
WHERE LastName LIKE '%son';
```
**Result:**

| FirstName | LastName |
|-----------|----------|
| Bob       | Johnson  |
| Anna      | Anderson |

*   **Find all employees whose `FirstName` contains 'a' anywhere in the name:**
```sql
SELECT FirstName, LastName
FROM Employees
WHERE FirstName LIKE '%a%';
```
**Result:**

| FirstName | LastName |
|-----------|----------|
| Alice     | Smith    |
| Carol     | Davis    |
| David     | Brown    |
| Frank     | Green    |
| Grace     | Hopper   |
| Charlie   | Chaplin  |
| Anna      | Anderson |

#### 2. The Underscore (`_`)

The underscore wildcard represents **any single character**. It's useful when you know the exact length of a part of the pattern but not the specific character.

**Purpose:** To match exactly one character.

**Examples:**

*   **Find all employees whose `FirstName` is 3 characters long:**
```sql
SELECT FirstName, LastName
FROM Employees
WHERE FirstName LIKE '___';
```
**Result:**

| FirstName | LastName |
|-----------|----------|
| Bob       | Johnson  |
| Eve       | White    |

*   **Find all employees whose `LastName` starts with 'S' and has 'i' as the third character:**
```sql
SELECT FirstName, LastName
FROM Employees
WHERE LastName LIKE 'S_i%';
```
**Result:**

| FirstName | LastName |
|-----------|----------|
| Alice     | Smith    |

*   **Find names where the second letter is 'o':**
```sql
SELECT FirstName, LastName
FROM Employees
WHERE FirstName LIKE '_o%';
```
**Result:**

| FirstName | LastName |
|-----------|----------|
| Bob       | Johnson  |

#### 3. The Bracket List (`[]`)

The bracket list wildcard matches **any single character within the specified set or range**.

**Purpose:** To match one character from a defined list or range.

**Examples:**

*   **Find employees whose `FirstName` starts with 'A', 'C', or 'D':**
```sql
SELECT FirstName, LastName
FROM Employees
WHERE FirstName LIKE '[ACD]%';
```
**Result:**

| FirstName | LastName |
|-----------|----------|
| Alice     | Smith    |
| Carol     | Davis    |
| David     | Brown    |
| Charlie   | Chaplin  |
| Anna      | Anderson |

*   **Find employees whose `LastName` starts with any letter from 'A' to 'G':**
```sql
SELECT FirstName, LastName
FROM Employees
WHERE LastName LIKE '[A-G]%';
```
**Result:**

| FirstName | LastName |
|-----------|----------|
| Carol     | Davis    |
| David     | Brown    |
| Frank     | Green    |
| Grace     | Hopper   |
| Anna      | Anderson |

#### 4. The Caret in Bracket List (`[^]`)

The caret in a bracket list matches **any single character *not* within the specified set or range**.

**Purpose:** To exclude characters from a defined list or range.

**Examples:**

*   **Find employees whose `FirstName` does *not* start with 'A', 'B', or 'C':**
```sql
SELECT FirstName, LastName
FROM Employees
WHERE FirstName LIKE '[^ABC]%';
```
**Result:**

| FirstName | LastName |
|-----------|----------|
| David     | Brown    |
| Eve       | White    |
| Frank     | Green    |
| Grace     | Hopper   |

*   **Find employees whose `LastName` does *not* start with a letter from 'A' to 'M':**
```sql
SELECT FirstName, LastName
FROM Employees
WHERE LastName LIKE '[^A-M]%';
```
**Result:**

| FirstName | LastName |
|-----------|----------|
| Alice     | Smith    |
| Bob       | Johnson  |
| Eve       | White    |
| Frank     | Green    |
| Grace     | Hopper   |
| Charlie   | Chaplin  |

#### Escaping Wildcard Characters

What if you need to search for a literal percent sign (`%`), underscore (`_`), or bracket (`[`, `]`, `^`) within your data? You can't use them directly in the `LIKE` pattern because they'll be interpreted as wildcards. To search for these characters literally, you need to **escape** them using an `ESCAPE` clause.

**Syntax:**
```sql
SELECT column_list
FROM TableName
WHERE ColumnName LIKE 'pattern_with_escaped_wildcard' ESCAPE 'escape_character';
```
You can choose any character as your `escape_character`, but it's common to use a backslash (`\`) or a dollar sign (`$`).

**Example Scenario:** Let's imagine we have a `ProductCodes` table with codes like `ABC_123`, `XYZ%456`, `P[Q]R`.

**`ProductCodes` Table:**

| ProductCode |
|-------------|
| ABC_123     |
| XYZ%456     |
| P[Q]R       |
| DATA_RAW    |
| 100%SALE    |

*   **Find product codes that contain a literal underscore (`_`):**
```sql
SELECT ProductCode
FROM ProductCodes
WHERE ProductCode LIKE '%\_%' ESCAPE '\';
```
**Result:**

| ProductCode |
|-------------|
| ABC_123     |
| DATA_RAW    |

*Explanation:* The `\` before `_` tells SQL Server to treat the underscore as a literal character, not a wildcard.

*   **Find product codes that contain a literal percent sign (`%`):**
```sql
SELECT ProductCode
FROM ProductCodes
WHERE ProductCode LIKE '%$%%' ESCAPE '$';
```
**Result:**

| ProductCode |
|-------------|
| XYZ%456     |
| 100%SALE    |

*Explanation:* Here, we chose `$` as our escape character. The first `$` escapes the `%`, making it a literal. The second `%` is a wildcard.

*   **Find product codes that contain a literal opening square bracket (`[`):**
```sql
SELECT ProductCode
FROM ProductCodes
WHERE ProductCode LIKE '%\[%' ESCAPE '\';
```
**Result:**

| ProductCode |
|-------------|
| P[Q]R       |

--- 
#### Another way without using escape character
**you can indeed use square brackets `[]` to treat wildcard characters (`%`, `_`, `[`, `]`, `^`) as literal characters within a `LIKE` pattern.**

### Escaping Wildcards with Square Brackets `[]`

The `[]` wildcard in SQL Server's `LIKE` operator is designed to match *any single character within a specified set or range*. When you place a character that would normally be a wildcard inside these brackets, it loses its special meaning and is interpreted as a literal character that must be matched.

**How it works:**

*   **`%` (Percent Sign):** When inside `[]`, `[%]` matches a literal percent sign.
*   **`_` (Underscore):** When inside `[]`, `[_]` matches a literal underscore.
*   **`[` (Opening Bracket):** When inside `[]`, `[[]` matches a literal opening square bracket.
*   **`]` (Closing Bracket):** When inside `[]`, `[]]` matches a literal closing square bracket. (Note: The closing bracket is special; if it's the *first* character after the opening bracket, it's treated as a literal. Otherwise, it closes the set. `[abc]]` would match 'a', 'b', 'c', or ']').
*   **`^` (Caret):** When inside `[]` and *not* at the beginning, `[^]` matches a literal caret. If it's at the beginning, `[^...]` negates the set.

Let's revisit your examples and expand on them using our `ProductCodes` table:

**`ProductCodes` Table:**

| ProductCode |
|-------------|
| ABC_123     |
| XYZ%456     |
| P[Q]R       |
| DATA_RAW    |
| 100%SALE    |
| [ERROR]     |
| ^START      |

**1. Matching a literal underscore (`_`):**

Your example `'%[_]%'` is perfectly valid.

```sql
SELECT ProductCode
FROM ProductCodes
WHERE ProductCode LIKE '%[_]%';
```

**Result:**

| ProductCode |
|-------------|
| ABC_123     |
| DATA_RAW    |

**Explanation:** `[_]` tells SQL Server to look for a literal underscore character, not "any single character."

**2. Matching a literal percent sign (`%`):**

Your example `'%[%]'` is also correct for matching strings that *contain* a percent sign. If you want to match strings *ending* with a percent sign, you'd use `'%[%]'`.

```sql
SELECT ProductCode
FROM ProductCodes
WHERE ProductCode LIKE '%[%]%'; -- Matches codes containing a literal %
```

**Result:**

| ProductCode |
|-------------|
| XYZ%456     |
| 100%SALE    |

**Explanation:** `[%]` tells SQL Server to look for a literal percent sign character, not "zero or more characters."

**Additional Examples:**

*   **Matching a literal opening square bracket (`[`):**
```sql
SELECT ProductCode
FROM ProductCodes
WHERE ProductCode LIKE '%[[]%';
```
**Result:**

| ProductCode |
|-------------|
| P[Q]R       |
| [ERROR]     |

**Explanation:** `[[]` matches a literal `[`.

*   **Matching a literal closing square bracket (`]`):**
```sql
SELECT ProductCode
FROM ProductCodes
WHERE ProductCode LIKE '%[]]%';
```
**Result:**

| ProductCode |
|-------------|
| P[Q]R       |
| [ERROR]     |

**Explanation:** `[]]` matches a literal `]`. Note that if `]` is the first character after `[`, it's treated as a literal.

*   **Matching a literal caret (`^`):**
```sql
SELECT ProductCode
FROM ProductCodes
WHERE ProductCode LIKE '%[^]%'; -- Matches codes containing a literal ^
```
**Result:**

| ProductCode |
|-------------|
| ^START      |

**Explanation:** `[^]` matches a literal `^` (when not at the beginning of the character set).

### Comparison: `[]` vs. `ESCAPE` Clause

Both methods achieve the goal of matching literal wildcard characters, but they have different use cases and characteristics:

**Using `[]` (Bracket List):**
*   **Pros:**
    *   Often more concise and readable for single wildcard characters.
    *   No need to define an `ESCAPE` character, which can sometimes conflict with data.
    *   Part of the standard `LIKE` pattern syntax.
*   **Cons:**
    *   Only works for single wildcard characters. You can't use `[%_]` to match a literal percent *followed by* a literal underscore; it would mean "match a single character that is either `%` or `_`".
    *   Can become less readable if you're trying to match multiple different literal wildcards in a complex pattern.
    *   Requires careful handling for `[` and `]` themselves (e.g., `[[]` and `[]]`).

**Using `ESCAPE` Clause:**
*   **Pros:**
    *   More explicit about the escaping mechanism.
    *   Can escape any character, including sequences of characters if the escape character is placed before each.
    *   Necessary if you need to escape a character that is *not* one of the `LIKE` wildcards but might be interpreted specially in other contexts (though `LIKE` only cares about `%, _, [], ^`).
*   **Cons:**
    *   Requires an extra `ESCAPE 'character'` clause.
    *   The chosen escape character must not appear literally in your data where it's not intended as an escape.

### When to Use Which?

*   **For simple literal wildcard matches (like `_`, `%`, `^`):** Using `[_]`, `[%]`, `[^]` is often preferred for its conciseness and clarity.
*   **For matching literal `[` or `]`:** Using `[[]` and `[]]` is the standard way.
*   **For complex patterns or when you prefer explicit control:** The `ESCAPE` clause provides a clear, separate mechanism for escaping. It's particularly useful if you have a dynamic pattern where you might not know which characters need escaping beforehand, and you can programmatically prepend the escape character.

As a senior developer, I often use the `[]` method for single character wildcards because it integrates seamlessly into the pattern itself. However, I'm always mindful of the `ESCAPE` clause for situations where it offers better clarity or is the only viable option (e.g., if I needed to match a literal `[` and `]` in a way that `[]` syntax makes ambiguous). Both are valuable tools in your SQL pattern matching toolkit!
### Important Considerations and Best Practices

1.  **Case Sensitivity:** The behavior of `LIKE` regarding case sensitivity depends on the collation of the database or the specific column. If your database uses a case-insensitive collation (e.g., `SQL_Latin1_General_CP1_CI_AS`), then `LIKE 'a%'` will match 'Alice', 'alice', 'ALICE', etc. If it's case-sensitive, it will only match 'Alice'. You can explicitly control this using `COLLATE` in your query if needed.
```sql
-- Force case-sensitive search
SELECT FirstName FROM Employees WHERE FirstName COLLATE Latin1_General_CS_AS LIKE 'alice%';
```

2.  **Performance:**
    *   Using `LIKE` with a leading wildcard (e.g., `LIKE '%pattern'`) can be very inefficient because it prevents SQL Server from using indexes on the column. The database often has to perform a full table scan.
    *   `LIKE 'pattern%'` (without a leading wildcard) can utilize an index if one exists on the column, as it can efficiently scan the beginning of the string.
    *   For very large text fields and complex pattern matching, consider using SQL Server's `FULLTEXT` search capabilities, which are specifically designed and optimized for this purpose and can be significantly faster than `LIKE`.

3.  **Alternatives:**
    *   For exact matches, use the `=` operator.
    *   For multiple specific patterns, `IN` can be more efficient than multiple `OR LIKE` conditions.
    *   For more complex regular expression matching (beyond what `LIKE` offers), you might need to explore CLR functions in SQL Server or process the data in an application layer.

Understanding `LIKE` and its wildcards is a fundamental skill for any database professional. It empowers you to write flexible queries that can adapt to varying data patterns, making your data retrieval more robust and intelligent. Always be mindful of performance implications, especially with large datasets, and choose the most appropriate tool for the job.