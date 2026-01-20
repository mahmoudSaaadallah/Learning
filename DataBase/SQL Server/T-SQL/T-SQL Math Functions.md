### SQL Server Built-in Math Functions: A Deep Dive

SQL Server provides a rich set of mathematical functions that operate on numeric input values and return numeric output values. Understanding these functions is key to writing efficient and robust T-SQL code.

#### 1. Absolute Value: `ABS()`

-   **Purpose**: Returns the absolute (positive) value of a given numeric expression. It effectively removes any negative sign.
-   **Syntax**: `ABS(numeric_expression)`
-   **Return Type**: The same data type as the input `numeric_expression`.
-   **Example**:
```sql
SELECT
	ABS(-10.5) AS AbsolutePositive,
	ABS(20) AS AbsolutePositiveAgain,
	ABS(0) AS AbsoluteZero;
```

| AbsolutePositive | AbsolutePositiveAgain | AbsoluteZero |
|---|---|---|
| 10.5 | 20 | 0 |

#### 2. Ceiling: `CEILING()`

-   **Purpose**: Returns the smallest integer greater than or equal to the specified numeric expression. It always rounds *up* to the next whole number.
-   **Syntax**: `CEILING(numeric_expression)`
-   **Return Type**: `float` for `float` input, otherwise `numeric` with a scale of 0.
-   **Example**:
```sql
SELECT
	CEILING(12.34) AS Ceiling1,
	CEILING(12.99) AS Ceiling2,
	CEILING(-12.34) AS CeilingNegative;
```

| Ceiling1 | Ceiling2 | CeilingNegative |
|---|---|---|
| 13 | 13 | -12 |

#### 3. Floor: `FLOOR()`

-   **Purpose**: Returns the largest integer less than or equal to the specified numeric expression. It always rounds *down* to the previous whole number.
-   **Syntax**: `FLOOR(numeric_expression)`
-   **Return Type**: `float` for `float` input, otherwise `numeric` with a scale of 0.
-   **Example**:
```sql
SELECT
	FLOOR(12.34) AS Floor1,
	FLOOR(12.99) AS Floor2,
	FLOOR(-12.34) AS FloorNegative;
```

| Floor1 | Floor2 | FloorNegative |
|---|---|---|
| 12 | 12 | -13 |

#### 4. Rounding: `ROUND()`

-   **Purpose**: Rounds a numeric expression to a specified length or precision.
-   **Syntax**: `ROUND(numeric_expression, length [, function])`
    -   `numeric_expression`: The number to be rounded.
    -   `length`: The precision to which `numeric_expression` is to be rounded. A positive `length` rounds to the right of the decimal point, a negative `length` rounds to the left.
    -   `function` (optional): If `function` is 0 (or omitted), `numeric_expression` is rounded. If `function` is any other value, `numeric_expression` is truncated.
-   **Return Type**: The same data type as the input `numeric_expression`.
-   **Example**:
```sql
SELECT
	ROUND(123.456, 2) AS RoundToTwoDecimals,
	ROUND(123.456, 0) AS RoundToWholeNumber,
	ROUND(123.456, -1) AS RoundToTens,
	ROUND(123.456, 2, 1) AS TruncateToTwoDecimals; -- Truncates instead of rounds
```

| RoundToTwoDecimals | RoundToWholeNumber | RoundToTens | TruncateToTwoDecimals |
|---|---|---|---|
| 123.46 | 123.00 | 120.00 | 123.45 |

#### 5. Power: `POWER()`

-   **Purpose**: Returns the value of a specified expression raised to a specified power.
-   **Syntax**: `POWER(base, exponent)`
-   **Return Type**: `float`.
-   **Example**:
```sql
SELECT
	POWER(2, 3) AS TwoCubed,
	POWER(9, 0.5) AS SquareRootOfNine; -- Equivalent to SQRT(9)
```

| TwoCubed | SquareRootOfNine |
|---|---|
| 8.0 | 3.0 |

#### 6. Square Root: `SQRT()`

-   **Purpose**: Returns the square root of a specified float expression.
-   **Syntax**: `SQRT(float_expression)`
-   **Return Type**: `float`.
-   **Note**: The input `float_expression` must be non-negative.
-   **Example**:
```sql
SELECT
	SQRT(25) AS SquareRoot25,
	SQRT(2) AS SquareRoot2;
```

| SquareRoot25 | SquareRoot2 |
|---|---|
| 5.0 | 1.4142135623730951 |

#### 7. Exponential: `EXP()`

