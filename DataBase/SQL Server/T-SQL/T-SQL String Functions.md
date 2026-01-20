### 3. String Functions

String manipulation is a cornerstone of almost any database application. From formatting names to parsing addresses, SQL Server's string functions provide a rich toolkit for working with textual data.

#### a. `LEN (string_expression)`

- **Description:** Returns the number of characters of the specified string expression, excluding trailing blanks.
- **Usage:** Useful for validating input length, calculating display widths, or determining if a string is empty (though `DATALENGTH` might be better for empty strings with spaces).
- **Example:**

```sql
SELECT
  LEN('Hello World') AS Length1,
  LEN('  SQL Server  ') AS Length2, -- Trailing spaces are ignored
  LEN('') AS Length3,
  LEN(' ') AS Length4;
```

**Output:**

| Length1 | Length2 | Length3 | Length4 |
| :------ | :------ | :------ | :------ |
| 11      | 12      | 0       | 0       |

#### b. `DATALENGTH (expression)`

- **Description:** Returns the number of bytes used to represent an expression. For `VARCHAR` and `NVARCHAR`, this can differ from `LEN`.
- **Usage:** Essential for understanding storage requirements and for distinguishing between an empty string (`''`) and a string containing only spaces (`' '`), as `LEN` would return 0 for both, but `DATALENGTH` would show the byte count for spaces.
- **Example:**

```sql
SELECT
  DATALENGTH('Hello World') AS DataLength1, -- 11 bytes for VARCHAR
  DATALENGTH(N'Hello World') AS DataLength2, -- 22 bytes for NVARCHAR (2 bytes per char)
  DATALENGTH('') AS DataLength3, -- 0 bytes
  DATALENGTH(' ') AS DataLength4; -- 1 byte for VARCHAR, 2 bytes for NVARCHAR
```

**Output:**

| DataLength1 | DataLength2 | DataLength3 | DataLength4 |
| :---------- | :---------- | :---------- | :---------- |
| 11          | 22          | 0           | 1           |

#### c. `LEFT (string_expression, length)`

- **Description:** Returns the left part of a character string with the specified number of characters.
- **Usage:** Extracting prefixes, initial characters of codes, or short descriptions.
- **Example:**

```sql
SELECT LEFT('SQL Server', 3) AS LeftPart;
```

**Output:**

| LeftPart |
| :------- |
| SQL      |

#### d. `RIGHT (string_expression, length)`

- **Description:** Returns the right part of a character string with the specified number of characters.
- **Usage:** Extracting suffixes, file extensions, or the last few digits of an identifier.
- **Example:**

```sql
SELECT RIGHT('SQL Server', 6) AS RightPart;
```

**Output:**

| RightPart |
| :-------- |
| Server    |

#### e. `SUBSTRING (string_expression, start, length)`

- **Description:** Returns part of a character, binary, text, or image expression. `start` is the starting position (1-based), and `length` is the number of characters to return.
- **Usage:** The most versatile function for extracting any segment of a string.
- **Example:**

```sql
SELECT SUBSTRING('SQL Server Database', 5, 6) AS SubstringPart;
```

**Output:**

| SubstringPart |
| :------------ |
| Server        |

#### f. `CHARINDEX (substring, expression [, start_location])`

- **Description:** Returns the starting position of the specified `substring` within `expression`. Returns 0 if `substring` is not found. `start_location` is optional.
- **Usage:** Finding the position of a delimiter, a specific word, or a pattern within a string. Often used in conjunction with `SUBSTRING`.
- **Example:**

```sql
SELECT
  CHARINDEX('Server', 'SQL Server Database') AS Position1,
  CHARINDEX('DB', 'SQL Server Database') AS Position2;
```

**Output:**

| Position1 | Position2 |
| :-------- | :-------- |
| 5         | 0         |

#### g. `PATINDEX ('%pattern%', expression)`

- **Description:** Returns the starting position of the first occurrence of a `pattern` in a specified `expression`. The `pattern` can include wildcard characters (`%`, `_`, `[]`, `[^]`). Returns 0 if the pattern is not found.
- **Usage:** More powerful than `CHARINDEX` for pattern matching, allowing for flexible searches using SQL Server's LIKE operator wildcards.
- **Example:**

