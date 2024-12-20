+++
title = 'Expressing LU factorization using rank one matrices'
date = 2022-05-02T00:00:00+00:00
+++

In the [last article](/posts/2022-01-03-matrix-multiplication-rank-one/), we looked at expressing matrix multiplication using rank one matrices. In this article, we will study LU factorization using rank one matrices.

## Prerequisites
We are assuming that you are familiar with linear algebra corresponding to a typical undergrad course. In particular, we are assuming that you are familiar with Gaussian elimination and LU factorization.

## Running example
In LU factorization, we take a square matrix and turn it into an upper triangular matrix. As our running example, we will look at the following matrix. Let's turn it into an upper triangular matrix using a sequence of row operations.

<div>
  $$
  \begin{align*}
    A = \begin{bmatrix}
      1 & -1 & -1 & 1 \\
      0 & -1 & -1 & 0 \\
      3 & -3 & -2 & 4 \\
      2 & 0 & 2 & 0
    \end{bmatrix}.
  \end{align*}
  $$
</div>

The sequence of row operations along with the intermediate matrices are as follows.

<div>
  $$
  \begin{align*}
    &\begin{bmatrix}
      1 & -1 & -1 & 1 \\
      0 & -1 & -1 & 0 \\
      3 & -3 & -2 & 4 \\
      2 & 0 & 2 & 0
    \end{bmatrix} \\ \\
    \overset{row2 = row2 + 0.row1}{\longrightarrow}
    &\begin{bmatrix}
      1 & -1 & -1 & 1 \\
      0 & -1 & -1 & 0 \\
      3 & -3 & -2 & 4 \\
      2 & 0 & 2 & 0
    \end{bmatrix} \\ \\
    \overset{row3 = row3 + (-3).row1}{\longrightarrow}
    &\begin{bmatrix}
      1 & -1 & -1 & 1 \\
      0 & -1 & -1 & 0 \\
      0 & 0 & 1 & 1 \\
      2 & 0 & 2 & 0
    \end{bmatrix} \\ \\
    \overset{row4 = row4 + (-2).row1}{\longrightarrow}
    &\begin{bmatrix}
      1 & -1 & -1 & 1 \\
      0 & -1 & -1 & 0 \\
      0 & 0 & 1 & 1 \\
      0 & 2 & 4 & -2
    \end{bmatrix} \\ \\
    \overset{row3 = row3 + 0.row2}{\longrightarrow}
    &\begin{bmatrix}
      1 & -1 & -1 & 0 \\
      0 & -1 & -1 & 0 \\
      0 & 0 & 1 & 1 \\
      0 & 2 & 4 & -2
    \end{bmatrix} \\ \\
    \overset{row4 = row4 + 2.row2}{\longrightarrow}
    &\begin{bmatrix}
      1 & -1 & -1 & 1 \\
      0 & -1 & -1 & 0 \\
      0 & 0 & 1 & 1 \\
      0 & 0 & 2 & -2
    \end{bmatrix} \\ \\
    \overset{row4 = row4 + (-2).row3}{\longrightarrow}
    &\begin{bmatrix}
      1 & -1 & -1 & 1 \\
      0 & -1 & -1 & 0 \\
      0 & 0 & 1 & 1 \\
      0 & 0 & 0 & -4
    \end{bmatrix}.
  \end{align*}
  $$
</div>

We reduced a square matrix into an upper triangular matrix:

<div>
  $$
  \begin{align*}
    \begin{bmatrix}
      1 & -1 & -1 & 1 \\
      0 & -1 & -1 & 0 \\
      3 & -3 & -2 & 4 \\
      2 & 0 & 2 & 0
    \end{bmatrix}
    \rightarrow
    \begin{bmatrix}
      1 & -1 & -1 & 1 \\
      0 & -1 & -1 & 0 \\
      0 & 0 & 1 & 1 \\
      0 & 0 & 0 & -4
    \end{bmatrix}
  \end{align*}
  $$
