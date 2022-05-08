---
layout: post
title: Expressing matrix multiplications using rank one matrices
categories: [compilers, project, matrices, rank-one, notes]
---
<h1 class="text-center">{{ page.title }}</h1>
<br/>
*This is the first article in the study of rank one matrices. You can find all the articles related to rank one matrices [here](/categories/rank-one/).*

Matrix multiplication is a fundamental operation in many areas of computer science. In this article, we will explore a new technique of doing matrix multiplications. We will first get some intuition by going through some examples. We will then formally prove that the new technique gives the same result as the standard matrix multiplication. Finally, we will discuss its possible applications.

## Prerequisites
I am assuming that you are familiar with linear algebra corresponding to a typical undergrad course. In particular, I make use of addition between matrices, multiplication between matrices, and the concept of the rank of a matrix.

## Examples
<div>
  <p>
    Suppose we multiply two \(2 \times 2\) matrices in the standard way.
  </p>
  $$
  \begin{align*}
    & \begin{bmatrix}
      1 & 3 \\
      2 & 4
    \end{bmatrix}
    \begin{bmatrix}
      5 & 7 \\
      6 & 8
    \end{bmatrix} \\ \\
    = &
    \begin{bmatrix}
      1.5 + 3.6 & 1.7 + 3.8 \\
      2.5 + 4.6 & 2.7 + 4.8
    \end{bmatrix}.
  \end{align*}
  $$
</div>
Since each element is a sum of two numbers, we can express the last multiplication as
<div>
  $$
  \begin{align*}
    &
    \begin{bmatrix}
      1.5 & 1.7 \\
      2.5 & 2.7
    \end{bmatrix} +
    \begin{bmatrix}
      3.6 & 3.8 \\
      4.6 & 4.8
    \end{bmatrix}.
  \end{align*}
  $$
</div>

In the first operand,
<div>
  $$
  \begin{align*}
    &
    \begin{bmatrix}
      1.5 & 1.7 \\
      2.5 & 2.7
    \end{bmatrix},
  \end{align*}
  $$
</div>
all the rows are multiples of
<div>
  $$
  \begin{align*}
    &
    \begin{bmatrix}
      5 & 7
    \end{bmatrix}
  \end{align*}
  $$
</div>
and all the columns are multiples of
<div>
  $$
  \begin{align*}
    &
    \begin{bmatrix}
      1 \\
      2
    \end{bmatrix}
  \end{align*}.
  $$
</div>
These kind of matrices can be written as a multiplication of a column and a row.

<div>
  $$
  \begin{align*}
    \begin{bmatrix}
      1 \\
      2
    \end{bmatrix}
    \begin{bmatrix}
      5 & 7
    \end{bmatrix}
  \end{align*}.
  $$
</div>

Similarly, we can express the second operand,
<div>
  $$
  \begin{align*}
    &
    \begin{bmatrix}
      3.6 & 3.8 \\
      4.6 & 4.8
    \end{bmatrix}
  \end{align*},
  $$
</div>
as
<div>
  $$
  \begin{align*}
    &
    \begin{bmatrix}
      3 \\
      4
    \end{bmatrix}
    \begin{bmatrix}
      6 & 8
    \end{bmatrix}.
  \end{align*}
  $$
</div>

<div>
  Thus, we can write our multiplication of two \(2 \times 2\) matrices as
</div>

<div>
  $$
  \begin{align*}
    \begin{bmatrix}
      1 & 3 \\
      2 & 4
    \end{bmatrix}
    \begin{bmatrix}
      5 & 7 \\
      6 & 8
    \end{bmatrix}
    =
    \begin{bmatrix}
      1 \\
      2
    \end{bmatrix}
    \begin{bmatrix}
      5 & 7
    \end{bmatrix}
    +
    \begin{bmatrix}
      3 \\
      4
    \end{bmatrix}
    \begin{bmatrix}
      6 & 8
    \end{bmatrix}.
  \end{align*}
  $$