```sql
SELECT
  PATINDEX('%[0-9]%', 'Product A123') AS Position1, -- Find first digit
  PATINDEX('%[a-z]%', '123ABC') AS Position2, -- Find first letter
  PATINDEX('%[^0-9]%', '12345') AS Position3; -- Find first non-digit
```

**Output:**

| Position1 | Position2 | Position3 |
| :-------- | :-------- | :-------- |
| 10        | 4         | 0         |

#### h. `REPLACE (string_expression, string_pattern, string_replacement)`

- **Description:** Replaces all occurrences of a specified `string_pattern` within `string_expression` with `string_replacement`.
- **Usage:** Cleaning data, standardizing formats (e.g., replacing hyphens with spaces), or redacting sensitive information.
- **Example:**

```sql
SELECT REPLACE('Hello World', 'World', 'SQL') AS ReplacedString;
```

**Output:**

| ReplacedString |
| :------------- |
| Hello SQL      |

#### i. `STUFF (character_expression, start, length, replacement_character_expression)`

- **Description:** Deletes a specified `length` of characters from `character_expression` at a `start` position and then inserts `replacement_character_expression` into `character_expression` at the `start` position.
- **Usage:** More complex string modification than `REPLACE`. Useful for inserting characters into a specific position, or replacing a segment of a string with another.
- **Example:**

```sql
SELECT STUFF('SQL Server', 1, 3, 'Microsoft') AS StuffedString; -- Replaces 'SQL' with 'Microsoft'
```

**Output:**

| StuffedString    |
| :--------------- |
| Microsoft Server |

#### j. `UPPER (string_expression)`

- **Description:** Converts all characters in the specified `string_expression` to uppercase.
- **Usage:** Standardizing case for comparisons, display, or data entry.
- **Example:**

```sql
SELECT UPPER('hello world') AS UpperCaseString;
```

**Output:**

| UpperCaseString |
| :-------------- |
| HELLO WORLD     |

#### k. `LOWER (string_expression)`

- **Description:** Converts all characters in the specified `string_expression` to lowercase.
- **Usage:** Standardizing case for comparisons, display, or data entry.
- **Example:**

```sql
SELECT LOWER('HELLO WORLD') AS LowerCaseString;
```

**Output:**

| LowerCaseString |
| :-------------- |
| hello world     |

#### l. `LTRIM (string_expression)`

- **Description:** Removes leading blanks (spaces) from `string_expression`.
- **Usage:** Cleaning data, especially from external sources, to ensure consistent formatting and accurate comparisons. Removes the extra spaces at the beginning of a string, doesn't affect the trailing(end) spaces.
- **Example:**

```sql
SELECT LTRIM('   Hello World   ') AS TrimmedString;
```

**Output:**

| TrimmedString |
| :------------ |
| Hello World   |

#### m. `RTRIM (string_expression)`

- **Description:** Removes trailing blanks (spaces) from `string_expression`.
- **Usage:** Cleaning data, preventing issues with string comparisons where trailing spaces might cause mismatches. Removes the extra spaces at the end of a string, doesn't affect the Leading (beginning) spaces.
- **Example:**

```sql
SELECT RTRIM('   Hello World   ') AS TrimmedString;
```

**Output:**

| TrimmedString |
| :------------ |
| Hello World   |

#### n. `TRIM ([characters FROM] string_expression)` (SQL Server 2017+)

- **Description:** Removes the specified `characters` (or spaces by default) from both the beginning and end of `string_expression`.
- **Usage:** A more versatile trimming function, allowing removal of specific characters, not just spaces, from both ends.
- **Example:**

```sql
SELECT
  TRIM('   Hello World   ') AS TrimmedSpaces,
  TRIM('.,! ' FROM '.,!Hello World!., ') AS TrimmedChars;
```

**Output:**

| TrimmedSpaces | TrimmedChars |
| :------------ | :----------- |
| Hello World   | Hello World  |

