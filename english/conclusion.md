> Voilà, c'est fini ...
Source:[Jean-Louis Aubert, *Bleu Blanc Vert*, 1989]

... for this introduction to the proof of C programs using Frama-C and WP.

Along this tutorial, we have seen how we can use these tools to specify what
we except of our programs and verify that the source code we have produced
indeed corresponds to the specification we have provided. This specification
is provided using annotations of our functions that includes the contract they
must respect. These contracts are properties required about the input to ensure
that the function will correctly work, which is specified by properties about
the output of the function.

Starting from specified programs, WP is able to produce the weakest
precondition of our programs, provided what we want in postcondition, and to
ask some provers if the specified precondition is compatible with the computed
one. The reasoning is completely modular, which allows to prove function in
isolation from each other and to compose the results. WP cannot currently
work with dynamic allocation. A function that would use it could not be proved.

However, even without dynamic allocation, a lot of function can be proved since
they work with data-structures that are already allocated. And these functions
can then be called with the certainty that they perform a correct job. If we
cannot or do not want to prove calling code, we can still write something like
this:

```c
/*@
  requires some_properties_on(a);
  requires some_other_on(b);

  assigns ...
  ensures ...
*/
void ma_fonction(int* a, int b){
  //this is indeed the "assert" defined in "assert.h"
  assert(/*properties on a*/ && "must respect properties on a");  
  assert(/*properties on b*/ && "must respect properties on b");
}
```

Which allows us to benefit from the robustness of our function having the
possibility to debug and incorrect call in a source code that we cannot or
do want to prove.

Writing specifications is sometimes long or tedious. Higher-level features
of ACSL (predicates, logic functions, axiomatizations) allow us to lighten
this work, as well as our programming languages allow us to define types
containing other types, functions call functions, bringing us to the final
program. But, despite this, write specification in a formal language, no
matter which one, is generally a hard task.

However, this **formalization** of our need is **crucial**. Concretely, such
a formalization is a work every developer should do. And not only in order
to prove a program. Even the definition of tests for a function requires to
correctly understand its goal if we want to test what is necessary and only
what is necessary. And writing specification in a formal language is
incredibly useful (even if it can be sometimes frustrating) to get a clear
specification.

Formal languages, that are close to mathematics, are precise. Mathematics have
this: they are precise. What is more terrible that reading a specification
written in a natural language, with complex sentences, using conditional forms,
imprecise terms, ambiguities, compiled in administrative documents composed of
hundreds of pages, and where we need to determine, "so, what this function is
supposed to do ? And have I to validate about it ?".

Formal methods are probably not enough used currently. Sometimes because of
mistrust, sometimes because of ignorance, sometimes becuase of prejudices
based on ideas born at the begining of the tools, 20 years ago. Our tools
evolve, the way we develop change, probably faster than in any other technical
domain. Saying that these tools could never be used for real life programs
would certainly be a too big shortcut. After all, we see everyday how much
development is different from what it were 10 years, 20 years, 40 years ago
and can barely imagine how much it will be different in 10 years, 20 years,
40 years.

During the past few years, safety and security questions have become more and
more actual and crucial. Formal methods also progressed a lot, and the
improvement they bring for these questions are greatly appreciated. For example,
this [hyperlink](http://sfm.seas.harvard.edu/report.html) brings to the report
of a conference about security that brought together people from academic and
industrial world, in which we can read:

> Adoption of formal methods in various areas (including verification of hardware
> and embedded systems, and analysis and testing of software) has dramatically 
> improved the quality of computer systems.  We anticipate that formal methods 
> can provide similar improvement in the security of computer systems.
>
> [...]
>
> **Without broad use of formal methods, security will always remain fragile.**
Source:[Formal Methods for Security, 2016]

# Going further

## With Frama-C

Frama-C provides different ways to analyse programs. In these tools, the most
commonly used and interesting to know from a static and dynamic verification
point of view are certainly those ones:

- abstract interpretation analysis using [EVA](http://frama-c.com/value.html),
- the transformation of annotation into runtime verification using
  [E-ACSL](http://frama-c.com/eacsl.html).

The goal of the first one is to compute the domain of the different variables at
each program point. When we precisely know these domains, we can determine if
these variables can produce errors when they are used. However this analysis is
executed on the whole program and not modularly, it is also stongly dependent of
the type of domain we use (we will not enter into details here) and it is not
so good at keeping the relations between variables. On the other side, it is
really completely automatic, we do not even need to give loop invariant ! The
most manual part of the work is to determine whether or not an alarm is a true
error or a false positive.

The second analysis allows to generate from an original program, a new program
where the assertions are transformed into runtime verifications. If these
assertions fail, the program fails. If they are valid, the program have the same
behavior it would have without the assertions. An example of use is to generate
the verification of absence of runtime errors as assertions using `-rte` and
then to use E-ACSL to generate the program containing the runtime verification
that these assertions do not fail.

There exists different other plugins for very different tasks:

- impact analysis of a modification,
- data-flow analysis to visit it efficiently,
- ...

Finally, a last possibility that will motivate the use of Frama-C is the ability
to develop their own analysis plugins using the API provided by the Frama-C
kernel. A lot of tasks can be realized by the analysis of the source code and
Frama-C allows to build different analyses.

## With deductive proof

Along this tutorial we used WP to generate proof obligation starting from
programs with their specification. Next we have used automatic solvers to assure
that these properties were indeed verified.

When we use other solvers than Alt-Ergo and Coq, the communication with this
solver in provided by a translation to the Why3 language that will next be used
to bridge the gap to automatic solvers. But this is not the only way to use Why3.
It can also be used itself to write programs and prove them. It especially
provides a set of theories for some common data structures.

There is some proofs that cannot be discharged by automatic solvers. In such a
case, we have to provide these proofs interactively. WP, like Why3, can extract
its goals to Coq, and it is very interesting to study this language. In the
context of Frama-C, we produce lemmas libraries already proved that we can reuse.
But Coq can also be used for many different tasks, including programming. Note
that Why3 can also extract its proof obligations to Isabelle or PVS that are
also proof assistants.

Finally, there exists other program logics, for example separation logic or
concurrent program logics. Again this notion are interesting to know in the
context of Frama-C: if we cannot directly use them, they can inspire the way
we specify our program in Frama-C for the proof with WP. They could also be
implemented into new plugins to Frama-C.

A whole new world of methods to explore.