</div>
Let's look at another example where we multiply a square matrix with a rectangular matrix.
<div>
  $$
  \begin{align*}
    & \begin{bmatrix}
      1 & 3 \\
      2 & 4
    \end{bmatrix}
    \begin{bmatrix}
      5 & 7 & 8 \\
      9 & 10 & 11
    \end{bmatrix} \\ \\
    = &
    \begin{bmatrix}
      1.5 + 3.9 & 1.7 + 3.10 & 1.8 + 3.11 \\
      2.5 + 4.9 & 2.7 + 4.10 & 2.8 + 4.11
    \end{bmatrix} \\ \\
    = &
    \begin{bmatrix}
      1.5 & 1.7 & 1.8 \\
      2.5 & 2.7 & 2.8
    \end{bmatrix}
    +
    \begin{bmatrix}
      3.9 & 3.10 & 3.11 \\
      4.9 & 4.10 & 4.11
    \end{bmatrix} \\ \\
    = &
    \begin{bmatrix}
      1 \\
      2
    \end{bmatrix}
    \begin{bmatrix}
      5 & 7 & 8
    \end{bmatrix}
    +
    \begin{bmatrix}
      3 \\
      4
    \end{bmatrix}
    \begin{bmatrix}
      9 & 10 & 11
    \end{bmatrix}.
  \end{align*}
  $$
</div>

If we see these two multiplications,
<div>
  $$
  \begin{align*}
    & \begin{bmatrix}
      1 & 3 \\
      2 & 4
    \end{bmatrix}
    \begin{bmatrix}
      5 & 7 \\
      6 & 8
    \end{bmatrix}
  \end{align*}
  =
  \begin{bmatrix}
      1 \\
      2
    \end{bmatrix}
    \begin{bmatrix}
      5 & 7
    \end{bmatrix}
    +
    \begin{bmatrix}
      3 \\
      4
    \end{bmatrix}
    \begin{bmatrix}
      6 & 8
    \end{bmatrix} \\
  $$
  and
  $$
    \begin{bmatrix}
      1 & 3 \\
      2 & 4
    \end{bmatrix}
    \begin{bmatrix}
      5 & 7 & 8 \\
      9 & 10 & 11
    \end{bmatrix} =
    \begin{bmatrix}
      1 \\
      2
    \end{bmatrix}
    \begin{bmatrix}
      5 & 7 & 8
    \end{bmatrix}
    +
    \begin{bmatrix}
      3 \\
      4
    \end{bmatrix}
    \begin{bmatrix}
      9 & 10 & 11
    \end{bmatrix},
  $$
</div>
we notice that the general rule is to multiply each column of the first matrix with the corresponding row of the second matrix and add the results of
these multiplications. Moreover, the multiplication of a column matrix and a row matrix results in a matrix where all columns are multiples of a column
and all rows are multiples of a row, that is, they are rank one matrices. These examples indicate that matrix multiplication can be expressed as a sum of rank one matrices.

We are now going to prove that this result holds for any matrix. Before moving
on to the proof, we will describe the notation we will follow in the proof.

## Notation
<div>
  <p>
    A matrix will be denoted by a capital letter with its dimensions on the subscript. For example, \(A_{m \times n}\) and \(B_{n \times p}\).
  </p>
  <p>
    An element of a particular matrix is given by the corresponding lowercase letter of the matrix with the index of the element as the subscript. For example, the \(i, j^{th}\) element of \(A_{m \times n}\) will be denoted by \(a_{i,j}\).
  </p>
  <p>
    If we want to abstractly refer to an element inside a matrix, we use the element notation inside square brackets. For example, to express that all elements of \(C_{m \times p}\) equal the sum of its indices, we write
    $$
      [c_{i,j}] = [i + j].
    $$
  </p>
</div>

## Theorem
We first define the concepts we are going to need in the proof.

<div class="definition">
  <p class="definition-title">Definition 1: Matrix-matrix addition</p>
  <p>
    If we have two matrices \(A_{m \times n}\) and \(B_{m \times n}\), then their addition is
    $$
      C_{m \times n} = A_{m \times n} + B_{m \times n}
    $$
    where
    $$
      [c_{i,j}] = [a_{i,j} + b_{i,j}].
    $$
  </p>
</div>

<div class="definition">
  <p class="definition-title">Definition 2: Matrix-matrix multiplication</p>
  <p>
    If we have two matrices \(A_{m \times n}\) and \(B_{n \times p}\), then their multiplication is
    $$
      C_{m \times p} = A_{m \times n}B_{n \times p}
    $$
    where
    $$
      [c_{i,j}] = [\sum_{k=1}^{n} a_{i,k}.b_{k,j}].
    $$
  </p>
</div>

Let's now state and prove our theorem.

<div class="definition">
  <p class="definition-title">
    Theorem: Matrix-matrix multiplication in terms of rank one matrices
  </p>
  <p>
    If we have two matrices \(A_{m \times n}\) and \(B_{n \times p}\), then their multiplication is
    $$
      C_{m \times p} = A_{m \times n}B_{n \times p}
    $$
    where
    $$
      C_{m \times p} = \sum_{k=1}^{n} \left( \begin{bmatrix}
        a_{1,k} \\
        a_{2,k} \\
        . \\
        . \\
        . \\
        a_{m,k}
      \end{bmatrix}
      \begin{bmatrix}
        b_{k,1} & b_{k,2} & . & . & . &b_{k,p}
      \end{bmatrix} \right).
    $$
  </p>
