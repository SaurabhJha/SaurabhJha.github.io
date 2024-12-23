+++
title = 'How I moved to compilers'
date = 2024-12-23T00:00:00+00:00
+++

In this article, I describe how I made the transition from working on web applications to working on compilers. I've
written this as a chronology. The chronology has gaps as it's not a continuous timeline.

I use some terminology that might be unfamiliar. I used it to add some flavour to my descriptions and I hope it won't 
deter you from reading.

## October 2019
I decided that I want to work on compilers. I had been working on web applications since early 2014 and wanted to
move on to more low-level areas of software engineering. The options I considered were operating systems, databases,
and compilers. I chose compilers as it also had a maths flavour in it.

## February - April 2020
I started reading an introductory compiler textbook, nicknamed the [dragon book](https://suif.stanford.edu/dragonbook/).
This book is widely considered the definitive introduction to the field of programming languages.

My plan was to learn about compilers from the dragon book and apply that to a problem. The problem I was looking at
was designing a language for linear algebra. In this language, the fundamental data types would be matrices, and the
matrix operations would be implicitly parallelized.

By March, I broadly knew about the three parts of a compiler:
1. Front-end, which takes program text and converts it into a data structure. This data structure is called
   intermediate representation (IR).
2. Middle-end, which takes the IR and modifies it so that the program it represents runs faster.
3. Back-end, which takes the modified IR and converts it into assembly text.

I tried to define my matrix problem and implemented a [prototype](https://github.com/SaurabhJha/roady-lang). By April,
I realised that the focus of this problem is going to be on the middle-end - that's where all the compiler
optimisations are done.

## June - August 2020
I attended an academic conference called Programming Languages Design and Implementation ([PLDI](https://pldi20.sigplan.org/track/pldi-2020-papers)) in June.
I did not know enough to appreciate the technical content. But I ended up talking to a lot of people, all of whom
were very encouraging. Concretely, when I shared my interests and my plan for building my programming languages
background, they all broadly agreed that the plan was good and also gave me an overview of the kind of problems
people were working on.

This conference was where I learned about the work happening in deep learning libraries. In deep learning,
the fundamental data structure is a multi-dimensional array called tensor. Tensors are a generalisation of matrices.
So my problem was in the area of deep learning libraries.

By this time, I had a good understanding of the compiler front-end from my reading of the dragon book. For practice,
I wrote a [regular expression compiler and a parser generator](https://github.com/SaurabhJha/lexpar/tree/main).

## October 2020
I finished going through about 75% of the dragon book. I did not finish it as I wanted to cover the remaining material
from more recent sources.

## January 2021
I attended an academic conference called Principles of Programming Languages ([POPL](https://popl21.sigplan.org)).
Here, I got an excellent suggestion that I should contribute to some open source compiler project to get more
real-world experience, given that I had built some background in programming languages.

I spent the last two weeks of January shopping around what I could contribute to. I seriously considered two
projects: Pytorch and llvm. I couldn't compile Pytorch on my machine. I could compile llvm. So I chose llvm and
started looking for easy-to-fix issues in that project.

## February 2021 - March 2021
I started working on a small [feature](https://bugs.llvm.org/show_bug.cgi?id=46164) in llvm. This feature was about
adding compound assignment operators, +=, -=, and *=, for matrix data types. The llvm compiler used to crash when
these operators were used. To debug it, I took the core dump of the crash and debugged it using lldb. This was the
first time I debugged a core dump. This work was commited in llvm in March.

In March, I got a mentor. I signed up for [SIGPLAN-M](https://www.sigplan.org/LongTermMentoring/), where newbies can
get mentorship from experienced folks. Me and my mentor started meeting every three months. I finally found someone
I could talk to about these things on a regular basis.

## April 2021 - June 2021
I continued working on llvm matrix types. In May, I got commit access to llvm, which increased my confidence that I
could work in this area.

So far, my work in llvm, and my knowledge of compilers, was focussed towards the front-end part of the compiler. I
thought about two choices at this point: continue working on the llvm front-end or build my background in compiler
middle-end. I chose the second alternative.

I started building my middle-end background by going through a Cornell [course](https://www.cs.cornell.edu/courses/cs6120/2020fa/self-guided/).
In the intro of this course, the instructor talked about the idea of domain-specific-architectures. I realised that
I can approach my matrix problem as designing a domain specific language for a domain specific archietcture.

## July 2021 - October 2021
I continued building my middle-end background from the Cornell course. From this course, I learned more about llvm
intermediate representation, llvm IR.

Around this time, I found the [computer architecture podcast](https://open.spotify.com/show/6fff71ppMFLosWyquhibcU).
I only had a rudimentary understanding of computer architecture so I couldn't understand a lot of the content. But
it exposed me to the kinds of problems people are interested in and what new opportunities are opening up for people
with a background in programming languages.

At the same time, I was clarifying my matrix language problem and looking for a possible direction of attack.
My idea now was to construct a matrix-specific IR so that I could implement some matrix-specific optimisations. This
strategy of building a domain specific IR was also pursued in an llvm project, [mlir](https://mlir.llvm.org). So it
gave me some confidence that this could be the right path to follow.

Also at this time, I started building my mathematical deep learning background. I thought it would help me in my
matrix problem. I started studying this [book](https://math.mit.edu/~gs/learningfromdata/). I didn't read beyond the
first 20% of this book but I got an important idea from it. I got to know about decomposition of a matrix
factorisation into rank one components.

## November 2021 - January 2022
I wasn't able to find a concrete attack to my matrix problem and was getting discouraged by the lack of progress
there. I was also having a hard time getting compiler jobs. So I shelved my matrix problem and started looking for
something where I could have impact in short term. With that in mind, I started looking at llvm mlir.

I picked up [benchmarking sparse dialect](https://discourse.llvm.org/t/sparse-compiler-starter-projects/4588) of mlir.
This turned into a [general framework](https://discourse.llvm.org/t/mlir-benchmarks/4945/3) for benchmarking in mlir.
I spent most of the Nov '21 - Jan '22 on mlir.

This was a very good experience and it exposed me to how people are designing domain specific IRs for specific
problems.

Around the time the mlir work was finishing, I found a direction of attack for my matrix problem. My idea now was
to design and implement a rank-one based IR for matrices.

## August 2022
I continued working on my matrix problem till about August. My strategy was to express various matrix factorisations
as decompositions into rank one components.

Around August, I became disillusioned with this approach. I spent many months with no progress. Moreover, since a
lot of people were already working on similar problems, I wasn't sure if I could come up with anything that is
different from what other people were doing. Gradually, I stopped working on the matrix problem altogether.

I was getting more and more interested in computational biology. One origin of my interests was this computer
architecture podcast [episode](https://open.spotify.com/episode/0LMeEQMdYPL8iELp3LzrLC),
which discussed domain specific architectures for genomics. Another origin was the unfortunate Covid-19 pandemic,
specifically the Omicron wave in Dec '21 - Jan '22. These interests came to the forefront for me as I was looking
for a new problem to work on. My new goal now was to understand biology from the lens of programming languages.

My thinking was I had enough open source contributions to get a compiler job so I could now take some time to
study biology. So I started building my biology background.

## August 2023
I finally started a job where I could work on compilers full time! I was very pleased because, instead of
talking about these things once every few months, I could talk about these things five days a week.
