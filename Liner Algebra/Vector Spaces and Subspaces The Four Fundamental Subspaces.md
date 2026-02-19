### Recap: Vector Spaces and Subspaces

Imagine a "vector space" as a universe where vectors live. For example, $\mathbb{R}^3$ is the space of all 3D vectors. A "subspace" is like a smaller, self-contained universe within the larger one. It must always contain the zero vector, and if you add any two vectors from the subspace, or multiply a vector by a scalar, the result must still be in that subspace.

### The Four Fundamental Subspaces of a Matrix $A$

For any matrix $A$ (let's say it's an $m \times n$ matrix, meaning $m$ rows and $n$ columns), there are four special subspaces associated with it. These subspaces reveal a lot about how the matrix transforms vectors.

Let's use a new example matrix $A$:
$A = \begin{bmatrix} 1 & 2 & 3 \\ 2 & 4 & 6 \end{bmatrix}$

Here, $m=2$ (rows) and $n=3$ (columns).

---

#### 1. Column Space $C(A)$
-   **What it is**: The set of all possible output vectors $\mathbf{b}$ you can get when you multiply $A$ by *any* vector $\mathbf{x}$ (i.e., $A\mathbf{x} = \mathbf{b}$). It's also the span of the columns of $A$.
-   **Where it lives**: In $\mathbb{R}^m$ (the space of the output vectors).
-   **How to find it (and its dimension)**: Look at the columns of $A$. Find a basis for the space they span. The number of vectors in this basis is the dimension, which is the **rank ($r$)** of the matrix.

