### 1. The Absolute Basics: Vectors, Matrices, and Systems of Equations

We begin with the fundamental building blocks: **vectors**.

**What is a vector?**

Think of a vector as an arrow in space. It has both a **magnitude** (length) and a **direction**. In linear algebra, we often represent them as lists of numbers, either as a column:

$v = \begin{pmatrix} 3 \\ 1 \end{pmatrix}$

or as a row:

$v^T = \begin{pmatrix} 3 & 1 \end{pmatrix}$

For now, let's stick to column vectors. This vector $v$ could represent a point $(3,1)$ in a 2D plane, or a displacement from the origin to that point.

**How do we add vectors?**

Vector addition is straightforward: you add their corresponding components. Geometrically, it's like placing the tail of the second vector at the head of the first.

Let $u = \begin{pmatrix} 1 \\ 2 \end{pmatrix}$ and $v = \begin{pmatrix} 3 \\ 1 \end{pmatrix}$.

Then $u + v = \begin{pmatrix} 1+3 \\ 2+1 \end{pmatrix} = \begin{pmatrix} 4 \\ 3 \end{pmatrix}$.

Imagine walking 1 unit right and 2 units up (vector $u$), and then from there, walking 3 units right and 1 unit up (vector $v$). Your final position from the start is 4 units right and 3 units up (vector $u+v$). It's the "resultant" vector.

**How do we multiply vectors by scalars?**

A **scalar** is just a single number. When you multiply a vector by a scalar, you scale its length. If the scalar is positive, the direction stays the same. If it's negative, the direction reverses.

Let $v = \begin{pmatrix} 3 \\ 1 \end{pmatrix}$ and let the scalar $c = 2$.

Then $c v = 2 \begin{pmatrix} 3 \\ 1 \end{pmatrix} = \begin{pmatrix} 2 \times 3 \\ 2 \times 1 \end{pmatrix} = \begin{pmatrix} 6 \\ 2 \end{pmatrix}$.

The vector $2v$ points in the same direction as $v$ but is twice as long.

What about $c = -1$? Then $-1 v = \begin{pmatrix} -3 \\ -1 \end{pmatrix}$, which points in the exact opposite direction.

These two operations – vector addition and scalar multiplication – are the bedrock of linear algebra. They allow us to combine vectors to create new ones, forming what we call **linear combinations**. A linear combination of vectors $v_1, v_2, \dots, v_n$ is any expression of the form $c_1 v_1 + c_2 v_2 + \dots + c_n v_n$, where $c_i$ are scalars.

This concept of linear combinations is absolutely central to everything we'll do. It's how we build up spaces, understand solutions, and see the action of matrices.

### What is a Matrix?

Think of a matrix as a rectangular array of numbers. It's a way to organize information, but more profoundly, it's a way to represent a **linear transformation**. A matrix takes a vector and transforms it into another vector. It's an *operator*, an *action*!

A matrix with $m$ rows and $n$ columns is called an $m \times n$ matrix.

