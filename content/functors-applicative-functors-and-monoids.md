Functors, Applicative Functors and Monoids
==========================================

Haskell's combination of purity, higher order functions, parameterized
algebraic data types, and typeclasses allows us to implement
polymorphism on a much higher level than possible in other languages. We
don't have to think about types belonging to a big hierarchy of types.
Instead, we think about what the types can act like and then connect
them with the appropriate typeclasses. An <span class="fixed">Int</span>
can act like a lot of things. It can act like an equatable thing, like
an ordered thing, like an enumerable thing, etc.

Typeclasses are open, which means that we can define our own data type,
think about what it can act like and connect it with the typeclasses
that define its behaviors. Because of that and because of Haskell's
great type system that allows us to know a lot about a function just by
knowing its type declaration, we can define typeclasses that define
behavior that's very general and abstract. We've met typeclasses that
define operations for seeing if two things are equal or comparing two
things by some ordering. Those are very abstract and elegant behaviors,
but we just don't think of them as anything very special because we've
been dealing with them for most of our lives. We recently met functors,
which are basically things that can be mapped over. That's an example of
a useful and yet still pretty abstract property that typeclasses can
describe. In this chapter, we'll take a closer look at functors, along
with slightly stronger and more useful versions of functors called
applicative functors. We'll also take a look at monoids, which are sort
of like socks.

Functors redux
--------------

