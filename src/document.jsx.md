# The Bel Language

### Paul Graham, 12 Oct 2019

In 1960 John McCarthy described a new type of programming language
called Lisp. I say "new type" because Lisp represented not just a new
language but a new way of describing languages. He defined Lisp by 
starting with a small set of operators, akin to axioms, and then 
using them to write an interpreter for the language in itself.

His goal was not to define a programming language in our sense of
the word: a language used to tell computers what to do. The Lisp
in his 1960 paper was meant to be a formal model of computation,
like a Turing Machine. McCarthy didn't realize it could be used on
computers till his graduate student Steve Russell suggested it.

The Lisp in the 1960 paper was missing features you'd need in a 
programming language. There were no numbers, for example, or errors, 
or I/O. So when people used it as the basis for languages used to 
program computers, such things had to be added. And when they were, 
the axiomatic approach was dropped.

So the development of Lisp happened in two parts (though they seem 
to have been interleaved somewhat): a formal phase, represented by 
the 1960 paper, and an implementation phase, in which this language
was adapted and extended to run on computers. Most of the work, as
measured by features, took place in the implementation phase. The
Lisp in the 1960 paper, translated into Common Lisp, is only 53 lines 
of code. It does only as much as it has to in order to interpret 
expressions. Everything else got added in the implementation phase.

My hypothesis is that, though an accident of history, it was a good 
thing for Lisp that its development happened in two phases-- that
the initial exercise of defining the language by writing an 
interpreter for it in itself is responsible for a lot of Lisp's best 
qualities. And if so, why not do more of it?

Bel is an attempt to answer the question: what happens if, instead of
switching from the formal to the implementation phase as soon as 
possible, you try to delay that switch for as long as possible? If 
you keep using the axiomatic approach till you have something close 
to a complete programming language, what axioms do you need, and what
does the resulting language look like?

I want to be clear about what Bel is and isn't. Although it has a lot
more features than McCarthy's 1960 Lisp, it's still only the product
of the formal phase. This is not a language you can use to program 
computers, just as the Lisp in the 1960 paper wasn't. Mainly because, 
like McCarthy's Lisp, it is not at all concerned with efficiency. 
When I define append in Bel, I'm saying what append means, not trying 
to provide an efficient implementation of it.

Why do this? Why prolong the formal phase? One answer is that it's
an interesting exercise in itself to see where the axiomatic approach 
leads. If computers were as powerful as we wanted, what would 
languages look like?

But I also believe that it will be possible to write efficient
implementations based on Bel, by adding restrictions. If you want a 
language with expressive power, clarity, and efficiency, it may work
better to start with expressive power and clarity, and then add 
restrictions, than to approach from another direction.

So if you'd like to try writing an implementation based on Bel, 
please do. I'll be one of your first users.

I've ended up reproducing a number of things in previous dialects.
Either their designers got it right, or I'm too influenced by
dialects I've used to see the right answer; time will tell. I've also 
tried to avoid gratuitous departures from existing Lisp conventions. 
Which means if you see a departure from existing conventions, there 
is probably a reason for it.


## Data

Bel has four fundamental data types: **symbols**, **pairs**, **characters**, and **streams**.

Symbols are words:

    foo

The names of symbols are case-sensitive, so `foo` and `Foo` are distinct
symbols.

Pairs are pairs of any two things, and are represented thus:

    (foo . bar)

That's a pair of two symbols, `foo` and `bar`, but the two halves of a
pair can be anything, including pairs:

    (foo . (bar . baz))

That's a pair of the symbol `foo`, and the pair `(bar . baz)`.

A character is represented by prepending a backslash to its name. So 
the letter a is represented as

    \a

Characters that aren't letters may have longer names. For example the
bell character, after which Bel is named, is

    \bel

There is no way of representing a stream. If Bel has to display a
stream, it prints something that will cause an error if it's read
back in.

Anything that's not a pair is called an **atom**. So **symbols**, **characters**,
and **streams** are **atoms**.

Instances of the four fundamental types are called **objects**.