</div>

#### Proof
We start by writing down the definition of matrix multiplication.
<div>
  $$
    \begin{align*}
      [c_{i,j}] = [\sum_{k=1}^{n} a_{i,k}.b_{k,j}].
    \end{align*}
  $$
  <p>We can write down the complete matrix \(C_{m \times p}\) as</p>
  $$
    \begin{align*}
      C_{m \times p} = \begin{bmatrix}
        \sum_{k=1}^{n} a_{1,k}.b_{k,1} & . & . & \sum_{k=1}^{n} a_{1,k}.b_{k,p} \\
        . & . & . & . \\
        . & . & . & . \\
        . & . & . & . \\
        \sum_{k=1}^{n} a_{m,k}.b_{k,1} & . & . & \sum_{k=1}^{n} a_{m,k}.b_{k,p} \\
      \end{bmatrix}.
    \end{align*}
  $$
  <p>Using the definition of matrix addition, we can write</p>
  $$
    \begin{align*}
      C_{m \times p} = \begin{bmatrix}
        a_{1,1}.b_{1,1} & . & . & a_{1,1}.b_{1,p} \\
        . & . & . & . \\
        . & . & . & . \\
        . & . & . & . \\
        a_{m,1}.b_{1,1} & . & . & a_{m,1}.b_{1,p} \\
      \end{bmatrix} +
      \begin{bmatrix}
          a_{1,2}.b_{2,1} & . & . & a_{1,2}.b_{2,p} \\
          . & . & . & . \\
          . & . & . & . \\
          . & . & . & . \\
          a_{m,2}.b_{2,1} & . & . & a_{m,2}.b_{2,p} \\
      \end{bmatrix} +
      \begin{bmatrix}
          a_{1,3}.b_{3,1} & . & . & a_{1,3}.b_{3,p} \\
          . & . & . & . \\
          . & . & . & . \\
          . & . & . & . \\
          a_{m,3}.b_{3,1} & . & . & a_{m,3}.b_{3,p} \\
      \end{bmatrix} + ... +
      \begin{bmatrix}
          a_{1,n}.b_{n,1} & . & . & a_{1,n}.b_{n,p} \\
          . & . & . & . \\
          . & . & . & . \\
          . & . & . & . \\
          a_{m,n}.b_{n,1} & . & . & a_{m,n}.b_{n,p} \\
      \end{bmatrix}.
    \end{align*}
  $$
  The above sum can be written as
  $$
    \begin{align*}
      C_{m \times p} =
      \sum_{k=1}^{n} \begin{bmatrix}
            a_{1,k}.b_{k,1} & . & . & a_{1,k}.b_{k,p} \\
                  . & . & . & . \\
                  . & . & . & . \\
                  . & . & . & . \\
                  a_{m,k}.b_{k,1} & . & . & a_{m,k}.b_{k,p}
                \end{bmatrix}.
    \end{align*}
  $$
</div>

Each term in the summation can be expressed as a multiplication of the column
<div>
  $$
    \begin{bmatrix}
      a_{1,k} \\
      . \\
      . \\
      . \\
      a_{m,k}
    \end{bmatrix}
  $$
</div>
and the row
<div>
  $$
    \begin{bmatrix}
      b_{k,1} & . & . & . & b_{k,p}
    \end{bmatrix}.
  $$
</div>

We can thus finally write

<div>
  $$
    \begin{align*}
      C_{m \times p} = \sum_{k=1}^{n} \left( \begin{bmatrix}
        a_{1,k} \\
        a_{2,k} \\
        . \\
        . \\
        . \\
        a_{m,k}
      \end{bmatrix}
      \begin{bmatrix}
        b_{k,1} & b_{k,2} & . & . & . &b_{k,p}
      \end{bmatrix} \right) \\
      \square
    \end{align*}.
  $$
</div>

## Discussion
What are the advantages of viewing multiplications like this? One area where
matrix multiplications are used is matrix factorizations. So it might be worth
investigating what matrix factorizations look like in terms of the sum of rank one matrices. In fact, we can assume that we have a hardware that works in terms
of rank one matrices and explore what a compiler for that hardware would look
like. We can see whether the "human way of thinking" about matrix factorizations can be translated into sum of rank one matrices. This is the topic I will explore in the follow up articles.