| Function | Removes Leading Spaces | Removes Trailing Spaces |
| -------- | ---------------------- | ----------------------- |
| LTRIM    | Yes                    | No                      |
| RTRIM    | No                     | Yes                     |
| TRIM     | Yes                    | Yes                     |

#### o. `REPLICATE (string_expression, integer_expression)`

- **Description:** Repeats `string_expression` a specified number of times.
- **Usage:** Padding strings, generating repeating patterns, or creating visual separators.
- **Example:**

```sql
SELECT REPLICATE('-', 10) AS Separator;
```

**Output:**

| Separator  |
| :--------- |
| ---------- |

#### p. `SPACE (integer_expression)`

- **Description:** Returns a string of spaces of the specified length.
- **Usage:** Creating formatted output, adding indentation, or separating columns in a concatenated string.
- **Example:**

```sql
SELECT 'First' + SPACE(5) + 'Last' AS SpacedString;
```

**Output:**

| SpacedString   |
| :------------- |
| First     Last |

#### q. `CONCAT (string_value1, string_value2 [, string_valueN])` (SQL Server 2012+)

- **Description:** Concatenates two or more string values end-to-end. It implicitly converts non-string values to strings and treats NULLs as empty strings.
- **Usage:** A modern and convenient way to combine multiple string columns or literals, especially when dealing with potential NULLs, as it handles them gracefully.
- **Example:**

```sql
SELECT CONCAT('Hello', ' ', 'World', '!') AS ConcatenatedString;
```

**Output:**

| ConcatenatedString |
| :----------------- |
| Hello World!       |

#### r. `FORMAT (value, format [, culture])` (SQL Server 2012+)

- **Description:** Formats a value with the specified format string and optional culture. Primarily used for formatting numbers and dates into strings.
- **Usage:** Provides powerful and flexible formatting capabilities, especially for presenting data in a user-friendly, localized manner.
- **Example:**

  ```sql
  SELECT
      FORMAT(12345.6789, 'C', 'en-US') AS CurrencyUS, -- Currency format for US
      FORMAT(GETDATE(), 'dddd, MMMM dd, yyyy') AS LongDate,
      FORMAT(0.25, 'P') AS Percentage,
      FORMAT(129.3298, '.00') as DecimalNumber; 
  ```

**Output (assuming GETDATE() is '2023-01-20'):**

| CurrencyUS | LongDate                 | Percentage | DecimalNumber |
| :--------- | :----------------------- | :--------- | ------------- |
| $12,345.68 | Friday, January 20, 2023 | 25.00 %    | 129.33        |

#### s. `STRING_SPLIT (string, separator)` (SQL Server 2016+)

- **Description:** A table-valued function that splits a string by a specified separator character and returns a table of substrings.
- **Usage:** Invaluable for parsing delimited lists stored in a single string column into individual rows, making them queryable.
- **Example:**

```sql
SELECT value
FROM STRING_SPLIT('apple,banana,orange', ',');
```

**Output:**

| value  |
| :----- |
| apple  |
| banana |
| orange |

#### t. `STRING_AGG (expression, separator)` (SQL Server 2017+)

- **Description:** An aggregate function that concatenates the values of string expressions and places separator values between them.
- **Usage:** The inverse of `STRING_SPLIT`. Useful for consolidating multiple related rows into a single delimited string, for example, listing all products for an order.
- **Example:**

```sql
-- Assume a table 'OrderItems'
CREATE TABLE OrderItems (
  OrderID INT,
  ItemName NVARCHAR(100)
);

INSERT INTO OrderItems (OrderID, ItemName) VALUES
(1, 'Laptop'),
(1, 'Mouse'),
(2, 'Keyboard'),
(2, 'Monitor');

SELECT
  OrderID,
  STRING_AGG(ItemName, ', ') AS ItemsList
FROM OrderItems
GROUP BY OrderID;
```

**Output:**

| OrderID | ItemsList         |
| :------ | :---------------- |
| 1       | Laptop, Mouse     |
| 2       | Keyboard, Monitor |

