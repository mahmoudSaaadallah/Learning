We'll cover `FORMAT()`, `DAY()`, `MONTH()`, `YEAR()`, and `EOMONTH()`.
--

### 1. `FORMAT()` Function

The `FORMAT()` function is a powerful tool for presenting date/time and numeric values in a specified format, leveraging the .NET Framework's formatting capabilities. It's particularly useful for generating user-friendly output strings.

*   **Purpose:** Formats a value with a specified format string and an optional culture.
*   **Syntax:** `FORMAT ( value, format [, culture ] )`
    *   `value`: The expression to be formatted (e.g., a `DATETIME`, `DATE`, `NUMERIC`).
    *   `format`: A string specifying the format pattern (e.g., 'yyyy-MM-dd', 'MM/dd/yyyy HH:mm:ss', 'D' for long date, 'd' for short date).
    *   `culture` (optional): A string specifying the culture (e.g., 'en-US', 'fr-FR', 'de-DE'). If omitted, the current session's culture is used.

**Detailed Examples:**

Let's use `GETDATE()` as our base, which currently returns `2026-01-14 22:40:00.000`.

```sql
DECLARE @CurrentDateTime DATETIME = GETDATE();

-- Basic date formatting
SELECT FORMAT(@CurrentDateTime, 'yyyy-MM-dd') AS 'YYYY-MM-DD';
SELECT FORMAT(@CurrentDateTime, 'MM/dd/yyyy') AS 'MM/DD/YYYY';
SELECT FORMAT(@CurrentDateTime, 'dd-MMM-yyyy') AS 'DD-MON-YYYY';

-- Date and time formatting
SELECT FORMAT(@CurrentDateTime, 'HH:mm:ss') AS '24HourTime';
SELECT FORMAT(@CurrentDateTime, 'hh:mm tt') AS '12HourTime';
SELECT FORMAT(@CurrentDateTime, 'yyyy-MM-dd HH:mm:ss.fff') AS 'FullDateTime';

-- Using standard format specifiers (case-sensitive)
SELECT FORMAT(@CurrentDateTime, 'D') AS 'LongDate';  -- e.g., Wednesday, January 14, 2026
SELECT FORMAT(@CurrentDateTime, 'd') AS 'ShortDate'; -- e.g., 1/14/2026
SELECT FORMAT(@CurrentDateTime, 'F') AS 'FullDateTimeLong'; -- e.g., Wednesday, January 14, 2026 10:40:00 PM

-- Formatting with culture
SELECT FORMAT(@CurrentDateTime, 'D', 'fr-FR') AS 'FrenchLongDate'; -- e.g., mercredi 14 janvier 2026
SELECT FORMAT(@CurrentDateTime, 'd', 'de-DE') AS 'GermanShortDate'; -- e.g., 14.01.2026
```
**Example Output (based on 2026-01-14 22:40:00.000):**

| Column Name          | Result                               |
|----------------------|--------------------------------------|
| YYYY-MM-DD           | 2026-01-14                           |
| MM/DD/YYYY           | 01/14/2026                           |
| DD-MON-YYYY          | 14-Jan-2026                          |
| 24HourTime           | 22:40:00                             |
| 12HourTime           | 10:40 PM                             |
| FullDateTime         | 2026-01-14 22:40:00.000              |
| LongDate             | Wednesday, January 14, 2026          |
| ShortDate            | 1/14/2026                            |
| FullDateTimeLong     | Wednesday, January 14, 2026 10:40:00 PM |
| FrenchLongDate       | mercredi 14 janvier 2026             |
| GermanShortDate      | 14.01.2026                           |

**Best Practices and Considerations:**
*   **Readability:** `FORMAT()` makes your code very readable for complex formatting requirements.
*   **Performance:** For simple, common date formats, `CONVERT()` with a style code (as we discussed in [[T-SQL Casting]]) can sometimes be more performant than `FORMAT()`, as `FORMAT()` involves the .NET Framework Common Language Runtime (CLR). Use `FORMAT()` when you need its specific flexibility or culture-aware formatting.
*   **Data Type:** The output of `FORMAT()` is always `NVARCHAR`.

---

### 2. `DAY()` Function

The `DAY()` function is a straightforward way to extract the day of the month from a date expression.

*   **Purpose:** Returns the day of the month as an integer (1 to 31).
*   **Syntax:** `DAY ( date )`
    *   `date`: An expression that can resolve to a `DATE`, `DATETIME`, `DATETIME2`, or other date/time data type.

**Detailed Examples:**

```sql
DECLARE @SpecificDate DATETIME = '2026-01-14 10:30:00';
DECLARE @AnotherDate DATE = '2025-12-25';

SELECT DAY(@SpecificDate) AS 'DayFromDateTime';
SELECT DAY(@AnotherDate) AS 'DayFromDate';
SELECT DAY(GETDATE()) AS 'CurrentDay';
SELECT DAY('2024-02-29') AS 'LeapYearDay';
SELECT DAY(NULL) AS 'DayFromNull';
```
**Example Output:**

| Column Name   | Result |
|---------------|--------|
| DayFromDateTime | 14     |
| DayFromDate   | 25     |
| CurrentDay    | 14     |
| LeapYearDay   | 29     |
| DayFromNull   | NULL   |

**Best Practices and Considerations:**
*   `DAY()` is functionally equivalent to `DATEPART(day, date)`.
*   It returns `NULL` if the input `date` is `NULL`.
*   It only cares about the day component; time components are ignored.

---

### 3. `MONTH()` Function

