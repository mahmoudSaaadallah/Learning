### What is the Determinant?

The determinant is a special scalar value that can be computed from the elements of a **square matrix**. It's denoted as $\det(A)$ or $|A|$. While it's a single number, it carries a tremendous amount of information about the matrix, particularly regarding its invertibility and the geometric transformations it represents.

Think of the determinant as a scaling factor. If you consider a matrix as a linear transformation, the absolute value of its determinant represents the scaling factor by which the area (for $2 \times 2$ matrices) or volume (for $3 \times 3$ matrices) of a region changes after the transformation. The sign of the determinant tells you if the orientation of the space is preserved or reversed.

### How to Calculate the Determinant

The method for calculating the determinant depends on the size of the matrix.

#### 1. For a $1 \times 1$ Matrix

If $A = \begin{pmatrix} a \end{pmatrix}$, then $\det(A) = a$. Simple enough!

#### 2. For a $2 \times 2$ Matrix

This is the simplest non-trivial case. If $A = \begin{pmatrix} a & b \\ c & d \end{pmatrix}$, then the determinant is calculated as:

$\det(A) = ad - bc$

**Example 1:**
Let $A = \begin{pmatrix} 3 & 1 \\ 5 & 2 \end{pmatrix}$.
$\det(A) = (3)(2) - (1)(5) = 6 - 5 = 1$.

**Example 2:**
Let $B = \begin{pmatrix} 4 & -2 \\ 3 & 5 \end{pmatrix}$.
$\det(B) = (4)(5) - (-2)(3) = 20 - (-6) = 20 + 6 = 26$.

#### 3. For a $3 \times 3$ Matrix (Cofactor Expansion or Sarrus' Rule)

For $3 \times 3$ matrices, there are a couple of common methods.

**a) Cofactor Expansion (General Method for any $n \times n$ matrix)**

This method can be used for matrices of any size $n \times n$, but it gets computationally intensive quickly. The idea is to pick a row or a column, and for each element in that row/column, multiply it by its **cofactor**.

The **minor** $M_{ij}$ of an element $a_{ij}$ is the determinant of the submatrix formed by deleting the $i$-th row and $j$-th column.

The **cofactor** $C_{ij}$ of an element $a_{ij}$ is given by $C_{ij} = (-1)^{i+j} M_{ij}$.

Then, the determinant of $A$ is the sum of the products of the elements of any row or column with their corresponding cofactors. For example, expanding along the first row:

If $A = \begin{pmatrix} a_{11} & a_{12} & a_{13} \\ a_{21} & a_{22} & a_{23} \\ a_{31} & a_{32} & a_{33} \end{pmatrix}$, then
$\det(A) = a_{11}C_{11} + a_{12}C_{12} + a_{13}C_{13}$
$= a_{11}(-1)^{1+1}M_{11} + a_{12}(-1)^{1+2}M_{12} + a_{13}(-1)^{1+3}M_{13}$
$= a_{11}\det\begin{pmatrix} a_{22} & a_{23} \\ a_{32} & a_{33} \end{pmatrix} - a_{12}\det\begin{pmatrix} a_{21} & a_{23} \\ a_{31} & a_{33} \end{pmatrix} + a_{13}\det\begin{pmatrix} a_{21} & a_{22} \\ a_{31} & a_{32} \end{pmatrix}$

**Example 3: Using Cofactor Expansion**
Let $A = \begin{pmatrix} 1 & 2 & 3 \\ 2 & 5 & 3 \\ 1 & 0 & 8 \end{pmatrix}$. Let's expand along the first row.

$M_{11} = \det\begin{pmatrix} 5 & 3 \\ 0 & 8 \end{pmatrix} = (5)(8) - (3)(0) = 40 - 0 = 40$. $C_{11} = (-1)^{1+1}M_{11} = 1 \times 40 = 40$.
$M_{12} = \det\begin{pmatrix} 2 & 3 \\ 1 & 8 \end{pmatrix} = (2)(8) - (3)(1) = 16 - 3 = 13$. $C_{12} = (-1)^{1+2}M_{12} = -1 \times 13 = -13$.
$M_{13} = \det\begin{pmatrix} 2 & 5 \\ 1 & 0 \end{pmatrix} = (2)(0) - (5)(1) = 0 - 5 = -5$. $C_{13} = (-1)^{1+3}M_{13} = 1 \times (-5) = -5$.

$\det(A) = a_{11}C_{11} + a_{12}C_{12} + a_{13}C_{13}$
$= (1)(40) + (2)(-13) + (3)(-5)$
$= 40 - 26 - 15 = 40 - 41 = -1$.

**b) Sarrus' Rule (Only for $3 \times 3$ matrices)**

