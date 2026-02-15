### Gaussian Elimination

**What is Gaussian Elimination?**

At its core, Gaussian Elimination is a systematic process for solving systems of linear equations by transforming the system's augmented matrix into an **upper triangular form** (or more generally, a **row echelon form**) using a set of elementary row operations. The beauty is that these operations do not change the solution set of the system.

**The Three Elementary Row Operations:**

These are the only moves we're allowed to make, and they preserve the solutions:
1.  **Row Swap:** Exchange two rows ($R_i \leftrightarrow R_j$). This is like reordering your equations.
2.  **Scalar Multiplication:** Multiply a row by a non-zero scalar ($cR_i \rightarrow R_i$, where $c \neq 0$). This is like multiplying an equation by a constant.
3.  **Row Addition:** Add a multiple of one row to another row ($R_i + cR_j \rightarrow R_i$). This is like adding a multiple of one equation to another.

**The Goal: Row Echelon Form**

Our aim is to transform the augmented matrix $\begin{pmatrix} A & | & b \end{pmatrix}$ into a form where:
*   All non-zero rows are above any rows of all zeros.
*   The first non-zero element in each non-zero row (called the **pivot**) is to the right of the pivot of the row above it.
*   All entries in a column below a pivot are zero.

If we go a step further to **Reduced Row Echelon Form (RREF)**, we also ensure:
*   Each pivot is 1.
*   Each pivot is the only non-zero entry in its column.

For solving systems, reaching row echelon form and then using back-substitution is often sufficient.

---

### Understanding Solutions: Pivots and Free Variables

The key to understanding the type of solution lies in identifying **pivots** and **free variables** after Gaussian Elimination.

*   **Pivots:** These are the first non-zero entries in each row of the row echelon form. Each pivot corresponds to a **pivot variable** (or basic variable). These variables will have their values determined.
*   **Free Variables:** If a column in the coefficient matrix (the $A$ part) does *not* contain a pivot, the corresponding variable is a **free variable**. Free variables can take on any value, and the other variables will be expressed in terms of them.

---

### Types of Solutions with Examples (Beyond 2 Dimensions!)

Let's explore the three possibilities with systems of 3 equations in 3 unknowns.

#### Case 1: Unique Solution (A Pivot in Every Column)

This occurs when, after Gaussian Elimination, every column of the coefficient matrix $A$ has a pivot. This means every variable is a pivot variable, and its value will be uniquely determined.

**Example 1:**
Solve the system:
$x_1 + 2x_2 + x_3 = 2$
$3x_1 + 8x_2 + x_3 = 12$
$4x_1 + 10x_2 + x_3 = 14$

**Step 1: Form the augmented matrix.**
$\begin{pmatrix} 1 & 2 & 1 & | & 2 \\ 3 & 8 & 1 & | & 12 \\ 4 & 10 & 1 & | & 14 \end{pmatrix}$

**Step 2: Forward Elimination to Row Echelon Form.**
*   Eliminate entries below the first pivot (1 in $R_1C_1$):
    *   $R_2 \leftarrow R_2 - 3R_1$
    *   $R_3 \leftarrow R_3 - 4R_1$
    $\begin{pmatrix} 1 & 2 & 1 & | & 2 \\ 0 & 2 & -2 & | & 6 \\ 0 & 2 & -3 & | & 6 \end{pmatrix}$

*   Eliminate entries below the second pivot (2 in $R_2C_2$):
    *   $R_3 \leftarrow R_3 - R_2$
    $\begin{pmatrix} 1 & 2 & 1 & | & 2 \\ 0 & 2 & -2 & | & 6 \\ 0 & 0 & -1 & | & 0 \end{pmatrix}$

We now have an upper triangular matrix. The pivots are 1 (in $C_1$), 2 (in $C_2$), and -1 (in $C_3$). Every column of the coefficient matrix has a pivot. This indicates a unique solution.

**Step 3: Back-Substitution.**
The transformed system is:
$x_1 + 2x_2 + x_3 = 2$
$2x_2 - 2x_3 = 6$
$-x_3 = 0$

From the last equation:
$-x_3 = 0 \implies x_3 = 0$

Substitute $x_3=0$ into the second equation:
$2x_2 - 2(0) = 6 \implies 2x_2 = 6 \implies x_2 = 3$

Substitute $x_3=0$ and $x_2=3$ into the first equation:
$x_1 + 2(3) + 0 = 2 \implies x_1 + 6 = 2 \implies x_1 = -4$

**Unique Solution:** $x = \begin{pmatrix} -4 \\ 3 \\ 0 \end{pmatrix}$.