</div>
or
<div>
  $$
    A \rightarrow U.
  $$
</div>

We can express the above sequence of operations using the following matrix multiplication:

<div>
  $$
  \begin{align*}
    \begin{bmatrix}
      1 & -1 & -1 & 1 \\
      0 & -1 & -1 & 0 \\
      3 & -3 & -2 & 4 \\
      2 & 0 & 2 & 0
    \end{bmatrix}
    =
    \begin{bmatrix}
      1 & 0 & 0 & 0 \\
      0 & 1 & 0 & 0 \\
      3 & 0 & 1 & 0 \\
      2 & -2 & 2 & 1
    \end{bmatrix}
    \begin{bmatrix}
      1 & -1 & -1 & 1 \\
      0 & -1 & -1 & 0 \\
      0 & 0 & 1 & 1 \\
      0 & 0 & 0 & -4
    \end{bmatrix}
  \end{align*}
  $$
  <p>or</p>
  $$
  \begin{align*}
    A = LU.
  \end{align*}
  $$
</div>

That was LU factorization using row operations. We will now look at an alternative way that uses rank one matrices.

## Computing rank one components
When we say that we want to express LU factorization using rank one matrices, we are saying that we want to achieve

<div>
  $$
  \begin{align*}
    A = LU = \sum_{k=1}^{n} l_ku_k,
  \end{align*}
  $$
</div>

<div>
  <p>
    where each summation term, $l_ku_k$, is of rank one.
  </p>
  <p>
   Our strategy is going to be to succesively extract rank one components from a matrix and reduce it's rank on each extraction till we are left with a rank zero matrix. For example, if we denote a rank n matrix by $A_{n}$, where the subscript denotes the matrix rank, it would look like this.
  </p>
  $$
  \begin{align*}
    & \space A_{n} \\
    = & \space l_1u_1 + A_{n - 1} \\
    = & \space l_1u_1 + l_2u_2 + A_{n - 2} \\
    & \space ... \\
    = & \space \sum_{k=1}^{n} l_ku_k.
  \end{align*}
  $$
</div>

To achieve this, we will look at the row operations that we did above and reorganize them so that they extract rank one components. The row operations in the above example are these.

```python
row2 = row2 + 0.row1
row3 = row3 + (-3).row1
row4 = row4 + (-2).row1
row3 = row3 + 0.row2
row4 = row4 + 2.row2
row4 = row4 + (-2).row3
```

In each of these operations, we use a pivot row to manipulate the rows below the pivot rows. For example, in the first three instructions, the pivot row is `row1`, in the next two instructions, the pivot row is `row2` and in the last instruction, the pivot row is `row3`.

We can group these instructions by the pivot row involved. That is, we can group them by the second operand of addition. This results in the following groups.

Group1:
```python
row2 = row2 + 0.row1
row3 = row3 + (-3).row1
row4 = row4 + (-2).row1
```

Group2:
```python
row3 = row3 + 0.row2
row4 = row4 + 2.row2
```

Group3:
```python
row4 = row4 + (-2).row3
```

As we will see, each of these groups is going to lead to the extraction of a rank one component. Our other requirement was to reduce the rank of the matrix by one on each rank one extraction. To achieve that, we turn the pivot row into zero. This results in the following four groups.

Group1:
```python
row1 = row1 + (-1).row1
row2 = row2 + 0.row1
row3 = row3 + (-3).row1
row4 = row4 + (-2).row1
```

Group2:
```python
row2 = row2 + (-1).row2
row3 = row3 + 0.row2
row4 = row4 + 2.row2
```

Group3:
```python
row3 = row3 + (-1).row3
row4 = row4 + (-2).row3
```

Group4:
```python
row4 = row4 + (-1).row4
```

Let's apply these operations to our example matrix group by group. After applying group1, the result is

