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


## Lists

We can use pairs to build lots of different data structures, but the 
most fundamental way they're used is to make lists, as follows:

1. The symbol `nil` represents the empty list.

2. If y is a list, then the pair `(x . y)` is a list of `x` followed by 
   the elements of `y`.

Here's a list made from a pair:

    (a . nil)

According to rule 2, this is a list of the symbol `a` followed by the
elements of `nil`, of which according to rule 1 there are none. So it 
is a list of one element, the symbol `a`.

By nesting such pairs we can create lists of any length. Here is a
list of two elements, the symbols `a` and `b`:

    (a . (b . nil))

And here is a list of `a`, `b`, and `c`:

    (a . (b . (c . nil)))

This would be an awkward way to express lists, but there is an
abbreviated notation that's more convenient:

1. The symbol `nil` can also be represented as `()`.

2. When the second half of a pair is a list, you can omit the dot 
   before it and the parentheses around it. So `(a . (b ...))` can 
   be written as `(a b ...)`.

By repeated application of these two rules we can transform

    (a . (b . (c . nil)))

into

    (a b c)

In other words, a list can be expressed as its elements within
parentheses. You wouldn't use dot notation for a list like `(a b c)`
unless there was some special reason to.

Because any object can be part of a pair, the elements of lists can
themselves be lists. All these are lists too:

    (a (b) c)

    ((a b c))

    (nil)

Pairs like these, where if you keep looking at the second half you
eventually hit a nil, are called proper lists. This is a proper
list:

    (a b c)

and this is not:

    (a b . c)

The empty list is also a proper list.

A pair that's not a proper list is called a dotted list (because
you need to use dot notation to represent it).

A proper list of characters is called a string, and can also be
represented as those characters within double-quotes. So the list

    (\h \e \l \l \o)

can also be represented as

    "hello"

and will when possible be displayed that way.


## Truth

The symbol **nil** represents falsity as well as the empty list. The 
symbol **t** is the default representation for truth, but any object
other than **nil** also counts as *true*.

It may seem strange to use the same value to represent both falsity
and the empty list, but in practice it works well. Lisp functions
often return sets of answers, and the empty set of answers is 
falsity.


## Functions

Bel programs consist mostly of functions. Functions take zero or more
objects as arguments, perhaps do something (e.g. print a message), 
and return one object.

Functions are represented using lists. For example, here is a
function that takes one argument, and returns that plus 1.

    (lit clo nil (x) (+ x 1))

The first element, **lit**, says that this is a literal object, not to be 
evaluated.

The second, **clo**, says what kind of literal it is: a closure.

The third is the local enviroment, a list of variables that already
have values from having been parameters of functions. This example 
has an empty environment.

The fourth, **(x)**, is the function's parameters. When the function is 
called, the value of **x** will be whatever it was called with.

The fifth and last element, **(+ x 1)**, defines the value that the
function returns.

You would not ordinarily express a function using its literal 
representation. Usually you'd say

    (fn (x) (+ x 1))

which yields the function above.


## Evaluation

The execution of a Bel program consists of the evaluation of
expressions. All Bel objects are expressions, so the word "expression"
is merely a statement of intention: it means an object that you
expect to be evaluated.

When an expression is evaluated, there are three possible outcomes:

1. It can return a value: `(+ 1 2)` returns `3`.

2. It can cause an error: `(/ 1 0)` will.

3. It can fail to terminate: `(while t)` will.

Some expressions also do things in the process of being evaluated.
For example,

    (prn 1)

will return `1`, but before doing so will print it.

Some atoms evaluate to themselves. All characters and streams do,
along with the symbols `nil`, `t`, `o`, and `apply`. All other symbols
are variable names, and either evaluate to some value, or cause an
error if they don't have a value.

A proper list whose first element evaluates to a function is called
a function call. For example, the expression

    (+ x 1)

is a function call, because the value of `+` is a function. The value 
of a function call is the object that the function returns.

Function calls are evaluated left to right. For example, when

    (+ 8 5)

is evaluated,

1. First `+` is evaluated, returning a function that returns the sum
   of its arguments.

2. Then `8` is evaluated, returning itself.

3. Then `5` is evaluated, also returning itself.

4. Finally, the two numbers are passed to the function, which returns `13`.

If we want to show what expressions evaluate to, it's conventional to 
show them being evaluated in a [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop):

    > (+ 8 5)
    13

Expressions can be nested. The rule of evaluating left to right means 
that nested function calls are evaluated depth-first. For example, 
when

    (+ (- 5 2) 7)

is evaluated, the subexpressions that get evaluated are, in order,
`+`, `(- 5 2)`, `-`, `5`, `2`, `7`.

Not all expressions are evaluated left to right, however. There is a 
small set of symbols called special forms, and an expression whose
first element is a special form is evaluated according to rules
defined for that special form.

For example, `if` is a special form, and when an expression of the form

    (if test then else)

is evaluated, only one of the last two elements will be evaluated,
depending on whether the second element, `test`, returns true or not.

Things meant to be used as the first element of an expression are
called operators. So functions and special forms are operators. But
like the term "expression," this is just a statement of intention.
You can put anything first in an expression so long as you specify 
what happens when you do.


