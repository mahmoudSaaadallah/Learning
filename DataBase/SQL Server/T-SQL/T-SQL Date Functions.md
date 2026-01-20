
### 4. Date Functions

Working with dates and times is a common requirement in database systems. SQL Server provides a comprehensive set of functions to retrieve, manipulate, and format date/time values, handling everything from simple date extraction to complex interval calculations.

#### a. `GETDATE()`

- **Description:** Returns the current database system date and time as a `DATETIME` value. The precision is limited to milliseconds.
- **Usage:** Obtaining the current timestamp for logging, auditing, or setting default values.
- **Example:**

```sql
SELECT GETDATE() AS CurrentDateTime;
```

**Output (example):**

| CurrentDateTime         |
| :---------------------- |
| 2023-01-20 10:30:45.123 |

#### b. `SYSDATETIME()` (SQL Server 2008+)

- **Description:** Returns the current system date and time as a `DATETIME2` value with higher precision (up to 100 nanoseconds) than `GETDATE()`.
- **Usage:** When high precision for date and time is critical, such as in scientific applications or high-frequency trading systems.
- **Example:**

```sql
SELECT SYSDATETIME() AS CurrentSystemDateTime;
```

**Output (example):**

| CurrentSystemDateTime       |
| :-------------------------- |
| 2023-01-20 10:30:45.1234567 |

#### c. `GETUTCDATE()`

- **Description:** Returns the current UTC (Coordinated Universal Time) date and time as a `DATETIME` value.
- **Usage:** Essential for applications that operate across different time zones, ensuring a consistent, time-zone-independent timestamp.
- **Example:**

```sql
SELECT GETUTCDATE() AS CurrentUtcDateTime;
```

**Output (example):**

| CurrentUtcDateTime      |
| :---------------------- |
| 2023-01-20 15:30:45.123 |

#### d. `CURRENT_TIMESTAMP`

- **Description:** Returns the current database system date and time as a `DATETIME` value. It is functionally equivalent to `GETDATE()`.
- **Usage:** An ANSI SQL standard equivalent to `GETDATE()`, often preferred for portability.
- **Example:**

```sql
SELECT CURRENT_TIMESTAMP AS CurrentTimestampValue;
```

**Output (example):**

| CurrentTimestampValue   |
| :---------------------- |
| 2023-01-20 10:30:45.123 |

#### e. `DATEADD (datepart, number, date)`

- **Description:** Adds a specified `number` (signed integer) to a specified `datepart` of an input `date`, and returns the modified date.
- **Usage:** Calculating future or past dates, such as due dates, expiration dates, or age calculations.
- **Example:**

```sql
SELECT
  DATEADD(day, 7, GETDATE()) AS DatePlus7Days,
  DATEADD(month, -1, GETDATE()) AS DateMinus1Month,
  DATEADD(year, 5, '2023-01-20') AS DatePlus5Years;
```

**Output (example, based on GETDATE() = '2023-01-20'):**

| DatePlus7Days           | DateMinus1Month         | DatePlus5Years          |
| :---------------------- | :---------------------- | :---------------------- |
| 2023-01-27 10:30:45.123 | 2022-12-20 10:30:45.123 | 2028-01-20 00:00:00.000 |

#### f. `DATEDIFF (datepart, startdate, enddate)`

- **Description:** Returns the count (signed integer) of the specified `datepart` boundaries crossed between `startdate` and `enddate`.
- **Usage:** Calculating durations, age, or the difference between two dates in terms of years, months, days, hours, etc.
- **Example:**

```sql
SELECT
  DATEDIFF(day, '2023-01-01', '2023-01-20') AS DaysDifference,
  DATEDIFF(month, '2022-12-15', '2023-01-10') AS MonthsDifference, -- Crosses 1 month boundary
  DATEDIFF(year, '1990-05-01', GETDATE()) AS AgeInYears;
```

**Output (example, based on GETDATE() = '2023-01-20'):**

| DaysDifference | MonthsDifference | AgeInYears |
| :------------- | :--------------- | :--------- |
| 19             | 1                | 33         |