-   **Purpose**: Returns the exponential value of a specified float expression. This is `e` (Euler's number, approximately 2.71828) raised to the power of the input expression.
-   **Syntax**: `EXP(float_expression)`
-   **Return Type**: `float`.
-   **Example**:
```sql
SELECT
	EXP(1) AS EulersNumber,
	EXP(0) AS EToThePowerOfZero;
```

| EulersNumber | EToThePowerOfZero |
|---|---|
| 2.718281828459045 | 1.0 |

#### 8. Logarithm: `LOG()` and `LOG10()`

-   **Purpose**:
    -   `LOG()`: Returns the natural logarithm (base `e`) of a specified float expression, or the logarithm to a specified base.
    -   `LOG10()`: Returns the base-10 logarithm of a specified float expression.
-   **Syntax**:
    -   `LOG(float_expression [, base])`
    -   `LOG10(float_expression)`
-   **Return Type**: `float`.
-   **Note**: The input `float_expression` must be positive.
-   **Example**:
```sql
SELECT
	LOG(10) AS NaturalLogOf10,
	LOG(100, 10) AS LogBase10Of100, -- Equivalent to LOG10(100)
	LOG10(1000) AS Log10Of1000;
```

| NaturalLogOf10 | LogBase10Of100 | Log10Of1000 |
|---|---|---|
| 2.302585092994046 | 2.0 | 3.0 |

#### 9. Sign: `SIGN()`

-   **Purpose**: Returns the positive (+1), zero (0), or negative (-1) sign of the specified numeric expression.
-   **Syntax**: `SIGN(numeric_expression)`
-   **Return Type**: The same data type as the input `numeric_expression`.
-   **Example**:
```sql
SELECT
	SIGN(15) AS SignPositive,
	SIGN(-15) AS SignNegative,
	SIGN(0) AS SignZero;
```

| SignPositive | SignNegative | SignZero |
|---|---|---|
| 1 | -1 | 0 |

#### 10. Pi: `PI()`

-   **Purpose**: Returns the constant value of PI.
-   **Syntax**: `PI()`
-   **Return Type**: `float`.
-   **Example**:
```sql
SELECT PI() AS ValueOfPi;
```

| ValueOfPi |
|---|
| 3.141592653589793 |

#### 11. Random Number: `RAND()`

-   **Purpose**: Returns a pseudo-random `float` value from 0 through 1, exclusive of 1.
-   **Syntax**: `RAND([seed])`
    -   `seed` (optional): An integer expression that provides a seed value. If `seed` is not specified, SQL Server assigns a seed value randomly. If a seed is specified, the result is deterministic for that seed.
-   **Return Type**: `float`.
-   **Example**:
```sql
SELECT
	RAND() AS RandomNumber1,
	RAND() AS RandomNumber2,
	RAND(123) AS SeededRandom1,
	RAND(123) AS SeededRandom2; -- Will be the same as SeededRandom1
```

| RandomNumber1 | RandomNumber2 | SeededRandom1 | SeededRandom2 |
|---|---|---|---|
| 0.123456789... | 0.987654321... | 0.710009623... | 0.710009623... |
*(Note: `RandomNumber1` and `RandomNumber2` will vary each time you execute the query without a seed.)*

#### 12. Trigonometric Functions

SQL Server also includes a comprehensive set of trigonometric functions, which are essential for geometry, physics, and engineering applications. All inputs and outputs for these functions are in radians.

-   **`SIN(float_expression)`**: Returns the sine of the specified angle in radians.
-   **`COS(float_expression)`**: Returns the cosine of the specified angle in radians.
-   **`TAN(float_expression)`**: Returns the tangent of the specified angle in radians.
-   **`ASIN(float_expression)`**: Returns the angle in radians whose sine is `float_expression`. (Arc sine)
-   **`ACOS(float_expression)`**: Returns the angle in radians whose cosine is `float_expression`. (Arc cosine)
-   **`ATAN(float_expression)`**: Returns the angle in radians whose tangent is `float_expression`. (Arc tangent)
-   **`ATN2(float_expression_y, float_expression_x)`**: Returns the angle in radians between the positive x-axis and the point (x, y). This function is particularly useful as it correctly handles all four quadrants.

-   **Example**:
```sql
SELECT
	SIN(PI()/2) AS SineOf90Degrees, -- PI()/2 radians = 90 degrees
	COS(0) AS CosineOf0Degrees,
	TAN(PI()/4) AS TangentOf45Degrees,
	ASIN(1) AS ArcSineOf1,
	ACOS(0) AS ArcCosineOf0,
	ATAN(1) AS ArcTangentOf1,
	ATN2(1, 1) AS ATN2_Quadrant1; -- Angle for (1,1)
```

| SineOf90Degrees | CosineOf0Degrees | TangentOf45Degrees | ArcSineOf1 | ArcCosineOf0 | ArcTangentOf1 | ATN2_Quadrant1 |
|---|---|---|---|---|---|---|
| 1.0 | 1.0 | 1.0 | 1.5707963267948966 | 1.5707963267948966 | 0.7853981633974483 | 0.7853981633974483 |
*(Note: 1.57079... is approximately PI/2, and 0.78539... is approximately PI/4)*

### Conclusion

These built-in mathematical functions are indispensable for any serious database developer. They allow for complex numerical operations to be performed directly within the database engine, often leading to more efficient and maintainable code compared to fetching data and processing it in an application layer. Always consider the data types involved and the specific behavior of each function (e.g., rounding rules, domain constraints for `SQRT` and `LOG`) to ensure the accuracy and reliability of your computations.

I hope this detailed discussion provides a solid foundation for your work with T-SQL math functions! Let me know if you have any further questions or specific scenarios you'd like to explore.