Similar to `DAY()`, the `MONTH()` function extracts the month number from a date expression.

*   **Purpose:** Returns the month of the year as an integer (1 to 12).
*   **Syntax:** `MONTH ( date )`
    *   `date`: An expression that can resolve to a `DATE`, `DATETIME`, `DATETIME2`, or other date/time data type.

**Detailed Examples:**

```sql
DECLARE @SpecificDate DATETIME = '2026-01-14 10:30:00';
DECLARE @AnotherDate DATE = '2025-07-15';

SELECT MONTH(@SpecificDate) AS 'MonthFromDateTime';
SELECT MONTH(@AnotherDate) AS 'MonthFromDate';
SELECT MONTH(GETDATE()) AS 'CurrentMonth';
SELECT MONTH('2023-11-01') AS 'NovemberMonth';
SELECT MONTH(NULL) AS 'MonthFromNull';
```
**Example Output:**

| Column Name     | Result |
|-----------------|--------|
| MonthFromDateTime | 1      |
| MonthFromDate   | 7      |
| CurrentMonth    | 1      |
| NovemberMonth   | 11     |
| MonthFromNull   | NULL   |

**Best Practices and Considerations:**
*   `MONTH()` is functionally equivalent to `DATEPART(month, date)`.
*   It returns `NULL` if the input `date` is `NULL`.

---

### 4. `YEAR()` Function

The `YEAR()` function extracts the year number from a date expression.

*   **Purpose:** Returns the year as an integer.
*   **Syntax:** `YEAR ( date )`
    *   `date`: An expression that can resolve to a `DATE`, `DATETIME`, `DATETIME2`, or other date/time data type.

**Detailed Examples:**

```sql
DECLARE @SpecificDate DATETIME = '2026-01-14 10:30:00';
DECLARE @AnotherDate DATE = '1999-07-15';

SELECT YEAR(@SpecificDate) AS 'YearFromDateTime';
SELECT YEAR(@AnotherDate) AS 'YearFromDate';
SELECT YEAR(GETDATE()) AS 'CurrentYear';
SELECT YEAR('1900-01-01') AS 'CenturyYear';
SELECT YEAR(NULL) AS 'YearFromNull';
```
**Example Output:**

| Column Name    | Result |
|----------------|--------|
| YearFromDateTime | 2026   |
| YearFromDate   | 1999   |
| CurrentYear    | 2026   |
| CenturyYear    | 1900   |
| YearFromNull   | NULL   |

**Best Practices and Considerations:**
*   `YEAR()` is functionally equivalent to `DATEPART(year, date)`.
*   It returns `NULL` if the input `date` is `NULL`.

---

### 5. `EOMONTH()` Function

The `EOMONTH()` function is incredibly useful for financial reporting, scheduling, and any scenario where you need to determine the last day of a given month.

*   **Purpose:** Returns the last day of the month of the specified `start_date`, with an optional offset in months.
*   **Syntax:** `EOMONTH ( start_date [, month_to_add ] )`
    *   `start_date`: A date expression that specifies the date for which to return the last day of the month.
    *   `month_to_add` (optional): An integer expression that specifies the number of months to add to `start_date`. A positive value returns a future month, a negative value returns a past month. If omitted, it defaults to 0 (the current month).

**Detailed Examples:**

```sql
DECLARE @CurrentDate DATE = GETDATE(); -- '2026-01-14'

-- Last day of the current month
SELECT EOMONTH(@CurrentDate) AS 'EndOfCurrentMonth';

-- Last day of the next month
SELECT EOMONTH(@CurrentDate, 1) AS 'EndOfNextMonth';

-- Last day of the previous month
SELECT EOMONTH(@CurrentDate, -1) AS 'EndOfPreviousMonth';

-- Last day of a month several months in the future
SELECT EOMONTH(@CurrentDate, 6) AS 'EndOfSixMonthsLater';

-- Last day of a month several months in the past
SELECT EOMONTH(@CurrentDate, -12) AS 'EndOfOneYearAgo';

-- EOMONTH with a date that is already the end of the month
SELECT EOMONTH('2026-02-28') AS 'EndOfFeb2026'; -- 2026 is not a leap year
SELECT EOMONTH('2024-02-15', 0) AS 'EndOfFeb2024'; -- 2024 is a leap year
```
**Example Output (based on @CurrentDate = '2026-01-14'):**

| Column Name         | Result     |
|---------------------|------------|
| EndOfCurrentMonth   | 2026-01-31 |
| EndOfNextMonth      | 2026-02-28 |
| EndOfPreviousMonth  | 2025-12-31 |
| EndOfSixMonthsLater | 2026-07-31 |
| EndOfOneYearAgo     | 2025-01-31 |
| EndOfFeb2026        | 2026-02-28 |
| EndOfFeb2024        | 2024-02-29 |

**Best Practices and Considerations:**
*   **Availability:** `EOMONTH()` was introduced in SQL Server 2012. If you're working with older versions, you'd have to use more complex date arithmetic (e.g., `DATEADD(dd, -1, DATEADD(mm, DATEDIFF(mm, 0, @date) + 1, 0))`).
*   **Return Type:** The return type is `DATE`.
*   **NULL Handling:** If `start_date` is `NULL`, `EOMONTH()` returns `NULL`.

---

These functions provide a robust toolkit for handling date and time data in SQL Server. By understanding their individual capabilities and how they can be combined, you'll be well-equipped to tackle a wide array of data manipulation challenges. Always consider the specific requirements of your task, including performance and compatibility, when choosing the most appropriate function.