This is a handy shortcut specifically for $3 \times 3$ matrices.
1.  Write out the matrix.
2.  Rewrite the first two columns to the right of the matrix.
3.  Multiply along the three main diagonals (top-left to bottom-right) and add these products.
4.  Multiply along the three anti-diagonals (top-right to bottom-left) and subtract these products.

For $A = \begin{pmatrix} a_{11} & a_{12} & a_{13} \\ a_{21} & a_{22} & a_{23} \\ a_{31} & a_{32} & a_{33} \end{pmatrix}$:

$\begin{pmatrix} a_{11} & a_{12} & a_{13} \\ a_{21} & a_{22} & a_{23} \\ a_{31} & a_{32} & a_{33} \end{pmatrix} \begin{matrix} a_{11} & a_{12} \\ a_{21} & a_{22} \\ a_{31} & a_{32} \end{matrix}$

$\det(A) = (a_{11}a_{22}a_{33} + a_{12}a_{23}a_{31} + a_{13}a_{21}a_{32}) - (a_{13}a_{22}a_{31} + a_{11}a_{23}a_{32} + a_{12}a_{21}a_{33})$

**Example 4: Using Sarrus' Rule**
Let $A = \begin{pmatrix} 1 & 2 & 3 \\ 2 & 5 & 3 \\ 1 & 0 & 8 \end{pmatrix}$.

$\begin{pmatrix} 1 & 2 & 3 \\ 2 & 5 & 3 \\ 1 & 0 & 8 \end{pmatrix} \begin{matrix} 1 & 2 \\ 2 & 5 \\ 1 & 0 \end{matrix}$

Positive diagonals:
$(1)(5)(8) = 40$
$(2)(3)(1) = 6$
$(3)(2)(0) = 0$
Sum = $40 + 6 + 0 = 46$

Negative diagonals:
$(3)(5)(1) = 15$
$(1)(3)(0) = 0$
$(2)(2)(8) = 32$
Sum = $15 + 0 + 32 = 47$

$\det(A) = 46 - 47 = -1$.
This matches the result from cofactor expansion!

#### 4. For Larger Matrices ($n \times n$ where $n > 3$)

For matrices larger than $3 \times 3$, cofactor expansion becomes very tedious. The most efficient method is to use **row reduction (Gaussian elimination)** to transform the matrix into an upper triangular or lower triangular matrix. The determinant of a triangular matrix is simply the product of its diagonal entries.

Remember these rules for row operations and their effect on the determinant:
*   **Swapping two rows:** Multiplies the determinant by $-1$.
*   **Multiplying a row by a scalar $k$:** Multiplies the determinant by $k$.
*   **Adding a multiple of one row to another row:** Does NOT change the determinant.

### Properties of the Determinant

These properties are incredibly useful for calculations and theoretical understanding:

*   **Determinant of the Identity Matrix:** $\det(I) = 1$.
*   **Determinant of a Product:** $\det(AB) = \det(A)\det(B)$.
*   **Determinant of a Scalar Multiple:** $\det(kA) = k^n \det(A)$, where $A$ is an $n \times n$ matrix.
*   **Determinant of the Transpose:** $\det(A^T) = \det(A)$.
*   **Determinant of the Inverse:** $\det(A^{-1}) = 1/\det(A)$, provided $A^{-1}$ exists.
*   **If a matrix has a row or column of all zeros, its determinant is 0.**
*   **If a matrix has two identical rows or columns, its determinant is 0.**
*   **If one row (or column) is a scalar multiple of another row (or column), its determinant is 0.**
*   **The determinant of a triangular matrix (upper or lower) is the product of its diagonal entries.**

### Significance of the Determinant

1.  **Invertibility:** This is perhaps the most important application we've discussed. A square matrix $A$ is invertible (non-singular) if and only if $\det(A) \neq 0$. If $\det(A) = 0$, the matrix is singular and has no inverse.

2.  **Solving Systems of Linear Equations:**
    *   If $\det(A) \neq 0$, the system $A\mathbf{x} = \mathbf{b}$ has a unique solution.
    *   If $\det(A) = 0$, the system either has no solutions or infinitely many solutions.
    *   Cramer's Rule uses determinants to find the solution to systems of linear equations, though it's generally less efficient than Gaussian elimination for larger systems.

3.  **Geometric Interpretation:** As mentioned, the determinant represents the scaling factor of area or volume under a linear transformation. A zero determinant means the transformation collapses space into a lower dimension (e.g., a 2D area collapses to a line or point, a 3D volume collapses to a plane, line, or point).

4.  **Eigenvalues:** Determinants are fundamental in finding the eigenvalues of a matrix, which are crucial in many applications.

So, the determinant is far more than just a number; it's a powerful diagnostic tool and a key concept that underpins much of linear algebra. Make sure you're comfortable calculating it for $2 \times 2$ and $3 \times 3$ matrices, and understand its properties.

Any further questions on this, class?