+++
title = 'A parser for an array language'
date = 2020-03-22T00:00:00+00:00
+++

In my previous [post]({{< ref "2020-02-22-problem-implicit-parallelism-array-languages" >}} "Problem: Implicit parallelism in array languages")
, we stated that the goal is to use an array language to explore the problems of implicit parallelism. To get started
with that, we need an infrastructure on which we can test our ideas. Specifically, we need to design and implement
an array language before we can tangibly do anything.

We are going to start easy and add features as we go along. In particular, we should keep reminding ourselves that we are not making a
product to be shipped but a prototype to evaluate our hypotheses. It's a means to an end, in particular, the compiler we are building
is a means to evaluate parallelism and nothing else.

This article is organized as follows. We will first design our array language and determine the constructs we need to support. We follow that
up by designing our lexical analyzer, the part that takes a program text and divides it into tokens. Finally, we design a parser that takes a
stream of tokens as input and imposes a grammatical structure on them.

## Language design
Let's start by deciding what our language will look like.

At first, we will restrict ourselves to simple arrays by only allowing positive integers as array elements. Later on, we will add
support for more data types. Concretely, we should add support for an array of arrays and an array of lists to support matrices
and graphs respectively.

The language is going to support basic arithmetic operations, that is, it can be used as a calculator.
```
> 3 * 2
6
> 213 * (123 + 123213)
26270568
```

We can support array arithmetic like this.
```
> [121, 122, 1232133] + [4, 5, 6]
[125, 127, 1232139]
```
We need to support the application of functions to arrays. There are two kinds of functions.
1. Those that take an integer as an argument and return an integer.
2. Those that take two integers as arguments and return an integer.

Let's say we have a function named `square` that takes an integer as an argument
and returns an integer. We can do something like this.
```
> square [1, 2, 3]
[1, 4, 9]
```
Now, let's say we have a function named `sum` that takes two integers as arguments and returns a single integer. We can then do something like this.
```
> sum [1, 2, 3]

```
We can put the following restrictions on these functions.
1. They must return an integer.
2. They must be commutative.

The commutativity is necessary for functions with two arguments to be applied to arrays. An interesting question is whether we can check these
two properties at compile time.

To apply multiple operators at once, we need to group them using parentheses.
```
> sum (square [1, 2, 3])
> square (sum (double [1, 2, 3])
```
We also support variables and assignments.
```
> a = [1, 2, 3]
> b = 42 * 241
```

Of course, we need to check whether the operations make sense. For instance, this is wrong.
```
> [1, 2] + [4, 5, 6, 7, 8]
```

But a parser cannot check these type errors. For that, we need a semantic analyzer, which is going to be a topic of the next post.

## Lexical analyzer
To build a lexical analyzer, we need a definition of the basic units of our language. These are called token types. Our tokens are the following:
1. (
2. )
3. positive integers
4. +
5. -
6. *
7. /
8. identifers/variables
9. =
10. ==

We represent these tokens using regular expressions. Concretely, we will do that using regular definitions, which are collections of regular
expressions such that each expression can use variables that are defined before it.

Finally, we need to implement these regular definitions using finite automata. The data structure we use to implement a finite automaton is called
a transition table.

The regular definition for positive integers is this.
```
digit  -> 1|2|3|4|5|6|7|8|9|0
digits -> digitdigit*
number -> digits
```
Using standard methods, we can construct a transition table for `number` like this.
```
      | digit |
------|-------|
   1  |   2   |   start_state: 1
------|-------|   final_states: 2, 3
   2  |   3   |
------|-------|
   3  |   3   |
------|-------|
```
The identifiers consist of letters, digits, and underscore, such that they start with a letter. The regular definition looks like this.
```
letter_ -> a|b|...|y|z|A|B|...|Z|_|
id      -> letter_(letter_|digit)*
```
And the transition table looks like this.
```
      | letter |  digit  |
------|--------|---------|
   1  |    2   |         |   start_state: 1
------|--------|---------|   final_states: 2, 3, 4
   2  |    4   |    3    |
------|--------|---------|
   3  |    4   |    3    |
------|--------|---------|
   4  |    4   |    3    |
------|--------|---------|
```
For single character tokens like `+`, `-`, `*`, `/`, `(`, `)`, and `=`, the transition tables are trivial. If we denote these characters as `chr`,
the table is this.
```
      |  chr  |
------|-------|
   1  |   2   |   initial_state: 1
------|-------|   final_states: 2
   2  |       |
------|-------|
```
The transition table for `==` is as follows.
```
      |   =   |
------|-------|
   1  |   2   |   initial_state: 1
------|-------|   final_states: 3
   2  |   3   |
------|-------|
```
Of course, we need to decide between the tokens like `=` and `==`. We will do this by using the policy of *maximal munch*; if multiple finite automata match
different prefixes of the same string, we choose the automaton with the longest matching prefix.

With transition tables in place, we can implement our lexical analyzer. We will give a link to the complete implementation in the conclusion of this post.

## Parser
Now let's turn to how can we design a parser for this. To build a parser, we need to construct context-free grammars to recognize the ways the above token
types can be combined. We will build a top-down LL(1) parser.

In the following, we will represent the empty string with the word `epsilon`.

Let's tackle arithmetic expressions first. The grammar that enforces operator precedence rules looks like this.
```
expr   --> expr + term | expr - term | term
term   --> term * factor | term / factor | factor
factor --> number | (expr)
```
Since we are building a top-down parser, we need to remove left-recursion from this grammar. The result looks like this.
```
expr   --> term expr'
expr'  --> + term expr' | - term expr' | epsilon
term   --> factor term'
term'  --> * factor term' | / factor term' | epsilon
factor --> number | (expr)
```

Now let's turn to arrays. The grammar for array looks like the following.
```
array  --> [ number rest ]
rest   --> , number rest | epsilon
```

We can support array arithmetic by changing the last production of the operator grammar from
```
factor --> number | (expr)
```
to
```
factor --> number | array | (expr)
```

To support assignment, the grammar is this.

```
assignment --> id = number | id = array
```
We need to left-factor this to make it amenable to top-down parsing. The result is this.

```
assignment --> id = assignee
assignee   --> number | array
```

These grammars cover most of the constructs we want to support. We can create a single LL(1) parsing table from these grammars using standard algorithms.

## Conclusion
The implementation is [here](https://github.com/saurabhjha/roady-lang). The components we
discussed above constitute roughly half of what is traditionally called a compiler frontend. The other half consists
of the semantic analyzer, which does type checking, and an intermediate code generator, which translates our parse
tree to an intermediate representation that can be more easily translated to a target language.

In the upcoming post, we will discuss the semantic analyzer of this language and see how can we add type checking to this language.