<div>
  $$
  \begin{align*}
    \begin{bmatrix}
      1 & -1 & -1 & 1 \\
      0 & -1 & -1 & 0 \\
      3 & -3 & -2 & 4 \\
      2 & 0 & 2 & 0
    \end{bmatrix}
    \overset{group1}\longrightarrow
    \begin{bmatrix}
      0 & 0 & 0 & 0 \\
      0 & -1 & -1 & 0 \\
      0 & 0 & 1 & 1 \\
      0 & 2 & 4 & -2
    \end{bmatrix}.
  \end{align*}
  $$
</div>

<div>
  <p>
    Let's denote the matrix on the right hand side of the arrow by $A_{group1}$, where $A_{group1}$ stands for "$A$ after group1".
  </p>
</div>

<div>
  $$
  \begin{align*}
    A \longrightarrow A_{group1}.
  \end{align*}
  $$
</div>

<div>
  <p>
    We can express the above transformation using the following matrix addition,
  </p>
  $$
  \begin{align*}
    & A = A_{group1} + O_{group1} \\
    \implies & O_{group1} = A - A_{group1},
  \end{align*}
  $$
  <p>
    where $O_{group1}$ stands for "operations of group1".
  </p>
</div>

<div>
  <p>Calculating $O_{group1}$ using $A$ and $A_{group1}$, we have</p>
  $$
  \begin{align*}
    O_{group1} = 
    & \begin{bmatrix}
      1 & -1 & -1 & 1 \\
      0 & 0 & 0 & 0 \\
      3 & -3 & -3 & 3 \\
      2 & -2 & -2 & 2
      \end{bmatrix}.
  \end{align*}
  $$
</div>

<div>
  <p>
    Notice that $O_{group1}$ is a rank one matrix. All of its rows are multiples of the first row. We can therefore express $O_{group1}$ in rank one form.
  </p>
  $$
  \begin{align*}
    O_{group1} = l_1u_1 =
    \begin{bmatrix}
      1 \\
      0 \\
      3 \\
      2
    \end{bmatrix}
    \begin{bmatrix}
      1 & -1 & -1 & 1
    \end{bmatrix}.
  \end{align*}
  $$
</div>

<div>
  We can thus write $A = A_{group1} + O_{group1}$ as
  $$
  \begin{align*}
    A = l_1u_1 + A_{group1}.
  \end{align*}
  $$
</div>

Now let's apply group2 operations which we repeat below.

```python
row2 = row2 + (-1).row2
row3 = row3 + 0.row2
row4 = row4 + 2.row2
```

<div>
  <p>
    We will apply group2 operations to $A_{group1}$. Doing that results in
  </p>
</div>

<div>
  $$
  \begin{align*}
    & \begin{bmatrix}
      0 & 0 & 0 & 0 \\
      0 & -1 & -1 & 0 \\
      0 & 0 & 1 & 1 \\
      0 & 2 & 4 & -2
    \end{bmatrix}
    \overset{group2}\longrightarrow
    \begin{bmatrix}
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & 0 \\
      0 & 0 & 1 & 1 \\
      0 & 0 & 2 & -2
    \end{bmatrix} \\
    \implies
    & A_{group1} \rightarrow A_{group2} \\
    \implies
    & A_{group1} = A_{group2} + O_{group2} \\
    \implies
    & O_{group2} = A_{group1} - A_{group2}.
  \end{align*}  
  $$

  <p>
    Evaluating $O_{group2}$, we have
  </p>
  $$
  \begin{align*}  
    & O_{group2} =
    \begin{bmatrix}
      0 & 0 & 0 & 0 \\
      0 & -1 & -1 & 0 \\
      0 & 0 & 0 & 0 \\
      0 & 2 & 2 & 0
    \end{bmatrix} \\
    & O_{group2} = l_2u_2 =
    \begin{bmatrix}
      0 \\
      1 \\
      0 \\
      -2
    \end{bmatrix}
    \begin{bmatrix}
      0 & -1 & -1 & 0
    \end{bmatrix}.
  \end{align*}
  $$

  <p>
    Earlier, we determined that $A = l_1u_1 + A_{group1}$. Substituting $A_{group1} = l_2u_2 + A_{group2}$, we have
  </p>
  $$
  \begin{align*}
    & A = l_1u_1 + l_2u_2 + A_{group2}.
  \end{align*}
  $$

  <p>
    Let's now apply group3 operations to $A_{group2}$. The group3 operations are these.
  </p>