#### g. `DATEPART (datepart, date)`

- **Description:** Returns an integer representing the specified `datepart` of the specified `date`.
- **Usage:** Extracting specific components of a date, such as the year, month, day, hour, minute, or second, for grouping, filtering, or analysis.
- **Example:**

```sql
SELECT
  DATEPART(year, GETDATE()) AS CurrentYear,
  DATEPART(month, GETDATE()) AS CurrentMonth,
  DATEPART(day, GETDATE()) AS CurrentDay,
  DATEPART(weekday, GETDATE()) AS CurrentWeekday; -- Sunday=1, Monday=2, etc.
```

  **Output (example, based on GETDATE() = '2023-01-20', a Friday):**

| CurrentYear | CurrentMonth | CurrentDay | CurrentWeekday |
| :---------- | :----------- | :--------- | :------------- |
| 2023        | 1            | 20         | 6              |

#### h. `DATENAME (datepart, date)`

- **Description:** Returns a `NVARCHAR` string representing the specified `datepart` of the specified `date`.
- **Usage:** Similar to `DATEPART`, but returns the name of the part (e.g., 'January' for month, 'Friday' for weekday), useful for display purposes.
- **Example:**

```sql
SELECT
  DATENAME(month, GETDATE()) AS MonthName,
  DATENAME(weekday, GETDATE()) AS WeekdayName;
```

**Output (example, based on GETDATE() = '2023-01-20', a Friday):**

| MonthName | WeekdayName |
| :-------- | :---------- |
| January   | Friday      |

#### i. `DAY (date)`

- **Description:** Returns the day of the month as an integer (1 to 31). Functionally equivalent to `DATEPART(day, date)`.
- **Usage:** A shorthand for extracting the day component.
- **Example:**

```sql
SELECT DAY(GETDATE()) AS CurrentDayOfMonth;
```

**Output (example, based on GETDATE() = '2023-01-20'):**

| CurrentDayOfMonth |
| :---------------- |
| 20                |

#### j. `MONTH (date)`

- **Description:** Returns the month as an integer (1 to 12). Functionally equivalent to `DATEPART(month, date)`.
- **Usage:** A shorthand for extracting the month component.
- **Example:**

```sql
SELECT MONTH(GETDATE()) AS CurrentMonthNumber;
```

**Output (example, based on GETDATE() = '2023-01-20'):**

| CurrentMonthNumber |
| :----------------- |
| 1                  |

#### k. `YEAR (date)`

- **Description:** Returns the year as an integer. Functionally equivalent to `DATEPART(year, date)`.
- **Usage:** A shorthand for extracting the year component.
- **Example:**

```sql
SELECT YEAR(GETDATE()) AS CurrentYearNumber;
```

**Output (example, based on GETDATE() = '2023-01-20'):**

| CurrentYearNumber |
| :---------------- |
| 2023              |

#### l. `ISDATE (expression)`

- **Description:** Returns 1 if the `expression` is a valid date, time, or datetime value; otherwise, returns 0.
- **Usage:** Validating string inputs before attempting to convert them to date/time types, preventing conversion errors.
- **Example:**

```sql
SELECT
  ISDATE('2023-01-20') AS IsValidDate1,
  ISDATE('NotADate') AS IsValidDate2,
  ISDATE('2023-13-01') AS IsValidDate3; -- Invalid month
```

**Output:**

| IsValidDate1 | IsValidDate2 | IsValidDate3 |
| :----------- | :----------- | :----------- |
| 1            | 0            | 0            |

#### m. `EOMONTH (start_date [, month_to_add])` (SQL Server 2012+)

- **Description:** Returns the last day of the month of the specified `start_date`, with an optional `month_to_add` offset.
- **Usage:** Calculating month-end dates, which is common in financial reporting and billing cycles.
- **Example:**

```sql
SELECT
  EOMONTH(GETDATE()) AS EndOfCurrentMonth,
  EOMONTH(GETDATE(), 1) AS EndOfNextMonth,
  EOMONTH('2023-02-15', 0) AS EndOfFeb2023;
```