## Bindings and Environments

There are three ways a variable can have a value. It can have a value 
**globally**, as for example `+` does, meaning that by default it has this 
value everywhere. Such a variable is said to be *globally bound*, and 
the set of global bindings is called the *global environment*.

Another way a variable can have a value is by being a *parameter* in a 
function. When the function

    (fn (x) (+ x 1))

is called, the variable `x` will, within it, have as its value whatever 
argument the function was called with. That's called a lexical 
binding, and the current set of lexical bindings is the lexical 
environment. 

Finally, variables can have *dynamic* bindings, which are visible
everywhere, like global bindings, but temporary: they persist only
during the evaluation of whatever expression created them.

Dynamic bindings take precendence over lexical bindings, which take 
precedence over global ones.

If you do an assignment to a variable that has one of the three kinds 
of bindings, you'll modify whichever binding is currently visible. If 
you do an assignment to a variable that's not bound, you'll create a 
global binding for it.


## Errors

Errors are signalled by calling `err` with one argument describing the 
error. Bel doesn't specify a global binding for `err`; this is 
something for a programming environment built on top of Bel to do. 
But some error-catching code in the Bel source does dynamically bind 
`err`.


## Axioms

Like McCarthy's Lisp, Bel is defined starting with a set of operators
that we have to assume already exist. Then more are defined in terms 
of these, till finally we can define a function that is a Bel 
interpreter, meaning a function that takes a Bel expression as an 
argument and evaluates it.

There are two main types of axioms: primitives and special forms.
There are also a few variables that come predefined.

In the following sections, the descriptions of primitives are their
definitions. The descriptions of special forms, however, are merely
summaries of their behavior; special forms are defined by the code
that implements them in the Bel source.


### Variables and Constants

1. `t` `nil` `o` `apply`

Evaluate to themselves.

2. `chars`

A list of all characters. Its elements are of the form `(c . b)`, where
`c` is a character and `b` is its binary representation in the form of a 
string of `\1` and `\0` characters. Bel doesn't specify which characters
are in chars, but obviously they should include at least those in the 
Bel source.

3. `globe` `scope`

The current global and lexical environments, represented as lists of 
`(var . val)` pairs of variables and their values.

4. `ins` `outs`

The default input and output streams. Initially `nil`, which represents
the initial input and output streams. Bel doesn't specify what those
are, but if you started Bel at a prompt you'd expect them to be the
terminal.


### Quote

The `quote` operator is a special form, but it has to be described 
first because so many code examples use it.

It returns its argument without evaluating it. Its purpose is to 
prevent evaluation.

    > (quote a)
    a

Prepending `'` to an expression is equivalent to wrapping a quote
around it.

    > 'a
    a

Why do you need to prevent evaluation? To distinguish between code
and data. If you want to talk about the symbol `a`, you have to quote
it. Otherwise it will be treated as a variable, and you'll get its
value. E.g. if `a` has been set to `10`:

    > a
    10
    > 'a
    a

Because the symbols `nil`, `t`, `o`, and `apply` evaluate to themselves, you 
don't have to quote them. Ditto for strings.


### Primitives

Primitives can be called like functions, but are assumed to exist, 
rather than defined in the Bel source. As with function calls, the
arguments in calls to primitives are all evaluated, left to right. 
Missing arguments default to nil. Extra arguments cause an error to 
be signalled.


1.  `(id x y)`

Returns true iff `x` and `y` are identical. 

    > (id 'a 'a)
    t
    > (id 'a 'b)
    nil

Identity is stricter than equality. While there is only one of each
symbol and character, there can be any number of different pairs with 
the same elements. So two pairs can look the same without being
identical:

    > (id '(a b) '(a b))
    nil

Because `id` is so strict, it's not the function you'd usually use to 
test for equality. Usually you'd use `=`.


2.  `(join x y)`

