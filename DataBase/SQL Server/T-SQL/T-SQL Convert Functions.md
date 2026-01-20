### 2. Convert Functions

Data type conversion is a fundamental operation in T-SQL. Whether you're combining data from different sources, preparing data for specific functions, or ensuring proper data storage, these functions are indispensable. SQL Server offers several robust conversion functions.

#### a. `CAST (expression AS data_type)`

- **Description:** Converts an `expression` of one data type to another `data_type`. `CAST` is part of the ANSI-SQL standard.
- **Usage:** Preferred for standard, straightforward data type conversions. It's generally more readable and portable than `CONVERT`.
- **Example:**

```sql
SELECT
  CAST(123.45 AS INT) AS CastToInt,
  CAST(GETDATE() AS DATE) AS CastToDateOnly,
  CAST('2023-01-15' AS DATETIME) AS CastToDateTime;
```

  **Output:**

| CastToInt | CastToDateOnly | CastToDateTime          |
| :-------- | :------------- | :---------------------- |
| 123       | 2023-01-15     | 2023-01-15 00:00:00.000 |

#### b. `CONVERT (data_type [(length)], expression [, style])`

- **Description:** Converts an `expression` of one data type to another `data_type`. `CONVERT` is SQL Server-specific and offers an optional `style` parameter for formatting date/time and numeric data during conversion.
- **Usage:** When you need specific formatting for date/time or numeric values during conversion, `CONVERT` with its `style` parameter is the go-to function.
- **Example:**

```sql
SELECT
  CONVERT(VARCHAR(10), GETDATE(), 101) AS DateStyle101, -- mm/dd/yyyy
  CONVERT(VARCHAR(10), GETDATE(), 103) AS DateStyle103, -- dd/mm/yyyy
  CONVERT(VARCHAR(20), GETDATE(), 120) AS DateStyle120, -- yyyy-mm-dd hh:mi:ss (ODBC canonical)
  CONVERT(DECIMAL(10, 2), 12345.6789) AS ConvertToDecimal;
```

  **Output (assuming GETDATE() is '2023-01-20 10:30:45.123'):**

| DateStyle101 | DateStyle103 | DateStyle120        | ConvertToDecimal |
| :----------- | :----------- | :------------------ | :--------------- |
| 01/20/2023   | 20/01/2023   | 2023-01-20 10:30:45 | 12345.68         |

#### c. `PARSE (string_expression AS data_type [USING culture])` (SQL Server 2012+)

- **Description:** Converts a string representation of a number or date/time to a specific numeric or date/time data type. It's locale-aware, meaning it can interpret strings based on a specified culture.
- **Usage:** Excellent for converting strings that might be formatted according to different cultural conventions (e.g., '1.234,56' vs '1,234.56' for numbers, or '15/01/2023' vs '01/15/2023' for dates).
- **Example:**

  ```sql
  SELECT
      PARSE('123.45' AS DECIMAL(10, 2)) AS ParseDecimal,
      PARSE('15/01/2023' AS DATE USING 'en-GB') AS ParseDate_GB,
      PARSE('January 15, 2023' AS DATETIME USING 'en-US') AS ParseDateTime_US;
  ```

  **Output:**

| ParseDecimal | ParseDate_GB | ParseDateTime_US        |
| :----------- | :----------- | :---------------------- |
| 123.45       | 2023-01-15   | 2023-01-15 00:00:00.000 |

#### d. `TRY_CAST (expression AS data_type)` (SQL Server 2012+)

- **Description:** Similar to `CAST`, but returns NULL if the cast fails, instead of raising an error.
- **Usage:** Crucial for handling potentially invalid data without halting query execution. Useful in ETL processes or when dealing with user-provided input that might not always conform to the expected data type.
- **Example:**

```sql
SELECT
  TRY_CAST('123' AS INT) AS ValidCast,
  TRY_CAST('abc' AS INT) AS InvalidCast,
  TRY_CAST('2023-01-15' AS DATE) AS ValidDateCast,
  TRY_CAST('NotADate' AS DATE) AS InvalidDateCast;
```

  **Output:**

| ValidCast | InvalidCast | ValidDateCast | InvalidDateCast |
| :-------- | :---------- | :------------ | :-------------- |
| 123       | NULL        | 2023-01-15    | NULL            |

#### e. `TRY_CONVERT (data_type [(length)], expression [, style])` (SQL Server 2012+)

- **Description:** Similar to `CONVERT`, but returns NULL if the conversion fails, instead of raising an error.
- **Usage:** Provides robust error handling for conversions, especially when using specific `style` parameters that might be sensitive to input format.
- **Example:**

```sql
SELECT
  TRY_CONVERT(DATE, '2023-01-15', 120) AS ValidConvert,
  TRY_CONVERT(DATE, '15-01-2023', 101) AS InvalidConvertStyle, -- 101 expects mm/dd/yyyy
  TRY_CONVERT(INT, 'Hello') AS InvalidConvert;
```

**Output:**

| ValidConvert | InvalidConvertStyle | InvalidConvert |
| :----------- | :------------------ | :------------- |
| 2023-01-15   | NULL                | NULL           |

#### f. `TRY_PARSE (string_expression AS data_type [USING culture])` (SQL Server 2012+)

- **Description:** Similar to `PARSE`, but returns NULL if the parsing fails, instead of raising an error.
- **Usage:** Combines the locale-awareness of `PARSE` with the error-handling capabilities of `TRY_` functions, making it ideal for parsing culturally formatted strings that might occasionally be malformed.
- **Example:**

```sql
SELECT
  TRY_PARSE('1.234,56' AS DECIMAL(10, 2) USING 'de-DE') AS ValidParse_DE,
  TRY_PARSE('1,234.56' AS DECIMAL(10, 2) USING 'de-DE') AS InvalidParse_DE,
  TRY_PARSE('Not a number' AS INT) AS InvalidParse;
```

**Output:**

| ValidParse_DE | InvalidParse_DE | InvalidParse |
| :------------ | :-------------- | :----------- |
| 1234.56       | NULL            | NULL         |