**Output (example, based on GETDATE() = '2023-01-20'):**

| EndOfCurrentMonth | EndOfNextMonth | EndOfFeb2023 |
| :---------------- | :------------- | :----------- |
| 2023-01-31        | 2023-02-28     | 2023-02-28   |

#### n. `DATEFROMPARTS (year, month, day)` (SQL Server 2012+)

- **Description:** Returns a `DATE` value for the specified year, month, and day.
- **Usage:** Constructing a `DATE` value from individual integer components, useful when date parts are stored separately.
- **Example:**

```sql
SELECT DATEFROMPARTS(2023, 1, 20) AS ConstructedDate;
```

**Output:**

| ConstructedDate |
| :-------------- |
| 2023-01-20      |

#### o. `DATETIME2FROMPARTS (year, month, day, hour, minute, seconds, fractions, precision)` (SQL Server 2012+)

- **Description:** Returns a `DATETIME2` value for the specified date and time components, with specified precision.
- **Usage:** Constructing a high-precision `DATETIME2` value from individual components.
- **Example:**

```sql
SELECT DATETIME2FROMPARTS(2023, 1, 20, 10, 30, 45, 1234567, 7) AS ConstructedDateTime2;
```

**Output:**

| ConstructedDateTime2        |
| :-------------------------- |
| 2023-01-20 10:30:45.1234567 |

#### p. `SMALLDATETIMEFROMPARTS (year, month, day, hour, minute)` (SQL Server 2012+)

- **Description:** Returns a `SMALLDATETIME` value for the specified date and time components.
- **Usage:** Constructing a `SMALLDATETIME` value from individual components, useful for older systems or when less precision is needed.
- **Example:**

```sql
SELECT SMALLDATETIMEFROMPARTS(2023, 1, 20, 10, 30) AS ConstructedSmallDateTime;
```

**Output:**

| ConstructedSmallDateTime |
| :----------------------- |
| 2023-01-20 10:30:00      |

#### q. `TIMEFROMPARTS (hour, minute, seconds, fractions, precision)` (SQL Server 2012+)

- **Description:** Returns a `TIME` value for the specified time components, with specified precision.
- **Usage:** Constructing a `TIME` value from individual components, useful for time-only data.
- **Example:**

```sql
SELECT TIMEFROMPARTS(10, 30, 45, 1234567, 7) AS ConstructedTime;
```

**Output:**

| ConstructedTime  |
| :--------------- |
| 10:30:45.1234567 |

#### r. `SWITCHOFFSET (datetimeoffset, time_zone_offset)` (SQL Server 2008+)

- **Description:** Changes the time zone offset of a `DATETIMEOFFSET` value without changing the UTC `DATETIME` value.
- **Usage:** Adjusting a `DATETIMEOFFSET` value to a different time zone for display or comparison, while preserving the underlying point in time.
- **Example:**

```sql
DECLARE @dt DATETIMEOFFSET = '2023-01-20 10:00:00 -05:00';
SELECT SWITCHOFFSET(@dt, '+01:00') AS AdjustedDateTimeOffset;
```

**Output:**

| AdjustedDateTimeOffset             |
| :--------------------------------- |
| 2023-01-20 16:00:00.0000000 +01:00 |

#### s. `TODATETIMEOFFSET (expression, time_zone_offset)` (SQL Server 2008+)

- **Description:** Converts a `DATETIME` or `SMALLDATETIME` value to a `DATETIMEOFFSET` value, applying a specified `time_zone_offset`.
- **Usage:** Assigning a time zone offset to a `DATETIME` value that was previously stored without time zone information, effectively defining its local time context.
- **Example:**

```sql
DECLARE @dt DATETIME = '2023-01-20 10:00:00';
SELECT TODATETIMEOFFSET(@dt, '-05:00') AS DateTimeWithOffset;
```

**Output:**

| DateTimeWithOffset                 |
| :--------------------------------- |
| 2023-01-20 10:00:00.0000000 -05:00 |
