---
layout: post
title: Implementing regular expressions
categories: [compilers, project]
---
<h1 class="text-center">{{ page.title }}</h1>
<br/>
## 1. Introduction
In this article, we will solve a set of problems around the theme of pattern matching. We will see how to represent string patterns,
how to determine whether a string matches a pattern, and how to classify substrings of a string given a set of patterns.

That last problem, classifying substrings of a string, is the most important problem treated here. It is often the first component of a
compiler where it is used to recognize lexical units or "words" in a program text.

The problems considered here have been studied for more than fifty years. They are now standard textbook materials. The
theories developed here are discussed in detail in [[1]](#1) and [[2]](#2). The applications of these techniques to language implementations are comprehensively
discussed in [[3]](#3).

This article is organized as follows. In section 2, we will give detailed descriptions of the problems we are solving. Then, in section 3,
we will discuss regular expressions and how they can be used to represent string patterns. After that, in section 4, we will discuss
the concept of finite automata. Finite automata are the key to implementing textual patterns and in section 5 we will see the technique to
convert regular expressions to finite automata. Finally, in section 6, we will see an application of these techniques to compute the lexical
units of a program text.

## 2. Problems
The main problem we are treating in this article is this.

<div class="definition">
  <p class="definition-title">Problem 1:</p>
  <p>
    Given a string, classify the substrings according to some string patterns.
  </p>
</div>

Here is an instance of this problem. We have a string, `123*53`, and we want to classify the substrings according to whether they are numbers 
or a multiplication operator. In this case, the classification would be this.
1. Two numbers `123` and `53`.
2. One multiplication operator `*`.

To solve this classification problem, we have to solve another related problem.

<div class="definition">
  <p class="definition-title">Problem 2:</p>
  <p>Given a string and a string pattern, determine whether that string matches that pattern.</p>
</div>

For example, if we have a pattern representing all digits, say `D`, then `123` matches `D` but `xyz` doesn't. But what is `D` and how is it represented?
This leads to another problem.

<div class="definition">
  <p class="definition-title">Problem 3:</p>
  <p>How do we represent string patterns?</p>
</div>

To solve problem 1, we need to solve problem 2. And to solve problem 2, we need to solve problem 3. Thus, we proceed as follows. We first
develop the concept of *regular expressions* to solve problem 3. Then, we solve problem 2 using the concept of *finite automata*. Finally,
we apply the techniques of regular expressions and finite automata to solve problem 1.

## 3. Regular expressions
Regular expressions are the notations to describe textual patterns. They were first introduced by Kleene[[4]](#4) and they are used to express statements like:
* A positive integer such that individual digits are between 1 and 4. Examples are `124`, `1`, and `323`.
* A string of alphabets and digits (alphanumeric characters) such that it begins with a letter. For example, `a12`, `z14`, and `hdf21`.

To build the concept of regular expressions, we need to develop some preliminaries. In particular, we need to define what a language is and what are
the language operators.

#### Language
The most basic object we are dealing with is a single string. A string is either empty or non-empty.

<div>
  <p>We represent the empty string with \(\epsilon\).</p>
</div>

We can now define the concept of a language.

<div class="definition">
  <p class="definition-title">Definition 1: Language</p>
  <p>
    A language is defined as a set of strings.
  </p>
</div>

For example, $$
X = \{a, b, c\}
$$ is a language consisting of three single character strings `a`, `b`, and `c`. The language $$
Y = \{abc, def\}
$$ consists of two strings, `abc` and `def`. The language $$
Z = \{xy\thinspace|\thinspace x \in \{a, b, c\} and \thinspace y \in \{d, e, f\}\}
$$ is the same as the language $$
W = \{ad, ae, af, bd, be, bf, cd, ce, cf\}.
$$

Languages are usually denoted by capital letters like A, B, X, and Y.

#### Language operators
How do we combine languags?. We combine languages using language operators which are defined below.

<div class="definition">
  <p class="definition-title">Definition 2: Union</p>
  <p>
    If we have two languages \(X\) and \(Y\), then their union, represented by \( X \cup Y \), is the set of strings that are either in \(X\) or in \(Y\).
  </p>
  <p>
    $$
    X \cup Y = \{w | w \in X or \thinspace w \in Y\}
    $$
  </p>
</div>

<div>
  <p>
    For example, if we have two languages \(X = \{ab, bc, cd\}\) and \(Y = \{wx, xy, yz\}\), then \(X \cup Y\) is \(\{ab, bc, cd, wx, xy, yz\}\).
  </p>
</div>

<div class="definition">
  <p class="definition-title">Definition 3: Concatenation</p>
  <p>If we have two languages \(X\) and \(Y\), then their concatenation, represented by \(X\).\(Y\), is the set of strings \(vw\) such that \(v\) is in \(X\) and \(w\) is in \(Y\).</p>
  <p>
    $$
    X.Y = \{vw | v \in X and \thinspace w \in Y\}
    $$
  </p>
</div>

<div>
  <p>
    Using the same example as above, for two languages \(X = \{ab, bc, cd\}\) and \(Y = \{wx, xy, yz\}\), \(X\).\(Y\)
    is \(\{abwx, abxy, abyz, bcwx, bcwy, bcyz,\) \(cdwx, cdxy, cdyz\}\).
  </p>
</div>

The next operator is a slightly trickier one. It is an application of concatenation on the strings from a single language. The number of concatenations can
be zero, in which it case it is an empty string, or it can be any number of times.

<div class="definition">
  <p class="definition-title">Definition 4: Kleene star</p>
  <p>
    If we have a language \(X\), then its kleene star, denoted by \(X*\), is a set of strings constructed by the concatenation of the strings of
    \(X\) zero or more times.
  </p>
  <p>
    $$
    X* = \{wx....yz | w, x, y, z... \in X \}
    $$
  </p>
</div>
<div>
  <p>
    For example, if \( X = \{abc, xyz\} \), then \(X* = \{\epsilon\ , abcabc, abcxyz, xyzxyz, abcabcxyz, abcxyzxyz, xyzxyzxyz....\}\).
  </p>
</div>

#### Regular expressions
Suppose we want to represent all strings that consist of the characters `a`, `b`, `c`, `1`, `2`, and `3`, such that they begin with `a`, `b`, or `c`.
These strings constitute a language and we want to come up with an expression using the operators above to describe that language.
<div>
  <p>
    Let \(X = \{a, b, c\}\) and \(Y = \{1, 2, 3\}\). A language of single character strings, where the characters come from \(X\) and \(Y\), is this.
  </p>
  <p>
    $$
    Z = (X\cup Y)
    $$
  </p>
  <p>If we want to extend \(Z\) to include multi-character strings, where the characters again come from \(X\) and \(Y\), we modify it like this.</p>
  <p>
    $$
    Z = (X\cup Y)*
    $$
  </p>
  <p>
    Finally, to restrict \(Z\) to only those strings which begin with <code class="language-plaintext highlighter-rogue">a</code>,
    <code class="language-plaintext highlighter-rogue">b</code>, and <code class="language-plaintext highlighter-rogue">c</code>, we do the following
    modification
  </p>
  <p>
    $$
    Z = X.(X\cup Y)*
    $$
  </p>
</div>

What we just saw was an example of constructing a regular expression. Regular expressions are a way to combine language operators to
represent string patterns.

<div class="definition">
  <p class="definition-title">Definition 5: Regular expressions</p>
  <p>The regular expressions are defined as follows.</p>
  <p class="definition-subtitle">Basis cases:</p>
  <ol>
    <li>To represent the empty string \(\epsilon\), the regular expression is \(\epsilon\).</li>
    <li>To represent a single character, say \(a\), the regular expression is that character itself, \(a\).</li>
  </ol>
  <p class="definition-subtitle">Inductive cases:</p>
  <ol>
    <li>To represent the union of the two regular expressions, say \(X\) and \(Y\), the regular expression is \(X|Y\).</li>
    <li>To represent the concatenation of the two regular expressions, say \(X\) and \(Y\), we write the expressions next to each other without any punctuation. So the resultant regular expression is \(XY\).</li>
    <li>To represent the kleene star of a regular expression, say \(X\), we write \(X*\).</li>
  </ol>
</div>

As an example, if we want to represent the pattern discussed above, that is, the strings of `a`, `b`, `c`, `1`, `2`, and `3`, beginning with `a`, `b`, and `c`, the regular expression constructed as follows.
```
x = a|b|c
y = 1|2|3
z = x((x|y)*)
```

You must have noticed that we changed the representation of regular expression operators. The definitions of individual operators used the
mathematical notations but we modified them to make them easier to type.

The fact that the definition of regular expressions is inductive is going to be the key to the implementation of regular expressions
using a recursive algorithm.

We can now define how regular expressions represent a language.

<div class="definition">
  <p class="definition-title">Definition 6: Language defined by a regular expression</p>
  The language define by a regular expression R, denoted by L(R), is the set of strings that are matched by that regular expression.
</div>

With the definition of regular expressions, we have solved problem 3. We have a way to represent textual patterns.

## 4. Finite Automata
The concept of finite automata was first introduced by McCullough and Pitts[[5]](#5) and later developed by Huffman [[6]](#6). Like regular expressions,
finite automata describe languages.

The notion of finite automata can be developed from the concept of directed graphs.

#### Transition tables and Transition graphs
Suppose we have a directed graph where each vertex has a label and each edge is labelled by a character. Here is an example.

<img src="{{site.url}}/static/img/compilers/first_automata.png" class="rounded mx-auto d-block" />

We represent graphs using a data structure called *adjacency lists*. The adjacency list representation for the above edge-labelled directed graph is this.
```
{
  1: {a: 2},
  2: {b: 3, c: 4},
  3: {d: 5},
  4: {d: 5}
}
```
In compiler terminology, adjacency lists are called the *transition tables* and vertices are called the *states*. The graphs themselves are called the
*transition graphs*.

#### Transition graphs with initial and final states
Let us designate one state as an initial state and one state as a final state. We represent an initial state in a transition graph by an arrow pointing
from nowhere to that state. We represent a final state by using concentric circles for that state. In the above transition graph, if we designate `1` as an initial
state and `5` as a final state, it would look like this.

<img src="{{site.url}}/static/img/compilers/first_automata_with_initial_final.png" class="rounded mx-auto d-block" />

Suppose we have a string, say `abd`, and we look at its characters one at a time. Also suppose that we traverse an edge of the above transition graph on each
input character, starting from initial state `1`. The sequence of states on each character would then look like this.

$$
  1 \xrightarrow a 2 \xrightarrow b 3 \xrightarrow d 5
$$

Let's construct another example, this time with multiple final states.

<img src="{{site.url}}/static/img/compilers/second_automata_multiple_final.png" class="rounded mx-auto d-block" />

For string `abd`, the transitions would look like this.
<div>
  $$
    1 \xrightarrow a 2 \xrightarrow b 3 \xrightarrow d 4
  $$
</div>
For `abe`, the transitions end at `ab`, because after making the transitions

$$
  1 \xrightarrow a 2 \xrightarrow b 3
$$

we end up at state 3 and there is no transition from state 3 on character `e`.

For `abd`, the transitions lead to a final state `4` but for `abe`, the transitions led to a non-final state `3`. This distinction is important
we will use it in what follows.

#### Transition graphs with empty string transitions
<div>
  <p>
    We add one last concept to our transition graphs. We allow transitions on empty strings, that is, \(\epsilon\). Here is an example where
    these \(\epsilon\)-transitions are represented by unlabelled edges.
  </p>
</div>

<img src="{{site.url}}/static/img/compilers/first_automata_epsilon.png" class="rounded mx-auto d-block" />

<div>
  <p>
    With \(\epsilon\)-transitions, we need to be more precise about how we describe the current state. In our earlier examples, we
    implicitly represented the current state using just one state. Now, we can be in multiple states at the same time. Let's traverse the above graph
    using a string to see how that can happen.
  </p>
</div>

<div>
  <p>
    Suppose we have a string <code class="language-plaintext highlighter-rogue">ab</code>. We start with the initial state
    <code class="language-plaintext highlighter-rouge">1</code>. But since we have two \(\epsilon\)-transitions out of state
    <code class="language-plaintext highlighter-rouge">1</code>, one to <code class="language-plaintext highlighter-rouge">2</code> and one to
    <code class="language-plaintext highlighter-rouge">5</code>, <em>we make these moves without reading any input character</em>. So our
    current state is not <code class="language-plaintext highlighter-rouge">1</code> but <code class="language-plaintext highlighter-rouge">
    {1, 2, 5}</code>.
  </p>
</div>

<div>
  <p>What we just saw was the computation of \(\epsilon\)-closure of a state. We formally define it as follows.</p>
</div>

<div class="definition">
  <p class="definition-title">Definition 7: \(\epsilon\)-closure of a state</p>
  <p>
    An \(\epsilon\)-closure of a state in a transition graph is the set of states that can be reached from that state by following \(\epsilon\)-transitions
    alone and that state itself.
  </p>
</div>

<div>
  <p>
    In our above example, the \(\epsilon\)-closure(<code class="language-plaintext highlighter-rouge">1</code>) is <code class="language-plaintext highlighter-rouge">{1, 2, 5}</code>.
  </p>
</div>

Going back to processing `ab`, we are now in `{1, 2, 5}` and the next input character is `a`. There is no transition out of state `1` and state `5`
with label `a` but there is one from state `2` to state `3` with that label. So the next state is `{3}`.

Once we are in `{3}`, the next input character is `b` so we move to `{4}`. For `ab`, the transitions look like this.

<div>
  $$
    \{1,2,5\} \xrightarrow a \{3\} \xrightarrow b \{4\}
  $$
</div>

<div>
  <p>
    We want to emphasise that \(\epsilon\)-closures are computed recursively. We need to follow <em>all</em> the \(\epsilon\)-transitions while
    computing an \(\epsilon\)-closure. The basis case of the recursion is when a state has no \(\epsilon\)-transition out of it. In that case, the
    \(\epsilon\)-closure of that state is that state itself.
  </p>
</div>

<div>
  <p>Here's a sketch of the algorithm to compute \(\epsilon\)-closure. Suppose we want to compute the \(\epsilon\)-closure of a state named `M`</p>
</div>

```
closure_set = {M}

if M has no transition out of it with label ""
  return closure_set
else
  for each state N such that edge M-N has label ""
    closure_set = Union of closure_set and compute_closure_for(M)

return closure_set
```

<div>
  <p>
    Let's apply the above algorithm to compute the \(\epsilon\)-closure of <code class="language-plaintext highlighter-rouge">1</code> in the below
    transition graph.
  </p>
</div>

<img src="{{site.url}}/static/img/compilers/second_automata_epsilon.png" class="rounded mx-auto d-block" />

At first, we initialise
$$closure(1)$$
to itself.

$$
  closure(1) = \{1\}
$$

Now there are two transitions out of `1` with label `""`, one to `2` and one to `5`. Therefore, we can write the following.

$$
  closure(1) = \{1\} \cup closure(2) \cup closure(5)
$$

Let's focus on the right hand side. There is nothing to be done for
$$
\{1\}
$$. But we need to compute closure(2) and closure(5).

Let's focus on closure(2). There is one transition out of 2 with label `""` to state 8. So,

$$
  closure(2) = \{2\} \cup closure(8).
$$

Let's compute closure(8). Since there is no transition out of `8` with label `""`, we have the following.

$$
  closure(8) = \{8\}
$$

Thus,

$$
\begin{align}
  closure(2) & = \{2\} \cup closure(8) \\
  \implies closure(2) & = \{2\} \cup \{8\}
\end{align}
$$

Let's go back to closure(5). Since there is no transition out of state 5 with label `""`,

$$
  closure(5) = \{5\}.
$$

Putting it all together, we can write the complete computation as follows.

$$
\begin{align*}
  closure(1) &= \{1\} \cup closure(2) \cup closure(5) \\
  \implies closure(1) &= \{1\} \cup \{2\} \cup closure(8) \cup closure(5) \\
  \implies closure(1) &= \{1\} \cup \{2\} \cup \{8\} \cup closure(5) \\
  \implies closure(1) &= \{1\} \cup \{2\} \cup \{8\} \cup \{5\} \\
  \implies closure(1) &= \{1\} \cup \{2\} \cup \{5, 8\} \\
  \implies closure(1) &= \{1\} \cup \{2, 5, 8\} \\
  \implies closure(1) &= \{1, 2, 5, 8\} \\
\end{align*}
$$

#### Finite automata
We now have all the ideas we need to define the concept of finite automata.

<div class="definition">
  <p class="definition-title">Definition 8: Finite Automaton</p>
    <p>A finite automaton is a set of five objects \(\{Q, \Sigma, q_0, F, T\thinspace \}\) defined as follows.</p>
  <ol>
    <li>A set of states \(Q\).</li>
    <li>A set of allowable input characters \(\Sigma \).</li>
    <li>A state designated as an initial state \(q_0\).</li>
    <li>A set of states designated as final states \(F\).</li>
    <li>A transition function \( \delta: Q \times \Sigma \rightarrow Q\).</li>
  </ol>
</div>

Let's look at an earlier example but in the context of a finite automaton. Suppose our transition graph looks like this.

<img src="{{site.url}}/static/img/compilers/second_automata_epsilon.png" class="rounded mx-auto d-block" />

In this case:

1.
$$
Q = \{1, 2, 3, 4, 5, 6, 7, 8\}
$$

2.
$$
q_0 = 1
$$

3.
$$
F = \{4, 7\}
$$

4.
$$
\Sigma = \{a, b, c, d\}
$$

5.
A transition table representing the transition function 
$$
\delta.
$$
```
  1: {"": {2, 5}},
  2: {a: {3}, "": {8}},
  3: {b: {4}},
  4: {d: {5}},
  5: {c: {6}},
  6: {d: {7}},
  7: {},
  8: {}
```

Like regular expressions, finite automata define languages.

<div class="definition">
  <p class="definition-title">Definition 9: Language defined by a finite automaton</p>
  The language defined by a finite automaton is the set of strings that take the automaton from the initial state to one of the final states.
</div>

Let's repeat the one of the above finite automata examples here.

<img src="{{site.url}}/static/img/compilers/second_automata_epsilon.png" class="rounded mx-auto d-block" />

The language defined by this automaton is `{ab, cd}`.

<div class="definition">
  <p class="definition-title">Definition 10: Acceptance criterion of a finite automaton</p>
  A finite automaton accepts a string if it takes the automaton from the initial state to one of the final states. It rejects all other strings.
</div>

## 5. Implementing regular expressions using finite automata.
What we have seen thus far is both finite automata and regular expressions define languages. More importantly, *every regular expression
has an equivalent finite automaton*. We won't give a proof of the equivalence of regular expressions and finite automata in this article. They are
standard textbook material[[1]](#1)[[2]](#2).

What we will do is translate our regular expressions into finite automata; these finite automata implement their equivalent regular expressions. The
algorithm discussed here is due to Thompson[[7]](#7).

In what follows, we will represent an arbitrary finite automata like this.

<img src="{{site.url}}/static/img/compilers/arbitrary_automata.png" class="rounded mx-auto d-block" />

Remember, we defined regular expressions inductively. We define the translation from regular expression to finite automaton for each case. We had two
basis cases and three inductive cases.

<h4>Basis case 1: Finite automata for regular expression \(\epsilon\)</h4>
<div>
  <p>For regular expression \(\epsilon\), the equivalent finite automata is this.</p>
  <img src="{{site.url}}/static/img/compilers/base_epsilon_automata.png" class="rounded mx-auto d-block" />
</div>

#### Basis case 2: Finite automata for single character regular expression
For a single character regular expression, say `x`, the equivalent finite automata is this.
<img src="{{site.url}}/static/img/compilers/base_single_character_automata.png" class="rounded mx-auto d-block" />

#### Inductive case 1: Union of regular expressions
If we have two regular expressions `r1` and `r2`, then their union, `r1|r2`, has the following equivalent finite automaton.
<img src="{{site.url}}/static/img/compilers/inductive_union_automata.png" class="rounded mx-auto d-block" />
That is, we do the following things:
<div>
  <ol>
    <li>We create a new start state.</li>
    <li>We add \(\epsilon\)-transitions from the new start state to each operand automaton's start state.</li>
    <li>We create a new final state.</li>
    <li>We add \(\epsilon\)-transitions from each operand automaton's final state to the new final state.</li>
  </ol>
</div>

#### Inductive case 2: Concatenation of regular expressions
If we have two regular expressions `r1` and `r2`, then their concatenation, `r1r2`, has the following equivalent finite automata.
<img src="{{site.url}}/static/img/compilers/inductive_concatenation_automata.png" class="rounded mx-auto d-block" />

<div>
  <p>That is, we add an \(\epsilon\)-transition from the first operand automaton's final state to the second operand automaton's initial state.</p>
</div>

#### Inductive case 3: Kleene star of regular expressions
If we have a regular expression `r1`, then its kleene star, `r1*`, has the following equivalent finite automata.
<img src="{{site.url}}/static/img/compilers/inductive_kleene_star_automata.png" class="rounded mx-auto d-block" />
That is, we do the following things:
<ol>
  <li>We create a new start state and add an \(\epsilon\)-transition from that new start state to the operand automaton's start state.</li>
  <li>We create a new final state and add an \(\epsilon\)-transition from the operand automaton's final state to the new final state.</li>
  <li>We add an \(\epsilon\)-transition from the operand's final state to the operand's start state.</li>
  <li>We add an \(\epsilon\)-transition from the new start state to the new final state. </li>
</ol>

#### Algorithm to implement regular expressions
What we have discussed so far is an inductive way to convert regular expressions into finite automata. We can readily translate these cases
into a recursive algorithm. The outline of the algorithm looks like this.
```
compile_regex_to_automaton(r)
  if r is an empty string, construct the empty string finite automaton and return it.

  if r is a single character string, construct the single character finite automaton and return it.

  else
    first_operand = get_first_operand(r)
    first_operand_automaton = compile_regex_to_automaton(first_operand)
  
    operator = get_operator(r)
  
    if operator is a kleene star
      // There is no second operand
      combined_automaton = apply_kleene_star(first_operand)
      return combined_automaton

    second_operand = get_second_operand(r)
    second_operand_automaton = compile_regex_to_automaton(second_operand)
    
    if operator is union
      combined_automaton = combine_using_union(first_operand_automaton, second_operand_automaton)
    else
      combined_automaton = combine_using_concatenation(first_operand_automaton, second_operand_automaton)
    
    return combined_automaton
```

Let's run this algorithm using an example. Suppose our regular expression is `(a|b)(c|d)*`. It describes the language `{ac, ad, bc, bd}`.
The recursion proceeds as follows.

1. Is `(a|b)(c|d)` an empty string? No.
2. Is `(a|b)(c|d)` a single character string? No.
3. Compute the first operand of `(a|b)(c|d)`. We do it by matching parentheses and its `(a|b)`. We trim the outermost parentheses.
    1. Is `a|b` an empty string? No.
    2. Is `a|b` a single character string? No.
    3. Compute the first operand of `a|b`. There are no parentheses so the first character, `a`, is the operand.
       1. Is `a` an empty string? No.
       2. Is `a` a single character string? Yes, so we construct the single character automaton and return it.
    4. Compute the operator for `a|b`. Its union `|`.
    5. Is the operator a kleene star? No.
    6. Compute the second operand of `a|b`. Its `b`.
       1. Is `b` is an empty string? No.
       2. Is `b` a single character string? Yes, so we construct the single character automaton and return it.
    7. Combine the automata for `a` and `b` using the union `|`.
4. Compute the operator for `(a|b)(c|d)`. Its concatenation.
5. Is the operator a kleene star? No.
6. Compute the second operand for `(a|b)(c|d)`. We just remove the first operand and the operator; the remaining substring is the second operand `(c|d)`.
   We trim the parentheses.
    1. Is `c|d` an empty string? No.
    2. Is `c|d` a single character string? No.
    3. Compute the first operand of `c|d`. There are no parentheses so the first character, `c`, is the operand.
       1. Is `c` an empty string? No.
       2. Is `c` a single character string? Yes, so we construct the single character automaton and return it.
    4. Compute the operator for `c|d`. Its union `|`.
    5. Is the operator a kleene star? No.
    6. Compute the second operand of `c|d`. Its `d`.
       1. Is `d` is an empty string? No.
       2. Is `d` a single character string? Yes, so we construct the single character automaton and return it.
    7. Combine the automata for `c` and `d` using the union `|`.
7. Combine the automata for `(a|b)` and `(c|d)` using concatenation.

With this algorithm, we have solved the problem 2 posed above: given a string and a string pattern, determine whether the string matches that pattern.
We proceed like this.

1. Compute the automaton for the pattern.
2. Traverse the automaton using the string to see whether it accepts or rejects the string.

We are in a position now to attack our capstone problem. The problem 1 was to classify the substrings of a string given some string patterns.
Let's see how can we do it and what is the role of the solution in programming language implementation.


## 6. Tokenization
Suppose we have a string and we want to classify the substrings of that string using regular expressions. For example, if we have a string `12+132`
and we are given regular expressions for positive integers and `+`, we will divide the string into `12`, `+`, and `132`.

This process of dividing the strings using regular expressions is called *Tokenization*.

The divided substrings are not represented as such but are used to construct objects called *Tokens*. Tokens represent the divided substrings
and contain two pieces of information.

1. The regular expression used to match that substring. It is called the *token type*.
2. The substring matched for this token. It is called the *lexeme*.

We will represent a token like `<Tokentype, Lexeme>`. For example, the tokens for `12+123` are going to be
`<positive_integer, 12>`, `<+, +>`, and `<positive_integer, 123>`.

We ignore all whitespace while tokenizing a string.

The way we do tokenization is as follows.
1. Construct the regular expressions on which we want to tokenize strings.
2. Starting from the left, bite off the substrings of a given string that match those regular expressions.

For example, for our above example, `12 + 123`, and using regular expressions representing positive integers and operator `+`, we proceed as follows.
1. Starting from the left of `12 + 123`, `12` will match the regular expression for positive integers. We will have `<positive_integer, 123>` as the first token and the
   remaining substring left is `+ 123`.
2. Starting from the left of `+ 123`, `+` will match the regular expression for `+`. We will have `<+, +>` as our second token and the remaining substring
   left is `123`.
3. Our regular expression for positive integers will match `123`, so our final token is going to be `<positive_integer, 123>`.

Therefore, the output of tokenizing `12 + 123` is `<positive_integer, 12>, <+, +>, and <positive_integer, 123>`.

Here's one problem with the above approach. Suppose we have the following regular expressions.
1. A regular expression for numbers: `(0|1|2|3|4|5|6|7|8|9)(0|1|2|3|4|5|6|7|8|9)*`.
2. A regular expression for single equals: `=`.
3. A regular expression for double equals: `==`.

If the string we want to tokenise is `123==123`, which of the following tokenization is correct.
1. `<number, 123>, <=, =>,<=, =>, <number, 123>`.
2. `<number, 123>, <==, ==>, <number, 123>`.

The answer is 2. At any point of the string, we choose that substring for the next token which is going to have the maximum length. This policy is
called the *Maximal Munch* policy. In our earlier example, after `123`, we match `=` with regular expression `=` and `==` with regular expression `==`. Using
maximal munch policy, the next token is going to be `<==, ==>`.

We can now sketch an algorithm to get the next token of a string given a string to tokenize and a string position from which we start the tokenization.

```
get_next_token(string, start_position)
  list_of_regexes = <regexes we want to use to tokenize strings>.
  
  next_lexeme = ""
  candidate_lexeme = ""
  for regex in list_of_regexes
    candidate_lexeme = regex.match(string, start_position)
    if candidate_lexeme is longer than next_lexeme
      next_lexeme = candidate_lexeme
  
  return next_lexeme
```

With this algorithm, we have solved the problem 1 stated above.

## References
<p id="1">
  [1] Sipser, Michael. Introduction to the Theory of Computation. 3rd edition. Cengage Learning. 2012
      <a href="#" onclick="window.history.back()">&uarr;</a>
</p>
<p id="2">
  [2] Hopcroft, John E., Motwani, Rajeev, &amp; Ullman, Jeffery D. Introduction to Automata Theory, Languages, and Computation. 3rd Edition. Pearson. 2007
      <a href="#" onclick="window.history.back()">&uarr;</a>
</p>
<p id="3">
  [3] Aho, Alfred V. Lam, Monica S., Sethi, Ravi, &amp; Ullman, Jeffery D. Compilers Principles, Techiniques and Tools, 2nd Edition. Pearson. 2007
      <a href="#" onclick="window.history.back()">&uarr;</a>
</p>
<p id="4">
  [4] Kleene, S. C. "Representation of events in nerve nets" in <em>Automata Studies</em>, edited by Shannon, C. &amp; McCarthy, J, 3-40.
      Princeton University Press, 1956.
      <a href="#" onclick="window.history.back()">&uarr;</a>
</p>
<p id="5">
  [5] McCullough, W.S and Pitts, W. "A logical calculus of the ideas immanent in nervous activity." Bull. Math. Biophysics 5, pp 115-133, 1943.
      Princeton University Press, 1956.
      <a href="#" onclick="window.history.back()">&uarr;</a>
</p>
<p id="6">
  [6] Huffman, D.A., "The synthesis of sequential machines," J. Franklin Inst. 257 (1954), pp. 3-4, 161, 190, 275-303.
      <a href="#" onclick="window.history.back()">&uarr;</a>
</p>
<p id="7">
  [7] Thompson, K., "Regular expression search algorithm," Comm. ACM 11:6 (1968), pp. 419-422.
      <a href="#" onclick="window.history.back()">&uarr;</a>
</p>