Returns a new pair whose first half is `x` and second half is `y`.

    > (join 'a 'b)
    (a . b)
    > (join 'a)
    (a)

A pair returned by `join` will not be `id` to any existing pair.

    > (id (join 'a 'b) (join 'a 'b))
    nil


3.  `(car x)`

Returns the first half of a pair:

    > (car '(a . b))
    a
    > (car '(a b))
    a

The `car` of `nil` is `nil`,

    > (car nil)
    nil

but calling `car` on any atom other than `nil` will cause an error.

The name "car" is McCarthy's. It's a reference to the architecture of 
the first computer Lisp ran on. But though the name is a historical 
accident, it works so well in practice that there's no reason to 
change it.


4.  `(cdr x)`

Returns the second half of a pair:

     > (cdr '(a . b))
    b
    > (cdr '(a b))
    (b)

As with `car`, calling it on `nil` yields `nil`, calling it on any other
atom causes an error, and the name is McCarthy's.

When operating on pairs used to represent lists, `car` and `cdr` get you 
the first element and the rest of the list respectively.


5.  `(type x)`

Returns either `symbol`, `pair`, `char`, or `stream` depending on the type
of `x`.

    > (type 'a)
    symbol
    > (type '(a))
    pair
    > (type \a)
    char


6.  `(xar x y)`

Replaces the `car` of `x` with `y`, returning `y`. Signals an error if `x` is 
not a pair.

If we assume that the value of `x` is `(a . b)`, then

    > x
    (a . b)
    > (xar x 'c)
    c
    > x
    (c . b)


7.  `(xdr x y)`

Like `xar`, except that it replaces the `cdr` of `x`.

    > x
    (c . b)
    > (xdr x 'd)
    d
    > x
    (c . d)


8.  `(sym x)`

Returns the symbol whose name is the elements of `x`. Signals an error
if `x` is not a string.

    > (sym "foo")
    foo


9.  `(nom x)`

Returns a fresh list of the characters in the name of `x`. Signals an 
error if `x` is not a symbol.

    > (nom 'foo)
   "foo"


10. `(wrb x y)`

Writes the bit `x` (represented by either `\1` or `\0`) to the stream `y`. 
Returns `x`. Signals an error if it can't or if `x` is not `\1` or `\0`. If `y` 
is `nil`, writes to the initial output stream. 


11. `(rdb x)`

Tries to read a bit from the stream `x`. Returns `\1` or `\0` if it finds 
one, `nil` if no bit is currently available, or `eof` if no more will be 
available. Signals an error if it can't. If `x` is `nil`, reads from the 
initial input stream.


12. `(ops x y)`

Returns a stream that writes to or reads from the place whose name is 
the string `x`, depending on whether `y` is `out` or `in` respectively. 
Signals an error if it can't, or if `y` is not `out` or `in`.


13. `(cls x)`

Closes the stream `x`. Signals an error if it can't.


14. `(stat x)`

Returns either `closed`, `in`, or `out` depending on whether the stream `x` 
is closed, or reading from or writing to something respectively. 
Signals an error if it can't.


15. `(coin)`

Returns either `t` or `nil` randomly.


16. `(sys x)`

Sends `x` as a command to the operating system. 


### Special Forms

Expressions beginning with special forms are not always evaluated in 
the usual left-to-right way.


1.  `(quote x)`

Described above.


2.  `(lit ...)`

Returns the whole `lit` expression without evaluating it. A `lit` is like
a persistent quote; evaluation strips the `quote` off a quote 
expression, but leaves a `lit` expression intact.

    > (quote a)
    a
    > (lit a)
    (lit a)

The name stands for literal, and it can take any number of arguments.
This is how you make things that evaluate to themselves, the way
characters or `nil` do. Functions are lits, for example, as are
numbers.

The value of a primitive `p` is `(lit prim p)`. 

    > car
    (lit prim car)


3.  `(if ...)`

An `if` expression with an odd number of arguments

    (if a1 a2 a3 a4 ... an)

is equivalent to

    if a1 then a2 else if a3 then a4 ... else an

I.e. the odd numbered arguments are evaluated in order till we either
reach the last, or one returns true.  In the former case, its value
is returned as the value of the if expression. In the latter, the
succeeding argument is evaluated and its value returned.

An `if` expression with an even number of arguments

    (if a1 a2 ... an)

is equivalent to

    (if a1 a2 ... an nil)

Falsity is represented by `nil`, and truth by any other value. Tests 
generally return the symbol `t` when they can't return anything more 
useful.

As a rule I've tried to make axioms as weak as possible. But while
a 3-argument if would have sufficed, an n-argument version didn't
require significantly more code, so it seemed gratuitously fussy to 
insist on 3 arguments.


4.  `(apply f ...)`

An expression of the form

    (apply f x y ... z)

is equivalent to

    (f 'a 'b ... 'c1 ... 'cn)

where `a` is the value of `x`, `b` the value of `y`, and the `ci` the elements 
of the value of `z`.

    > (join 'a 'b)
    (a . b)
    > (apply join '(a b))
    (a . b)
    > (apply join 'a '(b))
    (a . b)

The last argument to apply can be a dotted list if it matches the 
parameters of the first.


5.  `(where x)`

Evaluates `x`. If its value comes from a pair, returns a list of that
pair and either `a` or `d` depending on whether the value is stored in
the `car` or `cdr`. Signals an error if the value of `x` doesn't come from 
a pair.

For example, if `x` is `(a b c)`,

    > (where (cdr x))
    ((a b c) d)


6.  `(dyn v x y)`

Evaluates `x`, then causes `y` to be evaluated with the variable `v`
dynamically bound to the value of `x`. 

For example, if `x` is `a`,

    > x
    a
    > (dyn x 'z (join x 'b))
    (z . b)
    > x
    a


7.  `(after x y)`

Evaluates both its arguments in order. The second will be evaluated
even if the evaluation of the first is interrupted (e.g. by an
error). 


8.  `(ccc f)`

Evaluates `f` and calls its value on the current continuation. The
continuation, if called with one argument, will return it as the 
value of the ccc expression (even if you are no longer in the ccc 
expression or in code called by it).


9.  `(thread x)`

Starts a new thread in which `x` will be evaluated. Global bindings are 
shared between threads, but not dynamic ones.


