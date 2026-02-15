### Matrices

We've introduced matrices as rectangular arrays of numbers, powerful tools for organizing information and, more importantly, representing linear transformations. Now, let's explore some special types of matrices and the fundamental operations we can perform on them.

---

### Types of Matrices

Not all matrices are created equal! Certain structures give matrices special properties and roles.

1.  **Square Matrix:**
    *   **Definition:** A matrix with the same number of rows and columns ($n \times n$).
    *   **Why it's special:** Many important concepts like determinants, eigenvalues, and inverses are primarily defined for square matrices.
    *   **Example:**
        $A = \begin{pmatrix} 1 & 2 \\ 3 & 4 \end{pmatrix}$ (a $2 \times 2$ square matrix)

2.  **Identity Matrix ($I$):**
    *   **Definition:** A square matrix with ones on the main diagonal (from top-left to bottom-right) and zeros everywhere else.
    *   **Why it's special:** It's the "multiplicative identity" for matrices. Just like multiplying a number by 1 doesn't change it, multiplying a matrix by the identity matrix doesn't change it ($AI = IA = A$). It represents a transformation that does nothing!
    *   **Example:**
        $I_2 = \begin{pmatrix} 1 & 0 \\ 0 & 1 \end{pmatrix}$ (the $2 \times 2$ identity matrix)
        $I_3 = \begin{pmatrix} 1 & 0 & 0 \\ 0 & 1 & 0 \\ 0 & 0 & 1 \end{pmatrix}$ (the $3 \times 3$ identity matrix)

3.  **Zero Matrix ($0$):**
    *   **Definition:** A matrix where all entries are zero.
    *   **Why it's special:** It's the "additive identity" for matrices. Adding a zero matrix to any matrix doesn't change it ($A + 0 = A$).
    *   **Example:**
        $0 = \begin{pmatrix} 0 & 0 \\ 0 & 0 \\ 0 & 0 \end{pmatrix}$ (a $3 \times 2$ zero matrix)

4.  **Diagonal Matrix:**
    *   **Definition:** A square matrix where all non-diagonal entries are zero. The diagonal entries can be anything.
    *   **Why it's special:** They are very easy to work with, especially for multiplication and powers. Identity matrices are a special case of diagonal matrices.
    *   **Example:**
        $D = \begin{pmatrix} 5 & 0 & 0 \\ 0 & -2 & 0 \\ 0 & 0 & 7 \end{pmatrix}$

5.  **Symmetric Matrix:**
    *   **Definition:** A square matrix $A$ such that $A = A^T$ (where $A^T$ is the transpose, explained below). This means the entries are symmetric across the main diagonal ($a_{ij} = a_{ji}$).
    *   **Why it's special:** Symmetric matrices have many beautiful properties, especially concerning their eigenvalues and eigenvectors, which we'll explore later. They appear frequently in physics and optimization.
    *   **Example:**
        $S = \begin{pmatrix} 1 & 2 & 3 \\ 2 & 4 & 5 \\ 3 & 5 & 6 \end{pmatrix}$

6.  **Triangular Matrix (Upper and Lower):**
    *   **Definition:**
        *   **Upper Triangular:** A square matrix where all entries *below* the main diagonal are zero.
        *   **Lower Triangular:** A square matrix where all entries *above* the main diagonal are zero.
    *   **Why it's special:** Gaussian Elimination transforms matrices into upper triangular form, which makes solving systems of equations very easy via back-substitution.
    *   **Example:**
        Upper Triangular: $U = \begin{pmatrix} 1 & 2 & 3 \\ 0 & 4 & 5 \\ 0 & 0 & 6 \end{pmatrix}$
        Lower Triangular: $L = \begin{pmatrix} 1 & 0 & 0 \\ 2 & 3 & 0 \\ 4 & 5 & 6 \end{pmatrix}$

---

### Operations on Matrices

Just like numbers, we can perform operations on matrices. But be careful, matrix arithmetic has its own rules!

1.  **Matrix Addition and Subtraction:**
    *   **Rule:** You can only add or subtract matrices if they have the **exact same dimensions** ($m \times n$). You simply add or subtract corresponding entries.
    *   **Example:**
        Let $A = \begin{pmatrix} 1 & 2 \\ 3 & 4 \end{pmatrix}$ and $B = \begin{pmatrix} 5 & 6 \\ 7 & 8 \end{pmatrix}$.
        $A + B = \begin{pmatrix} 1+5 & 2+6 \\ 3+7 & 4+8 \end{pmatrix} = \begin{pmatrix} 6 & 8 \\ 10 & 12 \end{pmatrix}$
        $A - B = \begin{pmatrix} 1-5 & 2-6 \\ 3-7 & 4-8 \end{pmatrix} = \begin{pmatrix} -4 & -4 \\ -4 & -4 \end{pmatrix}$