**Example for $A = \begin{bmatrix} 1 & 2 & 3 \\ 2 & 4 & 6 \end{bmatrix}$**:
The columns are $\begin{bmatrix} 1 \\ 2 \end{bmatrix}$, $\begin{bmatrix} 2 \\ 4 \end{bmatrix}$, and $\begin{bmatrix} 3 \\ 6 \end{bmatrix}$.
Notice that:
-   $\begin{bmatrix} 2 \\ 4 \end{bmatrix} = 2 \times \begin{bmatrix} 1 \\ 2 \end{bmatrix}$
-   $\begin{bmatrix} 3 \\ 6 \end{bmatrix} = 3 \times \begin{bmatrix} 1 \\ 2 \end{bmatrix}$
All columns are multiples of the first column. So, the column space is just the line spanned by $\begin{bmatrix} 1 \\ 2 \end{bmatrix}$.
$C(A) = \text{span}\left\{ \begin{bmatrix} 1 \\ 2 \end{bmatrix} \right\}$
-   **Dimension**: $r = 1$. (The rank is 1 because there's only one linearly independent column).

---

#### 2. Nullspace $N(A)$
-   **What it is**: The set of all input vectors $\mathbf{x}$ that $A$ transforms into the zero vector (i.e., $A\mathbf{x} = \mathbf{0}$). These are the vectors that "disappear" when multiplied by $A$.
-   **Where it lives**: In $\mathbb{R}^n$ (the space of the input vectors).
-   **How to find it (and its dimension)**: Solve the equation $A\mathbf{x} = \mathbf{0}$. The number of "free variables" in the solution gives you the dimension. This dimension is $n - r$.

**Example for $A = \begin{bmatrix} 1 & 2 & 3 \\ 2 & 4 & 6 \end{bmatrix}$**:
We solve $A\mathbf{x} = \mathbf{0}$:
$\begin{bmatrix} 1 & 2 & 3 \\ 2 & 4 & 6 \end{bmatrix} \begin{bmatrix} x_1 \\ x_2 \\ x_3 \end{bmatrix} = \begin{bmatrix} 0 \\ 0 \end{bmatrix}$
Row reducing $A$:
$\begin{bmatrix} 1 & 2 & 3 \\ 0 & 0 & 0 \end{bmatrix}$ (Subtract 2 times Row 1 from Row 2)
The equation becomes $x_1 + 2x_2 + 3x_3 = 0$.
Here, $x_1$ is a pivot variable, and $x_2, x_3$ are free variables.
Let $x_2 = s$ and $x_3 = t$.
Then $x_1 = -2s - 3t$.
So, $\mathbf{x} = \begin{bmatrix} -2s - 3t \\ s \\ t \end{bmatrix} = s \begin{bmatrix} -2 \\ 1 \\ 0 \end{bmatrix} + t \begin{bmatrix} -3 \\ 0 \\ 1 \end{bmatrix}$.
$N(A) = \text{span}\left\{ \begin{bmatrix} -2 \\ 1 \\ 0 \end{bmatrix}, \begin{bmatrix} -3 \\ 0 \\ 1 \end{bmatrix} \right\}$
-   **Dimension**: $n - r = 3 - 1 = 2$. (There are two free variables, so the nullspace is a 2D plane in $\mathbb{R}^3$).

---

#### 3. Row Space $C(A^T)$
-   **What it is**: The set of all linear combinations of the rows of $A$. It's essentially the column space of $A^T$ (the transpose of $A$).
-   **Where it lives**: In $\mathbb{R}^n$ (the space of the input vectors).
-   **How to find it (and its dimension)**: Look at the rows of $A$. Find a basis for the space they span. The number of vectors in this basis is also the **rank ($r$)** of the matrix. (The dimension of the row space is always equal to the dimension of the column space).

**Example for $A = \begin{bmatrix} 1 & 2 & 3 \\ 2 & 4 & 6 \end{bmatrix}$**:
The rows are $(1, 2, 3)$ and $(2, 4, 6)$.
Notice that $(2, 4, 6) = 2 \times (1, 2, 3)$.
So, the row space is just the line spanned by $(1, 2, 3)$.
$C(A^T) = \text{span}\left\{ \begin{bmatrix} 1 \\ 2 \\ 3 \end{bmatrix} \right\}$ (writing it as a column vector for consistency with column space notation, but it's a subspace of $\mathbb{R}^3$).
-   **Dimension**: $r = 1$.

---

#### 4. Left Nullspace $N(A^T)$
-   **What it is**: The set of all vectors $\mathbf{y}$ such that $A^T\mathbf{y} = \mathbf{0}$. It's the nullspace of the transpose matrix. It's called "left" nullspace because if you write it as $\mathbf{y}^T A = \mathbf{0}^T$, the $\mathbf{y}^T$ is on the left.
-   **Where it lives**: In $\mathbb{R}^m$ (the space of the output vectors).
-   **How to find it (and its dimension)**: Solve $A^T\mathbf{y} = \mathbf{0}$. The dimension is $m - r$.

**Example for $A = \begin{bmatrix} 1 & 2 & 3 \\ 2 & 4 & 6 \end{bmatrix}$**:
First, find $A^T$:
$A^T = \begin{bmatrix} 1 & 2 \\ 2 & 4 \\ 3 & 6 \end{bmatrix}$
Now solve $A^T\mathbf{y} = \mathbf{0}$:
$\begin{bmatrix} 1 & 2 \\ 2 & 4 \\ 3 & 6 \end{bmatrix} \begin{bmatrix} y_1 \\ y_2 \end{bmatrix} = \begin{bmatrix} 0 \\ 0 \\ 0 \end{bmatrix}$
Row reducing $A^T$:
$\begin{bmatrix} 1 & 2 \\ 0 & 0 \\ 0 & 0 \end{bmatrix}$ (Subtract 2 times Row 1 from Row 2, and 3 times Row 1 from Row 3)
The equation becomes $y_1 + 2y_2 = 0$.
Let $y_2 = s$. Then $y_1 = -2s$.
So, $\mathbf{y} = \begin{bmatrix} -2s \\ s \end{bmatrix} = s \begin{bmatrix} -2 \\ 1 \end{bmatrix}$.
$N(A^T) = \text{span}\left\{ \begin{bmatrix} -2 \\ 1 \end{bmatrix} \right\}$
-   **Dimension**: $m - r = 2 - 1 = 1$.

---

### The Big Picture: Orthogonality

The most profound insight from these four subspaces is their relationship:
-   The **Row Space** $C(A^T)$ and the **Nullspace** $N(A)$ are orthogonal complements in $\mathbb{R}^n$. This means every vector in the row space is perpendicular to every vector in the nullspace, and together they span all of $\mathbb{R}^n$.
    -   In our example: $\text{span}\left\{ \begin{bmatrix} 1 \\ 2 \\ 3 \end{bmatrix} \right\}$ is orthogonal to $\text{span}\left\{ \begin{bmatrix} -2 \\ 1 \\ 0 \end{bmatrix}, \begin{bmatrix} -3 \\ 0 \\ 1 \end{bmatrix} \right\}$.
        -   Check: $(1,2,3) \cdot (-2,1,0) = -2+2+0 = 0$
        -   Check: $(1,2,3) \cdot (-3,0,1) = -3+0+3 = 0$
-   The **Column Space** $C(A)$ and the **Left Nullspace** $N(A^T)$ are orthogonal complements in $\mathbb{R}^m$.
    -   In our example: $\text{span}\left\{ \begin{bmatrix} 1 \\ 2 \end{bmatrix} \right\}$ is orthogonal to $\text{span}\left\{ \begin{bmatrix} -2 \\ 1 \end{bmatrix} \right\}$.
        -   Check: $(1,2) \cdot (-2,1) = -2+2 = 0$

This orthogonality is fundamental to understanding many concepts in linear algebra, including projections, least squares, and the Singular Value Decomposition.

Feel free to ask if you'd like to explore any of these subspaces or their relationships in more detail! You can add these insights to your note [[Vector Spaces and Subspaces The Four Fundamental Subspaces]].