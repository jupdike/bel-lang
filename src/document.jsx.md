# The Bel Language

### [Paul Graham](https://www.paulgraham.com/bel.html "Bel - Paul Graham's website")
### 12 Oct 2019

In 1960 [John McCarthy described a new type of programming language
called Lisp](https://paulgraham.com/rootsoflisp.html "The Roots of Listp - Paul Graham's website").
I say "new type" because Lisp represented not just a new
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
When I define `append` in Bel, I'm saying what `append` means, not trying 
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

There are three ways a variable can have a value. It can have a value **globally**,
as for example `+` does, meaning that by default it has this 
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


# Reading the Source

Starting with the foregoing 25 operators, we're going to define more,
till we can define a Bel interpreter. Then we'll continue, defining
numbers, I/O, and several other things one needs in programs.

These definition are in the Bel source, which is meant to be read in
parallel with this guide.

In the Bel source, when you see an expression of the form 

    (set v1 e1 ... vn en)

it means each `vi` is globally bound to the value of `ei`.

In the source I try not to use things before I've defined them, but 
I've made a handful of exceptions to make the code easier to read.

### `def` and `mac` Abbreviations

When you see

    (def n p e)

treat it as an abbreviation for 

    (set n (lit clo nil p e))

and when you see 

    (mac n p e)

treat it as an abbreviation for

    (set n (lit mac (lit clo nil p e)))

The actual `def` and `mac` operators are more powerful, but this is as 
much as we need to start with.

### Square-Bracket functions

Treat an expression in square brackets, e.g.

    [f _ x]

as an abbreviation for 

    (fn (_) (f _ x))

In Bel, underscore is an ordinary character and `_` is thus an ordinary 
variable.

### Backquote

Finally, treat an expression with a prepended backquote `(\`)` as a
quoted list, but with "holes," marked by commas, where evaluation 
is turned back on again. 

    > (set x 'a)
    a
    > `(x ,x y)
    (x a y)
    > `(x ,x y ,(+ 1 2))
    (x a y 3)

You can also use `,@` to get a value spliced into the surrounding
list:

    > (set y '(c d))
    (c d)
     > `(a b ,@y e f)
    (a b c d e f)

## Bel in Bel Source

(Actual code is shown in color, commentary in black and white. For a raw text
version of the original source code, see the link on [Paul Graham's page about Bel](https://www.paulgraham.com/bel.html "Bel - Paul Graham's website")).

<Bel>

    (def <dfn>no</dfn> (x)
      (id x nil))
</Bel>

Now let's look at the source. The first expression defines a function
no that takes one argument, `x`, and returns the result of using `id` to
compare it to `nil`. So `no` returns `t` if its argument is `nil`, and `nil` 
otherwise.

    > (no nil)
    t
    > (no 'a)
    nil

Since `nil` represents both falsity and the empty list, `no` is both
logical negation and the test for the empty list.

<Bel>

    (def <dfn>atom</dfn> (x)
      (no (id (type x) 'pair)))
</Bel>

The second function, `atom`, returns true [iff](https://en.wikipedia.org/wiki/If_and_only_if) its argument is not a 
pair. 

    > (atom \a)
    t
    > (atom nil)
    t
    > (atom 'a)
    t
    > (atom '(a))
    nil

<Bel>

    (def <dfn>all</dfn> (f xs)
      (if (no xs)      t
          (f (car xs)) (all f (cdr xs))
                       nil))

    (def <dfn>some</dfn> (f xs)
      (if (no xs)      nil
          (f (car xs)) xs
                       (some f (cdr xs))))
</Bel>

Next come a pair of similar functions, all and some. The former 
returns t iff its first argument returns true of all the elements of 
its second,

    > (all atom '(a b))
    t
    > (all atom nil)
    t
    > (all atom '(a (b c) d))
    nil
 
and the latter returns true iff its first argument returns true of 
any element of its second. However, when some returns true, it 
doesn't simply return `t`. It returns the remainder of the list 
starting from the point where `f` was true of the first element. 

    > (some atom '((a b) (c d)))
    nil
    > (some atom '((a b) c (d e)))
    (c (d e))

Logically, any value except `nil` counts as truth, so why not return 
the most informative result you can?

### More about `if`

In all and some we see the first use of `if`. Translated into English, 
the definition of all might be:

> If `xs` is empty, then return `t`. 

> Otherwise if `f` returns true of the first element, return the result 
> of calling all on `f` and the remaining elements. 

> Otherwise (in which case `xs` has at least one element of which `f`
> returns false), return `nil`.

This technique of doing something to the `car` of a list and then 
perhaps continuing down the `cdr` is very common.

Something else is new in `all` and `some`: these are the first functions
in the Bel source that you could cause an error by calling.

    > (all atom 'a)
    Error: can't call car on a non-nil atom.

I made up that error message; Bel doesn't specify more about errors
in primitives than when they occur, and doesn't specify anything 
about repls. But some error will be signalled if you call `all` with a
non-nil atom as the second argument, because in the second test 
within the if

    (f (car xs))

`car` is called on it, and it's an error to call `car` on anything except
a pair or `nil`.

One other thing to note about these definitions, now that we're
getting to more complex ones: these functions are not defined the
way they would be in idiomatic Bel. For example, if all didn't 
already exist in Bel you could define it as simply

    (def all (f xs)
      (~some ~f xs))

But since we haven't defined functional composition yet, I didn't use 
it.

<Bel>

    (def <dfn>reduce</dfn> (f xs)
      (if (no (cdr xs))
          (car xs)
          (f (car xs) (reduce f (cdr xs)))))
</Bel>

The next function, reduce, is for combining the elements of its 
second argument using nested calls to its first. For example 

    (reduce f '(a b c d))

is equivalent to

    (f 'a (f 'b (f 'c 'd)))

If `xs` has only one element, reduce returns it, and if it's empty,
`reduce` returns `nil`; since `(cdr nil)` is `nil`, we can check both these 
possibilities with `(no (cdr xs))`. Otherwise it calls `f` on the first
element and `reduce` of `f` and the remaining elements.

    > (reduce join '(a b c))
    (a b . c)

This is not the only way to reduce a list. Later we'll define two 
more, `foldl` and `foldr`.

### Indenting `if`s

The definition of reduce shows another way of indenting `if`s. 
Indentation isn't significant in Bel and only matters insofar as 
it helps humans read your code, but I've found three ways of 
indenting `if`s that work well. If an `if` has more than two tests and 
the arguments are sufficiently short, it works well to say

    (if test1 then1
        test2 then2
              else)

We saw this in all and some. But if you only have one test, or some 
arguments are too long to fit two on one line, then it works better 
to say

    (if test1
        then1
        test2
        then2
        else)

or if an `if` is long, 

    (if test1
         then1
        test2
         then2
        test3
         then3
         else)

<Bel>

    (def <dfn>cons</dfn> args
      (reduce join args))
</Bel>

The next function, `cons`, has the name that `join` had in McCarthy's
Lisp. It's the function you use to put things on the front of a list.

    > (cons 'a '(b c))
    (a b c)

If you only want to put one thing on the front of a list, you could
use `join`.

    > (join 'a '(b c))
    (a b c)

With `cons`, however, you can supply more than one thing to put on the
front:

    > (cons 'a 'b 'c '(d e f))
    (a b c d e f)

Since `cons` is a generalization of `join`, it's rare to see `join` in
programs.

### Function parameter forms

We see something new in the definition of `cons`: it has a single 
parameter, args, instead of a list of parameters. When a function has 
a single parameter, its value will be a list of all the arguments 
supplied when the function is called. So if we call `cons` thus

    (cons 'a 'b '(c d))

the value of `args` will be

    (a b (c d))

The parameter list of a Bel function can be a tree of any shape. If 
the arguments in the call match its shape, the parameters will get 
the corresponding values; otherwise an error is signalled.

So for example if a function `f` has the parameter list

    (x . y) 

and it's called

    (f 'a 'b 'c)

then `x` will be `a`, and `y` will be `(b c)`.

If the same function is called

    (f 'a)

then `x` will be `a`, and `y` will be `nil`. And if it's called

    (f)

you'll get an error because there is no value for `x`.

If a function `f` has the parameter list 

    ((x y) z)

and is called

    (f '(a (b c)) '(d))

then `x` will be a, `y` will be `(b c)`, and `z` will be `(d)`. Whereas if it's 
called

    (f '(a) '(d))

you'll get an error because there is no value for `y`, and if it's 
called

    (f '(a b c) '(d))

you'll get an error because there is no parameter for `c`.


<Bel>

    (def <dfn>append</dfn> args
      (if (no (cdr args)) (car args)
          (no (car args)) (apply append (cdr args))
                          (cons (car (car args))
                                (apply append (cdr (car args))
                                              (cdr args)))))
</Bel>

The next function, `append`, joins lists together:

    > (append '(a b c) '(d e f))
    (a b c d e f)
    > (append '(a) nil '(b c) '(d e f))
    (a b c d e f)

Its definition will be easier to understand if we look first at a 
two-argument version.

    (def append2 (xs ys)
      (if (no xs)
          ys
          (cons (car xs) (append2 (cdr xs) ys))))

In English, if `xs` is empty, then just return `ys`. Otherwise return the
result of `cons`ing the first element of `xs` onto `append` of the rest of 
`xs` and `ys`. I.e. 

    (append2 '(a b c) '(d e f))

becomes

    (cons 'a (append2 '(b c) '(d e f)))

and so on. The definition of append in the Bel source is the same 
principle applied to any number of arguments.

### More about `apply`

In it we see the first use of `apply`. Like `if`, `apply` is a special
form, meaning an operator whose behavior has to be defined as a
special case in the interpreter. Unlike if, apply has a value; it
evaluates to itself, like t and nil. This lets you use it as an
argument like an ordinary function. 

Its purpose is in effect to spread out the elements of a list as if 
they were the arguments in a function call. For example

    (apply f '(a b))

is equivalent to 

    (f 'a 'b)

In the general case `apply` can take one or more arguments, and is 
equivalent to calling `apply` on the first argument and all the 
intervening arguments `cons`ed onto the last. I.e.

    (apply f x y z)

is equivalent to

    (apply f (cons x y z))

It's common to use `apply` in functions like `append` that take any 
number of arguments. Using `apply` is in a sense the converse of using 
a single parameter to collect multiple arguments.

### Back to `append`

Now let's look at `append`. It takes any number of arguments. 
Collectively (i.e. as a list) they'll be the value of `args`. If `args` 
is empty or only has one element, then the result is `(car args)`. We 
saw the same sort of test in the first clause of `reduce`. That's two
base cases, and there is also a third: when `args` has more than one
element but the first element is `nil`. In that case we can ignore it,
and apply `append` to the rest of `args`.

Finally in the last clause we see the general case. It uses the 
same strategy we saw in `append2`: `cons` the first element of the
first argument onto a recursive call to `append` on the rest of the 
first argument and the remaining arguments. Unlike `append2`, `append` 
has to make this call using `apply`, because it has a varying number of
arguments in a list, instead of exactly two.


<Bel>

    (def <dfn>snoc</dfn> args
      (append (car args) (cdr args)))
       
    (def <dfn>list</dfn> args
      (append args nil))
</Bel>

Once we have append it's easy to define `snoc`, which as its name
suggests is like a reverse `cons`,

    > (snoc '(a b c) 'd 'e)
    (a b c d e)

and `list`, which returns a `list` of its arguments.

    > (list)
    nil
    > (list 'a)
    (a)
    > (list 'a 'b) 
    (a b)

Or more precisely, returns a newly made list of its arguments. If
you're wondering why we bother appending `args` to `nil` rather than 
simply returning it, the reason is that appending a list to `nil` will 
also copy it.

If we defined list as

    (def list args args)

and it was called thus

    (apply list x)

then the value that list returned would be the same list as `x`–not 
merely a list with the same elements, but the *same pair*–meaning if
we modified the value we got from list, we'd also be modifying the 
object up in the calling code.

<Bel>

    (def <dfn>map</dfn> (f . ls)
      (if (no ls)       nil
          (some no ls)  nil
          (no (cdr ls)) (cons (f (car (car ls)))
                              (map f (cdr (car ls))))
                        (cons (apply f (map car ls))
                              (apply map f (map cdr ls)))))
</Bel>

After list we see `map`, which in the simplest case returns a list of
calling its first argument on each element of its second.

    > (map car '((a b) (c d) (e f)))
    (a c e)

However, `map` can take any number of lists, and calls its first 
argument on successive sets of elements from the others.

    > (map cons '(a b c) '(1 2 3))
    ((a . 1) (b . 2) (c . 3))

It stops as soon as one list runs out

    > (map cons '(a b c) '(1 2))
    ((a . 1) (b . 2))

Like append, `map` is easier to understand if we start with a version 
that takes exactly two arguments.

    (def map2 (f xs)
      (if (no xs)
          nil
          (cons (f (car xs))
                (map2 f (cdr xs)))))

If there are no `xs` left, then return `nil`, otherwise `cons f` of the
first element onto `map2` of `f` and the remaining elements. Pretty 
simple.

All the additional complexity of `map` comes from the need to take 
multiple lists. The parameter list becomes `(f . ls)` so that all the 
lists can be collected in `ls`. We need an additional base case in case 
there are zero of them. Checking for the end of the list, which in 
`map2` was

    (no xs)

now becomes 

    (some no ls)

because we stop as soon as any list runs out.

Then we have yet another base case, the one in which we have just one
list. That's what `map2` does, and not surprisingly, the code is the 
same as in `map2` except that xs becomes `(car ls)`.

Finally in the general case we call `f` on all the first elements
(which we collect using `map`) and `cons` that onto `map` of `f` on all the 
rests of the lists.

Notice that `map` calls itself recursively in two ways: there is the
usual "do this to the rest of the list" recursive call in the last 
line. But in the preceding line we also use `(map car ls)` to collect 
the arguments for `f`. And that's why we need the single-list base 
case. Without it, we'd get an infinite recursion.


<Bel>

    (mac <dfn>fn</dfn> (parms . body)
      (if (no (cdr body))
          `(list 'lit 'clo scope ',parms ',(car body))
          `(list 'lit 'clo scope ',parms '(do ,@body))))

</Bel>

Next comes our first macro, `fn`. There are two concepts to explain 
first, though: **macros** and **scope**.

### Macros

A **macro** is essentially a function that generates code. I would have 
liked the first example of a macro to be something simpler, but `fn`
is the one we need first. So I'll introduce macros using a simpler 
macro that isn't part of Bel, then explain `fn`.

Here is a very simple macro:

    (mac nilwith (x)
      (list 'cons nil x))

This definition says that whenever you see an expression like

    (nilwith 'a)

transform it into 

    (cons nil 'a)

and then evaluate that and return its value as the value of the call 
to nilwith.

    > (nilwith 'a)
    (nil . a)

So unlike the evaluation of a function call, the evaluation of a 
macro call has two steps: 

1. First use the definition of the macro to generate an expression, 
   called the macro's expansion. In the case above the expansion is
   `(cons nil 'a)`.

2. Then evaluate the expansion and return the result. The expansion 
   above evaluates to `(nil . a)`.

Beneath the surface, what's going on is quite simple. A macro is in 
effect (or more precisely, contains) a function that generates 
expressions. This function is called on the (unevaluated) arguments 
in the macro call, and whatever it returns is the macro expansion.

For example, the value of `nilwith` will be equivalent to

    (lit mac (lit clo nil (x) (list 'cons nil x))))

If we look at the third element, we see the function that generates 
expansions

    (lit clo nil (x) (list 'cons nil x))

which looks just like the definition of `nilwith`.

Macros often use `backquote` to make the code that generates
expressions look as much as possible like the resulting expressions.
So if you defined `nilwith` you'd probably do it not as we did above
but as

    (mac nilwith (x)
      `(cons nil ,x))

### Simpler example: `fn-`

Now let's work our way up to `fn`, starting with the following 
simplified version:

    (mac fn- (parms expr)
      `(lit clo nil ,parms ,expr))

This is less powerful than the actual `fn` macro in two ways. 

1. It doesn't capture the local lexical environment, but instead
   simply inserts a nil environment.

2. It can only take a single expression.

But if we don't need either of these things, the functions made by
`fn-` work fine:

    > ((fn- (x y) (cons x y)) 'a 'b)
    (a . b)

All the extra complexity in the definition of `fn` is to get those two 
features, the local environment and a body of more than one 
expression.

### Closures

A function with a lexical environment stored within it is called a 
**closure**. That's why literal Bel functions begin `(lit clo ...)`; the
`clo` is for "closure." If a closure includes an environment with a 
value for `x`, then `x` will have a value within the closure even if it 
isn't a parameter.

So far the literal functions we've seen have had `nil` enviroments.
Let's try making one by hand with some variable bindings in it. An 
environment is a list of `(var . val)` pairs, so to make a closure we 
put such a list in the third position of a `clo`, like this:

    (lit clo ((x . a)) (y) (cons x y))

This closure includes an environment in which `x` has the value `a`. It 
has one parameter, `y`, and when called returns the value of `x` `cons`ed 
onto whatever we give as an argument.

    > ((lit clo ((x . a)) (y) (cons x y)) 'b)
    (a . b)

Notice that the `b` was all we passed to the function. The `a` came from
within it.

### More `fn`

It turns out to be very useful if functions include the lexical 
environment where they're created. And that is what the `fn` macro 
does.

In Bel you can get hold of the global and lexical environments using 
the variables `globe` and `scope` respectively. So for example if we 
define `foo` as

    (def foo (x)
      scope)

then it will work something like this

    > (foo 'a)
    ((x . a))
    > (foo 'b)
    ((x . b))

I say "something like" because a repl may have some variables of its 
own, but we know that scope will at least have a value for `x`.

If you compare the definitions of `fn-` and `fn`, you'll notice that
while `fn-` expands into a `lit` expression, `fn` expands into a call to
list that yields a `lit` expression. It works fine to use a call to
`list` rather than an actual list in a function call; functions are 
just lists after all.

    > ((list 'lit 'clo nil '(x) '(+ x 1)) 2)
    3

The reason the definition of `fn` expands into a call to `list` is so 
that we can incorporate the local environment, which we get by 
including `scope` in the arguments to list.

Here's an example where we do this manually:

    (def bar (x)
      ((list 'lit 'clo scope '(y) '(+ x y)) 2))

Within `bar` we call a hand-made closure that includes `scope`, which, as
we know from the example of foo above, will include a value for `x`.

    > (bar 3)
    5

The `fn` macro generates as its expansion exactly what we just made by 
hand. So this is equivalent to the definition above:

    (def bar (x)
      ((fn (y) (+ x y)) 2))

The `fn` macro has two different expansions depending on how many 
arguments we pass to it. That's so that functions can have bodies of
more than one expression.

If we call `fn` with two arguments, meaning a parameter list and an
expression, as in e.g.

    (fn (x) (cons 'a x))

then `(cdr body)` will be false, so the expansion will be

    (list 'lit 'clo scope '(x) '(cons 'a x))

If we call `fn` with three or more arguments, meaning a parameter list 
plus two or more expressions, e.g.

    (fn (x) 
      (prn 'hello) 
      (cons 'a x))

Then the expansion wraps a `do` around the expressions.

    (list 'lit 'clo scope '(x) '(do (prn 'hello) (cons 'a x)))

We haven't seen `do` yet, but it's coming soon. It makes multiple
expressions into a block of code. 


<Bel>

    (set <dfn>vmark</dfn> (join))
      
    (def <dfn>uvar</dfn> ()
      (list vmark))
</Bel>

Next comes something unusual: `vmark` is set to a newly created pair 
made by `join`. Missing arguments to primitives default to `nil`, so 
`(join)` is equivalent to `(join nil nil)`, and when you see a call like 
this, it's usually for the purpose of creating a fresh pair to mark 
the identity of something.

Any pair with `vmark` in its `car` is treated by Bel as a variable. The 
next function, `uvar`, thus returns a new, unique variable each time 
it's called. The reason we need such a thing is so that when we're
manipulating user code, we can add variables without worrying they'll 
accidentally share the names of variables created by users.


<Bel>

    (mac <dfn>do</dfn> args
      (reduce (fn (x y)
                (list (list 'fn (uvar) y) x))
              args))
</Bel>

Now we see the definition of `do`, which we used in the expansion of 
`fn`. The `do` macro uses nested function calls to represent blocks of 
code.

Suppose you want to evaluate two expressions in order and then return
the value of the last.

  e1
  e2

You can make this happen by embodying `e2` in a function that you then 
call on `e1`.

    ((fn x e2) e1)

When this expression is evaluated, `e1` will be evaluated first, and 
its value passed to the function in the car of the call:

    (fn x e2)

Then `e2` will be evaluated, ignoring the value of `e1` passed in the 
parameter, and its value returned. Result: `e1` and then `e2` get 
evaluated, as if in a block. (You cannot of course safely use `x` as 
the parameter in case it occurs within `e2`, but I'll explain how to 
deal with that in a minute.) 

This technique generalizes to blocks of any size. Here's one with 3 
expressions:

    ((fn x ((fn x e3) e2)) e1)

You can use `reduce` to generate this kind of expression as follows:

    (def block args
      (reduce (fn (x y)
                (list (list 'fn 'x y) x))
              args))
      
    > (block 'e1 'e2 'e3)
    ((fn x ((fn x e3) e2)) e1)

and this is almost exactly what the `do` macro does. If you look at
its definition, it's almost identical to that of `block`. 

One difference is that `do` is a macro rather than a function, which
means that the nested call gets evaluated after it's generated.

The other difference is that we call `uvar` to make the parameter 
instead of using `x`. We can't safely use any symbol as the parameter
in case it occurs in one of the expressions in the `do`. Since we're 
never going to look at the values passed in these function calls, we 
don't care what parameter we use, so long as it's unique.

<Bel>

    (mac <dfn>let</dfn> (parms val . body)
      `((fn (,parms) ,@body) ,val))
</Bel>

If you want to establish a lexical binding for some variable, you do
it with `let`, which is a very simple macro upon `fn`.

    > (let x 'a 
        (cons x 'b))
    (a . b)

Since `let` expands into a fn, you have the full power of Bel parameter 
lists in the first argument.

    > (let (x . y) '(a b c) 
        (list x y))
    (a (b c))

<Bel>

    (mac <dfn>macro</dfn> args
      `(list 'lit 'mac (fn ,@args)))
</Bel>

The macro `macro` is analogous to the `fn` macro in that it returns a
literal macro. You'll rarely use these directly, but you could if you 
wanted to.

    > ((macro (v) `(set ,v 'a)) x)
    a
    > x
    a

<Bel>

    (mac <dfn>def</dfn> (n . rest)
      `(set ,n (fn ,@rest)))
      
    (mac <dfn>mac</dfn> (n . rest)
      `(set ,n (macro ,@rest)))
</Bel>


Next we see the definition of `def` itself, which does nothing more
than `set` its first argument to a `fn` made using the rest of the
arguments, and also of `mac`, which does the same with macro.

(I like it when I can define new operators as thin, almost trivial
seeming layers on top of existing operators. It seems a sign of
orthogonality.)

If you were wondering why `fn` needs two cases-- why we don't just 
always wrap a `do` around the body-- the reason is that `do` calls 
`reduce`, which is defined using `def`, which expands into a `fn`. So to 
avoid an infinite recursion we either have to define `reduce` as a 
literal function, or make either `fn` or `do` consider the single 
expression case, and making `fn` do it was the least ugly.

<Bel>

    (mac <dfn>or</dfn> args
      (if (no args)
          nil
          (let v (uvar)
            `(let ,v ,(car args)
              (if ,v ,v (or ,@(cdr args)))))))
</Bel>

Now that we have `let`, we can define `or`, which returns the first 
non-`nil` value returned by one of its arguments. Like most `or`s in
programming languages, it only evaluates as many arguments as it
needs to, which means you can use it for control flow as well as
logical disjunction.

    > (or 'a (prn 'hello))
    a

The definition of `or` is the first recursive macro definition we've
seen. Unless it has no arguments, an `or` will expand into another `or`. 
This is fine so long as the recursion terminates, which this one 
will because each time we look at the `cdr` of the list of arguments, 
which will eventually be `nil`. (Though you could spoof `or` by 
constructing a circular list and applying `or` to it.)

In effect

    (or foo bar)

expands into

    (let x foo
      (if x 
          x
          (let y bar
            (if y
                y
                nil))))

except that we can't actually use variables like `x` and `y` to hold the 
values, and instead have to use `uvar`s.

Notice incidentally that the expression above could be optimized

    (let x foo
      (if x
          x
          bar))

but the definition of `or` doesn't try to; like every definition in
Bel, its purpose is to define what `or` means, not to provide an 
efficient implementation of it.

In Bel, macros are applyable just like functions are, though if you
do that you get only the logical aspect of `or` and not the control
aspect, since in a call to apply the arguments have already all been
evaluated.

    > (apply or '(nil nil))
    nil
    > (apply or '(nil a b))
    a

<Bel>

    (mac <dfn>and</dfn> args
      (reduce (fn es (cons 'if es))
              (or args '(t))))
</Bel>

The `and` macro is similar in spirit to `or`, but different in its 
implementation. Whereas `or` uses recursion to generate its expansion, 
`and` uses `reduce`. Since 

    (and w x y z) 

is equivalent to

    (if w (if x (if y z)))

it's an obvious candidate for `reduce`. 

Notice the function given to reduce has a single parameter. A 
function given as the first argument to `reduce` will only ever be 
called with two arguments, so usually such a function will have a 
list of two parameters, but in this case we just want to `cons` an if 
onto the front of the arguments each time.

The other interesting thing about `and` is what we do when it has no 
arguments. While we want (or) to return nil, we want (and) to return 
`t`. So in the second argument to reduce, we replace an empty `args` with 
`(t)`.


<Bel>

    (def = args
      (if (no (cdr args))  t
          (some atom args) (all [id _ (car args)] (cdr args))
                          (and (apply = (map car args))
                                (apply = (map cdr args)))))
</Bel>

The next function, `=`, is the one that programs usually use to test 
for equality. It returns true iff its arguments are trees of the same 
shape whose leaves are the same atoms.

    > (id '(a b) '(a b))
    nil
    > (= '(a b) '(a b))
    t

In Bel, everything that's not a symbol, character, or stream is a
pair. Numbers and strings are pairs, for example. So you'd never
want to use `id` for comparison unless you were specifically looking
for identical list structure.

In the definition of `=` we see the first instance of square bracket
notation.

    (all [id _ (car args)] (cdr args))

This is equivalent to

    (all (fn (x) (id x (car args))) 
        (cdr args))

I.e., is everything in the `cdr` of `args` `id` to the `car`? You know you
can use `id` to test equality at this point, because if one of the `args` is
an atom, they all have to be for them to be `=`, and you can use `id` 
to test equality of atoms.

If `id` took any number of arguments (it doesn't, because I want axioms
to be as weak as possible), the preceding `all` expression could have 
been simply

    (apply id args)


<Bel>

    (def <dfn>symbol</dfn> (x) (= (type x) 'symbol))
      
    (def <dfn>pair</dfn>   (x) (= (type x) 'pair))
      
    (def <dfn>char</dfn>   (x) (= (type x) 'char))
      
    (def <dfn>stream</dfn> (x) (= (type x) 'stream))
</Bel>

The next four functions are predicates for the four types. All use `=` 
for this test even though all could use `id`. My rule is to use `=`
unless I specifically need `id`. That way the appearance of `id` is a 
signal that code is looking for identical structure.


<Bel>

    (def <dfn>proper</dfn> (x)
      (or (no x)
          (and (pair x) (proper (cdr x)))))
</Bel>

Then we see `proper`, which tells us whether something is a proper
list. Informally, a proper list is one that we don't need a dot to
display.

    > (proper nil)
    t
    > (proper '(a . b))
    nil
    > (proper '(a b))
    t

Formally, something is a proper list if it's either `nil` or a pair
whose `cdr` is a proper list, which is exactly what the definition
says.


<Bel>

    (def <dfn>string</dfn> (x)
      (and (proper x) (all char x)))
</Bel>

In Bel, a `proper` list of characters is called a `string`, and has a 
special notation: zero or more characters within double quotes.

    > (string "foo")
    t

The next function, `mem`, tests for list membership.

    > (mem 'b '(a b c))
    (b c)
    > (mem 'e '(a b c))
    nil
    > (mem \a "foobar")
    "ar"

Since it uses `some`, it returns the rest of the list starting with the 
thing we're looking for, rather than simply `t`.