2.  **Scalar Multiplication:**
    *   **Rule:** To multiply a matrix by a scalar (a single number), you multiply *every entry* in the matrix by that scalar.
    *   **Example:**
        Let $A = \begin{pmatrix} 1 & 2 \\ 3 & 4 \end{pmatrix}$ and $c = 3$.
        $c A = 3 \begin{pmatrix} 1 & 2 \\ 3 & 4 \end{pmatrix} = \begin{pmatrix} 3 \times 1 & 3 \times 2 \\ 3 \times 3 & 3 \times 4 \end{pmatrix} = \begin{pmatrix} 3 & 6 \\ 9 & 12 \end{pmatrix}$

3.  **Matrix Multiplication ($AB$):**
    *   **Rule:** This is the most complex, but also the most important operation! To multiply two matrices $A$ and $B$ to get $C = AB$:
        *   The number of **columns in $A$ must equal the number of rows in $B$**.
        *   If $A$ is $m \times n$ and $B$ is $n \times p$, then the resulting matrix $C$ will be $m \times p$.
        *   The entry $c_{ij}$ (in row $i$, column $j$ of $C$) is found by taking the **dot product of row $i$ of $A$ with column $j$ of $B$**.

    *   **Example:**
        Let $A = \begin{pmatrix} 1 & 2 \\ 3 & 4 \end{pmatrix}$ (a $2 \times 2$ matrix) and $B = \begin{pmatrix} 5 & 6 \\ 7 & 8 \end{pmatrix}$ (a $2 \times 2$ matrix).
        The result $C$ will be a $2 \times 2$ matrix.

        $c_{11}$ (row 1 of A $\cdot$ column 1 of B) $= (1)(5) + (2)(7) = 5 + 14 = 19$
        $c_{12}$ (row 1 of A $\cdot$ column 2 of B) $= (1)(6) + (2)(8) = 6 + 16 = 22$
        $c_{21}$ (row 2 of A $\cdot$ column 1 of B) $= (3)(5) + (4)(7) = 15 + 28 = 43$
        $c_{22}$ (row 2 of A $\cdot$ column 2 of B) $= (3)(6) + (4)(8) = 18 + 32 = 50$

        So, $AB = \begin{pmatrix} 19 & 22 \\ 43 & 50 \end{pmatrix}$.

    *   **Important Note: Matrix multiplication is NOT commutative!** In general, $AB \neq BA$.
        Let's check $BA$ for the example above:
        $BA = \begin{pmatrix} 5 & 6 \\ 7 & 8 \end{pmatrix} \begin{pmatrix} 1 & 2 \\ 3 & 4 \end{pmatrix}$
        $b_{11} = (5)(1) + (6)(3) = 5 + 18 = 23$
        $b_{12} = (5)(2) + (6)(4) = 10 + 24 = 34$
        $b_{21} = (7)(1) + (8)(3) = 7 + 24 = 31$
        $b_{22} = (7)(2) + (8)(4) = 14 + 32 = 46$
        So, $BA = \begin{pmatrix} 23 & 34 \\ 31 & 46 \end{pmatrix}$. Clearly, $AB \neq BA$.

    *   **The Column Picture of Matrix Multiplication:**
        Just like $Ax$ is a linear combination of the columns of $A$, $AB$ can be seen as:
        *   Each column of $AB$ is $A$ multiplied by the corresponding column of $B$.
        *   If $B = \begin{pmatrix} | & | & | \\ b_1 & b_2 & \dots & b_p \\ | & | & | \end{pmatrix}$, then $AB = \begin{pmatrix} | & | & | \\ Ab_1 & Ab_2 & \dots & Ab_p \\ | & | & | \end{pmatrix}$.
        This means each column of $AB$ is a linear combination of the columns of $A$. This perspective is incredibly powerful for understanding transformations!

4.  **Transpose ($A^T$):**
    *   **Definition:** The transpose of a matrix $A$ is obtained by swapping its rows and columns. The entry $a_{ij}$ in $A$ becomes $a_{ji}$ in $A^T$.
    *   **Rule:** If $A$ is an $m \times n$ matrix, then $A^T$ is an $n \times m$ matrix.
    *   **Example:**
        Let $A = \begin{pmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \end{pmatrix}$ (a $2 \times 3$ matrix).
        Then $A^T = \begin{pmatrix} 1 & 4 \\ 2 & 5 \\ 3 & 6 \end{pmatrix}$ (a $3 \times 2$ matrix).

        For a square matrix:
        Let $S = \begin{pmatrix} 1 & 2 \\ 3 & 4 \end{pmatrix}$. Then $S^T = \begin{pmatrix} 1 & 3 \\ 2 & 4 \end{pmatrix}$.

---

These matrix types and operations are the building blocks for everything else we'll do. Understanding them thoroughly is crucial. Especially matrix multiplication – it's the engine of linear transformations!

Are you ready to move on to the grand concepts of Vector Spaces and Subspaces, or would you like more examples on any of these operations?