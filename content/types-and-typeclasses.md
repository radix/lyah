<div class="bgwrapper">

<div id="content">

<div class="footdiv" style="margin-bottom:25px;">

-   [Starting Out](starting-out)
-   [Table of contents](chapters)
-   [Syntax in Functions](syntax-in-functions)

</div>

Types and Typeclasses
=====================

Believe the type
----------------

![moo](http://s3.amazonaws.com/lyah/cow.png)
Previously we mentioned that Haskell has a static type system. The type
of every expression is known at compile time, which leads to safer code.
If you write a program where you try to divide a boolean type with some
number, it won't even compile. That's good because it's better to catch
such errors at compile time instead of having your program crash.
Everything in Haskell has a type, so the compiler can reason quite a lot
about your program before compiling it.

Unlike Java or Pascal, Haskell has type inference. If we write a number,
we don't have to tell Haskell it's a number. It can *infer* that on its
own, so we don't have to explicitly write out the types of our functions
and expressions to get things done. We covered some of the basics of
Haskell with only a very superficial glance at types. However,
understanding the type system is a very important part of learning
Haskell.

A type is a kind of label that every expression has. It tells us in
which category of things that expression fits. The expression <span
class="fixed">True</span> is a boolean, <span
class="fixed">"hello"</span> is a string, etc.

Now we'll use GHCI to examine the types of some expressions. We'll do
that by using the <span class="fixed">:t</span> command which, followed
by any valid expression, tells us its type. Let's give it a whirl.

``` {.haskell: .ghci name="code"}
ghci> :t 'a'
'a' :: Char
ghci> :t True
True :: Bool
ghci> :t "HELLO!"
"HELLO!" :: [Char]
ghci> :t (True, 'a')
(True, 'a') :: (Bool, Char)
ghci> :t 4 == 5
4 == 5 :: Bool
```

![bomb](http://s3.amazonaws.com/lyah/bomb.png) Here we see that doing
<span class="fixed">:t</span> on an expression prints out the expression
followed by <span class="fixed">::</span> and its type. <span
class="fixed">::</span> is read as "has type of". Explicit types are
always denoted with the first letter in capital case. <span
class="fixed">'a'</span>, as it would seem, has a type of <span
class="fixed">Char</span>. It's not hard to conclude that it stands for
*character*. <span class="fixed">True</span> is of a <span
class="fixed">Bool</span> type. That makes sense. But what's this?
Examining the type of <span class="fixed">"HELLO!"</span> yields a <span
class="fixed">[Char]</span>. The square brackets denote a list. So we
read that as it being *a list of characters*. Unlike lists, each tuple
length has its own type. So the expression of <span class="fixed">(True,
'a')</span> has a type of <span class="fixed">(Bool, Char)</span>,
whereas an expression such as <span class="fixed">('a','b','c')</span>
would have the type of <span class="fixed">(Char, Char, Char)</span>.
<span class="fixed">4 == 5</span> will always return <span
class="fixed">False</span>, so its type is <span
class="fixed">Bool</span>.

Functions also have types. When writing our own functions, we can choose
to give them an explicit type declaration. This is generally considered
to be good practice except when writing very short functions. From here
on, we'll give all the functions that we make type declarations.
Remember the list comprehension we made previously that filters a string
so that only caps remain? Here's how it looks like with a type
declaration.

``` {.haskell: .hs name="code"}
removeNonUppercase :: [Char] -> [Char]
removeNonUppercase st = [ c | c <- st, c `elem` ['A'..'Z']] 
```

<span class="fixed">removeNonUppercase</span> has a type of <span
class="fixed">[Char] -\> [Char]</span>, meaning that it maps from a
string to a string. That's because it takes one string as a parameter
and returns another as a result. The <span class="fixed">[Char]</span>
type is synonymous with <span class="fixed">String</span> so it's
clearer if we write <span class="fixed">removeNonUppercase :: String -\>
String</span>. We didn't have to give this function a type declaration
because the compiler can infer by itself that it's a function from a
string to a string but we did anyway. But how do we write out the type
of a function that takes several parameters? Here's a simple function
that takes three integers and adds them together:

``` {.haskell: .hs name="code"}
addThree :: Int -> Int -> Int -> Int
addThree x y z = x + y + z
```

The parameters are separated with <span class="fixed">-\></span> and
there's no special distinction between the parameters and the return
type. The return type is the last item in the declaration and the
parameters are the first three. Later on we'll see why they're all just
separated with <span class="fixed">-\></span> instead of having some
more explicit distinction between the return types and the parameters
like <span class="fixed">Int, Int, Int -\> Int</span> or something.

If you want to give your function a type declaration but are unsure as
to what it should be, you can always just write the function without it
and then check it with <span class="fixed">:t</span>. Functions are
expressions too, so <span class="fixed">:t</span> works on them without
a problem.

Here's an overview of some common types.

<span class="label type">Int</span> stands for integer. It's used for
whole numbers. <span class="fixed">7</span> can be an <span
class="fixed">Int</span> but <span class="fixed">7.2</span> cannot.
<span class="fixed">Int</span> is bounded, which means that it has a
minimum and a maximum value. Usually on 32-bit machines the maximum
possible <span class="fixed">Int</span> is 2147483647 and the minimum
is -2147483648.

<span class="label type">Integer</span> stands for, er … also integer.
The main difference is that it's not bounded so it can be used to
represent really really big numbers. I mean like really big. <span
class="fixed">Int</span>, however, is more efficient.

``` {.haskell: .hs name="code"}
factorial :: Integer -> Integer
factorial n = product [1..n]
```

``` {.haskell: .ghci name="code"}
ghci> factorial 50
30414093201713378043612608166064768844377641568960512000000000000
```

<span class="label type">Float</span> is a real floating point with
single precision.

``` {.haskell: .hs name="code"}
circumference :: Float -> Float
circumference r = 2 * pi * r
```

``` {.haskell: .ghci name="code"}
ghci> circumference 4.0
25.132742
```

<span class="label type">Double</span> is a real floating point with
double the precision!

``` {.haskell: .hs name="code"}
circumference' :: Double -> Double
circumference' r = 2 * pi * r
```

``` {.haskell: .ghci name="code"}
ghci> circumference' 4.0
25.132741228718345
```

<span class="label type">Bool</span> is a boolean type. It can have only
two values: <span class="fixed">True</span> and <span
class="fixed">False</span>.

<span class="label type">Char</span> represents a character. It's
denoted by single quotes. A list of characters is a string.

Tuples are types but they are dependent on their length as well as the
types of their components, so there is theoretically an infinite number
of tuple types, which is too many to cover in this tutorial. Note that
the empty tuple <span class="label type">()</span> is also a type which
can only have a single value: <span class="fixed">()</span>

Type variables
--------------

What do you think is the type of the <span class="fixed">head</span>
function? Because <span class="fixed">head</span> takes a list of any
type and returns the first element, so what could it be? Let's check!

``` {.haskell: .ghci name="code"}
ghci> :t head
head :: [a] -> a
```

![box](http://s3.amazonaws.com/lyah/box.png) Hmmm! What is this <span
class="fixed">a</span>? Is it a type? Remember that we previously stated
that types are written in capital case, so it can't exactly be a type.
Because it's not in capital case it's actually a *type variable*. That
means that <span class="fixed">a</span> can be of any type. This is much
like generics in other languages, only in Haskell it's much more
powerful because it allows us to easily write very general functions if
they don't use any specific behavior of the types in them. Functions
that have type variables are called *polymorphic functions*. The type
declaration of <span class="fixed">head</span> states that it takes a
list of any type and returns one element of that type.

Although type variables can have names longer than one character, we
usually give them names of a, b, c, d …

Remember <span class="fixed">fst</span>? It returns the first component
of a pair. Let's examine its type.

``` {.haskell: .ghci name="code"}
ghci> :t fst
fst :: (a, b) -> a
```

We see that <span class="fixed">fst</span> takes a tuple which contains
two types and returns an element which is of the same type as the pair's
first component. That's why we can use <span class="fixed">fst</span> on
a pair that contains any two types. Note that just because <span
class="fixed">a</span> and <span class="fixed">b</span> are different
type variables, they don't have to be different types. It just states
that the first component's type and the return value's type are the
same.

Typeclasses 101
---------------

![class](http://s3.amazonaws.com/lyah/classes.png)
A typeclass is a sort of interface that defines some behavior. If a type
is a part of a typeclass, that means that it supports and implements the
behavior the typeclass describes. A lot of people coming from OOP get
confused by typeclasses because they think they are like classes in
object oriented languages. Well, they're not. You can think of them kind
of as Java interfaces, only better.

What's the type signature of the <span class="fixed">==</span> function?

``` {.haskell: .ghci name="code"}
ghci> :t (==)
(==) :: (Eq a) => a -> a -> Bool
```

<div class="hintbox">

*Note*: the equality operator, <span class="fixed">==</span> is a
function. So are <span class="fixed">+</span>, <span
class="fixed">\*</span>, <span class="fixed">-</span>, <span
class="fixed">/</span> and pretty much all operators. If a function is
comprised only of special characters, it's considered an infix function
by default. If we want to examine its type, pass it to another function
or call it as a prefix function, we have to surround it in parentheses.

</div>

Interesting. We see a new thing here, the <span class="fixed">=\></span>
symbol. Everything before the <span class="fixed">=\></span> symbol is
called a *class constraint*. We can read the previous type declaration
like this: the equality function takes any two values that are of the
same type and returns a <span class="fixed">Bool</span>. The type of
those two values must be a member of the <span class="fixed">Eq</span>
class (this was the class constraint).

The <span class="fixed">Eq</span> typeclass provides an interface for
testing for equality. Any type where it makes sense to test for equality
between two values of that type should be a member of the <span
class="fixed">Eq</span> class. All standard Haskell types except for IO
(the type for dealing with input and output) and functions are a part of
the <span class="fixed">Eq</span> typeclass.

The <span class="fixed">elem</span> function has a type of <span
class="fixed">(Eq a) =\> a -\> [a] -\> Bool</span> because it uses <span
class="fixed">==</span> over a list to check whether some value we're
looking for is in it.

Some basic typeclasses:

<span class="label class">Eq</span> is used for types that support
equality testing. The functions its members implement are <span
class="fixed">==</span> and <span class="fixed">/=</span>. So if there's
an <span class="fixed">Eq</span> class constraint for a type variable in
a function, it uses <span class="fixed">==</span> or <span
class="fixed">/=</span> somewhere inside its definition. All the types
we mentioned previously except for functions are part of <span
class="fixed">Eq</span>, so they can be tested for equality.

``` {.haskell: .ghci name="code"}
ghci> 5 == 5
True
ghci> 5 /= 5
False
ghci> 'a' == 'a'
True
ghci> "Ho Ho" == "Ho Ho"
True
ghci> 3.432 == 3.432
True
```

<span class="label class">Ord</span> is for types that have an ordering.

``` {.haskell: .ghci name="code"}
ghci> :t (>)
(>) :: (Ord a) => a -> a -> Bool
```

All the types we covered so far except for functions are part of <span
class="fixed">Ord</span>. <span class="fixed">Ord</span> covers all the
standard comparing functions such as <span class="fixed">\></span>,
<span class="fixed">\<</span>, <span class="fixed">\>=</span> and <span
class="fixed">\<=</span>. The <span class="fixed">compare</span>
function takes two <span class="fixed">Ord</span> members of the same
type and returns an ordering. <span class="label type">Ordering</span>
is a type that can be <span class="fixed">GT</span>, <span
class="fixed">LT</span> or <span class="fixed">EQ</span>, meaning
*greater than*, *lesser than* and *equal*, respectively.

To be a member of <span class="fixed">Ord</span>, a type must first have
membership in the prestigious and exclusive <span
class="fixed">Eq</span> club.

``` {.haskell: .ghci name="code"}
ghci> "Abrakadabra" < "Zebra"
True
ghci> "Abrakadabra" `compare` "Zebra"
LT
ghci> 5 >= 2
True
ghci> 5 `compare` 3
GT
```

Members of <span class="label class">Show</span> can be presented as
strings. All types covered so far except for functions are a part of
<span class="fixed">Show</span>. The most used function that deals with
the <span class="fixed">Show</span> typeclass is <span
class="fixed">show</span>. It takes a value whose type is a member of
<span class="fixed">Show</span> and presents it to us as a string.

``` {.haskell: .ghci name="code"}
ghci> show 3
"3"
ghci> show 5.334
"5.334"
ghci> show True
"True"
```

<span class="label class">Read</span> is sort of the opposite typeclass
of <span class="fixed">Show</span>. The <span class="fixed">read</span>
function takes a string and returns a type which is a member of <span
class="fixed">Read</span>.

``` {.haskell: .ghci name="code"}
ghci> read "True" || False
True
ghci> read "8.2" + 3.8
12.0
ghci> read "5" - 2
3
ghci> read "[1,2,3,4]" ++ [3]
[1,2,3,4,3]
```

So far so good. Again, all types covered so far are in this typeclass.
But what happens if we try to do just <span class="fixed">read
"4"</span>?

``` {.haskell: .ghci name="code"}
ghci> read "4"
<interactive>:1:0:
    Ambiguous type variable `a' in the constraint:
      `Read a' arising from a use of `read' at <interactive>:1:0-7
    Probable fix: add a type signature that fixes these type variable(s)
```

What GHCI is telling us here is that it doesn't know what we want in
return. Notice that in the previous uses of <span
class="fixed">read</span> we did something with the result afterwards.
That way, GHCI could infer what kind of result we wanted out of our
<span class="fixed">read</span>. If we used it as a boolean, it knew it
had to return a <span class="fixed">Bool</span>. But now, it knows we
want some type that is part of the <span class="fixed">Read</span>
class, it just doesn't know which one. Let's take a look at the type
signature of <span class="fixed">read</span>.

``` {.haskell: .ghci name="code"}
ghci> :t read
read :: (Read a) => String -> a
```

See? It returns a type that's part of <span class="fixed">Read</span>
but if we don't try to use it in some way later, it has no way of
knowing which type. That's why we can use explicit *type annotations*.
Type annotations are a way of explicitly saying what the type of an
expression should be. We do that by adding <span class="fixed">::</span>
at the end of the expression and then specifying a type. Observe:

``` {.haskell: .ghci name="code"}
ghci> read "5" :: Int
5
ghci> read "5" :: Float
5.0
ghci> (read "5" :: Float) * 4
20.0
ghci> read "[1,2,3,4]" :: [Int]
[1,2,3,4]
ghci> read "(3, 'a')" :: (Int, Char)
(3, 'a')
```

Most expressions are such that the compiler can infer what their type is
by itself. But sometimes, the compiler doesn't know whether to return a
value of type <span class="fixed">Int</span> or <span
class="fixed">Float</span> for an expression like <span
class="fixed">read "5"</span>. To see what the type is, Haskell would
have to actually evaluate <span class="fixed">read "5"</span>. But since
Haskell is a statically typed language, it has to know all the types
before the code is compiled (or in the case of GHCI, evaluated). So we
have to tell Haskell: "Hey, this expression should have this type, in
case you don't know!".

<span class="label class">Enum</span> members are sequentially ordered
types — they can be enumerated. The main advantage of the <span
class="fixed">Enum</span> typeclass is that we can use its types in list
ranges. They also have defined successors and predecesors, which you can
get with the <span class="fixed">succ</span> and <span
class="fixed">pred</span> functions. Types in this class: <span
class="fixed">()</span>, <span class="fixed">Bool</span>, <span
class="fixed">Char</span>, <span class="fixed">Ordering</span>, <span
class="fixed">Int</span>, <span class="fixed">Integer</span>, <span
class="fixed">Float</span> and <span class="fixed">Double</span>.

``` {.haskell: .ghci name="code"}
ghci> ['a'..'e']
"abcde"
ghci> [LT .. GT]
[LT,EQ,GT]
ghci> [3 .. 5]
[3,4,5]
ghci> succ 'B'
'C'
```

<span class="label class">Bounded</span> members have an upper and a
lower bound.

``` {.haskell: .ghci name="code"}
ghci> minBound :: Int
-2147483648
ghci> maxBound :: Char
'\1114111'
ghci> maxBound :: Bool
True
ghci> minBound :: Bool
False
```

<span class="fixed">minBound</span> and <span
class="fixed">maxBound</span> are interesting because they have a type
of <span class="fixed">(Bounded a) =\> a</span>. In a sense they are
polymorphic constants.

All tuples are also part of <span class="fixed">Bounded</span> if the
components are also in it.

``` {.haskell: .ghci name="code"}
ghci> maxBound :: (Bool, Int, Char)
(True,2147483647,'\1114111')
```

<span class="label class">Num</span> is a numeric typeclass. Its members
have the property of being able to act like numbers. Let's examine the
type of a number.

``` {.haskell: .ghci name="code"}
ghci> :t 20
20 :: (Num t) => t
```

It appears that whole numbers are also polymorphic constants. They can
act like any type that's a member of the <span class="fixed">Num</span>
typeclass.

``` {.haskell: .ghci name="code"}
ghci> 20 :: Int
20
ghci> 20 :: Integer
20
ghci> 20 :: Float
20.0
ghci> 20 :: Double
20.0
```

Those are types that are in the <span class="fixed">Num</span>
typeclass. If we examine the type of <span class="fixed">\*</span>,
we'll see that it accepts all numbers.

``` {.haskell: .ghci name="code"}
ghci> :t (*)
(*) :: (Num a) => a -> a -> a
```

It takes two numbers of the same type and returns a number of that type.
That's why <span class="fixed">(5 :: Int) \* (6 :: Integer)</span> will
result in a type error whereas <span class="fixed">5 \* (6 ::
Integer)</span> will work just fine and produce an <span
class="fixed">Integer</span> because <span class="fixed">5</span> can
act like an <span class="fixed">Integer</span> or an <span
class="fixed">Int</span>.

To join <span class="fixed">Num</span>, a type must already be friends
with <span class="fixed">Show</span> and <span class="fixed">Eq</span>.

<span class="class label">Integral</span> is also a numeric typeclass.
<span class="fixed">Num</span> includes all numbers, including real
numbers and integral numbers, <span class="fixed">Integral</span>
includes only integral (whole) numbers. In this typeclass are <span
class="fixed">Int</span> and <span class="fixed">Integer</span>.

<span class="class label">Floating</span> includes only floating point
numbers, so <span class="fixed">Float</span> and <span
class="fixed">Double</span>.

A very useful function for dealing with numbers is <span
class="label function">fromIntegral</span>. It has a type declaration of
<span class="fixed">fromIntegral :: (Num b, Integral a) =\> a -\>
b</span>. From its type signature we see that it takes an integral
number and turns it into a more general number. That's useful when you
want integral and floating point types to work together nicely. For
instance, the <span class="fixed">length</span> function has a type
declaration of <span class="fixed">length :: [a] -\> Int</span> instead
of having a more general type of <span class="fixed">(Num b) =\> length
:: [a] -\> b</span>. I think that's there for historical reasons or
something, although in my opinion, it's pretty stupid. Anyway, if we try
to get a length of a list and then add it to <span
class="fixed">3.2</span>, we'll get an error because we tried to add
together an <span class="fixed">Int</span> and a floating point number.
So to get around this, we do <span class="fixed">fromIntegral (length
[1,2,3,4]) + 3.2</span> and it all works out.

Notice that <span class="fixed">fromIntegral</span> has several class
constraints in its type signature. That's completely valid and as you
can see, the class constraints are separated by commas inside the
parentheses.

<div class="footdiv">

-   [Starting Out](starting-out)
-   [Table of contents](chapters)
-   [Syntax in Functions](syntax-in-functions)

</div>

</div>

</div>