![frogs dont even need money](http://s3.amazonaws.com/lyah/frogtor.png)
We've already talked about functors in [their own little
section](making-our-own-types-and-typeclasses#the-functor-typeclass). If
you haven't read it yet, you should probably give it a glance right now,
or maybe later when you have more time. Or you can just pretend you read
it.

Still, here's a quick refresher: Functors are things that can be mapped
over, like lists, <span class="fixed">Maybe</span>s, trees, and such. In
Haskell, they're described by the typeclass <span
class="fixed">Functor</span>, which has only one typeclass method,
namely <span class="fixed">fmap</span>, which has a type of <span
class="fixed">fmap :: (a -\> b) -\> f a -\> f b</span>. It says: give me
a function that takes an <span class="fixed">a</span> and returns a
<span class="fixed">b</span> and a box with an <span
class="fixed">a</span> (or several of them) inside it and I'll give you
a box with a <span class="fixed">b</span> (or several of them) inside
it. It kind of applies the function to the element inside the box.

<div class="hintbox">

*A word of advice.* Many times the box analogy is used to help you get
some intuition for how functors work, and later, we'll probably use the
same analogy for applicative functors and monads. It's an okay analogy
that helps people understand functors at first, just don't take it too
literally, because for some functors the box analogy has to be stretched
really thin to still hold some truth. A more correct term for what a
functor is would be *computational context*. The context might be that
the computation can have a value or it might have failed (<span
class="fixed">Maybe</span> and <span class="fixed">Either a</span>) or
that there might be more values (lists), stuff like that.

</div>

If we want to make a type constructor an instance of <span
class="fixed">Functor</span>, it has to have a kind of <span
class="fixed">\* -\> \*</span>, which means that it has to take exactly
one concrete type as a type parameter. For example, <span
class="fixed">Maybe</span> can be made an instance because it takes one
type parameter to produce a concrete type, like <span
class="fixed">Maybe Int</span> or <span class="fixed">Maybe
String</span>. If a type constructor takes two parameters, like <span
class="fixed">Either</span>, we have to partially apply the type
constructor until it only takes one type parameter. So we can't write
<span class="fixed">instance Functor Either where</span>, but we can
write <span class="fixed">instance Functor (Either a) where</span> and
then if we imagine that <span class="fixed">fmap</span> is only for
<span class="fixed">Either a</span>, it would have a type declaration of
<span class="fixed">fmap :: (b -\> c) -\> Either a b -\> Either a
c</span>. As you can see, the <span class="fixed">Either a</span> part
is fixed, because <span class="fixed">Either a</span> takes only one
type parameter, whereas just <span class="fixed">Either</span> takes two
so <span class="fixed">fmap :: (b -\> c) -\> Either b -\> Either
c</span> wouldn't really make sense.

We've learned by now how a lot of types (well, type constructors really)
are instances of <span class="fixed">Functor</span>, like <span
class="fixed">[]</span>, <span class="fixed">Maybe</span>, <span
class="fixed">Either a</span> and a <span class="fixed">Tree</span> type
that we made on our own. We saw how we can map functions over them for
great good. In this section, we'll take a look at two more instances of
functor, namely <span class="fixed">IO</span> and <span
class="fixed">(-\>) r</span>.

If some value has a type of, say, <span class="fixed">IO String</span>,
that means that it's an I/O action that, when performed, will go out
into the real world and get some string for us, which it will yield as a
result. We can use <span class="fixed">\<-</span> in *do* syntax to bind
that result to a name. We mentioned that I/O actions are like boxes with
little feet that go out and fetch some value from the outside world for
us. We can inspect what they fetched, but after inspecting, we have to
wrap the value back in <span class="fixed">IO</span>. By thinking about
this box with little feet analogy, we can see how <span
class="fixed">IO</span> acts like a functor.

Let's see how <span class="fixed">IO</span> is an instance of <span
class="fixed">Functor</span>. When we <span class="fixed">fmap</span> a
function over an I/O action, we want to get back an I/O action that does
the same thing, but has our function applied over its result value.

``` {.haskell:hs name="code"}
instance Functor IO where
    fmap f action = do
        result <- action
        return (f result)
```

The result of mapping something over an I/O action will be an I/O
action, so right off the bat we use *do* syntax to glue two actions and
make a new one. In the implementation for <span
class="fixed">fmap</span>, we make a new I/O action that first performs
the original I/O action and calls its result <span
class="fixed">result</span>. Then, we do <span class="fixed">return (f
result)</span>. <span class="fixed">return</span> is, as you know, a
function that makes an I/O action that doesn't do anything but only
presents something as its result. The action that a *do* block produces
will always have the result value of its last action. That's why we use
return to make an I/O action that doesn't really do anything, it just
presents <span class="fixed">f result</span> as the result of the new
I/O action.

We can play around with it to gain some intuition. It's pretty simple
really. Check out this piece of code:

``` {.haskell:hs name="code"}
main = do line <- getLine
          let line' = reverse line
          putStrLn $ "You said " ++ line' ++ " backwards!"
          putStrLn $ "Yes, you really said" ++ line' ++ " backwards!"
```

The user is prompted for a line and we give it back to the user, only
reversed. Here's how to rewrite this by using <span
class="fixed">fmap</span>:

``` {.haskell:hs name="code"}
main = do line <- fmap reverse getLine
          putStrLn $ "You said " ++ line ++ " backwards!"
          putStrLn $ "Yes, you really said" ++ line ++ " backwards!"
```

![w00ooOoooOO](http://s3.amazonaws.com/lyah/alien.png)
Just like when we <span class="fixed">fmap</span> <span
class="fixed">reverse</span> over <span class="fixed">Just "blah"</span>
to get <span class="fixed">Just "halb"</span>, we can <span
class="fixed">fmap</span> <span class="fixed">reverse</span> over <span
class="fixed">getLine</span>. <span class="fixed">getLine</span> is an
I/O action that has a type of <span class="fixed">IO String</span> and
mapping <span class="fixed">reverse</span> over it gives us an I/O
action that will go out into the real world and get a line and then
apply <span class="fixed">reverse</span> to its result. Like we can
apply a function to something that's inside a <span
class="fixed">Maybe</span> box, we can apply a function to what's inside
an <span class="fixed">IO</span> box, only it has to go out into the
real world to get something. Then when we bind it to a name by using
<span class="fixed">\<-</span>, the name will reflect the result that
already has <span class="fixed">reverse</span> applied to it.

The I/O action <span class="fixed">fmap (++"!") getLine</span> behaves
just like <span class="fixed">getLine</span>, only that its result
always has <span class="fixed">"!"</span> appended to it!

If we look at what <span class="fixed">fmap</span>'s type would be if it
were limited to <span class="fixed">IO</span>, it would be <span
class="fixed">fmap :: (a -\> b) -\> IO a -\> IO b</span>. <span
class="fixed">fmap</span> takes a function and an I/O action and returns
a new I/O action that's like the old one, except that the function is
applied to its contained result.

If you ever find yourself binding the result of an I/O action to a name,
only to apply a function to that and call that something else, consider
using <span class="fixed">fmap</span>, because it looks prettier. If you
want to apply multiple transformations to some data inside a functor,
you can declare your own function at the top level, make a lambda
function or ideally, use function composition:

``` {.haskell:hs name="code"}
import Data.Char
import Data.List

main = do line <- fmap (intersperse '-' . reverse . map toUpper) getLine
          putStrLn line
```

``` {.plain name="code"}
$ runhaskell fmapping_io.hs
hello there
E-R-E-H-T- -O-L-L-E-H
```

As you probably know, <span class="fixed">intersperse '-' . reverse .
map toUpper</span> is a function that takes a string, maps <span
class="fixed">toUpper</span> over it, the applies <span
class="fixed">reverse</span> to that result and then applies <span
class="fixed">intersperse '-'</span> to that result. It's like writing
<span class="fixed">(\\xs -\> intersperse '-' (reverse (map toUpper
xs)))</span>, only prettier.

Another instance of <span class="fixed">Functor</span> that we've been
dealing with all along but didn't know was a <span
class="fixed">Functor</span> is <span class="fixed">(-\>) r</span>.
You're probably slightly confused now, since what the heck does <span
class="fixed">(-\>) r</span> mean? The function type <span
class="fixed">r -\> a</span> can be rewritten as <span
class="fixed">(-\>) r a</span>, much like we can write <span
class="fixed">2 + 3</span> as <span class="fixed">(+) 2 3</span>. When
we look at it as <span class="fixed">(-\>) r a</span>, we can see <span
class="fixed">(-\>)</span> in a slighty different light, because we see
that it's just a type constructor that takes two type parameters, just
like <span class="fixed">Either</span>. But remember, we said that a
type constructor has to take exactly one type parameter so that it can
be made an instance of <span class="fixed">Functor</span>. That's why we
can't make <span class="fixed">(-\>)</span> an instance of <span
class="fixed">Functor</span>, but if we partially apply it to <span
class="fixed">(-\>) r</span>, it doesn't pose any problems. If the
syntax allowed for type constructors to be partially applied with
sections (like we can partially apply <span class="fixed">+</span> by
doing <span class="fixed">(2+)</span>, which is the same as <span
class="fixed">(+) 2</span>), you could write <span class="fixed">(-\>)
r</span> as <span class="fixed">(r -\>)</span>. How are functions
functors? Well, let's take a look at the implementation, which lies in
<span class="fixed">Control.Monad.Instances</span>

<div class="hintbox">

We usually mark functions that take anything and return anything as
<span class="fixed">a -\> b</span>. <span class="fixed">r -\> a</span>
is the same thing, we just used different letters for the type
variables.

</div>

``` {.haskell:hs name="code"}
instance Functor ((->) r) where
    fmap f g = (\x -> f (g x))
```

If the syntax allowed for it, it could have been written as

``` {.haskell:hs name="code"}
instance Functor (r ->) where
    fmap f g = (\x -> f (g x))
```

But it doesn't, so we have to write it in the former fashion.

First of all, let's think about <span class="fixed">fmap</span>'s type.
It's <span class="fixed">fmap :: (a -\> b) -\> f a -\> f b</span>. Now
what we'll do is mentally replace all the <span
class="fixed">f</span>'s, which are the role that our functor instance
plays, with <span class="fixed">(-\>) r</span>'s. We'll do that to see
how <span class="fixed">fmap</span> should behave for this particular
instance. We get <span class="fixed">fmap :: (a -\> b) -\> ((-\>) r
a) -\> ((-\>) r b)</span>. Now what we can do is write the <span
class="fixed">(-\>) r a</span> and <span class="fixed">(-\> r b)</span>
types as infix <span class="fixed">r -\> a</span> and <span
class="fixed">r -\> b</span>, like we normally do with functions. What
we get now is <span class="fixed">fmap :: (a -\> b) -\> (r -\> a) -\>
(r -\> b)</span>.

Hmmm OK. Mapping one function over a function has to produce a function,
just like mapping a function over a <span class="fixed">Maybe</span> has
to produce a <span class="fixed">Maybe</span> and mapping a function
over a list has to produce a list. What does the type <span
class="fixed">fmap :: (a -\> b) -\> (r -\> a) -\> (r -\> b)</span> for
this instance tell us? Well, we see that it takes a function from <span
class="fixed">a</span> to <span class="fixed">b</span> and a function
from <span class="fixed">r</span> to <span class="fixed">a</span> and
returns a function from <span class="fixed">r</span> to <span
class="fixed">b</span>. Does this remind you of anything? Yes! Function
composition! We pipe the output of <span class="fixed">r -\> a</span>
into the input of <span class="fixed">a -\> b</span> to get a function
<span class="fixed">r -\> b</span>, which is exactly what function
composition is about. If you look at how the instance is defined above,
you'll see that it's just function composition. Another way to write
this instance would be:

``` {.haskell:hs name="code"}
instance Functor ((->) r) where
    fmap = (.)
```

This makes the revelation that using <span class="fixed">fmap</span>
over functions is just composition sort of obvious. Do <span
class="fixed">:m + Control.Monad.Instances</span>, since that's where
the instance is defined and then try playing with mapping over
functions.

``` {.haskell:hs name="code"}
ghci> :t fmap (*3) (+100)
fmap (*3) (+100) :: (Num a) => a -> a
ghci> fmap (*3) (+100) 1
303
ghci> (*3) `fmap` (+100) $ 1
303
ghci> (*3) . (+100) $ 1
303
ghci> fmap (show . (*3)) (*100) 1
"300"
```

We can call <span class="fixed">fmap</span> as an infix function so that
the resemblance to <span class="fixed">.</span> is clear. In the second
input line, we're mapping <span class="fixed">(\*3)</span> over <span
class="fixed">(+100)</span>, which results in a function that will take
an input, call <span class="fixed">(+100)</span> on that and then call
<span class="fixed">(\*3)</span> on that result. We call that function
with <span class="fixed">1</span>.

How does the box analogy hold here? Well, if you stretch it, it holds.
When we use <span class="fixed">fmap (+3)</span> over <span
class="fixed">Just 3</span>, it's easy to imagine the <span
class="fixed">Maybe</span> as a box that has some contents on which we
apply the function <span class="fixed">(+3)</span>. But what about when
we're doing <span class="fixed">fmap (\*3) (+100)</span>? Well, you can
think of the function <span class="fixed">(+100)</span> as a box that
contains its eventual result. Sort of like how an I/O action can be
thought of as a box that will go out into the real world and fetch some
result. Using <span class="fixed">fmap (\*3)</span> on <span
class="fixed">(+100)</span> will create another function that acts like
<span class="fixed">(+100)</span>, only before producing a result, <span
class="fixed">(\*3)</span> will be applied to that result. Now we can
see how <span class="fixed">fmap</span> acts just like <span
class="fixed">.</span> for functions.

The fact that <span class="fixed">fmap</span> is function composition
when used on functions isn't so terribly useful right now, but at least
it's very interesting. It also bends our minds a bit and let us see how
things that act more like computations than boxes (<span
class="fixed">IO</span> and <span class="fixed">(-\>) r</span>) can be
functors. The function being mapped over a computation results in the
same computation but the result of that computation is modified with the
function.

![lifting a function is easier than lifting a million
pounds](http://s3.amazonaws.com/lyah/lifter.png)
Before we go on to the rules that <span class="fixed">fmap</span> should
follow, let's think about the type of <span class="fixed">fmap</span>
once more. Its type is <span class="fixed">fmap :: (a -\> b) -\> f a -\>
f b</span>. We're missing the class constraint <span
class="fixed">(Functor f) =\></span>, but we left it out here for
brevity, because we're talking about functors anyway so we know what the
<span class="fixed">f</span> stands for. When we first learned about
[curried functions](higher-order-functions#curried-functions), we said
that all Haskell functions actually take one parameter. A function <span
class="fixed">a -\> b -\> c</span> actually takes just one parameter of
type <span class="fixed">a</span> and then returns a function <span
class="fixed">b -\> c</span>, which takes one parameter and returns a
<span class="fixed">c</span>. That's how if we call a function with too
few parameters (i.e. partially apply it), we get back a function that
takes the number of parameters that we left out (if we're thinking about
functions as taking several parameters again). So <span
class="fixed">a -\> b -\> c</span> can be written as <span
class="fixed">a -\> (b -\> c)</span>, to make the currying more
apparent.

In the same vein, if we write <span class="fixed">fmap :: (a -\> b) -\>
(f a -\> f b)</span>, we can think of <span class="fixed">fmap</span>
not as a function that takes one function and a functor and returns a
functor, but as a function that takes a function and returns a new
function that's just like the old one, only it takes a functor as a
parameter and returns a functor as the result. It takes an <span
class="fixed">a -\> b</span> function and returns a function <span
class="fixed">f a -\> f b</span>. This is called *lifting* a function.
Let's play around with that idea by using GHCI's <span
class="fixed">:t</span> command:

``` {.haskell:hs name="code"}
ghci> :t fmap (*2)
fmap (*2) :: (Num a, Functor f) => f a -> f a
ghci> :t fmap (replicate 3)
fmap (replicate 3) :: (Functor f) => f a -> f [a]
```

The expression <span class="fixed">fmap (\*2)</span> is a function that
takes a functor <span class="fixed">f</span> over numbers and returns a
functor over numbers. That functor can be a list, a <span
class="fixed">Maybe </span>, an <span class="fixed">Either
String</span>, whatever. The expression <span class="fixed">fmap
(replicate 3)</span> will take a functor over any type and return a
functor over a list of elements of that type.

<div class="hintbox">

When we say *a functor over numbers*, you can think of that as *a
functor that has numbers in it*. The former is a bit fancier and more
technically correct, but the latter is usually easier to get.

</div>

This is even more apparent if we partially apply, say, <span
class="fixed">fmap (++"!")</span> and then bind it to a name in GHCI.

You can think of <span class="fixed">fmap</span> as either a function
that takes a function and a functor and then maps that function over the
functor, or you can think of it as a function that takes a function and
lifts that function so that it operates on functors. Both views are
correct and in Haskell, equivalent.

The type <span class="fixed">fmap (replicate 3) :: (Functor f) =\> f
a -\> f [a]</span> means that the function will work on any functor.
What exactly it will do depends on which functor we use it on. If we use
<span class="fixed">fmap (replicate 3)</span> on a list, the list's
implementation for <span class="fixed">fmap</span> will be chosen, which
is just <span class="fixed">map</span>. If we use it on a <span
class="fixed">Maybe a</span>, it'll apply <span class="fixed">replicate
3</span> to the value inside the <span class="fixed">Just</span>, or if
it's <span class="fixed">Nothing</span>, then it stays <span
class="fixed">Nothing</span>.

``` {.haskell:hs name="code"}
ghci> fmap (replicate 3) [1,2,3,4]
[[1,1,1],[2,2,2],[3,3,3],[4,4,4]]
ghci> fmap (replicate 3) (Just 4)
Just [4,4,4]
ghci> fmap (replicate 3) (Right "blah")
Right ["blah","blah","blah"]
ghci> fmap (replicate 3) Nothing
Nothing
ghci> fmap (replicate 3) (Left "foo")
Left "foo"
```

Next up, we're going to look at the *functor laws*. In order for
something to be a functor, it should satisfy some laws. All functors are
expected to exhibit certain kinds of functor-like properties and
behaviors. They should reliably behave as things that can be mapped
over. Calling <span class="fixed">fmap</span> on a functor should just
map a function over the functor, nothing more. This behavior is
described in the functor laws. There are two of them that all instances
of <span class="fixed">Functor</span> should abide by. They aren't
enforced by Haskell automatically, so you have to test them out
yourself.

*The first functor law states that if we map the <span
class="fixed">id</span> function over a functor, the functor that we get
back should be the same as the original functor.* If we write that a bit
more formally, it means that <span class="label law">fmap id =
id</span>. So essentially, this says that if we do <span
class="fixed">fmap id</span> over a functor, it should be the same as
just calling <span class="fixed">id</span> on the functor. Remember,
<span class="fixed">id</span> is the identity function, which just
returns its parameter unmodified. It can also be written as <span
class="fixed">\\x -\> x</span>. If we view the functor as something that
can be mapped over, the <span class="label law">fmap id = id</span> law
seems kind of trivial or obvious.

Let's see if this law holds for a few values of functors.

``` {.haskell:hs name="code"}
ghci> fmap id (Just 3)
Just 3
ghci> id (Just 3)
Just 3
ghci> fmap id [1..5]
[1,2,3,4,5]
ghci> id [1..5]
[1,2,3,4,5]
ghci> fmap id []
[]
ghci> fmap id Nothing
Nothing
```

If we look at the implementation of <span class="fixed">fmap</span> for,
say, <span class="fixed">Maybe</span>, we can figure out why the first
functor law holds.

``` {.haskell:hs name="code"}
instance Functor Maybe where
    fmap f (Just x) = Just (f x)
    fmap f Nothing = Nothing
```

We imagine that <span class="fixed">id</span> plays the role of the
<span class="fixed">f</span> parameter in the implementation. We see
that if wee <span class="fixed">fmap id</span> over <span
class="fixed">Just x</span>, the result will be <span class="fixed">Just
(id x)</span>, and because <span class="fixed">id</span> just returns
its parameter, we can deduce that <span class="fixed">Just (id x)</span>
equals <span class="fixed">Just x</span>. So now we know that if we map
<span class="fixed">id</span> over a <span class="fixed">Maybe</span>
value with a <span class="fixed">Just</span> value constructor, we get
that same value back.

Seeing that mapping <span class="fixed">id</span> over a <span
class="fixed">Nothing</span> value returns the same value is trivial. So
from these two equations in the implementation for <span
class="fixed">fmap</span>, we see that the law <span class="fixed">fmap
id = id</span> holds.

![justice is blind, but so is my
dog](http://s3.amazonaws.com/lyah/justice.png)
*The second law says that composing two functions and then mapping the
resulting function over a functor should be the same as first mapping
one function over the functor and then mapping the other one.* Formally
written, that means that <span class="label law">fmap (f . g) = fmap f .
fmap g</span>. Or to write it in another way, for any functor *F*, the
following should hold: <span class="label law">fmap (f . g) F = fmap f
(fmap g F)</span>.

If we can show that some type obeys both functor laws, we can rely on it
having the same fundamental behaviors as other functors when it comes to
mapping. We can know that when we use <span class="fixed">fmap</span> on
it, there won't be anything other than mapping going on behind the
scenes and that it will act like a thing that can be mapped over, i.e. a
functor. You figure out how the second law holds for some type by
looking at the implementation of <span class="fixed">fmap</span> for
that type and then using the method that we used to check if <span
class="fixed">Maybe</span> obeys the first law.

If you want, we can check out how the second functor law holds for <span
class="fixed">Maybe</span>. If we do <span class="fixed">fmap (f .
g)</span> over <span class="fixed">Nothing</span>, we get <span
class="fixed">Nothing</span>, because doing a <span
class="fixed">fmap</span> with any function over <span
class="fixed">Nothing</span> returns <span class="fixed">Nothing</span>.
If we do <span class="fixed">fmap f (fmap g Nothing)</span>, we get
<span class="fixed">Nothing</span>, for the same reason. OK, seeing how
the second law holds for <span class="fixed">Maybe</span> if it's a
<span class="fixed">Nothing</span> value is pretty easy, almost trivial.

How about if it's a <span class="fixed">Just *something*</span> value?
Well, if we do <span class="fixed">fmap (f . g) (Just x)</span>, we see
from the implementation that it's implemented as <span
class="fixed">Just ((f . g) x)</span>, which is, of course, <span
class="fixed">Just (f (g x))</span>. If we do <span class="fixed">fmap f
(fmap g (Just x))</span>, we see from the implementation that <span
class="fixed">fmap g (Just x)</span> is <span class="fixed">Just (g
x)</span>. Ergo, <span class="fixed">fmap f (fmap g (Just x))</span>
equals <span class="fixed">fmap f (Just (g x))</span> and from the
implementation we see that this equals <span class="fixed">Just (f (g
x))</span>.

If you're a bit confused by this proof, don't worry. Be sure that you
understand how [function
composition](higher-order-functions#composition) works. Many times, you
can intuitively see how these laws hold because the types act like
containers or functions. You can also just try them on a bunch of
different values of a type and be able to say with some certainty that a
type does indeed obey the laws.

Let's take a look at a pathological example of a type constructor being
an instance of the <span class="fixed">Functor</span> typeclass but not
really being a functor, because it doesn't satisfy the laws. Let's say
that we have a type:

``` {.haskell:hs name="code"}
data CMaybe a = CNothing | CJust Int a deriving (Show)
```

The C here stands for *counter*. It's a data type that looks much like
<span class="fixed">Maybe a</span>, only the <span
class="fixed">Just</span> part holds two fields instead of one. The
first field in the <span class="fixed">CJust</span> value constructor
will always have a type of <span class="fixed">Int</span>, and it will
be some sort of counter and the second field is of type <span
class="fixed">a</span>, which comes from the type parameter and its type
will, of course, depend on the concrete type that we choose for <span
class="fixed">CMaybe a</span>. Let's play with our new type to get some
intuition for it.

``` {.haskell:hs name="code"}
ghci> CNothing
CNothing
ghci> CJust 0 "haha"
CJust 0 "haha"
ghci> :t CNothing
CNothing :: CMaybe a
ghci> :t CJust 0 "haha"
CJust 0 "haha" :: CMaybe [Char]
ghci> CJust 100 [1,2,3]
CJust 100 [1,2,3]
```

If we use the <span class="fixed">CNothing</span> constructor, there are
no fields, and if we use the <span class="fixed">CJust</span>
constructor, the first field is an integer and the second field can be
any type. Let's make this an instance of <span
class="fixed">Functor</span> so that everytime we use <span
class="fixed">fmap</span>, the function gets applied to the second
field, whereas the first field gets increased by 1.

``` {.haskell:hs name="code"}
instance Functor CMaybe where
    fmap f CNothing = CNothing
    fmap f (CJust counter x) = CJust (counter+1) (f x)
```

This is kind of like the instance implementation for <span
class="fixed">Maybe</span>, except that when we do <span
class="fixed">fmap</span> over a value that doesn't represent an empty
box (a <span class="fixed">CJust</span> value), we don't just apply the
function to the contents, we also increase the counter by 1. Everything
seems cool so far, we can even play with this a bit:

``` {.haskell:hs name="code"}
ghci> fmap (++"ha") (CJust 0 "ho")
CJust 1 "hoha"
ghci> fmap (++"he") (fmap (++"ha") (CJust 0 "ho"))
CJust 2 "hohahe"
ghci> fmap (++"blah") CNothing
CNothing
```

Does this obey the functor laws? In order to see that something doesn't
obey a law, it's enough to find just one counter-example.

``` {.haskell:hs name="code"}
ghci> fmap id (CJust 0 "haha")
CJust 1 "haha"
ghci> id (CJust 0 "haha")
CJust 0 "haha"
```

Ah! We know that the first functor law states that if we map <span
class="fixed">id</span> over a functor, it should be the same as just
calling <span class="fixed">id</span> with the same functor, but as
we've seen from this example, this is not true for our <span
class="fixed">CMaybe</span> functor. Even though it's part of the <span
class="fixed">Functor</span> typeclass, it doesn't obey the functor laws
and is therefore not a functor. If someone used our <span
class="fixed">CMaybe</span> type as a functor, they would expect it to
obey the functor laws like a good functor. But <span
class="fixed">CMaybe</span> fails at being a functor even though it
pretends to be one, so using it as a functor might lead to some faulty
code. When we use a functor, it shouldn't matter if we first compose a
few functions and then map them over the functor or if we just map each
function over a functor in succession. But with <span
class="fixed">CMaybe</span>, it matters, because it keeps track of how
many times it's been mapped over. Not cool! If we wanted <span
class="fixed">CMaybe</span> to obey the functor laws, we'd have to make
it so that the <span class="fixed">Int</span> field stays the same when
we use <span class="fixed">fmap</span>.

At first, the functor laws might seem a bit confusing and unnecessary,
but then we see that if we know that a type obeys both laws, we can make
certain assumptions about how it will act. If a type obeys the functor
laws, we know that calling <span class="fixed">fmap</span> on a value of
that type will only map the function over it, nothing more. This leads
to code that is more abstract and extensible, because we can use laws to
reason about behaviors that any functor should have and make functions
that operate reliably on any functor.

All the <span class="fixed">Functor</span> instances in the standard
library obey these laws, but you can check for yourself if you don't
believe me. And the next time you make a type an instance of <span
class="fixed">Functor</span>, take a minute to make sure that it obeys
the functor laws. Once you've dealt with enough functors, you kind of
intuitively see the properties and behaviors that they have in common
and it's not hard to intuitively see if a type obeys the functor laws.
But even without the intuition, you can always just go over the
implementation line by line and see if the laws hold or try to find a
counter-example.

We can also look at functors as things that output values in a context.
For instance, <span class="fixed">Just 3</span> outputs the value <span
class="fixed">3</span> in the context that it might or not output any
values at all. <span class="fixed">[1,2,3]</span> outputs three
values—<span class="fixed">1</span>, <span class="fixed">2</span>, and
<span class="fixed">3</span>, the context is that there may be multiple
values or no values. The function <span class="fixed">(+3)</span> will
output a value, depending on which parameter it is given.

If you think of functors as things that output values, you can think of
mapping over functors as attaching a transformation to the output of the
functor that changes the value. When we do <span class="fixed">fmap (+3)
[1,2,3]</span>, we attach the transformation <span
class="fixed">(+3)</span> to the output of <span
class="fixed">[1,2,3]</span>, so whenever we look at a number that the
list outputs, <span class="fixed">(+3)</span> will be applied to it.
Another example is mapping over functions. When we do <span
class="fixed">fmap (+3) (\*3)</span>, we attach the transformation <span
class="fixed">(+3)</span> to the eventual output of <span
class="fixed">(\*3)</span>. Looking at it this way gives us some
intuition as to why using <span class="fixed">fmap</span> on functions
is just composition (<span class="fixed">fmap (+3) (\*3)</span> equals
<span class="fixed">(+3) . (\*3)</span>, which equals <span
class="fixed">\\x -\> ((x\*3)+3)</span>), because we take a function
like <span class="fixed">(\*3)</span> then we attach the transformation
<span class="fixed">(+3)</span> to its output. The result is still a
function, only when we give it a number, it will be multiplied by three
and then it will go through the attached transformation where it will be
added to three. This is what happens with composition.

Applicative functors
--------------------

![disregard this analogy](http://s3.amazonaws.com/lyah/present.png)
In this section, we'll take a look at applicative functors, which are
beefed up functors, represented in Haskell by the <span
class="fixed">Applicative</span> typeclass, found in the <span
class="fixed">Control.Applicative</span> module.

As you know, functions in Haskell are curried by default, which means
that a function that seems to take several parameters actually takes
just one parameter and returns a function that takes the next parameter
and so on. If a function is of type <span class="fixed">a -\> b -\>
c</span>, we usually say that it takes two parameters and returns a
<span class="fixed">c</span>, but actually it takes an <span
class="fixed">a</span> and returns a function <span class="fixed">b -\>
c</span>. That's why we can call a function as <span class="fixed">f x
y</span> or as <span class="fixed">(f x) y</span>. This mechanism is
what enables us to partially apply functions by just calling them with
too few parameters, which results in functions that we can then pass on
to other functions.

So far, when we were mapping functions over functors, we usually mapped
functions that take only one parameter. But what happens when we map a
function like <span class="fixed">\*</span>, which takes two parameters,
over a functor? Let's take a look at a couple of concrete examples of
this. If we have <span class="fixed">Just 3</span> and we do <span
class="fixed">fmap (\*) (Just 3)</span>, what do we get? From the
instance implementation of <span class="fixed">Maybe</span> for <span
class="fixed">Functor</span>, we know that if it's a <span
class="fixed">Just *something*</span> value, it will apply the function
to the <span class="fixed">*something*</span> inside the <span
class="fixed">Just</span>. Therefore, doing <span class="fixed">fmap
(\*) (Just 3)</span> results in <span class="fixed">Just ((\*)
3)</span>, which can also be written as <span class="fixed">Just (\*
3)</span> if we use sections. Interesting! We get a function wrapped in
a <span class="fixed">Just</span>!

``` {.haskell:hs name="code"}
ghci> :t fmap (++) (Just "hey")
fmap (++) (Just "hey") :: Maybe ([Char] -> [Char])
ghci> :t fmap compare (Just 'a')
fmap compare (Just 'a') :: Maybe (Char -> Ordering)
ghci> :t fmap compare "A LIST OF CHARS"
fmap compare "A LIST OF CHARS" :: [Char -> Ordering]
ghci> :t fmap (\x y z -> x + y / z) [3,4,5,6]
fmap (\x y z -> x + y / z) [3,4,5,6] :: (Fractional a) => [a -> a -> a]
```

If we map <span class="fixed">compare</span>, which has a type of <span
class="fixed">(Ord a) =\> a -\> a -\> Ordering</span> over a list of
characters, we get a list of functions of type <span
class="fixed">Char -\> Ordering</span>, because the function <span
class="fixed">compare</span> gets partially applied with the characters
in the list. It's not a list of <span class="fixed">(Ord a) =\> a -\>
Ordering</span> function, because the first <span class="fixed">a</span>
that got applied was a <span class="fixed">Char</span> and so the second
<span class="fixed">a</span> has to decide to be of type <span
class="fixed">Char</span>.

We see how by mapping "multi-parameter" functions over functors, we get
functors that contain functions inside them. So now what can we do with
them? Well for one, we can map functions that take these functions as
parameters over them, because whatever is inside a functor will be given
to the function that we're mapping over it as a parameter.

``` {.haskell:hs name="code"}
ghci> let a = fmap (*) [1,2,3,4]
ghci> :t a
a :: [Integer -> Integer]
ghci> fmap (\f -> f 9) a
[9,18,27,36]
```

But what if we have a functor value of <span class="fixed">Just
(3 \*)</span> and a functor value of <span class="fixed">Just 5</span>
and we want to take out the function from <span class="fixed">Just
(3 \*)</span> and map it over <span class="fixed">Just 5</span>? With
normal functors, we're out of luck, because all they support is just
mapping normal functions over existing functors. Even when we mapped
<span class="fixed">\\f -\> f 9</span> over a functor that contained
functions inside it, we were just mapping a normal function over it. But
we can't map a function that's inside a functor over another functor
with what <span class="fixed">fmap</span> offers us. We could
pattern-match against the <span class="fixed">Just</span> constructor to
get the function out of it and then map it over <span class="fixed">Just
5</span>, but we're looking for a more general and abstract way of doing
that, which works across functors.

Meet the <span class="fixed">Applicative</span> typeclass. It lies in
the <span class="fixed">Control.Applicative</span> module and it defines
two methods, <span class="fixed">pure</span> and <span
class="fixed">\<\*\></span>. It doesn't provide a default implementation
for any of them, so we have to define them both if we want something to
be an applicative functor. The class is defined like so:

``` {.haskell:hs name="code"}
class (Functor f) => Applicative f where
    pure :: a -> f a
    (<*>) :: f (a -> b) -> f a -> f b
```

This simple three line class definition tells us a lot! Let's start at
the first line. It starts the definition of the <span
class="fixed">Applicative</span> class and it also introduces a class
constraint. It says that if we want to make a type constructor part of
the <span class="fixed">Applicative</span> typeclass, it has to be in
<span class="fixed">Functor</span> first. That's why if we know that if
a type constructor is part of the <span class="fixed">Applicative</span>
typeclass, it's also in <span class="fixed">Functor</span>, so we can
use <span class="fixed">fmap</span> on it.

The first method it defines is called <span class="fixed">pure</span>.
Its type declaration is <span class="fixed">pure :: a -\> f a</span>.
<span class="fixed">f</span> plays the role of our applicative functor
instance here. Because Haskell has a very good type system and because
everything a function can do is take some parameters and return some
value, we can tell a lot from a type declaration and this is no
exception. <span class="fixed">pure</span> should take a value of any
type and return an applicative functor with that value inside it. When
we say *inside it*, we're using the box analogy again, even though we've
seen that it doesn't always stand up to scrutiny. But the <span
class="fixed">a -\> f a</span> type declaration is still pretty
descriptive. We take a value and we wrap it in an applicative functor
that has that value as the result inside it.

A better way of thinking about <span class="fixed">pure</span> would be
to say that it takes a value and puts it in some sort of default (or
pure) context—a minimal context that still yields that value.

The <span class="fixed">\<\*\></span> function is really interesting. It
has a type declaration of <span class="fixed">f (a -\> b) -\> f a -\> f
b</span>. Does this remind you of anything? Of course, <span
class="fixed">fmap :: (a -\> b) -\> f a -\> f b</span>. It's a sort of a
beefed up <span class="fixed">fmap</span>. Whereas <span
class="fixed">fmap</span> takes a function and a functor and applies the
function inside the functor, <span class="fixed">\<\*\></span> takes a
functor that has a function in it and another functor and sort of
extracts that function from the first functor and then maps it over the
second one. When I say *extract*, I actually sort of mean *run* and then
extract, maybe even *sequence*. We'll see why soon.

Let's take a look at the <span class="fixed">Applicative</span> instance
implementation for <span class="fixed">Maybe</span>.

``` {.haskell:hs name="code"}
instance Applicative Maybe where
    pure = Just
    Nothing <*> _ = Nothing
    (Just f) <*> something = fmap f something
```

Again, from the class definition we see that the <span
class="fixed">f</span> that plays the role of the applicative functor
should take one concrete type as a parameter, so we write <span
class="fixed">instance Applicative Maybe where</span> instead of writing
<span class="fixed">instance Applicative (Maybe a) where</span>.

First off, <span class="fixed">pure</span>. We said earlier that it's
supposed to take something and wrap it in an applicative functor. We
wrote <span class="fixed">pure = Just</span>, because value constructors
like <span class="fixed">Just</span> are normal functions. We could have
also written <span class="fixed">pure x = Just x</span>.

Next up, we have the definition for <span class="fixed">\<\*\></span>.
We can't extract a function out of a <span class="fixed">Nothing</span>,
because it has no function inside it. So we say that if we try to
extract a function from a <span class="fixed">Nothing</span>, the result
is a <span class="fixed">Nothing</span>. If you look at the class
definition for <span class="fixed">Applicative</span>, you'll see that
there's a <span class="fixed">Functor</span> class constraint, which
means that we can assume that both of <span
class="fixed">\<\*\></span>'s parameters are functors. If the first
parameter is not a <span class="fixed">Nothing</span>, but a <span
class="fixed">Just</span> with some function inside it, we say that we
then want to map that function over the second parameter. This also
takes care of the case where the second parameter is <span
class="fixed">Nothing</span>, because doing <span
class="fixed">fmap</span> with any function over a <span
class="fixed">Nothing</span> will return a <span
class="fixed">Nothing</span>.

So for <span class="fixed">Maybe</span>, <span
class="fixed">\<\*\></span> extracts the function from the left value if
it's a <span class="fixed">Just</span> and maps it over the right value.
If any of the parameters is <span class="fixed">Nothing</span>, <span
class="fixed">Nothing</span> is the result.

OK cool great. Let's give this a whirl.

``` {.haskell:hs name="code"}
ghci> Just (+3) <*> Just 9
Just 12
ghci> pure (+3) <*> Just 10
Just 13
ghci> pure (+3) <*> Just 9
Just 12
ghci> Just (++"hahah") <*> Nothing
Nothing
ghci> Nothing <*> Just "woot"
Nothing
```

We see how doing <span class="fixed">pure (+3)</span> and <span
class="fixed">Just (+3)</span> is the same in this case. Use <span
class="fixed">pure</span> if you're dealing with <span
class="fixed">Maybe</span> values in an applicative context (i.e. using
them with <span class="fixed">\<\*\></span>), otherwise stick to <span
class="fixed">Just</span>. The first four input lines demonstrate how
the function is extracted and then mapped, but in this case, they could
have been achieved by just mapping unwrapped functions over functors.
The last line is interesting, because we try to extract a function from
a <span class="fixed">Nothing</span> and then map it over something,
which of course results in a <span class="fixed">Nothing</span>.

With normal functors, you can just map a function over a functor and
then you can't get the result out in any general way, even if the result
is a partially applied function. Applicative functors, on the other
hand, allow you to operate on several functors with a single function.
Check out this piece of code:

``` {.haskell:hs name="code"}
ghci> pure (+) <*> Just 3 <*> Just 5
Just 8
ghci> pure (+) <*> Just 3 <*> Nothing
Nothing
ghci> pure (+) <*> Nothing <*> Just 5
Nothing
```

![whaale](http://s3.amazonaws.com/lyah/whale.png)
What's going on here? Let's take a look, step by step. <span
class="fixed">\<\*\></span> is left-associative, which means that <span
class="fixed">pure (+) \<\*\> Just 3 \<\*\> Just 5</span> is the same as
<span class="fixed">(pure (+) \<\*\> Just 3) \<\*\> Just 5</span>.
First, the <span class="fixed">+</span> function is put in a functor,
which is in this case a <span class="fixed">Maybe</span> value that
contains the function. So at first, we have <span class="fixed">pure
(+)</span>, which is <span class="fixed">Just (+)</span>. Next, <span
class="fixed">Just (+) \<\*\> Just 3</span> happens. The result of this
is <span class="fixed">Just (3+)</span>. This is because of partial
application. Only applying <span class="fixed">3</span> to the <span
class="fixed">+</span> function results in a function that takes one
parameter and adds 3 to it. Finally, <span class="fixed">Just (3+)
\<\*\> Just 5</span> is carried out, which results in a <span
class="fixed">Just 8</span>.

Isn't this awesome?! Applicative functors and the applicative style of
doing <span class="fixed">pure f \<\*\> x \<\*\> y \<\*\> ...</span>
allow us to take a function that expects parameters that aren't
necessarily wrapped in functors and use that function to operate on
several values that are in functor contexts. The function can take as
many parameters as we want, because it's always partially applied step
by step between occurences of <span class="fixed">\<\*\></span>.

This becomes even more handy and apparent if we consider the fact that
<span class="fixed">pure f \<\*\> x</span> equals <span
class="fixed">fmap f x</span>. This is one of the applicative laws.
We'll take a closer look at them later, but for now, we can sort of
intuitively see that this is so. Think about it, it makes sense. Like we
said before, <span class="fixed">pure</span> puts a value in a default
context. If we just put a function in a default context and then extract
and apply it to a value inside another applicative functor, we did the
same as just mapping that function over that applicative functor.
Instead of writing <span class="fixed">pure f \<\*\> x \<\*\> y \<\*\>
...</span>, we can write <span class="fixed">fmap f x \<\*\> y \<\*\>
...</span>. This is why <span class="fixed">Control.Applicative</span>
exports a function called <span class="fixed">\<\$\></span>, which is
just <span class="fixed">fmap</span> as an infix operator. Here's how
it's defined:

``` {.haskell:hs name="code"}
(<$>) :: (Functor f) => (a -> b) -> f a -> f b
f <$> x = fmap f x
```

<div class="hintbox">

*Yo!* Quick reminder: type variables are independent of parameter names
or other value names. The <span class="fixed">f</span> in the function
declaration here is a type variable with a class constraint saying that
any type constructor that replaces <span class="fixed">f</span> should
be in the <span class="fixed">Functor</span> typeclass. The <span
class="fixed">f</span> in the function body denotes a function that we
map over <span class="fixed">x</span>. The fact that we used <span
class="fixed">f</span> to represent both of those doesn't mean that they
somehow represent the same thing.

</div>

By using <span class="fixed">\<\$\></span>, the applicative style really
shines, because now if we want to apply a function <span
class="fixed">f</span> between three applicative functors, we can write
<span class="fixed">f \<\$\> x \<\*\> y \<\*\> z</span>. If the
parameters weren't applicative functors but normal values, we'd write
<span class="fixed">f x y z</span>.

Let's take a closer look at how this works. We have a value of <span
class="fixed">Just "johntra"</span> and a value of <span
class="fixed">Just "volta"</span> and we want to join them into one
<span class="fixed">String</span> inside a <span
class="fixed">Maybe</span> functor. We do this:

``` {.haskell:hs name="code"}
ghci> (++) <$> Just "johntra" <*> Just "volta"
Just "johntravolta"
```

Before we see how this happens, compare the above line with this:

``` {.haskell:hs name="code"}
ghci> (++) "johntra" "volta"
"johntravolta"
```

Awesome! To use a normal function on applicative functors, just sprinkle
some <span class="fixed">\<\$\></span> and <span
class="fixed">\<\*\></span> about and the function will operate on
applicatives and return an applicative. How cool is that?

Anyway, when we do <span class="fixed">(++) \<\$\> Just "johntra" \<\*\>
Just "volta"</span>, first <span class="fixed">(++)</span>, which has a
type of <span class="fixed">(++) :: [a] -\> [a] -\> [a]</span> gets
mapped over <span class="fixed">Just "johntra"</span>, resulting in a
value that's the same as <span class="fixed">Just ("johntra"++)</span>
and has a type of <span class="fixed">Maybe ([Char] -\> [Char])</span>.
Notice how the first parameter of <span class="fixed">(++)</span> got
eaten up and how the <span class="fixed">a</span>s turned into <span
class="fixed">Char</span>s. And now <span class="fixed">Just
("johntra"++) \<\*\> Just "volta"</span> happens, which takes the
function out of the <span class="fixed">Just</span> and maps it over
<span class="fixed">Just "volta"</span>, resulting in <span
class="fixed">Just "johntravolta"</span>. Had any of the two values been
<span class="fixed">Nothing</span>, the result would have also been
<span class="fixed">Nothing</span>.

So far, we've only used <span class="fixed">Maybe</span> in our examples
and you might be thinking that applicative functors are all about <span
class="fixed">Maybe</span>. There are loads of other instances of <span
class="fixed">Applicative</span>, so let's go and meet them!

Lists (actually the list type constructor, <span
class="fixed">[]</span>) are applicative functors. What a suprise!
Here's how <span class="fixed">[]</span> is an instance of <span
class="fixed">Applicative</span>:

``` {.haskell:hs name="code"}
instance Applicative [] where
    pure x = [x]
    fs <*> xs = [f x | f <- fs, x <- xs]
```

Earlier, we said that <span class="fixed">pure</span> takes a value and
puts it in a default context. Or in other words, a minimal context that
still yields that value. The minimal context for lists would be the
empty list, <span class="fixed">[]</span>, but the empty list represents
the lack of a value, so it can't hold in itself the value that we used
<span class="fixed">pure</span> on. That's why <span
class="fixed">pure</span> takes a value and puts it in a singleton list.
Similarly, the minimal context for the <span class="fixed">Maybe</span>
applicative functor would be a <span class="fixed">Nothing</span>, but
it represents the lack of a value instead of a value, so <span
class="fixed">pure</span> is implemented as <span
class="fixed">Just</span> in the instance implementation for <span
class="fixed">Maybe</span>.

``` {.haskell:hs name="code"}
ghci> pure "Hey" :: [String]
["Hey"]
ghci> pure "Hey" :: Maybe String
Just "Hey"
```

What about <span class="fixed">\<\*\></span>? If we look at what <span
class="fixed">\<\*\></span>'s type would be if it were limited only to
lists, we get <span class="fixed">(\<\*\>) :: [a -\> b] -\> [a] -\>
[b]</span>. It's implemented with a [list
comprehension](starting-out#im-a-list-comprehension). <span
class="fixed">\<\*\></span> has to somehow extract the function out of
its left parameter and then map it over the right parameter. But the
thing here is that the left list can have zero functions, one function,
or several functions inside it. The right list can also hold several
values. That's why we use a list comprehension to draw from both lists.
We apply every possible function from the left list to every possible
value from the right list. The resulting list has every possible
combination of applying a function from the left list to a value in the
right one.

``` {.haskell:hs name="code"}
ghci> [(*0),(+100),(^2)] <*> [1,2,3]
[0,0,0,101,102,103,1,4,9]
```

The left list has three functions and the right list has three values,
so the resulting list will have nine elements. Every function in the
left list is applied to every function in the right one. If we have a
list of functions that take two parameters, we can apply those functions
between two lists.

``` {.haskell:hs name="code"}
ghci> [(+),(*)] <*> [1,2] <*> [3,4]
[4,5,5,6,3,4,6,8]
```

Because <span class="fixed">\<\*\></span> is left-associative, <span
class="fixed">[(+),(\*)] \<\*\> [1,2]</span> happens first, resulting in
a list that's the same as <span
class="fixed">[(1+),(2+),(1\*),(2\*)]</span>, because every function on
the left gets applied to every value on the right. Then, <span
class="fixed">[(1+),(2+),(1\*),(2\*)] \<\*\> [3,4]</span> happens, which
produces the final result.

Using the applicative style with lists is fun! Watch:

``` {.haskell:hs name="code"}
ghci> (++) <$> ["ha","heh","hmm"] <*> ["?","!","."]
["ha?","ha!","ha.","heh?","heh!","heh.","hmm?","hmm!","hmm."]
```

Again, see how we used a normal function that takes two strings between
two applicative functors of strings just by inserting the appropriate
applicative operators.

You can view lists as non-deterministic computations. A value like <span
class="fixed">100</span> or <span class="fixed">"what"</span> can be
viewed as a deterministic computation that has only one result, whereas
a list like <span class="fixed">[1,2,3]</span> can be viewed as a
computation that can't decide on which result it wants to have, so it
presents us with all of the possible results. So when you do something
like <span class="fixed">(+) \<\$\> [1,2,3] \<\*\> [4,5,6]</span>, you
can think of it as adding together two non-deterministic computations
with <span class="fixed">+</span>, only to produce another
non-deterministic computation that's even less sure about its result.

Using the applicative style on lists is often a good replacement for
list comprehensions. In the second chapter, we wanted to see all the
possible products of <span class="fixed">[2,5,10]</span> and <span
class="fixed">[8,10,11]</span>, so we did this:

``` {.haskell:hs name="code"}
ghci> [ x*y | x <- [2,5,10], y <- [8,10,11]]
[16,20,22,40,50,55,80,100,110]
```

We're just drawing from two lists and applying a function between every
combination of elements. This can be done in the applicative style as
well:

``` {.haskell:hs name="code"}
ghci> (*) <$> [2,5,10] <*> [8,10,11]
[16,20,22,40,50,55,80,100,110]
```

This seems clearer to me, because it's easier to see that we're just
calling <span class="fixed">\*</span> between two non-deterministic
computations. If we wanted all possible products of those two lists that
are more than 50, we'd just do:

``` {.haskell:hs name="code"}
ghci> filter (>50) $ (*) <$> [2,5,10] <*> [8,10,11]
[55,80,100,110]
```

It's easy to see how <span class="fixed">pure f \<\*\> xs</span> equals
<span class="fixed">fmap f xs</span> with lists. <span
class="fixed">pure f</span> is just <span class="fixed">[f]</span> and
<span class="fixed">[f] \<\*\> xs</span> will apply every function in
the left list to every value in the right one, but there's just one
function in the left list, so it's like mapping.

Another instance of <span class="fixed">Applicative</span> that we've
already encountered is <span class="fixed">IO</span>. This is how the
instance is implemented:

``` {.haskell:hs name="code"}
instance Applicative IO where
    pure = return
    a <*> b = do
        f <- a
        x <- b
        return (f x)
```

![ahahahah!](http://s3.amazonaws.com/lyah/knight.png)
Since <span class="fixed">pure</span> is all about putting a value in a
minimal context that still holds it as its result, it makes sense that
<span class="fixed">pure</span> is just <span
class="fixed">return</span>, because <span class="fixed">return</span>
does exactly that; it makes an I/O action that doesn't do anything, it
just yields some value as its result, but it doesn't really do any I/O
operations like printing to the terminal or reading from a file.

If <span class="fixed">\<\*\></span> were specialized for <span
class="fixed">IO</span> it would have a type of <span
class="fixed">(\<\*\>) :: IO (a -\> b) -\> IO a -\> IO b</span>. It
would take an I/O action that yields a function as its result and
another I/O action and create a new I/O action from those two that, when
performed, first performs the first one to get the function and then
performs the second one to get the value and then it would yield that
function applied to the value as its result. We used *do* syntax to
implement it here. Remember, *do* syntax is about taking several I/O
actions and gluing them into one, which is exactly what we do here.

With <span class="fixed">Maybe</span> and <span class="fixed">[]</span>,
we could think of <span class="fixed">\<\*\></span> as simply extracting
a function from its left parameter and then sort of applying it over the
right one. With <span class="fixed">IO</span>, extracting is still in
the game, but now we also have a notion of *sequencing*, because we're
taking two I/O actions and we're sequencing, or gluing, them into one.
We have to extract the function from the first I/O action, but to
extract a result from an I/O action, it has to be performed.

Consider this:

``` {.haskell:hs name="code"}
myAction :: IO String
myAction = do
    a <- getLine
    b <- getLine
    return $ a ++ b
```

This is an I/O action that will prompt the user for two lines and yield
as its result those two lines concatenated. We achieved it by gluing
together two <span class="fixed">getLine</span> I/O actions and a <span
class="fixed">return</span>, because we wanted our new glued I/O action
to hold the result of <span class="fixed">a ++ b</span>. Another way of
writing this would be to use the applicative style.

``` {.haskell:hs name="code"}
myAction :: IO String
myAction = (++) <$> getLine <*> getLine
```

What we were doing before was making an I/O action that applied a
function between the results of two other I/O actions, and this is the
same thing. Remember, <span class="fixed">getLine</span> is an I/O
action with the type <span class="fixed">getLine :: IO String</span>.
When we use <span class="fixed">\<\*\></span> between two applicative
functors, the result is an applicative functor, so this all makes sense.

If we regress to the box analogy, we can imagine <span
class="fixed">getLine</span> as a box that will go out into the real
world and fetch us a string. Doing <span class="fixed">(++) \<\$\>
getLine \<\*\> getLine</span> makes a new, bigger box that sends those
two boxes out to fetch lines from the terminal and then presents the
concatenation of those two lines as its result.

The type of the expression <span class="fixed">(++) \<\$\> getLine
\<\*\> getLine</span> is <span class="fixed">IO String</span>, which
means that this expression is a completely normal I/O action like any
other, which also holds a result value inside it, just like other I/O
actions. That's why we can do stuff like:

``` {.haskell:hs name="code"}
main = do
    a <- (++) <$> getLine <*> getLine
    putStrLn $ "The two lines concatenated turn out to be: " ++ a
```

If you ever find yourself binding some I/O actions to names and then
calling some function on them and presenting that as the result by using
<span class="fixed">return</span>, consider using the applicative style
because it's arguably a bit more concise and terse.

Another instance of <span class="fixed">Applicative</span> is <span
class="fixed">(-\>) r</span>, so functions. They are rarely used with
the applicative style outside of code golf, but they're still
interesting as applicatives, so let's take a look at how the function
instance is implemented.

<div class="hintbox">

If you're confused about what <span class="fixed">(-\>) r</span> means,
check out the previous section where we explain how <span
class="fixed">(-\>) r</span> is a functor.

</div>

``` {.haskell:hs name="code"}
instance Applicative ((->) r) where
    pure x = (\_ -> x)
    f <*> g = \x -> f x (g x)
```

When we wrap a value into an applicative functor with <span
class="fixed">pure</span>, the result it yields always has to be that
value. A minimal default context that still yields that value as a
result. That's why in the function instance implementation, <span
class="fixed">pure</span> takes a value and creates a function that
ignores its parameter and always returns that value. If we look at the
type for <span class="fixed">pure</span>, but specialized for the <span
class="fixed">(-\>) r</span> instance, it's <span class="fixed">pure ::
a -\> (r -\> a)</span>.

``` {.haskell:hs name="code"}
ghci> (pure 3) "blah"
3
```

Because of currying, function application is left-associative, so we can
omit the parentheses.

``` {.haskell:hs name="code"}
ghci> pure 3 "blah"
3
```

The instance implementation for <span class="fixed">\<\*\></span> is a
bit cryptic, so it's best if we just take a look at how to use functions
as applicative functors in the applicative style.

``` {.haskell:hs name="code"}
ghci> :t (+) <$> (+3) <*> (*100)
(+) <$> (+3) <*> (*100) :: (Num a) => a -> a
ghci> (+) <$> (+3) <*> (*100) $ 5
508
```

Calling <span class="fixed">\<\*\></span> with two applicative functors
results in an applicative functor, so if we use it on two functions, we
get back a function. So what goes on here? When we do <span
class="fixed">(+) \<\$\> (+3) \<\*\> (\*100)</span>, we're making a
function that will use <span class="fixed">+</span> on the results of
<span class="fixed">(+3)</span> and <span class="fixed">(\*100)</span>
and return that. To demonstrate on a real example, when we did <span
class="fixed">(+) \<\$\> (+3) \<\*\> (\*100) \$ 5</span>, the <span
class="fixed">5</span> first got applied to <span
class="fixed">(+3)</span> and <span class="fixed">(\*100)</span>,
resulting in <span class="fixed">8</span> and <span
class="fixed">500</span>. Then, <span class="fixed">+</span> gets called
with <span class="fixed">8</span> and <span class="fixed">500</span>,
resulting in <span class="fixed">508</span>.

``` {.haskell:hs name="code"}
ghci> (\x y z -> [x,y,z]) <$> (+3) <*> (*2) <*> (/2) $ 5
[8.0,10.0,2.5]
```

![SLAP](http://s3.amazonaws.com/lyah/jazzb.png)
Same here. We create a function that will call the function <span
class="fixed">\\x y z -\> [x,y,z]</span> with the eventual results from
<span class="fixed">(+3)</span>, <span class="fixed">(\*2)</span> and
<span class="fixed">(/2)</span>. The <span class="fixed">5</span> gets
fed to each of the three functions and then <span class="fixed">\\x y
z -\> [x, y, z]</span> gets called with those results.

You can think of functions as boxes that contain their eventual results,
so doing <span class="fixed">k \<\$\> f \<\*\> g</span> creates a
function that will call <span class="fixed">k</span> with the eventual
results from <span class="fixed">f</span> and <span
class="fixed">g</span>. When we do something like <span
class="fixed">(+) \<\$\> Just 3 \<\*\> Just 5</span>, we're using <span
class="fixed">+</span> on values that might or might not be there, which
also results in a value that might or might not be there. When we do
<span class="fixed">(+) \<\$\> (+10) \<\*\> (+5)</span>, we're using
<span class="fixed">+</span> on the future return values of <span
class="fixed">(+10)</span> and <span class="fixed">(+5)</span> and the
result is also something that will produce a value only when called with
a parameter.

We don't often use functions as applicatives, but this is still really
interesting. It's not very important that you get how the <span
class="fixed">(-\>) r</span> instance for <span
class="fixed">Applicative</span> works, so don't despair if you're not
getting this right now. Try playing with the applicative style and
functions to build up an intuition for functions as applicatives.

An instance of <span class="fixed">Applicative</span> that we haven't
encountered yet is <span class="fixed">ZipList</span>, and it lives in
<span class="fixed">Control.Applicative</span>.

It turns out there are actually more ways for lists to be applicative
functors. One way is the one we already covered, which says that calling
<span class="fixed">\<\*\></span> with a list of functions and a list of
values results in a list which has all the possible combinations of
applying functions from the left list to the values in the right list.
If we do <span class="fixed">[(+3),(\*2)] \<\*\> [1,2]</span>, <span
class="fixed">(+3)</span> will be applied to both <span
class="fixed">1</span> and <span class="fixed">2</span> and <span
class="fixed">(\*2)</span> will also be applied to both <span
class="fixed">1</span> and <span class="fixed">2</span>, resulting in a
list that has four elements, namely <span
class="fixed">[4,5,2,4]</span>.

However, <span class="fixed">[(+3),(\*2)] \<\*\> [1,2]</span> could also
work in such a way that the first function in the left list gets applied
to the first value in the right one, the second function gets applied to
the second value, and so on. That would result in a list with two
values, namely <span class="fixed">[4,4]</span>. You could look at it as
<span class="fixed">[1 + 3, 2 \* 2]</span>.

Because one type can't have two instances for the same typeclass, the
<span class="fixed">ZipList a</span> type was introduced, which has one
constructor <span class="fixed">ZipList</span> that has just one field,
and that field is a list. Here's the instance:

``` {.haskell:hs name="code"}
instance Applicative ZipList where
        pure x = ZipList (repeat x)
        ZipList fs <*> ZipList xs = ZipList (zipWith (\f x -> f x) fs xs)
```

<span class="fixed">\<\*\></span> does just what we said. It applies the
first function to the first value, the second function to the second
value, etc. This is done with <span class="fixed">zipWith (\\f x -\> f
x) fs xs</span>. Because of how <span class="fixed">zipWith</span>
works, the resulting list will be as long as the shorter of the two
lists.

<span class="fixed">pure</span> is also interesting here. It takes a
value and puts it in a list that just has that value repeating
indefinitely. <span class="fixed">pure "haha"</span> results in <span
class="fixed">ZipList (["haha","haha","haha"...</span>. This might be a
bit confusing since we said that <span class="fixed">pure</span> should
put a value in a minimal context that still yields that value. And you
might be thinking that an infinite list of something is hardly minimal.
But it makes sense with zip lists, because it has to produce the value
on every position. This also satisfies the law that <span
class="fixed">pure f \<\*\> xs</span> should equal <span
class="fixed">fmap f xs</span>. If <span class="fixed">pure 3</span>
just returned <span class="fixed">ZipList [3]</span>, <span
class="fixed">pure (\*2) \<\*\> ZipList [1,5,10]</span> would result in
<span class="fixed">ZipList [2]</span>, because the resulting list of
two zipped lists has the length of the shorter of the two. If we zip a
finite list with an infinite list, the length of the resulting list will
always be equal to the length of the finite list.

So how do zip lists work in an applicative style? Let's see. Oh, the
<span class="fixed">ZipList a</span> type doesn't have a <span
class="fixed">Show</span> instance, so we have to use the <span
class="label function">getZipList</span> function to extract a raw list
out of a zip list.

``` {.haskell:hs name="code"}
ghci> getZipList $ (+) <$> ZipList [1,2,3] <*> ZipList [100,100,100]
[101,102,103]
ghci> getZipList $ (+) <$> ZipList [1,2,3] <*> ZipList [100,100..]
[101,102,103]
ghci> getZipList $ max <$> ZipList [1,2,3,4,5,3] <*> ZipList [5,3,1,2]
[5,3,3,4]
ghci> getZipList $ (,,) <$> ZipList "dog" <*> ZipList "cat" <*> ZipList "rat"
[('d','c','r'),('o','a','a'),('g','t','t')]
```

<div class="hintbox">

The <span class="fixed">(,,)</span> function is the same as <span
class="fixed">\\x y z -\> (x,y,z)</span>. Also, the <span
class="fixed">(,)</span> function is the same as <span class="fixed">\\x
y -\> (x,y)</span>.

</div>

Aside from <span class="fixed">zipWith</span>, the standard library has
functions such as <span class="fixed">zipWith3</span>, <span
class="fixed">zipWith4</span>, all the way up to 7. <span
class="fixed">zipWith</span> takes a function that takes two parameters
and zips two lists with it. <span class="fixed">zipWith3</span> takes a
function that takes three parameters and zips three lists with it, and
so on. By using zip lists with an applicative style, we don't have to
have a separate zip function for each number of lists that we want to
zip together. We just use the applicative style to zip together an
arbitrary amount of lists with a function, and that's pretty cool.

<span class="fixed">Control.Applicative</span> defines a function that's
called <span class="label function">liftA2</span>, which has a type of
<span class="fixed">liftA2 :: (Applicative f) =\> (a -\> b -\> c) -\> f
a -\> f b -\> f c</span> . It's defined like this:

``` {.haskell:hs name="code"}
liftA2 :: (Applicative f) => (a -> b -> c) -> f a -> f b -> f c
liftA2 f a b = f <$> a <*> b
```

Nothing special, it just applies a function between two applicatives,
hiding the applicative style that we've become familiar with. The reason
we're looking at it is because it clearly showcases why applicative
functors are more powerful than just ordinary functors. With ordinary
functors, we can just map functions over one functor. But with
applicative functors, we can apply a function between several functors.
It's also interesting to look at this function's type as <span
class="fixed">(a -\> b -\> c) -\> (f a -\> f b -\> f c)</span>. When we
look at it like this, we can say that <span class="fixed">liftA2</span>
takes a normal binary function and promotes it to a function that
operates on two functors.

Here's an interesting concept: we can take two applicative functors and
combine them into one applicative functor that has inside it the results
of those two applicative functors in a list. For instance, we have <span
class="fixed">Just 3</span> and <span class="fixed">Just 4</span>. Let's
assume that the second one has a singleton list inside it, because
that's really easy to achieve:

``` {.haskell:hs name="code"}
ghci> fmap (\x -> [x]) (Just 4)
Just [4]
```

OK, so let's say we have <span class="fixed">Just 3</span> and <span
class="fixed">Just [4]</span>. How do we get <span class="fixed">Just
[3,4]</span>? Easy.

``` {.haskell:hs name="code"}
ghci> liftA2 (:) (Just 3) (Just [4])
Just [3,4]
ghci> (:) <$> Just 3 <*> Just [4]
Just [3,4]
```

Remember, <span class="fixed">:</span> is a function that takes an
element and a list and returns a new list with that element at the
beginning. Now that we have <span class="fixed">Just [3,4]</span>, could
we combine that with <span class="fixed">Just 2</span> to produce <span
class="fixed">Just [2,3,4]</span>? Of course we could. It seems that we
can combine any amount of applicatives into one applicative that has a
list of the results of those applicatives inside it. Let's try
implementing a function that takes a list of applicatives and returns an
applicative that has a list as its result value. We'll call it <span
class="fixed">sequenceA</span>.

``` {.haskell:hs name="code"}
sequenceA :: (Applicative f) => [f a] -> f [a]
sequenceA [] = pure []
sequenceA (x:xs) = (:) <$> x <*> sequenceA xs
```

Ah, recursion! First, we look at the type. It will transform a list of
applicatives into an applicative with a list. From that, we can lay some
groundwork for an edge condition. If we want to turn an empty list into
an applicative with a list of results, well, we just put an empty list
in a default context. Now comes the recursion. If we have a list with a
head and a tail (remember, <span class="fixed">x</span> is an
applicative and <span class="fixed">xs</span> is a list of them), we
call <span class="fixed">sequenceA</span> on the tail, which results in
an applicative with a list. Then, we just prepend the value inside the
applicative <span class="fixed">x</span> into that applicative with a
list, and that's it!

So if we do <span class="fixed">sequenceA [Just 1, Just 2]</span>,
that's <span class="fixed">(:) \<\$\> Just 1 \<\*\> sequenceA [Just 2]
</span>. That equals <span class="fixed">(:) \<\$\> Just 1 \<\*\> ((:)
\<\$\> Just 2 \<\*\> sequenceA [])</span>. Ah! We know that <span
class="fixed">sequenceA []</span> ends up as being <span
class="fixed">Just []</span>, so this expression is now <span
class="fixed">(:) \<\$\> Just 1 \<\*\> ((:) \<\$\> Just 2 \<\*\> Just
[])</span>, which is <span class="fixed">(:) \<\$\> Just 1 \<\*\> Just
[2]</span>, which is <span class="fixed">Just [1,2]</span>!

Another way to implement <span class="fixed">sequenceA</span> is with a
fold. Remember, pretty much any function where we go over a list element
by element and accumulate a result along the way can be implemented with
a fold.

``` {.haskell:hs name="code"}
sequenceA :: (Applicative f) => [f a] -> f [a]
sequenceA = foldr (liftA2 (:)) (pure [])
```

We approach the list from the right and start off with an accumulator
value of <span class="fixed">pure []</span>. We do <span
class="fixed">liftA2 (:)</span> between the accumulator and the last
element of the list, which results in an applicative that has a
singleton in it. Then we do <span class="fixed">liftA2 (:)</span> with
the now last element and the current accumulator and so on, until we're
left with just the accumulator, which holds a list of the results of all
the applicatives.

Let's give our function a whirl on some applicatives.

``` {.haskell:hs name="code"}
ghci> sequenceA [Just 3, Just 2, Just 1]
Just [3,2,1]
ghci> sequenceA [Just 3, Nothing, Just 1]
Nothing
ghci> sequenceA [(+3),(+2),(+1)] 3
[6,5,4]
ghci> sequenceA [[1,2,3],[4,5,6]]
[[1,4],[1,5],[1,6],[2,4],[2,5],[2,6],[3,4],[3,5],[3,6]]
ghci> sequenceA [[1,2,3],[4,5,6],[3,4,4],[]]
[]
```

Ah! Pretty cool. When used on <span class="fixed">Maybe</span> values,
<span class="fixed">sequenceA</span> creates a <span
class="fixed">Maybe</span> value with all the results inside it as a
list. If one of the values was <span class="fixed">Nothing</span>, then
the result is also a <span class="fixed">Nothing</span>. This is cool
when you have a list of <span class="fixed">Maybe</span> values and
you're interested in the values only if none of them is a <span
class="fixed">Nothing</span>.

When used with functions, <span class="fixed">sequenceA</span> takes a
list of functions and returns a function that returns a list. In our
example, we made a function that took a number as a parameter and
applied it to each function in the list and then returned a list of
results. <span class="fixed">sequenceA [(+3),(+2),(+1)] 3</span> will
call <span class="fixed">(+3)</span> with <span class="fixed">3</span>,
<span class="fixed">(+2)</span> with <span class="fixed">3</span> and
<span class="fixed">(+1)</span> with <span class="fixed">3</span> and
present all those results as a list.

Doing <span class="fixed">(+) \<\$\> (+3) \<\*\> (\*2)</span> will
create a function that takes a parameter, feeds it to both <span
class="fixed">(+3)</span> and <span class="fixed">(\*2)</span> and then
calls <span class="fixed">+</span> with those two results. In the same
vein, it makes sense that <span class="fixed">sequenceA
[(+3),(\*2)]</span> makes a function that takes a parameter and feeds it
to all of the functions in the list. Instead of calling <span
class="fixed">+</span> with the results of the functions, a combination
of <span class="fixed">:</span> and <span class="fixed">pure []</span>
is used to gather those results in a list, which is the result of that
function.

Using <span class="fixed">sequenceA</span> is cool when we have a list
of functions and we want to feed the same input to all of them and then
view the list of results. For instance, we have a number and we're
wondering whether it satisfies all of the predicates in a list. One way
to do that would be like so:

``` {.haskell:hs name="code"}
ghci> map (\f -> f 7) [(>4),(<10),odd]
[True,True,True]
ghci> and $ map (\f -> f 7) [(>4),(<10),odd]
True
```

Remember, <span class="fixed">and</span> takes a list of booleans and
returns <span class="fixed">True</span> if they're all <span
class="fixed">True</span>. Another way to achieve the same thing would
be with <span class="fixed">sequenceA</span>:

``` {.haskell:hs name="code"}
ghci> sequenceA [(>4),(<10),odd] 7
[True,True,True]
ghci> and $ sequenceA [(>4),(<10),odd] 7
True
```

<span class="fixed">sequenceA [(\>4),(\<10),odd]</span> creates a
function that will take a number and feed it to all of the predicates in
<span class="fixed">[(\>4),(\<10),odd]</span> and return a list of
booleans. It turns a list with the type <span class="fixed">(Num a) =\>
[a -\> Bool]</span> into a function with the type <span
class="fixed">(Num a) =\> a -\> [Bool]</span>. Pretty neat, huh?

Because lists are homogenous, all the functions in the list have to be
functions of the same type, of course. You can't have a list like <span
class="fixed">[ord, (+3)]</span>, because <span class="fixed">ord</span>
takes a character and returns a number, whereas <span
class="fixed">(+3)</span> takes a number and returns a number.

When used with <span class="fixed">[]</span>, <span
class="fixed">sequenceA</span> takes a list of lists and returns a list
of lists. Hmm, interesting. It actually creates lists that have all
possible combinations of their elements. For illustration, here's the
above done with <span class="fixed">sequenceA</span> and then done with
a list comprehension:

``` {.haskell:hs name="code"}
ghci> sequenceA [[1,2,3],[4,5,6]]
[[1,4],[1,5],[1,6],[2,4],[2,5],[2,6],[3,4],[3,5],[3,6]]
ghci> [[x,y] | x <- [1,2,3], y <- [4,5,6]]
[[1,4],[1,5],[1,6],[2,4],[2,5],[2,6],[3,4],[3,5],[3,6]]
ghci> sequenceA [[1,2],[3,4]]
[[1,3],[1,4],[2,3],[2,4]]
ghci> [[x,y] | x <- [1,2], y <- [3,4]]
[[1,3],[1,4],[2,3],[2,4]]
ghci> sequenceA [[1,2],[3,4],[5,6]]
[[1,3,5],[1,3,6],[1,4,5],[1,4,6],[2,3,5],[2,3,6],[2,4,5],[2,4,6]]
ghci> [[x,y,z] | x <- [1,2], y <- [3,4], z <- [5,6]]
[[1,3,5],[1,3,6],[1,4,5],[1,4,6],[2,3,5],[2,3,6],[2,4,5],[2,4,6]]
```

This might be a bit hard to grasp, but if you play with it for a while,
you'll see how it works. Let's say that we're doing <span
class="fixed">sequenceA [[1,2],[3,4]]</span>. To see how this happens,
let's use the <span class="fixed">sequenceA (x:xs) = (:) \<\$\> x \<\*\>
sequenceA xs</span> definition of <span class="fixed">sequenceA</span>
and the edge condition <span class="fixed">sequenceA [] = pure
[]</span>. You don't have to follow this evaluation, but it might help
you if have trouble imagining how <span class="fixed">sequenceA</span>
works on lists of lists, because it can be a bit mind-bending.

-   We start off with <span class="fixed">sequenceA [[1,2],[3,4]]</span>
-   That evaluates to <span class="fixed">(:) \<\$\> [1,2] \<\*\>
    sequenceA [[3,4]]</span>
-   Evaluating the inner <span class="fixed">sequenceA</span> further,
    we get <span class="fixed">(:) \<\$\> [1,2] \<\*\> ((:) \<\$\> [3,4]
    \<\*\> sequenceA [])</span>
-   We've reached the edge condition, so this is now <span
    class="fixed">(:) \<\$\> [1,2] \<\*\> ((:) \<\$\> [3,4] \<\*\>
    [[]])</span>
-   Now, we evaluate the <span class="fixed">(:) \<\$\> [3,4] \<\*\>
    [[]]</span> part, which will use <span class="fixed">:</span> with
    every possible value in the left list (possible values are <span
    class="fixed">3</span> and <span class="fixed">4</span>) with every
    possible value on the right list (only possible value is <span
    class="fixed">[]</span>), which results in <span
    class="fixed">[3:[], 4:[]]</span>, which is <span
    class="fixed">[[3],[4]]</span>. So now we have <span
    class="fixed">(:) \<\$\> [1,2] \<\*\> [[3],[4]]</span>
-   Now, <span class="fixed">:</span> is used with every possible value
    from the left list (<span class="fixed">1</span> and <span
    class="fixed">2</span>) with every possible value in the right list
    (<span class="fixed">[3]</span> and <span class="fixed">[4]</span>),
    which results in <span class="fixed">[1:[3], 1:[4], 2:[3],
    2:[4]]</span>, which is <span
    class="fixed">[[1,3],[1,4],[2,3],[2,4]</span>

Doing <span class="fixed">(+) \<\$\> [1,2] \<\*\> [4,5,6]</span>results
in a non-deterministic computation <span class="fixed">x + y</span>
where <span class="fixed">x</span> takes on every value from <span
class="fixed">[1,2]</span> and <span class="fixed">y</span> takes on
every value from <span class="fixed">[4,5,6]</span>. We represent that
as a list which holds all of the possible results. Similarly, when we do
<span class="fixed">sequence [[1,2],[3,4],[5,6],[7,8]]</span>, the
result is a non-deterministic computation <span
class="fixed">[x,y,z,w]</span>, where <span class="fixed">x</span> takes
on every value from <span class="fixed">[1,2]</span>, <span
class="fixed">y</span> takes on every value from <span
class="fixed">[3,4]</span> and so on. To represent the result of that
non-deterministic computation, we use a list, where each element in the
list is one possible list. That's why the result is a list of lists.

When used with I/O actions, <span class="fixed">sequenceA</span> is the
same thing as <span class="fixed">sequence</span>! It takes a list of
I/O actions and returns an I/O action that will perform each of those
actions and have as its result a list of the results of those I/O
actions. That's because to turn an <span class="fixed">[IO a]</span>
value into an <span class="fixed">IO [a]</span> value, to make an I/O
action that yields a list of results when performed, all those I/O
actions have to be sequenced so that they're then performed one after
the other when evaluation is forced. You can't get the result of an I/O
action without performing it.

``` {.haskell:hs name="code"}
ghci> sequenceA [getLine, getLine, getLine]
heyh
ho
woo
["heyh","ho","woo"]
```

Like normal functors, applicative functors come with a few laws. The
most important one is the one that we already mentioned, namely that
<span class="label law">pure f \<\*\> x = fmap f x</span> holds. As an
exercise, you can prove this law for some of the applicative functors
that we've met in this chapter.The other functor laws are:

-   <span class="label law">pure id \<\*\> v = v</span>
-   <span class="label law">pure (.) \<\*\> u \<\*\> v \<\*\> w = u
    \<\*\> (v \<\*\> w)</span>
-   <span class="label law">pure f \<\*\> pure x = pure (f x)</span>
-   <span class="label law">u \<\*\> pure y = pure (\$ y) \<\*\>
    u</span>

We won't go over them in detail right now because that would take up a
lot of pages and it would probably be kind of boring, but if you're up
to the task, you can take a closer look at them and see if they hold for
some of the instances.

In conclusion, applicative functors aren't just interesting, they're
also useful, because they allow us to combine different computations,
such as I/O computations, non-deterministic computations, computations
that might have failed, etc. by using the applicative style. Just by
using <span class="fixed">\<\$\></span> and <span
class="fixed">\<\*\></span> we can use normal functions to uniformly
operate on any number of applicative functors and take advantage of the
semantics of each one.

The newtype keyword
-------------------

![why\_ so serious?](http://s3.amazonaws.com/lyah/maoi.png)
So far, we've learned how to make our own algebraic data types by using
the *data* keyword. We've also learned how to give existing types
synonyms with the *type* keyword. In this section, we'll be taking a
look at how to make new types out of existing data types by using the
*newtype* keyword and why we'd want to do that in the first place.

In the previous section, we saw that there are actually more ways for
the list type to be an applicative functor. One way is to have <span
class="fixed">\<\*\></span> take every function out of the list that is
its left parameter and apply it to every value in the list that is on
the right, resulting in every possible combination of applying a
function from the left list to a value in the right list.

``` {.haskell:hs name="code"}
ghci> [(+1),(*100),(*5)] <*> [1,2,3]
[2,3,4,100,200,300,5,10,15]
```

The second way is to take the first function on the left side of <span
class="fixed">\<\*\></span> and apply it to the first value on the
right, then take the second function from the list on the left side and
apply it to the second value on the right, and so on. Ultimately, it's
kind of like zipping the two lists together. But lists are already an
instance of <span class="fixed">Applicative</span>, so how did we also
make lists an instance of <span class="fixed">Applicative</span> in this
second way? If you remember, we said that the <span
class="fixed">ZipList a</span> type was introduced for this reason,
which has one value constructor, <span class="fixed">ZipList</span>,
that has just one field. We put the list that we're wrapping in that
field. Then, <span class="fixed">ZipList</span> was made an instance of
<span class="fixed">Applicative</span>, so that when we want to use
lists as applicatives in the zipping manner, we just wrap them with the
<span class="fixed">ZipList</span> constructor and then once we're done,
unwrap them with <span class="fixed">getZipList</span>:

``` {.haskell:hs name="code"}
ghci> getZipList $ ZipList [(+1),(*100),(*5)] <*> ZipList [1,2,3]
[2,200,15]
```

So, what does this have to do with this *newtype* keyword? Well, think
about how we might write the data declaration for our <span
class="fixed">ZipList a</span> type. One way would be to do it like so:

``` {.haskell:hs name="code"}
data ZipList a = ZipList [a]
```

A type that has just one value constructor and that value constructor
has just one field that is a list of things. We might also want to use
record syntax so that we automatically get a function that extracts a
list from a <span class="fixed">ZipList</span>:

``` {.haskell:hs name="code"}
data ZipList a = ZipList { getZipList :: [a] }
```

This looks fine and would actually work pretty well. We had two ways of
making an existing type an instance of a type class, so we used the
*data* keyword to just wrap that type into another type and made the
other type an instance in the second way.

The *newtype* keyword in Haskell is made exactly for these cases when we
want to just take one type and wrap it in something to present it as
another type. In the actual libraries, <span class="fixed">ZipList
a</span> is defined like this:

``` {.haskell:hs name="code"}
newtype ZipList a = ZipList { getZipList :: [a] }
```

Instead of the *data* keyword, the *newtype* keyword is used. Now why is
that? Well for one, *newtype* is faster. If you use the *data* keyword
to wrap a type, there's some overhead to all that wrapping and
unwrapping when your program is running. But if you use *newtype*,
Haskell knows that you're just using it to wrap an existing type into a
new type (hence the name), because you want it to be the same internally
but have a different type. With that in mind, Haskell can get rid of the
wrapping and unwrapping once it resolves which value is of what type.

So why not just use *newtype* all the time instead of *data* then? Well,
when you make a new type from an existing type by using the *newtype*
keyword, you can only have one value constructor and that value
constructor can only have one field. But with *data*, you can make data
types that have several value constructors and each constructor can have
zero or more fields:

``` {.haskell:hs name="code"}
data Profession = Fighter | Archer | Accountant

data Race = Human | Elf | Orc | Goblin

data PlayerCharacter = PlayerCharacter Race Profession
```

When using *newtype*, you're restricted to just one constructor with one
field.

We can also use the *deriving* keyword with *newtype* just like we would
with *data*. We can derive instances for <span class="fixed">Eq</span>,
<span class="fixed">Ord</span>, <span class="fixed">Enum</span>, <span
class="fixed">Bounded</span>, <span class="fixed">Show</span> and <span
class="fixed">Read</span>. If we derive the instance for a type class,
the type that we're wrapping has to be in that type class to begin with.
It makes sense, because *newtype* just wraps an existing type. So now if
we do the following, we can print and equate values of our new type:

``` {.haskell:hs name="code"}
newtype CharList = CharList { getCharList :: [Char] } deriving (Eq, Show)
```

Let's give that a go:

``` {.haskell:hs name="code"}
ghci> CharList "this will be shown!"
CharList {getCharList = "this will be shown!"}
ghci> CharList "benny" == CharList "benny"
True
ghci> CharList "benny" == CharList "oisters"
False
```

In this particular *newtype*, the value constructor has the following
type:

``` {.haskell:hs name="code"}
CharList :: [Char] -> CharList
```

It takes a <span class="fixed">[Char]</span> value, such as <span
class="fixed">"my sharona"</span> and returns a <span
class="fixed">CharList</span> value. From the above examples where we
used the <span class="fixed">CharList</span> value constructor, we see
that really is the case. Conversely, the <span
class="fixed">getCharList</span> function, which was generated for us
because we used record syntax in our *newtype*, has this type:

``` {.haskell:hs name="code"}
getCharList :: CharList -> [Char]
```

It takes a <span class="fixed">CharList</span> value and converts it to
a <span class="fixed">[Char]</span> value. You can think of this as
wrapping and unwrapping, but you can also think of it as converting
values from one type to the other.

### Using newtype to make type class instances

Many times, we want to make our types instances of certain type classes,
but the type parameters just don't match up for what we want to do. It's
easy to make <span class="fixed">Maybe</span> an instance of <span
class="fixed">Functor</span>, because the <span
class="fixed">Functor</span> type class is defined like this:

``` {.haskell:hs name="code"}
class Functor f where
    fmap :: (a -> b) -> f a -> f b
```

So we just start out with:

``` {.haskell:hs name="code"}
instance Functor Maybe where
```

And then implement <span class="fixed">fmap</span>. All the type
parameters add up because the <span class="fixed">Maybe</span> takes the
place of <span class="fixed">f</span> in the definition of the <span
class="fixed">Functor</span> type class and so if we look at <span
class="fixed">fmap</span> like it only worked on <span
class="fixed">Maybe</span>, it ends up behaving like:

``` {.haskell:hs name="code"}
fmap :: (a -> b) -> Maybe a -> Maybe b
```

![wow, very evil](http://s3.amazonaws.com/lyah/krakatoa.png)
Isn't that just peachy? Now what if we wanted to make the tuple an
instance of <span class="fixed">Functor</span> in such a way that when
we <span class="fixed">fmap</span> a function over a tuple, it gets
applied to the first component of the tuple? That way, doing <span
class="fixed">fmap (+3) (1,1)</span> would result in <span
class="fixed">(4,1)</span>. It turns out that writing the instance for
that is kind of hard. With <span class="fixed">Maybe</span>, we just say
<span class="fixed">instance Functor Maybe where</span> because only
type constructors that take exactly one parameter can be made an
instance of <span class="fixed">Functor</span>. But it seems like
there's no way to do something like that with <span
class="fixed">(a,b)</span> so that the type parameter <span
class="fixed">a</span> ends up being the one that changes when we use
<span class="fixed">fmap</span>. To get around this, we can *newtype*
our tuple in such a way that the second type parameter represents the
type of the first component in the tuple:

``` {.haskell:hs name="code"}
newtype Pair b a = Pair { getPair :: (a,b) }
```

And now, we can make it an instance of <span
class="fixed">Functor</span> so that the function is mapped over the
first component:

``` {.haskell:hs name="code"}
instance Functor (Pair c) where
    fmap f (Pair (x,y)) = Pair (f x, y)
```

As you can see, we can pattern match on types defined with *newtype*. We
pattern match to get the underlying tuple, then we apply the function
<span class="fixed">f</span> to the first component in the tuple and
then we use the <span class="fixed">Pair</span> value constructor to
convert the tuple back to our <span class="fixed">Pair b a</span>. If we
imagine what the type <span class="fixed">fmap</span> would be if it
only worked on our new pairs, it would be:

``` {.haskell:hs name="code"}
fmap :: (a -> b) -> Pair c a -> Pair c b
```

Again, we said <span class="fixed">instance Functor (Pair c)
where</span> and so <span class="fixed">Pair c</span> took the place of
the <span class="fixed">f</span> in the type class definition for <span
class="fixed">Functor</span>:

``` {.haskell:hs name="code"}
class Functor f where
    fmap :: (a -> b) -> f a -> f b
```

So now, if we convert a tuple into a <span class="fixed">Pair b
a</span>, we can use <span class="fixed">fmap</span> over it and the
function will be mapped over the first component:

``` {.haskell:hs name="code"}
ghci> getPair $ fmap (*100) (Pair (2,3))
(200,3)
ghci> getPair $ fmap reverse (Pair ("london calling", 3))
("gnillac nodnol",3)
```

### On newtype laziness

We mentioned that *newtype* is usually faster than *data*. The only
thing that can be done with *newtype* is turning an existing type into a
new type, so internally, Haskell can represent the values of types
defined with *newtype* just like the original ones, only it has to keep
in mind that the their types are now distinct. This fact means that not
only is *newtype* faster, it's also lazier. Let's take a look at what
this means.

Like we've said before, Haskell is lazy by default, which means that
only when we try to actually print the results of our functions will any
computation take place. Furthemore, only those computations that are
necessary for our function to tell us the result will get carried out.
The <span class="fixed">undefined</span> value in Haskell represents an
erronous computation. If we try to evaluate it (that is, force Haskell
to actually compute it) by printing it to the terminal, Haskell will
throw a hissy fit (technically referred to as an exception):

``` {.haskell:hs name="code"}
ghci> undefined
*** Exception: Prelude.undefined
```

However, if we make a list that has some <span
class="fixed">undefined</span> values in it but request only the head of
the list, which is not <span class="fixed">undefined</span>, everything
will go smoothly because Haskell doesn't really need to evaluate any
other elements in a list if we only want to see what the first element
is:

``` {.haskell:hs name="code"}
ghci> head [3,4,5,undefined,2,undefined]
3
```

Now consider the following type:

``` {.haskell:hs name="code"}
data CoolBool = CoolBool { getCoolBool :: Bool }
```

It's your run-of-the-mill algebraic data type that was defined with the
*data* keyword. It has one value constructor, which has one field whose
type is <span class="fixed">Bool</span>. Let's make a function that
pattern matches on a <span class="fixed">CoolBool</span> and returns the
value <span class="fixed">"hello"</span> regardless of whether the <span
class="fixed">Bool</span> inside the <span class="fixed">CoolBool</span>
was <span class="fixed">True</span> or <span class="fixed">False</span>:

``` {.haskell:hs name="code"}
helloMe :: CoolBool -> String
helloMe (CoolBool _) = "hello"
```

Instead of applying this function to a normal <span
class="fixed">CoolBool</span>, let's throw it a curveball and apply it
to <span class="fixed">undefined</span>!

``` {.haskell:hs name="code"}
ghci> helloMe undefined
"*** Exception: Prelude.undefined
```

Yikes! An exception! Now why did this exception happen? Types defined
with the *data* keyword can have multiple value constructors (even
though <span class="fixed">CoolBool</span> only has one). So in order to
see if the value given to our function conforms to the <span
class="fixed">(CoolBool \_)</span> pattern, Haskell has to evaluate the
value just enough to see which value constructor was used when we made
the value. And when we try to evaluate an <span
class="fixed">undefined</span> value, even a little, an exception is
thrown.

Instead of using the *data* keyword for <span
class="fixed">CoolBool</span>, let's try using *newtype*:

``` {.haskell:hs name="code"}
newtype CoolBool = CoolBool { getCoolBool :: Bool }
```

We don't have to change our <span class="fixed">helloMe</span> function,
because the pattern matching syntax is the same if you use *newtype* or
*data* to define your type. Let's do the same thing here and apply <span
class="fixed">helloMe</span> to an <span class="fixed">undefined</span>
value:

``` {.haskell:hs name="code"}
ghci> helloMe undefined
"hello"
```

![top of the mornin to ya!!!](http://s3.amazonaws.com/lyah/shamrock.png)
It worked! Hmmm, why is that? Well, like we've said, when we use
*newtype*, Haskell can internally represent the values of the new type
in the same way as the original values. It doesn't have to add another
box around them, it just has to be aware of the values being of
different types. And because Haskell knows that types made with the
*newtype* keyword can only have one constructor, it doesn't have to
evaluate the value passed to the function to make sure that it conforms
to the <span class="fixed">(CoolBool \_)</span> pattern because
*newtype* types can only have one possible value constructor and one
field!

This difference in behavior may seem trivial, but it's actually pretty
important because it helps us realize that even though types defined
with *data* and *newtype* behave similarly from the programmer's point
of view because they both have value constructors and fields, they are
actually two different mechanisms. Whereas *data* can be used to make
your own types from scratch, *newtype* is for making a completely new
type out of an existing type. Pattern matching on *newtype* values isn't
like taking something out of a box (like it is with *data*), it's more
about making a direct conversion from one type to another.

### <span class="fixed">type</span> vs. <span class="fixed">newtype</span> vs. <span class="fixed">data</span>

At this point, you may be a bit confused about what exactly the
difference between *type*, *data* and *newtype* is, so let's refresh our
memory a bit.

The *type* keyword is for making type synonyms. What that means is that
we just give another name to an already existing type so that the type
is easier to refer to. Say we did the following:

``` {.haskell:hs name="code"}
type IntList = [Int]
```

All this does is to allow us to refer to the <span
class="fixed">[Int]</span> type as <span class="fixed">IntList</span>.
They can be used interchangeably. We don't get an <span
class="fixed">IntList</span> value constructor or anything like that.
Because <span class="fixed">[Int]</span> and <span
class="fixed">IntList</span> are only two ways to refer to the same
type, it doesn't matter which name we use in our type annotations:

``` {.haskell:hs name="code"}
ghci> ([1,2,3] :: IntList) ++ ([1,2,3] :: [Int])
[1,2,3,1,2,3]
```

We use type synonyms when we want to make our type signatures more
descriptive by giving types names that tell us something about their
purpose in the context of the functions where they're being used. For
instance, when we used an association list of type <span
class="fixed">[(String,String)]</span> to represent a phone book, we
gave it the type synonym of <span class="fixed">PhoneBook</span> so that
the type signatures of our functions were easier to read.

The *newtype* keyword is for taking existing types and wrapping them in
new types, mostly so that it's easier to make them instances of certain
type classes. When we use *newtype* to wrap an existing type, the type
that we get is separate from the original type. If we make the following
*newtype*:

``` {.haskell:hs name="code"}
newtype CharList = CharList { getCharList :: [Char] }
```

We can't use <span class="fixed">++</span> to put together a <span
class="fixed">CharList</span> and a list of type <span
class="fixed">[Char]</span>. We can't even use <span
class="fixed">++</span> to put together two <span
class="fixed">CharList</span>s, because <span class="fixed">++</span>
works only on lists and the <span class="fixed">CharList</span> type
isn't a list, even though it could be said that it contains one. We can,
however, convert two <span class="fixed">CharList</span>s to lists,
<span class="fixed">++</span> them and then convert that back to a <span
class="fixed">CharList</span>.

When we use record syntax in our *newtype* declarations, we get
functions for converting between the new type and the original type:
namely the value constructor of our *newtype* and the function for
extracting the value in its field. The new type also isn't automatically
made an instance of the type classes that the original type belongs to,
so we have to derive or manually write them.

In practice, you can think of *newtype* declarations as *data*
declarations that can only have one constructor and one field. If you
catch yourself writing such a *data* declaration, consider using
*newtype*.

The *data* keyword is for making your own data types and with them, you
can go hog wild. They can have as many constructors and fields as you
wish and can be used to implement any algebraic data type by yourself.
Everything from lists and <span class="fixed">Maybe</span>-like types to
trees.

If you just want your type signatures to look cleaner and be more
descriptive, you probably want type synonyms. If you want to take an
existing type and wrap it in a new type in order to make it an instance
of a type class, chances are you're looking for a *newtype*. And if you
want to make something completely new, odds are good that you're looking
for the *data* keyword.

Monoids
-------

![wow this is pretty much the gayest pirate ship
ever](http://s3.amazonaws.com/lyah/pirateship.png)
Type classes in Haskell are used to present an interface for types that
have some behavior in common. We started out with simple type classes
like <span class="fixed">Eq</span>, which is for types whose values can
be equated, and <span class="fixed">Ord</span>, which is for things that
can be put in an order and then moved on to more interesting ones, like
<span class="fixed">Functor</span> and <span
class="fixed">Applicative</span>.

When we make a type, we think about which behaviors it supports, i.e.
what it can act like and then based on that we decide which type classes
to make it an instance of. If it makes sense for values of our type to
be equated, we make it an instance of the <span class="fixed">Eq</span>
type class. If we see that our type is some kind of functor, we make it
an instance of <span class="fixed">Functor</span>, and so on.

Now consider the following: <span class="fixed">\*</span> is a function
that takes two numbers and multiplies them. If we multiply some number
with a <span class="fixed">1</span>, the result is always equal to that
number. It doesn't matter if we do <span class="fixed">1 \* x</span> or
<span class="fixed">x \* 1</span>, the result is always <span
class="fixed">x</span>. Similarly, <span class="fixed">++</span> is also
a function which takes two things and returns a third. Only instead of
multiplying numbers, it takes two lists and concatenates them. And much
like <span class="fixed">\*</span>, it also has a certain value which
doesn't change the other one when used with <span
class="fixed">++</span>. That value is the empty list: <span
class="fixed">[]</span>.

``` {.haskell:hs name="code"}
ghci> 4 * 1
4
ghci> 1 * 9
9
ghci> [1,2,3] ++ []
[1,2,3]
ghci> [] ++ [0.5, 2.5]
[0.5,2.5]
```

It seems that both <span class="fixed">\*</span> together with <span
class="fixed">1</span> and <span class="fixed">++</span> along with
<span class="fixed">[]</span> share some common properties:

-   The function takes two parameters.
-   The parameters and the returned value have the same type.
-   There exists such a value that doesn't change other values when used
    with the binary function.

There's another thing that these two operations have in common that may
not be as obvious as our previous observations: when we have three or
more values and we want to use the binary function to reduce them to a
single result, the order in which we apply the binary function to the
values doesn't matter. It doesn't matter if we do <span
class="fixed">(3 \* 4) \* 5</span> or <span class="fixed">3 \* (4 \*
5)</span>. Either way, the result is <span class="fixed">60</span>. The
same goes for <span class="fixed">++</span>:

``` {.haskell:hs name="code"}
ghci> (3 * 2) * (8 * 5)
240
ghci> 3 * (2 * (8 * 5))
240
ghci> "la" ++ ("di" ++ "da")
"ladida"
ghci> ("la" ++ "di") ++ "da"
"ladida"
```

We call this property *associativity*. <span class="fixed">\*</span> is
associative, and so is <span class="fixed">++</span>, but <span
class="fixed">-</span>, for example, is not. The expressions <span
class="fixed">(5 - 3) - 4</span> and <span class="fixed">5 - (3 -
4)</span> result in different numbers.

By noticing and writing down these properties, we have chanced upon
*monoids*! A monoid is when you have an associative binary function and
a value which acts as an identity with respect to that function. When
something acts as an identity with respect to a function, it means that
when called with that function and some other value, the result is
always equal to that other value. <span class="fixed">1</span> is the
identity with respect to <span class="fixed">\*</span> and <span
class="fixed">[]</span> is the identity with respect to <span
class="fixed">++</span>. There are a lot of other monoids to be found in
the world of Haskell, which is why the <span class="fixed">Monoid</span>
type class exists. It's for types which can act like monoids. Let's see
how the type class is defined:

``` {.haskell:hs name="code"}
class Monoid m where
    mempty :: m
    mappend :: m -> m -> m
    mconcat :: [m] -> m
    mconcat = foldr mappend mempty
```

![woof dee do!!!](http://s3.amazonaws.com/lyah/balloondog.png)
The <span class="fixed">Monoid</span> type class is defined in <span
class="fixed">import Data.Monoid</span>. Let's take some time and get
properly acquainted with it.

First of all, we see that only concrete types can be made instances of
<span class="fixed">Monoid</span>, because the <span
class="fixed">m</span> in the type class definition doesn't take any
type parameters. This is different from <span
class="fixed">Functor</span> and <span class="fixed">Applicative</span>,
which require their instances to be type constructors which take one
parameter.

The first function is <span class="fixed">mempty</span>. It's not really
a function, since it doesn't take parameters, so it's a polymorphic
constant, kind of like <span class="fixed">minBound</span> from <span
class="fixed">Bounded</span>. <span class="fixed">mempty</span>
represents the identity value for a particular monoid.

Next up, we have <span class="fixed">mappend</span>, which, as you've
probably guessed, is the binary function. It takes two values of the
same type and returns a value of that type as well. It's worth noting
that the decision to name <span class="fixed">mappend</span> as it's
named was kind of unfortunate, because it implies that we're appending
two things in some way. While <span class="fixed">++</span> does take
two lists and append one to the other, <span class="fixed">\*</span>
doesn't really do any appending, it just multiplies two numbers
together. When we meet other instances of <span
class="fixed">Monoid</span>, we'll see that most of them don't append
values either, so avoid thinking in terms of appending and just think in
terms of <span class="fixed">mappend</span> being a binary function that
takes two monoid values and returns a third.

The last function in this type class definition is <span
class="fixed">mconcat</span>. It takes a list of monoid values and
reduces them to a single value by doing <span
class="fixed">mappend</span> between the list's elements. It has a
default implementation, which just takes <span
class="fixed">mempty</span> as a starting value and folds the list from
the right with <span class="fixed">mappend</span>. Because the default
implementation is fine for most instances, we won't concern ourselves
with <span class="fixed">mconcat</span> too much from now on. When
making a type an instance of <span class="fixed">Monoid</span>, it
suffices to just implement <span class="fixed">mempty</span> and <span
class="fixed">mappend</span>. The reason <span
class="fixed">mconcat</span> is there at all is because for some
instances, there might be a more efficient way to implement <span
class="fixed">mconcat</span>, but for most instances the default
implementation is just fine.

Before moving on to specific instances of <span
class="fixed">Monoid</span>, let's take a brief look at the monoid laws.
We mentioned that there has to be a value that acts as the identity with
respect to the binary function and that the binary function has to be
associative. It's possible to make instances of <span
class="fixed">Monoid</span> that don't follow these rules, but such
instances are of no use to anyone because when using the <span
class="fixed">Monoid</span> type class, we rely on its instances acting
like monoids. Otherwise, what's the point? That's why when making
instances, we have to make sure they follow these laws:

-   <span class="label law">mempty \`mappend\` x = x</span>
-   <span class="label law">x \`mappend\` mempty = x</span>
-   <span class="label law">(x \`mappend\` y) \`mappend\` z = x
    \`mappend\` (y \`mappend\` z)</span>

The first two state that <span class="fixed">mempty</span> has to act as
the identity with respect to <span class="fixed">mappend</span> and the
third says that <span class="fixed">mappend</span> has to be associative
i.e. that it the order in which we use <span
class="fixed">mappend</span> to reduce several monoid values into one
doesn't matter. Haskell doesn't enforce these laws, so we as the
programmer have to be careful that our instances do indeed obey them.

### Lists are monoids

Yes, lists are monoids! Like we've seen, the <span
class="fixed">++</span> function and the empty list <span
class="fixed">[]</span> form a monoid. The instance is very simple:

``` {.haskell:hs name="code"}
instance Monoid [a] where
    mempty = []
    mappend = (++)
```

Lists are an instance of the <span class="fixed">Monoid</span> type
class regardless of the type of the elements they hold. Notice that we
wrote <span class="fixed">instance Monoid [a]</span> and not <span
class="fixed">instance Monoid []</span>, because <span
class="fixed">Monoid</span> requires a concrete type for an instance.

Giving this a test run, we encounter no surprises:

``` {.haskell:hs name="code"}
ghci> [1,2,3] `mappend` [4,5,6]
[1,2,3,4,5,6]
ghci> ("one" `mappend` "two") `mappend` "tree"
"onetwotree"
ghci> "one" `mappend` ("two" `mappend` "tree")
"onetwotree"
ghci> "one" `mappend` "two" `mappend` "tree"
"onetwotree"
ghci> "pang" `mappend` mempty
"pang"
ghci> mconcat [[1,2],[3,6],[9]]
[1,2,3,6,9]
ghci> mempty :: [a]
[]
```

![smug as hell](http://s3.amazonaws.com/lyah/smug.png)
Notice that in the last line, we had to write an explicit type
annotation, because if we just did <span class="fixed">mempty</span>,
GHCi wouldn't know which instance to use, so we had to say we want the
list instance. We were able to use the general type of <span
class="fixed">[a]</span> (as opposed to specifying <span
class="fixed">[Int]</span> or <span class="fixed">[String]</span>)
because the empty list can act as if it contains any type.

Because <span class="fixed">mconcat</span> has a default implementation,
we get it for free when we make something an instance of <span
class="fixed">Monoid</span>. In the case of the list, <span
class="fixed">mconcat</span> turns out to be just <span
class="fixed">concat</span>. It takes a list of lists and flattens it,
because that's the equivalent of doing <span class="fixed">++</span>
between all the adjecent lists in a list.

The monoid laws do indeed hold for the list instance. When we have
several lists and we <span class="fixed">mappend</span> (or <span
class="fixed">++</span>) them together, it doesn't matter which ones we
do first, because they're just joined at the ends anyway. Also, the
empty list acts as the identity so all is well. Notice that monoids
don't require that <span class="fixed">a \`mappend\` b</span> be equal
to <span class="fixed">b \`mappend\` a</span>. In the case of the list,
they clearly aren't:

``` {.haskell:hs name="code"}
ghci> "one" `mappend` "two"
"onetwo"
ghci> "two" `mappend` "one"
"twoone"
```

And that's okay. The fact that for multiplication <span
class="fixed">3 \* 5</span> and <span class="fixed">5 \* 3</span> are
the same is just a property of multiplication, but it doesn't hold for
all (and indeed, most) monoids.

### <span class="fixed">Product</span> and <span class="fixed">Sum</span>

We already examined one way for numbers to be considered monoids. Just
have the binary function be <span class="fixed">\*</span> and the
identity value <span class="fixed">1</span>. It turns out that that's
not the only way for numbers to be monoids. Another way is to have the
binary function be <span class="fixed">+</span> and the identity value
<span class="fixed">0</span>:

``` {.haskell:hs name="code"}
ghci> 0 + 4
4
ghci> 5 + 0
5
ghci> (1 + 3) + 5
9
ghci> 1 + (3 + 5)
9
```

The monoid laws hold, because if you add 0 to any number, the result is
that number. And addition is also associative, so we get no problems
there. So now that there are two equally valid ways for numbers to be
monoids, which way do choose? Well, we don't have to. Remember, when
there are several ways for some type to be an instance of the same type
class, we can wrap that type in a *newtype* and then make the new type
an instance of the type class in a different way. We can have our cake
and eat it too.

The <span class="fixed">Data.Monoid</span> module exports two types for
this, namely <span class="fixed">Product</span> and <span
class="fixed">Sum</span>. <span class="fixed">Product</span> is defined
like this:

``` {.haskell:hs name="code"}
newtype Product a =  Product { getProduct :: a }
    deriving (Eq, Ord, Read, Show, Bounded)
```

Simple, just a *newtype* wrapper with one type parameter along with some
derived instances. Its instance for <span class="fixed">Monoid</span>
goes a little something like this:

``` {.haskell:hs name="code"}
instance Num a => Monoid (Product a) where
    mempty = Product 1
    Product x `mappend` Product y = Product (x * y)
```

<span class="fixed">mempty</span> is just <span class="fixed">1</span>
wrapped in a <span class="fixed">Product</span> constructor. <span
class="fixed">mappend</span> pattern matches on the <span
class="fixed">Product</span> constructor, multiplies the two numbers and
then wraps the resulting number back. As you can see, there's a <span
class="fixed">Num a</span> class constraint. So this means that <span
class="fixed">Product a</span> is an instance of <span
class="fixed">Monoid</span> for all <span class="fixed">a</span>'s that
are already an instance of <span class="fixed">Num</span>. To use <span
class="fixed">Producta a</span> as a monoid, we have to do some
*newtype* wrapping and unwrapping:

``` {.haskell:hs name="code"}
ghci> getProduct $ Product 3 `mappend` Product 9
27
ghci> getProduct $ Product 3 `mappend` mempty
3
ghci> getProduct $ Product 3 `mappend` Product 4 `mappend` Product 2
24
ghci> getProduct . mconcat . map Product $ [3,4,2]
24
```

This is nice as a showcase of the <span class="fixed">Monoid</span> type
class, but no one in their right mind would use this way of multiplying
numbers instead of just writing <span class="fixed">3 \* 9</span> and
<span class="fixed">3 \* 1</span>. But a bit later, we'll see how these
<span class="fixed">Monoid</span> instances that may seem trivial at
this time can come in handy.

<span class="fixed">Sum</span> is defined like <span
class="fixed">Product</span> and the instance is similar as well. We use
it in the same way:

``` {.haskell:hs name="code"}
ghci> getSum $ Sum 2 `mappend` Sum 9
11
ghci> getSum $ mempty `mappend` Sum 3
3
ghci> getSum . mconcat . map Sum $ [1,2,3]
6
```

### <span class="fixed">Any</span> and <span class="fixed">All</span>

Another type which can act like a monoid in two distinct but equally
valid ways is <span class="fixed">Bool</span>. The first way is to have
the *or* function <span class="fixed">||</span> act as the binary
function along with <span class="fixed">False</span> as the identity
value. The way *or* works in logic is that if any of its two parameters
is <span class="fixed">True</span>, it returns <span
class="fixed">True</span>, otherwise it returns <span
class="fixed">False</span>. So if we use <span
class="fixed">False</span> as the identity value, it will return <span
class="fixed">False</span> when *or*-ed with <span
class="fixed">False</span> and <span class="fixed">True</span> when
*or*-ed with <span class="fixed">True</span>. The <span
class="fixed">Any</span> *newtype* constructor is an instance of <span
class="fixed">Monoid</span> in this fashion. It's defined like this:

``` {.haskell:hs name="code"}
newtype Any = Any { getAny :: Bool }
    deriving (Eq, Ord, Read, Show, Bounded)
```

Its instance looks goes like so:

``` {.haskell:hs name="code"}
instance Monoid Any where
        mempty = Any False
        Any x `mappend` Any y = Any (x || y)
```

The reason it's called <span class="fixed">Any</span> is because <span
class="fixed">x \`mappend\` y</span> will be <span
class="fixed">True</span> if *any* one of those two is <span
class="fixed">True</span>. Even if three or more <span
class="fixed">Any</span> wrapped <span class="fixed">Bool</span>s are
<span class="fixed">mappend</span>ed together, the result will hold
<span class="fixed">True</span> if any of them are <span
class="fixed">True</span>:

``` {.haskell:hs name="code"}
ghci> getAny $ Any True `mappend` Any False
True
ghci> getAny $ mempty `mappend` Any True
True
ghci> getAny . mconcat . map Any $ [False, False, False, True]
True
ghci> getAny $ mempty `mappend` mempty
False
```

The other way for <span class="fixed">Bool</span> to be an instance of
<span class="fixed">Monoid</span> is to kind of do the opposite: have
<span class="fixed">&&</span> be the binary function and then make <span
class="fixed">True</span> the identity value. Logical *and* will return
<span class="fixed">True</span> only if both of its parameters are <span
class="fixed">True</span>. This is the *newtype* declaration, nothing
fancy:

``` {.haskell:hs name="code"}
newtype All = All { getAll :: Bool }
        deriving (Eq, Ord, Read, Show, Bounded)
```

And this is the instance:

``` {.haskell:hs name="code"}
instance Monoid All where
        mempty = All True
        All x `mappend` All y = All (x && y)
```

When we <span class="fixed">mappend</span> values of the <span
class="fixed">All</span> type, the result will be <span
class="fixed">True</span> only if *all* the values used in the <span
class="fixed">mappend</span> operations are <span
class="fixed">True</span>:

``` {.haskell:hs name="code"}
ghci> getAll $ mempty `mappend` All True
True
ghci> getAll $ mempty `mappend` All False
False
ghci> getAll . mconcat . map All $ [True, True, True]
True
ghci> getAll . mconcat . map All $ [True, True, False]
False
```

Just like with multiplication and addition, we usually explicitly state
the binary functions instead of wrapping them in *newtype*s and then
using <span class="fixed">mappend</span> and <span
class="fixed">mempty</span>. <span class="fixed">mconcat</span> seems
useful for <span class="fixed">Any</span> and <span
class="fixed">All</span>, but usually it's easier to use the <span
class="fixed">or</span> and <span class="fixed">and</span> functions,
which take lists of <span class="fixed">Bool</span>s and return <span
class="fixed">True</span> if any of them are <span
class="fixed">True</span> or if all of them are <span
class="fixed">True</span>, respectively.

### The <span class="fixed">Ordering</span> monoid

Hey, remember the <span class="fixed">Ordering</span> type? It's used as
the result when comparing things and it can have three values: <span
class="fixed">LT</span>, <span class="fixed">EQ</span> and <span
class="fixed">GT</span>, which stand for *less than*, *equal* and
*greater than* respectively:

``` {.haskell:hs name="code"}
ghci> 1 `compare` 2
LT
ghci> 2 `compare` 2
EQ
ghci> 3 `compare` 2
GT
```

With lists, numbers and boolean values, finding monoids was just a
matter of looking at already existing commonly used functions and seeing
if they exhibit some sort of monoid behavior. With <span
class="fixed">Ordering</span>, we have to look a bit harder to recognize
a monoid, but it turns out that its <span class="fixed">Monoid</span>
instance is just as intuitive as the ones we've met so far and also
quite useful:

``` {.haskell:hs name="code"}
instance Monoid Ordering where
    mempty = EQ
    LT `mappend` _ = LT
    EQ `mappend` y = y
    GT `mappend` _ = GT
```

![did anyone ORDER pizza?!?! I can't BEAR these
puns!](http://s3.amazonaws.com/lyah/bear.png)
The instance is set up like this: when we <span
class="fixed">mappend</span> two <span class="fixed">Ordering</span>
values, the one on the left is kept, unless the value on the left is
<span class="fixed">EQ</span>, in which case the right one is the
result. The identity is <span class="fixed">EQ</span>. At first, this
may seem kind of arbitrary, but it actually resembles the way we
alphabetically compare words. We compare the first two letters and if
they differ, we can already decide which word would go first in a
dictionary. However, if the first two letters are equal, then we move on
to comparing the next pair of letters and repeat the process.

For instance, if we were to alphabetically compare the words <span
class="fixed">"ox"</span> and <span class="fixed">"on"</span>, we'd
first compare the first two letters of each word, see that they are
equal and then move on to comparing the second letter of each word. We
see that <span class="fixed">'x'</span> is alphabetically greater than
<span class="fixed">'n'</span>, and so we know how the words compare. To
gain some intuition for <span class="fixed">EQ</span> being the
identity, we can notice that if we were to cram the same letter in the
same position in both words, it wouldn't change their alphabetical
ordering. <span class="fixed">"oix"</span> is still alphabetically
greater than and <span class="fixed">"oin"</span>.

It's important to note that in the <span class="fixed">Monoid</span>
instance for <span class="fixed">Ordering</span>, <span class="fixed">x
\`mappend\` y</span> doesn't equal <span class="fixed">y \`mappend\`
x</span>. Because the first parameter is kept unless it's <span
class="fixed">EQ</span>, <span class="fixed">LT \`mappend\` GT</span>
will result in <span class="fixed">LT</span>, whereas <span
class="fixed">GT \`mappend\` LT</span> will result in <span
class="fixed">GT</span>:

``` {.haskell:hs name="code"}
ghci> LT `mappend` GT
LT
ghci> GT `mappend` LT
GT
ghci> mempty `mappend` LT
LT
ghci> mempty `mappend` GT
GT
```

OK, so how is this monoid useful? Let's say you were writing a function
that takes two strings, compares their lengths, and returns an <span
class="fixed">Ordering</span>. But if the strings are of the same
length, then instead of returning <span class="fixed">EQ</span> right
away, we want to compare them alphabetically. One way to write this
would be like so:

``` {.haskell:hs name="code"}
lengthCompare :: String -> String -> Ordering
lengthCompare x y = let a = length x `compare` length y
                        b = x `compare` y
                    in  if a == EQ then b else a
```

We name the result of comparing the lengths <span class="fixed">a</span>
and the result of the alphabetical comparison <span
class="fixed">b</span> and then if it turns out that the lengths were
equal, we return their alphabetical ordering.

But by employing our understanding of how <span
class="fixed">Ordering</span> is a monoid, we can rewrite this function
in a much simpler manner:

``` {.haskell:hs name="code"}
import Data.Monoid

lengthCompare :: String -> String -> Ordering
lengthCompare x y = (length x `compare` length y) `mappend`
                    (x `compare` y)
```

We can try this out:

``` {.haskell:hs name="code"}
ghci> lengthCompare "zen" "ants"
LT
ghci> lengthCompare "zen" "ant"
GT
```

Remember, when we use <span class="fixed">mappend</span>, its left
parameter is always kept unless it's <span class="fixed">EQ</span>, in
which case the right one is kept. That's why we put the comparison that
we consider to be the first, more important criterion as the first
parameter. If we wanted to expand this function to also compare for the
number of vowels and set this to be the second most important criterion
for comparison, we'd just modify it like this:

``` {.haskell:hs name="code"}
import Data.Monoid

lengthCompare :: String -> String -> Ordering
lengthCompare x y = (length x `compare` length y) `mappend`
                    (vowels x `compare` vowels y) `mappend`
                    (x `compare` y)
    where vowels = length . filter (`elem` "aeiou")
```

We made a helper function, which takes a string and tells us how many
vowels it has by first filtering it only for letters that are in the
string <span class="fixed">"aeiou"</span> and then applying <span
class="fixed">length</span> to that.

``` {.haskell:hs name="code"}
ghci> lengthCompare "zen" "anna"
LT
ghci> lengthCompare "zen" "ana"
LT
ghci> lengthCompare "zen" "ann"
GT
```

Very cool. Here, we see how in the first example the lengths are found
to be different and so <span class="fixed">LT</span> is returned,
because the length of <span class="fixed">"zen"</span> is less than the
length of <span class="fixed">"anna"</span>. In the second example, the
lengths are the same, but the second string has more vowels, so <span
class="fixed">LT</span> is returned again. In the third example, they
both have the same length and the same number of vowels, so they're
compared alphabetically and <span class="fixed">"zen"</span> wins.

The <span class="fixed">Ordering</span> monoid is very cool because it
allows us to easily compare things by many different criteria and put
those criteria in an order themselves, ranging from the most important
to the least.

### <span class="fixed">Maybe</span> the monoid

Let's take a look at the various ways that <span class="fixed">Maybe
a</span> can be made an instance of <span class="fixed">Monoid</span>
and what those instances are useful for.

One way is to treat <span class="fixed">Maybe a</span> as a monoid only
if its type parameter <span class="fixed">a</span> is a monoid as well
and then implement <span class="fixed">mappend</span> in such a way that
it uses the <span class="fixed">mappend</span> operation of the values
that are wrapped with <span class="fixed">Just</span>. We use <span
class="fixed">Nothing</span> as the identity, and so if one of the two
values that we're <span class="fixed">mappend</span>ing is <span
class="fixed">Nothing</span>, we keep the other value. Here's the
instance declaration:

``` {.haskell:hs name="code"}
instance Monoid a => Monoid (Maybe a) where
    mempty = Nothing
    Nothing `mappend` m = m
    m `mappend` Nothing = m
    Just m1 `mappend` Just m2 = Just (m1 `mappend` m2)
```

Notice the class constraint. It says that <span class="fixed">Maybe
a</span> is an instance of <span class="fixed">Monoid</span> only if
<span class="fixed">a</span> is an instance of <span
class="fixed">Monoid</span>. If we <span class="fixed">mappend</span>
something with a <span class="fixed">Nothing</span>, the result is that
something. If we <span class="fixed">mappend</span> two <span
class="fixed">Just</span> values, the contents of the <span
class="fixed">Just</span>s get <span class="fixed">mappended</span> and
then wrapped back in a <span class="fixed">Just</span>. We can do this
because the class constraint ensures that the type of what's inside the
<span class="fixed">Just</span> is an instance of <span
class="fixed">Monoid</span>.

``` {.haskell:hs name="code"}
ghci> Nothing `mappend` Just "andy"
Just "andy"
ghci> Just LT `mappend` Nothing
Just LT
ghci> Just (Sum 3) `mappend` Just (Sum 4)
Just (Sum {getSum = 7})
```

This comes in use when you're dealing with monoids as results of
computations that may have failed. Because of this instance, we don't
have to check if the computations have failed by seeing if they're a
<span class="fixed">Nothing</span> or <span class="fixed">Just</span>
value; we can just continue to treat them as normal monoids.

But what if the type of the contents of the <span
class="fixed">Maybe</span> aren't an instance of <span
class="fixed">Monoid</span>? Notice that in the previous instance
declaration, the only case where we have to rely on the contents being
monoids is when both parameters of <span class="fixed">mappend</span>
are <span class="fixed">Just</span> values. But if we don't know if the
contents are monoids, we can't use <span class="fixed">mappend</span>
between them, so what are we to do? Well, one thing we can do is to just
discard the second value and keep the first one. For this, the <span
class="fixed">First a</span> type exists and this is its definition:

``` {.haskell:hs name="code"}
newtype First a = First { getFirst :: Maybe a }
    deriving (Eq, Ord, Read, Show)
```

We take a <span class="fixed">Maybe a</span> and we wrap it with a
*newtype*. The <span class="fixed">Monoid</span> instance is as follows:

``` {.haskell:hs name="code"}
instance Monoid (First a) where
    mempty = First Nothing
    First (Just x) `mappend` _ = First (Just x)
    First Nothing `mappend` x = x
```

Just like we said. <span class="fixed">mempty</span> is just a <span
class="fixed">Nothing</span> wrapped with the <span
class="fixed">First</span> *newtype* constructor. If <span
class="fixed">mappend</span>'s first parameter is a <span
class="fixed">Just</span> value, we ignore the second one. If the first
one is a <span class="fixed">Nothing</span>, then we present the second
parameter as a result, regardless of whether it's a <span
class="fixed">Just</span> or a <span class="fixed">Nothing</span>:

``` {.haskell:hs name="code"}
ghci> getFirst $ First (Just 'a') `mappend` First (Just 'b')
Just 'a'
ghci> getFirst $ First Nothing `mappend` First (Just 'b')
Just 'b'
ghci> getFirst $ First (Just 'a') `mappend` First Nothing
Just 'a'
```

<span class="fixed">First</span> is useful when we have a bunch of <span
class="fixed">Maybe</span> values and we just want to know if any of
them is a <span class="fixed">Just</span>. The <span
class="fixed">mconcat</span> function comes in handy:

``` {.haskell:hs name="code"}
ghci> getFirst . mconcat . map First $ [Nothing, Just 9, Just 10]
Just 9
```

If we want a monoid on <span class="fixed">Maybe a</span> such that the
second parameter is kept if both parameters of <span
class="fixed">mappend</span> are <span class="fixed">Just</span> values,
<span class="fixed">Data.Monoid</span> provides a the <span
class="fixed">Last a</span> type, which works like <span
class="fixed">First a</span>, only the last non-<span
class="fixed">Nothing</span> value is kept when <span
class="fixed">mappend</span>ing and using <span
class="fixed">mconcat</span>:

``` {.haskell:hs name="code"}
ghci> getLast . mconcat . map Last $ [Nothing, Just 9, Just 10]
Just 10
ghci> getLast $ Last (Just "one") `mappend` Last (Just "two")
Just "two"
```

### Using monoids to fold data structures

One of the more interesting ways to put monoids to work is to make them
help us define folds over various data structures. So far, we've only
done folds over lists, but lists aren't the only data structure that can
be folded over. We can define folds over almost any data structure.
Trees especially lend themselves well to folding.

Because there are so many data structures that work nicely with folds,
the <span class="label class">Foldable</span> type class was introduced.
Much like <span class="fixed">Functor</span> is for things that can be
mapped over, <span class="fixed">Foldable</span> is for things that can
be folded up! It can be found in <span
class="fixed">Data.Foldable</span> and because it export functions whose
names clash with the ones from the <span class="fixed">Prelude</span>,
it's best imported qualified (and served with basil):

``` {.haskell:hs name="code"}
import qualified Foldable as F
```

To save ourselves precious keystrokes, we've chosen to import it
qualified as <span class="fixed">F</span>. Alright, so what are some of
the functions that this type class defines? Well, among them are <span
class="fixed">foldr</span>, <span class="fixed">foldl</span>, <span
class="fixed">foldr1</span> and <span class="fixed">foldl1</span>. Huh?
But we already know these functions, what's so new about this? Let's
compare the types of <span class="fixed">Foldable</span>'s <span
class="fixed">foldr</span> and the <span class="fixed">foldr</span> from
the <span class="fixed">Prelude</span> to see how they differ:

``` {.haskell:hs name="code"}
ghci> :t foldr
foldr :: (a -> b -> b) -> b -> [a] -> b
ghci> :t F.foldr
F.foldr :: (F.Foldable t) => (a -> b -> b) -> b -> t a -> b
```

Ah! So whereas <span class="fixed">foldr</span> takes a list and folds
it up, the <span class="fixed">foldr</span> from <span
class="fixed">Data.Foldable</span> accepts any type that can be folded
up, not just lists! As expected, both <span class="fixed">foldr</span>
functions do the same for lists:

``` {.haskell:hs name="code"}
ghci> foldr (*) 1 [1,2,3]
6
ghci> F.foldr (*) 1 [1,2,3]
6
```

Okay then, what are some other data structures that support folds? Well,
there's the <span class="fixed">Maybe</span> we all know and love!

``` {.haskell:hs name="code"}
ghci> F.foldl (+) 2 (Just 9)
11
ghci> F.foldr (||) False (Just True)
True
```

But folding over a <span class="fixed">Maybe</span> value isn't terribly
interesting, because when it comes to folding, it just acts like a list
with one element if it's a <span class="fixed">Just</span> value and as
an empty list if it's <span class="fixed">Nothing</span>. So let's
examine a data structure that's a little more complex then.

Remember the tree data structure from the [Making Our Own Types and
Typeclasses](making-our-own-types-and-typeclasses#recursive-data-structures)
chapter? We defined it like this:

``` {.haskell:hs name="code"}
data Tree a = Empty | Node a (Tree a) (Tree a) deriving (Show, Read, Eq)
```

We said that a tree is either an empty tree that doesn't hold any values
or it's a node that holds one value and also two other trees. After
defining it, we made it an instance of <span
class="fixed">Functor</span> and with that we gained the ability to
<span class="fixed">fmap</span> functions over it. Now, we're going to
make it an instance of <span class="fixed">Foldable</span> so that we
get the abilty to fold it up. One way to make a type constructor an
instance of <span class="fixed">Foldable</span> is to just directly
implement <span class="fixed">foldr</span> for it. But another, often
much easier way, is to implement the <span class="fixed">foldMap</span>
function, which is also a part of the <span
class="fixed">Foldable</span> type class. The <span
class="fixed">foldMap</span> function has the following type:

``` {.haskell:hs name="code"}
foldMap :: (Monoid m, Foldable t) => (a -> m) -> t a -> m
```

Its first parameter is a function that takes a value of the type that
our foldable structure contains (denoted here with <span
class="fixed">a</span>) and returns a monoid value. Its second parameter
is a foldable structure that contains values of type <span
class="fixed">a</span>. It maps that function over the foldable
structure, thus producing a foldable structure that contains monoid
values. Then, by doing <span class="fixed">mappend</span> between those
monoid values, it joins them all into a single monoid value. This
function may sound kind of odd at the moment, but we'll see that it's
very easy to implement. What's also cool is that implementing this
function is all it takes for our type to be made an instance of <span
class="fixed">Foldable</span>. So if we just implement <span
class="fixed">foldMap</span> for some type, we get <span
class="fixed">foldr</span> and <span class="fixed">foldl</span> on that
type for free!

This is how we make <span class="fixed">Tree</span> an instance of <span
class="fixed">Foldable</span>:

``` {.haskell:hs name="code"}
instance F.Foldable Tree where
    foldMap f Empty = mempty
    foldMap f (Node x l r) = F.foldMap f l `mappend`
                             f x           `mappend`
                             F.foldMap f r
```

![find the visual pun or
whatever](http://s3.amazonaws.com/lyah/accordion.png)
We think like this: if we are provided with a function that takes an
element of our tree and returns a monoid value, how do we reduce our
whole tree down to one single monoid value? When we were doing <span
class="fixed">fmap</span> over our tree, we applied the function that we
were mapping to a node and then we recursively mapped the function over
the left sub-tree as well as the right one. Here, we're tasked with not
only mapping a function, but with also joining up the results into a
single monoid value by using <span class="fixed">mappend</span>. First
we consider the case of the empty tree — a sad and lonely tree that has
no values or sub-trees. It doesn't hold any value that we can give to
our monoid-making function, so we just say that if our tree is empty,
the monoid value it becomes is <span class="fixed">mempty</span>.

The case of a non-empty node is a bit more interesting. It contains two
sub-trees as well as a value. In this case, we recursively <span
class="fixed">foldMap</span> the same function <span
class="fixed">f</span> over the left and the right sub-trees. Remember,
our <span class="fixed">foldMap</span> results in a single monoid value.
We also apply our function <span class="fixed">f</span> to the value in
the node. Now we have three monoid values (two from our sub-trees and
one from applying <span class="fixed">f</span> to the value in the node)
and we just have to bang them together into a single value. For this
purpose we use <span class="fixed">mappend</span>, and naturally the
left sub-tree comes first, then the node value and then the right
sub-tree.

Notice that we didn't have to provide the function that takes a value
and returns a monoid value. We receive that function as a parameter to
<span class="fixed">foldMap</span> and all we have to decide is where to
apply that function and how to join up the resulting monoids from it.

Now that we have a <span class="fixed">Foldable</span> instance for our
tree type, we get <span class="fixed">foldr</span> and <span
class="fixed">foldl</span> for free! Consider this tree:

``` {.haskell:hs name="code"}
testTree = Node 5
            (Node 3
                (Node 1 Empty Empty)
                (Node 6 Empty Empty)
            )
            (Node 9
                (Node 8 Empty Empty)
                (Node 10 Empty Empty)
            )
```

It has <span class="fixed">5</span> at its root and then its left node
is has <span class="fixed">3</span> with <span class="fixed">1</span> on
the left and <span class="fixed">6</span> on the right. The root's right
node has a <span class="fixed">9</span> and then an <span
class="fixed">8</span> to its left and a <span class="fixed">10</span>
on the far right side. With a <span class="fixed">Foldable</span>
instance, we can do all of the folds that we can do on lists:

``` {.haskell:hs name="code"}
ghci> F.foldl (+) 0 testTree
42
ghci> F.foldl (*) 1 testTree
64800
```

And also, <span class="fixed">foldMap</span> isn't only useful for
making new instances of <span class="fixed">Foldable</span>; it comes in
handy for reducing our structure to a single monoid value. For instance,
if we want to know if any number in our tree is equal to <span
class="fixed">3</span>, we can do this:

``` {.haskell:hs name="code"}
ghci> getAny $ F.foldMap (\x -> Any $ x == 3) testTree
True
```

Here, <span class="fixed">\\x -\> Any \$ x == 3</span> is a function
that takes a number and returns a monoid value, namely a <span
class="fixed">Bool</span> wrapped in <span class="fixed">Any</span>.
<span class="fixed">foldMap</span> applies this function to every
element in our tree and then reduces the resulting monoids into a single
monoid with <span class="fixed">mappend</span>. If we do this:

``` {.haskell:hs name="code"}
ghci> getAny $ F.foldMap (\x -> Any $ x > 15) testTree
False
```

All of the nodes in our tree would hold the value <span
class="fixed">Any False</span> after having the function in the lambda
applied to them. But to end up <span class="fixed">True</span>, <span
class="fixed">mappend</span> for <span class="fixed">Any</span> has to
have at least one <span class="fixed">True</span> value as a parameter.
That's why the final result is <span class="fixed">False</span>, which
makes sense because no value in our tree is greater than <span
class="fixed">15</span>.

We can also easily turn our tree into a list by doing a <span
class="fixed">foldMap</span> with the <span class="fixed">\\x -\>
[x]</span> function. By first projecting that function onto our tree,
each element becomes a singleton list. The <span
class="fixed">mappend</span> action that takes place between all those
singleton list results in a single list that holds all of the elements
that are in our tree:

``` {.haskell:hs name="code"}
ghci> F.foldMap (\x -> [x]) testTree
[1,3,6,5,8,9,10]
```

What's cool is that all of these trick aren't limited to trees, they
work on any instance of <span class="fixed">Foldable</span>.