</div>

```python
row3 = row3 + (-1).row3
row4 = row4 + (-2).row3
```

This results in
<div>
  $$
  \begin{align*}
    & \begin{bmatrix}
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & 0 \\
      0 & 0 & 1 & 1 \\
      0 & 0 & 2 & -2
    \end{bmatrix}
    \overset{group3}\longrightarrow
    \begin{bmatrix}
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & -4
    \end{bmatrix} \\
    \implies
    & A_{group2} = O_{group3} + A_{group3} \\
    \implies
    & O_{group2} = A_{group2} - A_{group3} \\
    \implies
    & O_{group3} =
    \begin{bmatrix}
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & 0 \\
      0 & 0 & 1 & 1 \\
      0 & 0 & 2 & 2
    \end{bmatrix} \\
    \implies
    & O_{group3} = l_3u_3 =
    \begin{bmatrix}
      0 \\
      0 \\
      1 \\
      2 \\
    \end{bmatrix}
    \begin{bmatrix}
      0 & 0 & 1 & 1
    \end{bmatrix}.
  \end{align*}
  $$
</div>

The situation now looks like

<div>
  $$
  \begin{align*}
    A = l_1u_1 + l_2u_2 + l_3u_3 + A_{group3}.
  \end{align*}
  $$

  Finally, $A_{group3}$ is
  $$
  \begin{align*}
    A_{group3} = \begin{bmatrix}
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & -4
    \end{bmatrix}.
  \end{align*}
  $$
</div>

The group4 operations is a single operation.

```python
row4 = row4 + (-1).row4
```

<div>
  Applying group4 to $A_{group3}$ results in
  $$
  \begin{align*}
    & \begin{bmatrix}
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & -4
    \end{bmatrix}
    \overset{group4}\longrightarrow
    \begin{bmatrix}
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & 0
    \end{bmatrix} \\
    \implies
    & A_{group3} = O_{group4} + A_{group4} \\
    \implies
    & O_{group4} = A_{group3} - A_{group4} \\
    \implies
    & O_{group4} = \begin{bmatrix}
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & -4
    \end{bmatrix} \\
    \implies
    & O_{group4} = l_4u_4 =
    \begin{bmatrix}
      0 \\
      0 \\
      0 \\
      1
    \end{bmatrix}
    \begin{bmatrix}
      0 & 0 & 0 & -4
    \end{bmatrix}.
  \end{align*}
  $$

  <p>
    After applying all four groups, it looks like
  </p>
  $$
  \begin{align*}
    A = l_1u_1 + l_2u_2 + l_3u_3 + l_4u_4 + A_{group4}.
  \end{align*}
  $$
  <p>
    Since $A_{group4}$ is a zero matrix, we have
  </p>
  $$
  \begin{align*}
    A = l_1u_1 + l_2u_2 + l_3u_3 + l_4u_4
  \end{align*}
  $$
  $$
  \begin{align*}
    \implies
    A =
    \begin{bmatrix}
      1 \\
      0 \\
      3 \\
      2
    \end{bmatrix}
    \begin{bmatrix}
      1 & -1 & -1 & 1
    \end{bmatrix} +
    \begin{bmatrix}
      0 \\
      1 \\
      0 \\
      -2
    \end{bmatrix}
    \begin{bmatrix}
      0 & -1 & -1 & 0
    \end{bmatrix} +
    \begin{bmatrix}
      0 \\
      0 \\
      1 \\
      2 \\
    \end{bmatrix}
    \begin{bmatrix}
      0 & 0 & 1 & 1
    \end{bmatrix} +
    \begin{bmatrix}
      0 \\
      0 \\
      0 \\
      1
    \end{bmatrix}
    \begin{bmatrix}
      0 & 0 & 0 & -4
    \end{bmatrix}.
  \end{align*}
  $$
  <p>Using rank one multiplication, we have the answer</p>
  $$
  \begin{align*}
    A =
    \begin{bmatrix}
      1 & 0 & 0 & 0 \\
      0 & 1 & 0 & 0 \\
      3 & 0 & 1 & 0 \\
      2 & -2 & 2 & 1
    \end{bmatrix}
    \begin{bmatrix}
      1 & -1 & -1 & 1 \\
      0 & -1 & -1 & 0 \\
      0 & 0 & 1 & 1 \\
      0 & 0 & 0 & -4
    \end{bmatrix}.
  \end{align*}
  $$