**Example:**
Here's a $2 \times 3$ matrix $A$:
$A = \begin{pmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \end{pmatrix}$

And here's a $3 \times 2$ matrix $B$:
$B = \begin{pmatrix} 7 & 8 \\ 9 & 10 \\ 11 & 12 \end{pmatrix}$

### Matrix-Vector Multiplication: $Ax$

This is where matrices come alive! When a matrix $A$ multiplies a vector $x$, it produces a new vector $b$. This operation, $Ax=b$, is the heart of linear algebra.

There are two fundamental ways to look at matrix-vector multiplication, and both are incredibly important.

#### Perspective 1: Row Picture (Dot Products)

You can think of each row of the matrix $A$ as a vector. When you multiply $A$ by $x$, you're taking the dot product of each row of $A$ with the vector $x$.

Let $A = \begin{pmatrix} a_{11} & a_{12} \\ a_{21} & a_{22} \end{pmatrix}$ and $x = \begin{pmatrix} x_1 \\ x_2 \end{pmatrix}$.

Then $Ax = \begin{pmatrix} a_{11} & a_{12} \\ a_{21} & a_{22} \end{pmatrix} \begin{pmatrix} x_1 \\ x_2 \end{pmatrix} = \begin{pmatrix} a_{11}x_1 + a_{12}x_2 \\ a_{21}x_1 + a_{22}x_2 \end{pmatrix}$.

Each component of the resulting vector $b$ is the dot product of a row of $A$ with $x$.

**Example:**
Let $A = \begin{pmatrix} 2 & -1 \\ 1 & 3 \end{pmatrix}$ and $x = \begin{pmatrix} 4 \\ 2 \end{pmatrix}$.

$Ax = \begin{pmatrix} 2 & -1 \\ 1 & 3 \end{pmatrix} \begin{pmatrix} 4 \\ 2 \end{pmatrix} = \begin{pmatrix} (2)(4) + (-1)(2) \\ (1)(4) + (3)(2) \end{pmatrix} = \begin{pmatrix} 8 - 2 \\ 4 + 6 \end{pmatrix} = \begin{pmatrix} 6 \\ 10 \end{pmatrix}$.

So, the matrix $A$ transformed the vector $\begin{pmatrix} 4 \\ 2 \end{pmatrix}$ into $\begin{pmatrix} 6 \\ 10 \end{pmatrix}$.

#### Perspective 2: Column Picture (Linear Combinations of Columns)

This is often the more insightful perspective! When you multiply $A$ by $x$, the result $Ax$ is a **linear combination of the columns of $A$**, where the coefficients are the components of $x$.

Let $A = \begin{pmatrix} | & | \\ v_1 & v_2 \\ | & | \end{pmatrix}$ where $v_1$ and $v_2$ are the column vectors of $A$.
And $x = \begin{pmatrix} x_1 \\ x_2 \end{pmatrix}$.

Then $Ax = x_1 v_1 + x_2 v_2$.

**Example (using the same $A$ and $x$):**
$A = \begin{pmatrix} 2 & -1 \\ 1 & 3 \end{pmatrix}$. Its columns are $v_1 = \begin{pmatrix} 2 \\ 1 \end{pmatrix}$ and $v_2 = \begin{pmatrix} -1 \\ 3 \end{pmatrix}$.
$x = \begin{pmatrix} 4 \\ 2 \end{pmatrix}$.

$Ax = 4 \begin{pmatrix} 2 \\ 1 \end{pmatrix} + 2 \begin{pmatrix} -1 \\ 3 \end{pmatrix} = \begin{pmatrix} 8 \\ 4 \end{pmatrix} + \begin{pmatrix} -2 \\ 6 \end{pmatrix} = \begin{pmatrix} 8-2 \\ 4+6 \end{pmatrix} = \begin{pmatrix} 6 \\ 10 \end{pmatrix}$.

Notice we got the exact same result! This column picture is incredibly powerful because it tells us that $Ax=b$ is asking: "Can $b$ be formed by a linear combination of the columns of $A$?" If so, what are the coefficients ($x_1, x_2, \dots$)?

### Systems of Equations: $Ax = b$

This is the core problem we want to solve! We're given a matrix $A$ and a vector $b$, and we want to find the vector $x$ that satisfies the equation $Ax=b$.

Let's write out a system of linear equations:
$2x_1 - x_2 = 6$
$x_1 + 3x_2 = 10$

We can immediately see this is exactly our $Ax=b$ example from above!
$A = \begin{pmatrix} 2 & -1 \\ 1 & 3 \end{pmatrix}$, $x = \begin{pmatrix} x_1 \\ x_2 \end{pmatrix}$, $b = \begin{pmatrix} 6 \\ 10 \end{pmatrix}$.

Now, let's look at solving this system from our two perspectives:

#### The Row Picture: Intersections of Lines (or Planes)

Each equation in the system represents a line (in 2D) or a plane (in 3D) or a hyperplane (in higher dimensions). The solution $(x_1, x_2)$ is the point where all these lines (or planes) intersect.

For our example:
1.  $2x_1 - x_2 = 6$
2.  $x_1 + 3x_2 = 10$

If you were to graph these two lines, the point where they cross would be our solution $x = \begin{pmatrix} x_1 \\ x_2 \end{pmatrix}$.
Let's find it:
From (1), $x_2 = 2x_1 - 6$.
Substitute into (2): $x_1 + 3(2x_1 - 6) = 10$
$x_1 + 6x_1 - 18 = 10$
$7x_1 = 28$
$x_1 = 4$

Now find $x_2$: $x_2 = 2(4) - 6 = 8 - 6 = 2$.
So the solution is $x = \begin{pmatrix} 4 \\ 2 \end{pmatrix}$. This is the point $(4,2)$ where the two lines intersect.

**Geometric Interpretation:**
-   The first line passes through, for example, $(3,0)$ and $(0,-6)$.
-   The second line passes through, for example, $(10,0)$ and $(1,3)$.
-   They both pass through $(4,2)$.

#### The Column Picture: Linear Combinations of Column Vectors

This perspective asks: Can we find coefficients $x_1$ and $x_2$ such that a linear combination of the columns of $A$ equals $b$?

$x_1 \begin{pmatrix} 2 \\ 1 \end{pmatrix} + x_2 \begin{pmatrix} -1 \\ 3 \end{pmatrix} = \begin{pmatrix} 6 \\ 10 \end{pmatrix}$

**Geometric Interpretation:**
-   We have two column vectors: $v_1 = \begin{pmatrix} 2 \\ 1 \end{pmatrix}$ and $v_2 = \begin{pmatrix} -1 \\ 3 \end{pmatrix}$.
-   We want to know if the target vector $b = \begin{pmatrix} 6 \\ 10 \end{pmatrix}$ can be reached by scaling $v_1$ by $x_1$ and $v_2$ by $x_2$ and then adding them.
-   In our example, we found $x_1=4$ and $x_2=2$. So, $4 \begin{pmatrix} 2 \\ 1 \end{pmatrix} + 2 \begin{pmatrix} -1 \\ 3 \end{pmatrix} = \begin{pmatrix} 8 \\ 4 \end{pmatrix} + \begin{pmatrix} -2 \\ 6 \end{pmatrix} = \begin{pmatrix} 6 \\ 10 \end{pmatrix}$.
-   This means we take 4 steps in the direction of $v_1$ and 2 steps in the direction of $v_2$, and we land exactly on $b$.

The column picture is often more powerful because it directly relates to the idea of **span** – the set of all possible linear combinations of the columns. If $b$ is in the span of the columns of $A$, then a solution $x$ exists.

### Key Tool: Gaussian Elimination
[[Gaussian Elimination]]
Now, how do we systematically find this $x$? For small systems, substitution works, but for larger systems, we need a robust algorithm. That's where **Gaussian Elimination** comes in!

Gaussian Elimination is not just an algorithm; it's a window into the structure of matrices. It's a systematic way to solve $Ax=b$ by transforming the system into an equivalent, simpler form that's easy to solve.

The idea is to use elementary row operations to turn the matrix $A$ into an **upper triangular matrix** (or more generally, a row echelon form). The three allowed row operations are:
1.  Swapping two rows.
2.  Multiplying a row by a non-zero scalar.
3.  Adding a multiple of one row to another row.

These operations don't change the solution set of the system.

Let's take our example:
$2x_1 - x_2 = 6$
$x_1 + 3x_2 = 10$

We write this in an **augmented matrix** form:
$\begin{pmatrix} 2 & -1 & | & 6 \\ 1 & 3 & | & 10 \end{pmatrix}$

**Step 1: Get a '1' in the top-left corner (pivot).**
Swap Row 1 and Row 2 (Operation 1):
$\begin{pmatrix} 1 & 3 & | & 10 \\ 2 & -1 & | & 6 \end{pmatrix}$

**Step 2: Eliminate the entry below the first pivot.**
Subtract 2 times Row 1 from Row 2 (Operation 3):
$R_2 \leftarrow R_2 - 2R_1$
$\begin{pmatrix} 1 & 3 & | & 10 \\ 2 - 2(1) & -1 - 2(3) & | & 6 - 2(10) \end{pmatrix} = \begin{pmatrix} 1 & 3 & | & 10 \\ 0 & -7 & | & -14 \end{pmatrix}$

Now we have an upper triangular matrix! This corresponds to the system:
$x_1 + 3x_2 = 10$
$-7x_2 = -14$

**Step 3: Back-substitution.**
From the second equation:
$-7x_2 = -14 \implies x_2 = 2$.

Substitute $x_2=2$ into the first equation:
$x_1 + 3(2) = 10$
$x_1 + 6 = 10$
$x_1 = 4$.

So, $x = \begin{pmatrix} 4 \\ 2 \end{pmatrix}$, just as we found before!

**Pivots and Free Variables:**
-   The **pivots** are the first non-zero entries in each row after Gaussian elimination (in our example, the '1' in the first row and the '-7' in the second row). These correspond to variables that are uniquely determined.
-   Sometimes, you might end up with a column that doesn't have a pivot. The variables corresponding to these columns are called **free variables**. They can take any value, leading to infinitely many solutions. If you get a row like $\begin{pmatrix} 0 & 0 & | & 5 \end{pmatrix}$, it means $0=5$, which is impossible, indicating **no solution**.

Gaussian Elimination is the workhorse for understanding when solutions exist, how many there are, and how to find them. It's the foundation for understanding concepts like rank, null space, and invertibility, which we'll get to soon!

How does that feel for a start? We've covered vectors, matrices, matrix-vector multiplication from two crucial perspectives, and the fundamental method for solving systems. Ready for more?