#### Case 2: Infinitely Many Solutions (At Least One Free Variable)

This occurs when, after Gaussian Elimination, there is at least one column in the coefficient matrix $A$ that does *not* contain a pivot. The variables corresponding to these columns are free variables.

**Example 2:**
Solve the system:
$x_1 + 2x_2 + 3x_3 = 6$
$2x_1 + 4x_2 + 6x_3 = 12$
$3x_1 + 6x_2 + 9x_3 = 18$

**Step 1: Form the augmented matrix.**
$\begin{pmatrix} 1 & 2 & 3 & | & 6 \\ 2 & 4 & 6 & | & 12 \\ 3 & 6 & 9 & | & 18 \end{pmatrix}$

**Step 2: Forward Elimination.**
*   Eliminate entries below the first pivot (1 in $R_1C_1$):
    *   $R_2 \leftarrow R_2 - 2R_1$
    *   $R_3 \leftarrow R_3 - 3R_1$
    $\begin{pmatrix} 1 & 2 & 3 & | & 6 \\ 0 & 0 & 0 & | & 0 \\ 0 & 0 & 0 & | & 0 \end{pmatrix}$

Here, we have pivots only in the first column (1). Columns 2 and 3 do not have pivots. This means $x_2$ and $x_3$ are **free variables**. The row of zeros $\begin{pmatrix} 0 & 0 & 0 & | & 0 \end{pmatrix}$ indicates that _one equation was a multiple of another_, providing no new information.

**Step 3: Express the solution in terms of free variables.**
The transformed system is simply:
$x_1 + 2x_2 + 3x_3 = 6$

Let $x_2 = s$ and $x_3 = t$, where $s$ and $t$ can be any real numbers.
Then, $x_1 = 6 - 2x_2 - 3x_3 = 6 - 2s - 3t$.

**Infinite Solutions:** The solution vector $x$ can be written as:
$x = \begin{pmatrix} x_1 \\ x_2 \\ x_3 \end{pmatrix} = \begin{pmatrix} 6 - 2s - 3t \\ s \\ t \end{pmatrix}$

We can also decompose this into a particular solution and a combination of nullspace vectors (which we'll discuss more later!):
$x = \begin{pmatrix} 6 \\ 0 \\ 0 \end{pmatrix} + s \begin{pmatrix} -2 \\ 1 \\ 0 \end{pmatrix} + t \begin{pmatrix} -3 \\ 0 \\ 1 \end{pmatrix}$

This shows that there are infinitely many solutions, forming a plane in 3D space.

#### Case 3: No Solution (A Contradiction Arises)

This occurs when, during Gaussian Elimination, we arrive at a row that represents a contradictory statement, such as $0 = \text{non-zero number}$.

**Example 3:**
Solve the system:
$x_1 + x_2 = 1$
$x_1 + x_2 = 2$

This is clearly impossible, as $x_1+x_2$ cannot simultaneously be 1 and 2.

**Step 1: Form the augmented matrix.**
$\begin{pmatrix} 1 & 1 & | & 1 \\ 1 & 1 & | & 2 \end{pmatrix}$

**Step 2: Forward Elimination.**
*   $R_2 \leftarrow R_2 - R_1$
    $\begin{pmatrix} 1 & 1 & | & 1 \\ 0 & 0 & | & 1 \end{pmatrix}$

The last row translates to $0x_1 + 0x_2 = 1$, which simplifies to $0 = 1$. This is a **contradiction**!

**No Solution:** When Gaussian Elimination leads to a row like $\begin{pmatrix} 0 & 0 & \dots & 0 & | & \text{non-zero number} \end{pmatrix}$, the system has no solution. Geometrically, this means the lines or planes represented by the equations do not intersect at a common point.

---

**Summary of Solution Types:**

| Outcome of Gaussian Elimination | Pivots | Free Variables | Type of Solution |
| :------------------------------ | :----- | :------------- | :--------------- |
| No contradiction, pivot in every column | All columns have pivots | None | Unique Solution |
| No contradiction, at least one column without a pivot | Fewer pivots than columns | At least one | Infinitely Many Solutions |
| A row like $\begin{pmatrix} 0 & \dots & 0 & | & c \end{pmatrix}$ where $c \neq 0$ | (Irrelevant, as system is inconsistent) | (Irrelevant) | No Solution |

---

Gaussian Elimination is your first true algorithmic friend in linear algebra. Master it, and you'll have a powerful tool for understanding the solvability and structure of any linear system.

Are you ready to move on to the grand concepts of Vector Spaces and Subspaces? Let me know!