</div>

## Algorithm
<div>
  <p>
    In our above discussion, we reused the existing row operations to extract rank one components. Let's try to come up with the $l_ku_k$ components in a different way.
  </p>
  <p>
    What happens when we extract $l_1u_1$ from our matrix $A$? The original matrix was
  </p>
  $$
  \begin{align*}
    A = \begin{bmatrix}
      1 & -1 & -1 & 1 \\
      0 & -1 & -1 & 0 \\
      3 & -3 & -2 & 4 \\
      2 & 0 & 2 & 0
    \end{bmatrix}.
  \end{align*}
  $$
  <p>
    After subtracting $l_1u_1$ from A, we need zeroes on the pivot row and on the pivot column. In LU factorization, this is done using the pivot row, which is currently the first row of A. What we want is
  </p>
  $$
  \begin{align*}
    A =
    \begin{bmatrix}
      a \\
      b \\
      c \\
      d
    \end{bmatrix}
    \begin{bmatrix}
      1 & -1 & -1 & 1
    \end{bmatrix} +
    \begin{bmatrix}
      0 & 0 & 0 & 0 \\
      0 & -1 & -1 & 0 \\
      0 & 0 & 1 & 1 \\
      0 & 2 & 4 & -2
    \end{bmatrix},
  \end{align*}
  $$
  <p>
    that is, we need the right multipliers $a$, $b$, $c$, and $d$. We can get these numbers by dividing the first column entries by the pivot. So that looks like
  </p>
  $$
  \begin{align*}
    A & =
    \begin{bmatrix}
      1 / 1 \\
      0 / 1 \\
      3 / 1 \\
      2 / 1
    \end{bmatrix}
    \begin{bmatrix}
      1 & -1 & -1 & 1
    \end{bmatrix} +
    \begin{bmatrix}
      0 & 0 & 0 & 0 \\
      0 & -1 & -1 & 0 \\
      0 & 0 & 1 & 1 \\
      0 & 2 & 4 & -2
    \end{bmatrix} \\ \\
    & =
    \begin{bmatrix}
      1 \\
      0 \\
      3 \\
      2 \\
    \end{bmatrix}
    \begin{bmatrix}
      1 & -1 & -1 & 1
    \end{bmatrix} +
    \begin{bmatrix}
      0 & 0 & 0 & 0 \\
      0 & -1 & -1 & 0 \\
      0 & 0 & 1 & 1 \\
      0 & 2 & 4 & -2
    \end{bmatrix}.
  \end{align*}
  $$
  <p>
    The first term here is $l_1u_1$. We now need to extract a rank one component from the second matrix such that its second row and its second row column is zero. We use the second pivot row to do it. It looks like
  </p>
  $$
  \begin{align*}
     A & =
    \begin{bmatrix}
      1 \\
      0 \\
      3 \\
      2 \\
    \end{bmatrix}
    \begin{bmatrix}
      1 & -1 & -1 & 1
    \end{bmatrix} +
    \begin{bmatrix}
      a \\
      b \\
      c \\
      d
    \end{bmatrix}
    \begin{bmatrix}
      0 & -1 & -1 & 0
    \end{bmatrix} +
    \begin{bmatrix}
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & 0 \\
      0 & 0 & 1 & 1 \\
      0 & 0 & 2 & -2
    \end{bmatrix}.
  \end{align*}
  $$
  <p>
    The second pivot is -1 so dividing entries in second column by -1 gives us $a$, $b$, $c$, and $d$.
  </p>
  $$
  \begin{align*}
     A & =
    \begin{bmatrix}
      1 \\
      0 \\
      3 \\
      2 \\
    \end{bmatrix}
    \begin{bmatrix}
      1 & -1 & -1 & 1
    \end{bmatrix} +
    \begin{bmatrix}
      0 / -1 \\
      -1 / -1 \\
      0 / -1 \\
      2 / -1
    \end{bmatrix}
    \begin{bmatrix}
      0 & -1 & -1 & 0
    \end{bmatrix} +
    \begin{bmatrix}
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & 0 \\
      0 & 0 & 1 & 1 \\
      0 & 0 & 2 & -2
    \end{bmatrix} \\ \\
    & =
    \begin{bmatrix}
      1 \\
      0 \\
      3 \\
      2 \\
    \end{bmatrix}
    \begin{bmatrix}
      1 & -1 & -1 & 1
    \end{bmatrix} +
    \begin{bmatrix}
      0 \\
      1 \\
      0 \\
      -2 \\
    \end{bmatrix}
    \begin{bmatrix}
      0 & -1 & -1 & 0
    \end{bmatrix} +
    \begin{bmatrix}
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & 0 \\
      0 & 0 & 1 & 1 \\
      0 & 0 & 2 & -2
    \end{bmatrix}
  \end{align*}
  $$
  <p>
    We now need to extract a rank one component from the third matrix. The third pivot is 1 so we can write
  </p>
  $$
  \begin{align*}
    A & =
    \begin{bmatrix}
      1 \\
      0 \\
      3 \\
      2 \\
    \end{bmatrix}
    \begin{bmatrix}
      1 & -1 & -1 & 1
    \end{bmatrix} +
    \begin{bmatrix}
      0 \\
      1 \\
      0 \\
      -2 \\
    \end{bmatrix}
    \begin{bmatrix}
      0 & -1 & -1 & 0
    \end{bmatrix} +
    \begin{bmatrix}
      0 \\
      0 \\
      1 \\
      2 \\
    \end{bmatrix}
    \begin{bmatrix}
      0 & 0 & 1 & 1
    \end{bmatrix} +
    \begin{bmatrix}
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & 0 \\
      0 & 0 & 0 & -4
    \end{bmatrix}.
  \end{align*}
  $$
  <p>
    Finally, we need to extract a rank one component from the fourth matrix. The pivot is -4 and it looks like. We omit the remaining zero matrix.
  </p>
  $$
  \begin{align*}
    A = 
    \begin{bmatrix}
      1 \\
      0 \\
      3 \\
      2 \\
    \end{bmatrix}
    \begin{bmatrix}
      1 & -1 & -1 & 1
    \end{bmatrix} +
    \begin{bmatrix}
      0 \\
      1 \\
      0 \\
      -2 \\
    \end{bmatrix}
    \begin{bmatrix}
      0 & -1 & -1 & 0
    \end{bmatrix} +
    \begin{bmatrix}
      0 \\
      0 \\
      1 \\
      2 \\
    \end{bmatrix}
    \begin{bmatrix}
      0 & 0 & 1 & 1
    \end{bmatrix} +
    \begin{bmatrix}
      0 \\
      0 \\
      0 \\
      1
    \end{bmatrix}
    \begin{bmatrix}
      0 & 0 & 0 & -4
    \end{bmatrix}.
  \end{align*}
  $$
</div>

The above procedure can be described recursively as follows:
1. Base case: The matrix is zero rank.
2. Recursive case: Extract a rank one component and reduce the rank of the matrix by one.

<div>
  <p>
    The $u_k$ factor of the rank one component is the current pivot row. We compute the $l_k$ factor by dividing all elements of the pivot column by the pivot.
  </p>
</div>

## Implementation
A python implementation of the above ideas is this

```python
import copy


class Matrix:
    def __init__(self, body):
        self.body = body

    def get_number_of_rows(self):
        return len(self.body)

    def get_number_of_columns(self):
        return len(self.body[0])

    def get_element(self, row_idx, col_idx):
        return self.body[row_idx][col_idx]

    def get_row(self, row_idx):
        return Matrix([self.body[row_idx]])

    def get_column(self, col_idx):
        return Matrix([[row[col_idx]] for row in self.body])

    def __add__(self, other_matrix):
        new_matrix = copy.deepcopy(self)
        for row_idx, (row1, row2) in enumerate(zip(self.body, other_matrix.body)):
            new_matrix.body[row_idx] = [num1 + num2 for num1, num2 in zip(row1, row2)]
        return new_matrix

    def __mul__(self, other_object):
        if isinstance(other_object, int) or isinstance(other_object, float):
            new_matrix = copy.deepcopy(self)
            for row_idx, row in enumerate(self.body):
                for col_idx, element in enumerate(row):
                    new_matrix.body[row_idx][col_idx] = other_object * element
        else:
            new_matrix = Matrix([[0] * len(other_object.body[0]) for _ in range(len(self.body))])
            for row_idx in range(len(self.body)):
                for col_idx in range(len(other_object.body[0])):
                    row1_list = self.body[row_idx]
                    col2_list = [row[col_idx] for row in other_object.body]
                    new_matrix.body[row_idx][col_idx] = sum([num1 * num2 for num1, num2 in zip(row1_list, col2_list)])
        return new_matrix

    def __sub__(self, other_matrix):
        return self + (other_matrix * -1)

    def __truediv__(self, scalar):
        if isinstance(scalar, int) or  isinstance(scalar, float):
            return self * (1 / scalar)
        raise ValueError("Cannot divide a matrix by anything other than an int or a float")

    def __repr__(self):
        result = []
        for row in self.body:
            row_string_list = [str(element) if element < 0 else f" {element}" for element in row]
            result.append(" ".join(row_string_list))
        return "\n".join(result)


def lu(matrix, current_pivot_idx=0):
    if current_pivot_idx == matrix.get_number_of_rows():
        return []

    pivot = matrix.get_element(current_pivot_idx, current_pivot_idx)
    current_u = matrix.get_row(current_pivot_idx)
    current_l = matrix.get_column(current_pivot_idx) / pivot
    result = [(current_l, current_u)]
    remaining_matrix = matrix - current_l * current_u
    result.extend(lu(remaining_matrix, current_pivot_idx + 1))
    return result

```

## Conclusion
The original idea behind expressing factorization using the rank one approach was to automatically infer parallelization oppurtunities. Indeed, in our recursive algorithm, each recursive step has some
potential parallelization.

What actually ended up happening was viewing factorization using the rank one approach helped in algorithm design. The recursive structure of LU decomposition was more prominent for me in this framework.

It is worthwhile to go further in this direction. I am not sure where will it go and whether it will lead to anything. My strategy is not to go broad in the sense of covering other factorizations like QR or SVD but instead to go deep and implement this with all the automatic parallelization possible.

One thing I am unhappy about is the algorithm is too detailed about where the data is coming from and going to. We need a higher level language or notation to express data dependencies that can be somehow be compiled down to actual operations. Maybe I will explore this in a future article.
