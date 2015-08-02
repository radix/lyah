Modules
=======

Loading modules
---------------

![modules](http://s3.amazonaws.com/lyah/modules.png)
A Haskell module is a collection of related functions, types and
typeclasses. A Haskell program is a collection of modules where the main
module loads up the other modules and then uses the functions defined in
them to do something. Having code split up into several modules has
quite a lot of advantages. If a module is generic enough, the functions
it exports can be used in a multitude of different programs. If your own
code is separated into self-contained modules which don't rely on each
other too much (we also say they are loosely coupled), you can reuse
them later on. It makes the whole deal of writing code more manageable
by having it split into several parts, each of which has some sort of
purpose.

The Haskell standard library is split into modules, each of them
contains functions and types that are somehow related and serve some
common purpose. There's a module for manipulating lists, a module for
concurrent programming, a module for dealing with complex numbers, etc.
All the functions, types and typeclasses that we've dealt with so far
were part of the <span class="fixed">Prelude</span> module, which is
imported by default. In this chapter, we're going to examine a few
useful modules and the functions that they have. But first, we're going
to see how to import modules.

The syntax for importing modules in a Haskell script is <span
class="fixed">import \<module name\></span>. This must be done before
defining any functions, so imports are usually done at the top of the
file. One script can, of course, import several modules. Just put each
import statement into a separate line. Let's import the <span
class="fixed">Data.List</span> module, which has a bunch of useful
functions for working with lists and use a function that it exports to
create a function that tells us how many unique elements a list has.

``` {.haskell:hs name="code"}
import Data.List

numUniques :: (Eq a) => [a] -> Int
numUniques = length . nub
```

When you do <span class="fixed">import Data.List</span>, all the
functions that <span class="fixed">Data.List</span> exports become
available in the global namespace, meaning that you can call them from
wherever in the script. <span class="fixed">nub</span> is a function
defined in <span class="fixed">Data.List</span> that takes a list and
weeds out duplicate elements. Composing <span
class="fixed">length</span> and <span class="fixed">nub</span> by doing
<span class="fixed">length . nub</span> produces a function that's the
equivalent of <span class="fixed">\\xs -\> length (nub xs)</span>.

You can also put the functions of modules into the global namespace when
using GHCI. If you're in GHCI and you want to be able to call the
functions exported by <span class="fixed">Data.List</span>, do this:

``` {.haskell:ghci name="code"}
ghci> :m + Data.List
```

If we want to load up the names from several modules inside GHCI, we
don't have to do <span class="fixed">:m +</span> several times, we can
just load up several modules at once.

``` {.haskell:ghci name="code"}
ghci> :m + Data.List Data.Map Data.Set
```

However, if you've loaded a script that already imports a module, you
don't need to use <span class="fixed">:m +</span> to get access to it.

If you just need a couple of functions from a module, you can
selectively import just those functions. If we wanted to import only the
<span class="fixed">nub</span> and <span class="fixed">sort</span>
functions from <span class="fixed">Data.List</span>, we'd do this:

``` {.haskell:hs name="code"}
import Data.List (nub, sort)
```

You can also choose to import all of the functions of a module except a
few select ones. That's often useful when several modules export
functions with the same name and you want to get rid of the offending
ones. Say we already have our own function that's called <span
class="fixed">nub</span> and we want to import all the functions from
<span class="fixed">Data.List</span> except the <span
class="fixed">nub</span> function:

``` {.haskell:hs name="code"}
import Data.List hiding (nub)
```

Another way of dealing with name clashes is to do qualified imports. The
<span class="fixed">Data.Map</span> module, which offers a data
structure for looking up values by key, exports a bunch of functions
with the same name as <span class="fixed">Prelude</span> functions, like
<span class="fixed">filter</span> or <span class="fixed">null</span>. So
when we import <span class="fixed">Data.Map</span> and then call <span
class="fixed">filter</span>, Haskell won't know which function to use.
Here's how we solve this:

``` {.haskell:hs name="code"}
import qualified Data.Map
```

This makes it so that if we want to reference <span
class="fixed">Data.Map</span>'s <span class="fixed">filter</span>
function, we have to do <span class="fixed">Data.Map.filter</span>,
whereas just <span class="fixed">filter</span> still refers to the
normal <span class="fixed">filter</span> we all know and love. But
typing out <span class="fixed">Data.Map</span> in front of every
function from that module is kind of tedious. That's why we can rename
the qualified import to something shorter:

``` {.haskell:hs name="code"}
import qualified Data.Map as M
```

Now, to reference <span class="fixed">Data.Map</span>'s <span
class="fixed">filter</span> function, we just use <span
class="fixed">M.filter</span>.

Use [this handy
reference](http://www.haskell.org/ghc/docs/latest/html/libraries/) to
see which modules are in the standard library. A great way to pick up
new Haskell knowledge is to just click through the standard library
reference and explore the modules and their functions. You can also view
the Haskell source code for each module. Reading the source code of some
modules is a really good way to learn Haskell and get a solid feel for
it.

To search for functions or to find out where they're located, use
[Hoogle](http://haskell.org/hoogle). It's a really awesome Haskell
search engine, you can search by name, module name or even type
signature.

Data.List
---------

The <span class="fixed">Data.List</span> module is all about lists,
obviously. It provides some very useful functions for dealing with them.
We've already met some of its functions (like <span
class="fixed">map</span> and <span class="fixed">filter</span>) because
the <span class="fixed">Prelude</span> module exports some functions
from <span class="fixed">Data.List</span> for convenience. You don't
have to import <span class="fixed">Data.List</span> via a qualified
import because it doesn't clash with any <span
class="fixed">Prelude</span> names except for those that <span
class="fixed">Prelude</span> already steals from <span
class="fixed">Data.List</span>. Let's take a look at some of the
functions that we haven't met before.

<span class="label function">intersperse</span> takes an element and a
list and then puts that element in between each pair of elements in the
list. Here's a demonstration:

``` {.haskell:ghci name="code"}
ghci> intersperse '.' "MONKEY"
"M.O.N.K.E.Y"
ghci> intersperse 0 [1,2,3,4,5,6]
[1,0,2,0,3,0,4,0,5,0,6]
```

<span class="label function">intercalate</span> takes a list of lists
and a list. It then inserts that list in between all those lists and
then flattens the result.

``` {.haskell:ghci name="code"}
ghci> intercalate " " ["hey","there","guys"]
"hey there guys"
ghci> intercalate [0,0,0] [[1,2,3],[4,5,6],[7,8,9]]
[1,2,3,0,0,0,4,5,6,0,0,0,7,8,9]
```

<span class="label function">transpose</span> transposes a list of
lists. If you look at a list of lists as a 2D matrix, the columns become
the rows and vice versa.

``` {.haskell:ghci name="code"}
ghci> transpose [[1,2,3],[4,5,6],[7,8,9]]
[[1,4,7],[2,5,8],[3,6,9]]
ghci> transpose ["hey","there","guys"]
["htg","ehu","yey","rs","e"]
```

Say we have the polynomials *3x^2^ + 5x + 9*, *10x^3^ + 9* and *8x^3^ +
5x^2^ + x - 1* and we want to add them together. We can use the lists
<span class="fixed">[0,3,5,9]</span>, <span
class="fixed">[10,0,0,9]</span> and <span
class="fixed">[8,5,1,-1]</span> to represent them in Haskell. Now, to
add them, all we have to do is this:

``` {.haskell:ghci name="code"}
ghci> map sum $ transpose [[0,3,5,9],[10,0,0,9],[8,5,1,-1]]
[18,8,6,17]
```

When we transpose these three lists, the third powers are then in the
first row, the second powers in the second one and so on. Mapping <span
class="fixed">sum</span> to that produces our desired result.

![shopping lists](http://s3.amazonaws.com/lyah/legolists.png)
<span class="label function">foldl'</span> and <span
class="label function">foldl1'</span> are stricter versions of their
respective lazy incarnations. When using lazy folds on really big lists,
you might often get a stack overflow error. The culprit for that is that
due to the lazy nature of the folds, the accumulator value isn't
actually updated as the folding happens. What actually happens is that
the accumulator kind of makes a promise that it will compute its value
when asked to actually produce the result (also called a thunk). That
happens for every intermediate accumulator and all those thunks overflow
your stack. The strict folds aren't lazy buggers and actually compute
the intermediate values as they go along instead of filling up your
stack with thunks. So if you ever get stack overflow errors when doing
lazy folds, try switching to their strict versions.

<span class="label function">concat</span> flattens a list of lists into
just a list of elements.

``` {.haskell:ghci name="code"}
ghci> concat ["foo","bar","car"]
"foobarcar"
ghci> concat [[3,4,5],[2,3,4],[2,1,1]]
[3,4,5,2,3,4,2,1,1]
```

It will just remove one level of nesting. So if you want to completely
flatten <span class="fixed">[[[2,3],[3,4,5],[2]],[[2,3],[3,4]]]</span>,
which is a list of lists of lists, you have to concatenate it twice.

Doing <span class="label function">concatMap</span> is the same as first
mapping a function to a list and then concatenating the list with <span
class="fixed">concat</span>.

``` {.haskell:ghci name="code"}
ghci> concatMap (replicate 4) [1..3]
[1,1,1,1,2,2,2,2,3,3,3,3]
```

<span class="label function">and</span> takes a list of boolean values
and returns <span class="fixed">True</span> only if all the values in
the list are <span class="fixed">True</span>.

``` {.haskell:ghci name="code"}
ghci> and $ map (>4) [5,6,7,8]
True
ghci> and $ map (==4) [4,4,4,3,4]
False
```

<span class="label function">or</span> is like <span
class="fixed">and</span>, only it returns <span
class="fixed">True</span> if any of the boolean values in a list is
<span class="fixed">True</span>.

``` {.haskell:ghci name="code"}
ghci> or $ map (==4) [2,3,4,5,6,1]
True
ghci> or $ map (>4) [1,2,3]
False
```

<span class="label function">any</span> and <span
class="label function">all</span> take a predicate and then check if any
or all the elements in a list satisfy the predicate, respectively.
Usually we use these two functions instead of mapping over a list and
then doing <span class="fixed">and</span> or <span
class="fixed">or</span>.

``` {.haskell:ghci name="code"}
ghci> any (==4) [2,3,5,6,1,4]
True
ghci> all (>4) [6,9,10]
True
ghci> all (`elem` ['A'..'Z']) "HEYGUYSwhatsup"
False
ghci> any (`elem` ['A'..'Z']) "HEYGUYSwhatsup"
True
```

<span class="label function">iterate</span> takes a function and a
starting value. It applies the function to the starting value, then it
applies that function to the result, then it applies the function to
that result again, etc. It returns all the results in the form of an
infinite list.

``` {.haskell:ghci name="code"}
ghci> take 10 $ iterate (*2) 1
[1,2,4,8,16,32,64,128,256,512]
ghci> take 3 $ iterate (++ "haha") "haha"
["haha","hahahaha","hahahahahaha"]
```

<span class="label function">splitAt</span> takes a number and a list.
It then splits the list at that many elements, returning the resulting
two lists in a tuple.

``` {.haskell:ghci name="code"}
ghci> splitAt 3 "heyman"
("hey","man")
ghci> splitAt 100 "heyman"
("heyman","")
ghci> splitAt (-3) "heyman"
("","heyman")
ghci> let (a,b) = splitAt 3 "foobar" in b ++ a
"barfoo"
```

<span class="label function">takeWhile</span> is a really useful little
function. It takes elements from a list while the predicate holds and
then when an element is encountered that doesn't satisfy the predicate,
it's cut off. It turns out this is very useful.

``` {.haskell:ghci name="code"}
ghci> takeWhile (>3) [6,5,4,3,2,1,2,3,4,5,4,3,2,1]
[6,5,4]
ghci> takeWhile (/=' ') "This is a sentence"
"This"
```

Say we wanted to know the sum of all third powers that are under 10,000.
We can't map <span class="fixed">(\^3)</span> to <span
class="fixed">[1..]</span>, apply a filter and then try to sum that up
because filtering an infinite list never finishes. You may know that all
the elements here are ascending but Haskell doesn't. That's why we can
do this:

``` {.haskell:ghci name="code"}
ghci> sum $ takeWhile (<10000) $ map (^3) [1..]
53361
```

We apply <span class="fixed">(\^3)</span> to an infinite list and then
once an element that's over 10,000 is encountered, the list is cut off.
Now we can sum it up easily.

<span class="label function">dropWhile</span> is similar, only it drops
all the elements while the predicate is true. Once predicate equates to
<span class="fixed">False</span>, it returns the rest of the list. An
extremely useful and lovely function!

``` {.haskell:ghci name="code"}
ghci> dropWhile (/=' ') "This is a sentence"
" is a sentence"
ghci> dropWhile (<3) [1,2,2,2,3,4,5,4,3,2,1]
[3,4,5,4,3,2,1]
```

We're given a list that represents the value of a stock by date. The
list is made of tuples whose first component is the stock value, the
second is the year, the third is the month and the fourth is the date.
We want to know when the stock value first exceeded one thousand
dollars!

``` {.haskell:ghci name="code"}
ghci> let stock = [(994.4,2008,9,1),(995.2,2008,9,2),(999.2,2008,9,3),(1001.4,2008,9,4),(998.3,2008,9,5)]
ghci> head (dropWhile (\(val,y,m,d) -> val < 1000) stock)
(1001.4,2008,9,4)
```

<span class="label function">span</span> is kind of like <span
class="fixed">takeWhile</span>, only it returns a pair of lists. The
first list contains everything the resulting list from <span
class="fixed">takeWhile</span> would contain if it were called with the
same predicate and the same list. The second list contains the part of
the list that would have been dropped.

``` {.haskell:ghci name="code"}
ghci> let (fw, rest) = span (/=' ') "This is a sentence" in "First word:" ++ fw ++ ", the rest:" ++ rest
"First word: This, the rest: is a sentence"
```

Whereas <span class="fixed">span</span> spans the list while the
predicate is true, <span class="label function">break</span> breaks it
when the predicate is first true. Doing <span class="fixed">break
p</span> is the equivalent of doing <span class="fixed">span (not .
p)</span>.

``` {.haskell:ghci name="code"}
ghci> break (==4) [1,2,3,4,5,6,7]
([1,2,3],[4,5,6,7])
ghci> span (/=4) [1,2,3,4,5,6,7]
([1,2,3],[4,5,6,7])
```

When using <span class="fixed">break</span>, the second list in the
result will start with the first element that satisfies the predicate.

<span class="label function">sort</span> simply sorts a list. The type
of the elements in the list has to be part of the <span
class="fixed">Ord</span> typeclass, because if the elements of a list
can't be put in some kind of order, then the list can't be sorted.

``` {.haskell:ghci name="code"}
ghci> sort [8,5,3,2,1,6,4,2]
[1,2,2,3,4,5,6,8]
ghci> sort "This will be sorted soon"
"    Tbdeehiillnooorssstw"
```

<span class="label function">group</span> takes a list and groups
adjacent elements into sublists if they are equal.

``` {.haskell:ghci name="code"}
ghci> group [1,1,1,1,2,2,2,2,3,3,2,2,2,5,6,7]
[[1,1,1,1],[2,2,2,2],[3,3],[2,2,2],[5],[6],[7]]
```

If we sort a list before grouping it, we can find out how many times
each element appears in the list.

``` {.haskell:ghci name="code"}
ghci> map (\l@(x:xs) -> (x,length l)) . group . sort $ [1,1,1,1,2,2,2,2,3,3,2,2,2,5,6,7]
[(1,4),(2,7),(3,2),(5,1),(6,1),(7,1)]
```

<span class="label function">inits</span> and <span
class="label function">tails</span> are like <span
class="fixed">init</span> and <span class="fixed">tail</span>, only they
recursively apply that to a list until there's nothing left. Observe.

``` {.haskell:ghci name="code"}
ghci> inits "w00t"
["","w","w0","w00","w00t"]
ghci> tails "w00t"
["w00t","00t","0t","t",""]
ghci> let w = "w00t" in zip (inits w) (tails w)
[("","w00t"),("w","00t"),("w0","0t"),("w00","t"),("w00t","")]
```

Let's use a fold to implement searching a list for a sublist.

``` {.haskell:hs name="code"}
search :: (Eq a) => [a] -> [a] -> Bool
search needle haystack =
    let nlen = length needle
    in  foldl (\acc x -> if take nlen x == needle then True else acc) False (tails haystack)
```

First we call <span class="fixed">tails</span> with the list in which
we're searching. Then we go over each tail and see if it starts with
what we're looking for.

With that, we actually just made a function that behaves like <span
class="label function">isInfixOf</span>. <span
class="fixed">isInfixOf</span> searches for a sublist within a list and
returns <span class="fixed">True</span> if the sublist we're looking for
is somewhere inside the target list.

``` {.haskell:ghci name="code"}
ghci> "cat" `isInfixOf` "im a cat burglar"
True
ghci> "Cat" `isInfixOf` "im a cat burglar"
False
ghci> "cats" `isInfixOf` "im a cat burglar"
False
```

<span class="label function">isPrefixOf</span> and <span
class="label function">isSuffixOf</span> search for a sublist at the
beginning and at the end of a list, respectively.

``` {.haskell:ghci name="code"}
ghci> "hey" `isPrefixOf` "hey there!"
True
ghci> "hey" `isPrefixOf` "oh hey there!"
False
ghci> "there!" `isSuffixOf` "oh hey there!"
True
ghci> "there!" `isSuffixOf` "oh hey there"
False
```

<span class="label function">elem</span> and <span
class="label function">notElem</span> check if an element is or isn't
inside a list.

<span class="label function">partition</span> takes a list and a
predicate and returns a pair of lists. The first list in the result
contains all the elements that satisfy the predicate, the second
contains all the ones that don't.

``` {.haskell:ghci name="code"}
ghci> partition (`elem` ['A'..'Z']) "BOBsidneyMORGANeddy"
("BOBMORGAN","sidneyeddy")
ghci> partition (>3) [1,3,5,6,3,2,1,0,3,7]
([5,6,7],[1,3,3,2,1,0,3])
```

It's important to understand how this is different from <span
class="fixed">span</span> and <span class="fixed">break</span>:

``` {.haskell:ghci name="code"}
ghci> span (`elem` ['A'..'Z']) "BOBsidneyMORGANeddy"
("BOB","sidneyMORGANeddy")
```

While <span class="fixed">span</span> and <span
class="fixed">break</span> are done once they encounter the first
element that doesn't and does satisfy the predicate, <span
class="fixed">partition</span> goes through the whole list and splits it
up according to the predicate.

<span class="label function">find</span> takes a list and a predicate
and returns the first element that satisfies the predicate. But it
returns that element wrapped in a <span class="fixed">Maybe</span>
value. We'll be covering algebraic data types more in depth in the next
chapter but for now, this is what you need to know: a <span
class="fixed">Maybe</span> value can either be <span class="fixed">Just
something</span> or <span class="fixed">Nothing</span>. Much like a list
can be either an empty list or a list with some elements, a <span
class="fixed">Maybe</span> value can be either no elements or a single
element. And like the type of a list of, say, integers is <span
class="fixed">[Int]</span>, the type of maybe having an integer is <span
class="fixed">Maybe Int</span>. Anyway, let's take our <span
class="fixed">find</span> function for a spin.

``` {.haskell:ghci name="code"}
ghci> find (>4) [1,2,3,4,5,6]
Just 5
ghci> find (>9) [1,2,3,4,5,6]
Nothing
ghci> :t find
find :: (a -> Bool) -> [a] -> Maybe a
```

Notice the type of <span class="fixed">find</span>. Its result is <span
class="fixed">Maybe a</span>. That's kind of like having the type of
<span class="fixed">[a]</span>, only a value of the type <span
class="fixed">Maybe</span> can contain either no elements or one
element, whereas a list can contain no elements, one element or several
elements.

Remember when we were searching for the first time our stock went over
\$1000. We did <span class="fixed">head (dropWhile (\\(val,y,m,d) -\>
val \< 1000) stock)</span>. Remember that <span
class="fixed">head</span> is not really safe. What would happen if our
stock never went over \$1000? Our application of <span
class="fixed">dropWhile</span> would return an empty list and getting
the head of an empty list would result in an error. However, if we
rewrote that as <span class="fixed">find (\\(val,y,m,d) -\> val \> 1000)
stock</span>, we'd be much safer. If our stock never went over \$1000
(so if no element satisfied the predicate), we'd get back a <span
class="fixed">Nothing</span>. But there was a valid answer in that list,
we'd get, say, <span class="fixed">Just (1001.4,2008,9,4)</span>.

<span class="label function">elemIndex</span> is kind of like <span
class="fixed">elem</span>, only it doesn't return a boolean value. It
maybe returns the index of the element we're looking for. If that
element isn't in our list, it returns a <span
class="fixed">Nothing</span>.

``` {.haskell:ghci name="code"}
ghci> :t elemIndex
elemIndex :: (Eq a) => a -> [a] -> Maybe Int
ghci> 4 `elemIndex` [1,2,3,4,5,6]
Just 3
ghci> 10 `elemIndex` [1,2,3,4,5,6]
Nothing
```

<span class="label function">elemIndices</span> is like <span
class="fixed">elemIndex</span>, only it returns a list of indices, in
case the element we're looking for crops up in our list several times.
Because we're using a list to represent the indices, we don't need a
<span class="fixed">Maybe</span> type, because failure can be
represented as the empty list, which is very much synonymous to <span
class="fixed">Nothing</span>.

``` {.haskell:ghci name="code"}
ghci> ' ' `elemIndices` "Where are the spaces?"
[5,9,13]
```

<span class="label function">findIndex</span> is like find, but it maybe
returns the index of the first element that satisfies the predicate.
<span class="label function">findIndices</span> returns the indices of
all elements that satisfy the predicate in the form of a list.

``` {.haskell:ghci name="code"}
ghci> findIndex (==4) [5,3,2,1,6,4]
Just 5
ghci> findIndex (==7) [5,3,2,1,6,4]
Nothing
ghci> findIndices (`elem` ['A'..'Z']) "Where Are The Caps?"
[0,6,10,14]
```

We already covered <span class="fixed">zip</span> and <span
class="fixed">zipWith</span>. We noted that they zip together two lists,
either in a tuple or with a binary function (meaning such a function
that takes two parameters). But what if we want to zip together three
lists? Or zip three lists with a function that takes three parameters?
Well, for that, we have <span class="label function">zip3</span>, <span
class="label function">zip4</span>, etc. and <span
class="label function">zipWith3</span>, <span
class="label function">zipWith4</span>, etc. These variants go up to 7.
While this may look like a hack, it works out pretty fine, because there
aren't many times when you want to zip 8 lists together. There's also a
very clever way for zipping infinite numbers of lists, but we're not
advanced enough to cover that just yet.

``` {.haskell:ghci name="code"}
ghci> zipWith3 (\x y z -> x + y + z) [1,2,3] [4,5,2,2] [2,2,3]
[7,9,8]
ghci> zip4 [2,3,3] [2,2,2] [5,5,3] [2,2,2]
[(2,2,5,2),(3,2,5,2),(3,2,3,2)]
```

Just like with normal zipping, lists that are longer than the shortest
list that's being zipped are cut down to size.

<span class="label function">lines</span> is a useful function when
dealing with files or input from somewhere. It takes a string and
returns every line of that string in a separate list.

``` {.haskell:ghci name="code"}
ghci> lines "first line\nsecond line\nthird line"
["first line","second line","third line"]
```

<span class="fixed">'\\n'</span> is the character for a unix newline.
Backslashes have special meaning in Haskell strings and characters.

<span class="label function">unlines</span> is the inverse function of
<span class="fixed">lines</span>. It takes a list of strings and joins
them together using a <span class="fixed">'\\n'</span>.

``` {.haskell:ghci name="code"}
ghci> unlines ["first line", "second line", "third line"]
"first line\nsecond line\nthird line\n"
```

<span class="label function">words</span> and <span
class="label function">unwords</span> are for splitting a line of text
into words or joining a list of words into a text. Very useful.

``` {.haskell:ghci name="code"}
ghci> words "hey these are the words in this sentence"
["hey","these","are","the","words","in","this","sentence"]
ghci> words "hey these           are    the words in this\nsentence"
["hey","these","are","the","words","in","this","sentence"]
ghci> unwords ["hey","there","mate"]
"hey there mate"
```

We've already mentioned <span class="label function">nub</span>. It
takes a list and weeds out the duplicate elements, returning a list
whose every element is a unique snowflake! The function does have a kind
of strange name. It turns out that "nub" means a small lump or essential
part of something. In my opinion, they should use real words for
function names instead of old-people words.

``` {.haskell:ghci name="code"}
ghci> nub [1,2,3,4,3,2,1,2,3,4,3,2,1]
[1,2,3,4]
ghci> nub "Lots of words and stuff"
"Lots fwrdanu"
```

<span class="label function">delete</span> takes an element and a list
and deletes the first occurence of that element in the list.

``` {.haskell:ghci name="code"}
ghci> delete 'h' "hey there ghang!"
"ey there ghang!"
ghci> delete 'h' . delete 'h' $ "hey there ghang!"
"ey tere ghang!"
ghci> delete 'h' . delete 'h' . delete 'h' $ "hey there ghang!"
"ey tere gang!"
```

<span class="label function">\\\\</span> is the list difference
function. It acts like a set difference, basically. For every element in
the right-hand list, it removes a matching element in the left one.

``` {.haskell:ghci name="code"}
ghci> [1..10] \\ [2,5,9]
[1,3,4,6,7,8,10]
ghci> "Im a big baby" \\ "big"
"Im a  baby"
```

Doing <span class="fixed">[1..10] \\\\ [2,5,9]</span> is like doing
<span class="fixed">delete 2 . delete 5 . delete 9 \$ [1..10]</span>.

<span class="label function">union</span> also acts like a function on
sets. It returns the union of two lists. It pretty much goes over every
element in the second list and appends it to the first one if it isn't
already in yet. Watch out though, duplicates are removed from the second
list!

``` {.haskell:ghci name="code"}
ghci> "hey man" `union` "man what's up"
"hey manwt'sup"
ghci> [1..7] `union` [5..10]
[1,2,3,4,5,6,7,8,9,10]
```

<span class="label function">intersect</span> works like set
intersection. It returns only the elements that are found in both lists.

``` {.haskell:ghci name="code"}
ghci> [1..7] `intersect` [5..10]
[5,6,7]
```

<span class="label function">insert</span> takes an element and a list
of elements that can be sorted and inserts it into the last position
where it's still less than or equal to the next element. In other words,
<span class="fixed">insert</span> will start at the beginning of the
list and then keep going until it finds an element that's equal to or
greater than the element that we're inserting and it will insert it just
before the element.

``` {.haskell:ghci name="code"}
ghci> insert 4 [3,5,1,2,8,2]
[3,4,5,1,2,8,2]
ghci> insert 4 [1,3,4,4,1]
[1,3,4,4,4,1]
```

The <span class="fixed">4</span> is inserted right after the <span
class="fixed">3</span> and before the <span class="fixed">5</span> in
the first example and in between the <span class="fixed">3</span> and
<span class="fixed">4</span> in the second example.

If we use <span class="fixed">insert</span> to insert into a sorted
list, the resulting list will be kept sorted.
``` {.haskell:ghci name="code"}
ghci> insert 4 [1,2,3,5,6,7]
[1,2,3,4,5,6,7]
ghci> insert 'g' $ ['a'..'f'] ++ ['h'..'z']
"abcdefghijklmnopqrstuvwxyz"
ghci> insert 3 [1,2,4,3,2,1]
[1,2,3,4,3,2,1]
```

What <span class="fixed">length</span>, <span class="fixed">take</span>,
<span class="fixed">drop</span>, <span class="fixed">splitAt</span>,
<span class="fixed">!!</span> and <span class="fixed">replicate</span>
have in common is that they take an <span class="fixed">Int</span> as
one of their parameters (or return an <span class="fixed">Int</span>),
even though they could be more generic and usable if they just took any
type that's part of the <span class="fixed">Integral</span> or <span
class="fixed">Num</span> typeclasses (depending on the functions). They
do that for historical reasons. However, fixing that would probably
break a lot of existing code. That's why <span
class="fixed">Data.List</span> has their more generic equivalents, named
<span class="label function">genericLength</span>, <span
class="label function">genericTake</span>, <span
class="label function">genericDrop</span>, <span
class="label function">genericSplitAt</span>, <span
class="label function">genericIndex</span> and <span
class="label function">genericReplicate</span>. For instance, <span
class="fixed">length</span> has a type signature of <span
class="fixed">length :: [a] -\> Int</span>. If we try to get the average
of a list of numbers by doing <span class="fixed">let xs = [1..6] in sum
xs / length xs</span>, we get a type error, because you can't use <span
class="fixed">/</span> with an <span class="fixed">Int</span>. <span
class="fixed">genericLength</span>, on the other hand, has a type
signature of <span class="fixed">genericLength :: (Num a) =\> [b] -\>
a</span>. Because a <span class="fixed">Num</span> can act like a
floating point number, getting the average by doing <span
class="fixed">let xs = [1..6] in sum xs / genericLength xs</span> works
out just fine.

The <span class="fixed">nub</span>, <span class="fixed">delete</span>,
<span class="fixed">union</span>, <span class="fixed">intersect</span>
and <span class="fixed">group</span> functions all have their more
general counterparts called <span class="label function">nubBy</span>,
<span class="label function">deleteBy</span>, <span
class="label function">unionBy</span>, <span
class="label function">intersectBy</span> and <span
class="label function">groupBy</span>. The difference between them is
that the first set of functions use <span class="fixed">==</span> to
test for equality, whereas the *By* ones also take an equality function
and then compare them by using that equality function. <span
class="fixed">group</span> is the same as <span class="fixed">groupBy
(==)</span>.

For instance, say we have a list that describes the value of a function
for every second. We want to segment it into sublists based on when the
value was below zero and when it went above. If we just did a normal
<span class="fixed">group</span>, it would just group the equal adjacent
values together. But what we want is to group them by whether they are
negative or not. That's where <span class="fixed">groupBy</span> comes
in! The equality function supplied to the *By* functions should take two
elements of the same type and return <span class="fixed">True</span> if
it considers them equal by its standards.

``` {.haskell:ghci name="code"}
ghci> let values = [-4.3, -2.4, -1.2, 0.4, 2.3, 5.9, 10.5, 29.1, 5.3, -2.4, -14.5, 2.9, 2.3]
ghci> groupBy (\x y -> (x > 0) == (y > 0)) values
[[-4.3,-2.4,-1.2],[0.4,2.3,5.9,10.5,29.1,5.3],[-2.4,-14.5],[2.9,2.3]]
```

From this, we clearly see which sections are positive and which are
negative. The equality function supplied takes two elements and then
returns <span class="fixed">True</span> only if they're both negative or
if they're both positive. This equality function can also be written as
<span class="fixed">\\x y -\> (x \> 0) && (y \> 0) || (x \<= 0) && (y
\<= 0)</span>, although I think the first way is more readable. An even
clearer way to write equality functions for the *By* functions is if you
import the <span class="label function">on</span> function from <span
class="fixed">Data.Function</span>. <span class="fixed">on</span> is
defined like this:

``` {.haskell:ghci name="code"}
on :: (b -> b -> c) -> (a -> b) -> a -> a -> c
f `on` g = \x y -> f (g x) (g y)
```

So doing <span class="fixed">(==) \`on\` (\> 0)</span> returns an
equality function that looks like <span class="fixed">\\x y -\> (x \> 0)
== (y \> 0)</span>. <span class="fixed">on</span> is used a lot with the
*By* functions because with it, we can do:

``` {.haskell:ghci name="code"}
ghci> groupBy ((==) `on` (> 0)) values
[[-4.3,-2.4,-1.2],[0.4,2.3,5.9,10.5,29.1,5.3],[-2.4,-14.5],[2.9,2.3]]
```

Very readable indeed! You can read it out loud: Group this by equality
on whether the elements are greater than zero.

Similarly, the <span class="fixed">sort</span>, <span
class="fixed">insert</span>, <span class="fixed">maximum</span> and
<span class="fixed">minimum</span> also have their more general
equivalents. Functions like <span class="fixed">groupBy</span> take a
function that determines when two elements are equal. <span
class="label function">sortBy</span>, <span
class="label function">insertBy</span>, <span
class="label function">maximumBy</span> and <span
class="label function">minimumBy</span> take a function that determine
if one element is greater, smaller or equal to the other. The type
signature of <span class="fixed">sortBy</span> is <span
class="fixed">sortBy :: (a -\> a -\> Ordering) -\> [a] -\> [a]</span>.
If you remember from before, the <span class="fixed">Ordering</span>
type can have a value of <span class="fixed">LT</span>, <span
class="fixed">EQ</span> or <span class="fixed">GT</span>. <span
class="fixed">sort</span> is the equivalent of <span
class="fixed">sortBy compare</span>, because compare just takes two
elements whose type is in the <span class="fixed">Ord</span> typeclass
and returns their ordering relationship.

Lists can be compared, but when they are, they are compared
lexicographically. What if we have a list of lists and we want to sort
it not based on the inner lists' contents but on their lengths? Well, as
you've probably guessed, we'll use the <span class="fixed">sortBy</span>
function.

``` {.haskell:ghci name="code"}
ghci> let xs = [[5,4,5,4,4],[1,2,3],[3,5,4,3],[],[2],[2,2]]
ghci> sortBy (compare `on` length) xs
[[],[2],[2,2],[1,2,3],[3,5,4,3],[5,4,5,4,4]]
```

Awesome! <span class="fixed">compare \`on\` length</span> ... man, that
reads almost like real English! If you're not sure how exactly the <span
class="fixed">on</span> works here, <span class="fixed">compare \`on\`
length</span> is the equivalent of <span class="fixed">\\x y -\> length
x \`compare\` length y</span>. When you're dealing with *By* functions
that take an equality function, you usually do <span class="fixed">(==)
\`on\` something</span> and when you're dealing with *By* functions that
take an ordering function, you usually do <span class="fixed">compare
\`on\` something</span>.

Data.Char
---------

![lego char](http://s3.amazonaws.com/lyah/legochar.png)
The <span class="fixed">Data.Char</span> module does what its name
suggests. It exports functions that deal with characters. It's also
helpful when filtering and mapping over strings because they're just
lists of characters.

<span class="fixed">Data.Char</span> exports a bunch of predicates over
characters. That is, functions that take a character and tell us whether
some assumption about it is true or false. Here's what they are:

<span class="label function">isControl</span> checks whether a character
is a control character.

<span class="label function">isSpace</span> checks whether a character
is a white-space characters. That includes spaces, tab characters,
newlines, etc.

<span class="label function">isLower</span> checks whether a character
is lower-cased.

<span class="label function">isUpper</span> checks whether a character
is upper-cased.

<span class="label function">isAlpha</span> checks whether a character
is a letter.

<span class="label function">isAlphaNum</span> checks whether a
character is a letter or a number.

<span class="label function">isPrint</span> checks whether a character
is printable. Control characters, for instance, are not printable.

<span class="label function">isDigit</span> checks whether a character
is a digit.

<span class="label function">isOctDigit</span> checks whether a
character is an octal digit.

<span class="label function">isHexDigit</span> checks whether a
character is a hex digit.

<span class="label function">isLetter</span> checks whether a character
is a letter.

<span class="label function">isMark</span> checks for Unicode mark
characters. Those are characters that combine with preceding letters to
form latters with accents. Use this if you are French.

<span class="label function">isNumber</span> checks whether a character
is numeric.

<span class="label function">isPunctuation</span> checks whether a
character is punctuation.

<span class="label function">isSymbol</span> checks whether a character
is a fancy mathematical or currency symbol.

<span class="label function">isSeparator</span> checks for Unicode
spaces and separators.

<span class="label function">isAscii</span> checks whether a character
falls into the first 128 characters of the Unicode character set.

<span class="label function">isLatin1</span> checks whether a character
falls into the first 256 characters of Unicode.

<span class="label function">isAsciiUpper</span> checks whether a
character is ASCII and upper-case.

<span class="label function">isAsciiLower</span> checks whether a
character is ASCII and lower-case.

All these predicates have a type signature of <span
class="fixed">Char -\> Bool</span>. Most of the time you'll use this to
filter out strings or something like that. For instance, let's say we're
making a program that takes a username and the username can only be
comprised of alphanumeric characters. We can use the <span
class="fixed">Data.List</span> function <span class="fixed">all</span>
in combination with the <span class="fixed">Data.Char</span> predicates
to determine if the username is alright.

``` {.haskell:ghci name="code"}
ghci> all isAlphaNum "bobby283"
True
ghci> all isAlphaNum "eddy the fish!"
False
```

Kewl. In case you don't remember, <span class="fixed">all</span> takes a
predicate and a list and returns <span class="fixed">True</span> only if
that predicate holds for every element in the list.

We can also use <span class="fixed">isSpace</span> to simulate the <span
class="fixed">Data.List</span> function <span
class="fixed">words</span>.

``` {.haskell:ghci name="code"}
ghci> words "hey guys its me"
["hey","guys","its","me"]
ghci> groupBy ((==) `on` isSpace) "hey guys its me"
["hey"," ","guys"," ","its"," ","me"]
ghci>
```

Hmmm, well, it kind of does what <span class="fixed">words</span> does
but we're left with elements of only spaces. Hmm, whatever shall we do?
I know, let's filter that sucker.

``` {.haskell:ghci name="code"}
ghci> filter (not . any isSpace) . groupBy ((==) `on` isSpace) $ "hey guys its me"
["hey","guys","its","me"]
```

Ah.

The <span class="fixed">Data.Char</span> also exports a datatype that's
kind of like <span class="fixed">Ordering</span>. The <span
class="fixed">Ordering</span> type can have a value of <span
class="fixed">LT</span>, <span class="fixed">EQ</span> or <span
class="fixed">GT</span>. It's a sort of enumeration. It describes a few
possible results that can arise from comparing two elements. The <span
class="fixed">GeneralCategory</span> type is also an enumeration. It
presents us with a few possible categories that a character can fall
into. The main function for getting the general category of a character
is <span class="fixed">generalCategory</span>. It has a type of <span
class="fixed">generalCategory :: Char -\> GeneralCategory</span>. There
are about 31 categories so we won't list them all here, but let's play
around with the function.

``` {.haskell:ghci name="code"}
ghci> generalCategory ' '
Space
ghci> generalCategory 'A'
UppercaseLetter
ghci> generalCategory 'a'
LowercaseLetter
ghci> generalCategory '.'
OtherPunctuation
ghci> generalCategory '9'
DecimalNumber
ghci> map generalCategory " \t\nA9?|"
[Space,Control,Control,UppercaseLetter,DecimalNumber,OtherPunctuation,MathSymbol]
```

Since the <span class="fixed">GeneralCategory</span> type is part of the
<span class="fixed">Eq</span> typeclass, we can also test for stuff like
<span class="fixed">generalCategory c == Space</span>.

<span class="label function">toUpper</span> converts a character to
upper-case. Spaces, numbers, and the like remain unchanged.

<span class="label function">toLower</span> converts a character to
lower-case.

<span class="label function">toTitle</span> converts a character to
title-case. For most characters, title-case is the same as upper-case.

<span class="label function">digitToInt</span> converts a character to
an <span class="fixed">Int</span>. To succeed, the character must be in
the ranges <span class="fixed">'0'..'9'</span>, <span
class="fixed">'a'..'f'</span> or <span class="fixed">'A'..'F'</span>.

``` {.haskell:ghci name="code"}
ghci> map digitToInt "34538"
[3,4,5,3,8]
ghci> map digitToInt "FF85AB"
[15,15,8,5,10,11]
```

<span class="label function">intToDigit</span> is the inverse function
of <span class="fixed">digitToInt</span>. It takes an <span
class="fixed">Int</span> in the range of <span
class="fixed">0..15</span> and converts it to a lower-case character.

``` {.haskell:ghci name="code"}
ghci> intToDigit 15
'f'
ghci> intToDigit 5
'5'
```

The <span class="label function">ord</span> and <span
class="fixed">chr</span> functions convert characters to their
corresponding numbers and vice versa:

``` {.haskell:ghci name="code"}
ghci> ord 'a'
97
ghci> chr 97
'a'
ghci> map ord "abcdefgh"
[97,98,99,100,101,102,103,104]
```

The difference between the <span class="fixed">ord</span> values of two
characters is equal to how far apart they are in the Unicode table.

The Caesar cipher is a primitive method of encoding messages by shifting
each character in them by a fixed number of positions in the alphabet.
We can easily create a sort of Caesar cipher of our own, only we won't
constrict ourselves to the alphabet.

``` {.haskell:hs name="code"}
encode :: Int -> String -> String
encode shift msg =
    let ords = map ord msg
        shifted = map (+ shift) ords
    in  map chr shifted
```

Here, we first convert the string to a list of numbers. Then we add the
shift amount to each number before converting the list of numbers back
to characters. If you're a composition cowboy, you could write the body
of this function as <span class="fixed">map (chr . (+ shift) . ord)
msg</span>. Let's try encoding a few messages.

``` {.haskell:ghci name="code"}
ghci> encode 3 "Heeeeey"
"Khhhhh|"
ghci> encode 4 "Heeeeey"
"Liiiii}"
ghci> encode 1 "abcd"
"bcde"
ghci> encode 5 "Marry Christmas! Ho ho ho!"
"Rfww~%Hmwnxyrfx&%Mt%mt%mt&"
```

That's encoded alright. Decoding a message is basically just shifting it
back by the number of places it was shifted by in the first place.

``` {.haskell:hs name="code"}
decode :: Int -> String -> String
decode shift msg = encode (negate shift) msg
```

``` {.haskell:ghci name="code"}
ghci> encode 3 "Im a little teapot"
"Lp#d#olwwoh#whdsrw"
ghci> decode 3 "Lp#d#olwwoh#whdsrw"
"Im a little teapot"
ghci> decode 5 . encode 5 $ "This is a sentence"
"This is a sentence"
```

Data.Map
--------

Association lists (also called dictionaries) are lists that are used to
store key-value pairs where ordering doesn't matter. For instance, we
might use an association list to store phone numbers, where phone
numbers would be the values and people's names would be the keys. We
don't care in which order they're stored, we just want to get the right
phone number for the right person.

The most obvious way to represent association lists in Haskell would be
by having a list of pairs. The first component in the pair would be the
key, the second component the value. Here's an example of an association
list with phone numbers:

``` {.haskell:hs name="code"}
phoneBook =
    [("betty","555-2938")
    ,("bonnie","452-2928")
    ,("patsy","493-2928")
    ,("lucille","205-2928")
    ,("wendy","939-8282")
    ,("penny","853-2492")
    ]
```

Despite this seemingly odd indentation, this is just a list of pairs of
strings. The most common task when dealing with association lists is
looking up some value by key. Let's make a function that looks up some
value given a key.

``` {.haskell:hs name="code"}
findKey :: (Eq k) => k -> [(k,v)] -> v
findKey key xs = snd . head . filter (\(k,v) -> key == k) $ xs
```

Pretty simple. The function that takes a key and a list, filters the
list so that only matching keys remain, gets the first key-value that
matches and returns the value. But what happens if the key we're looking
for isn't in the association list? Hmm. Here, if a key isn't in the
association list, we'll end up trying to get the head of an empty list,
which throws a runtime error. However, we should avoid making our
programs so easy to crash, so let's use the <span
class="fixed">Maybe</span> data type. If we don't find the key, we'll
return a <span class="fixed">Nothing</span>. If we find it, we'll return
<span class="fixed">Just something</span>, where something is the value
corresponding to that key.

``` {.haskell:hs name="code"}
findKey :: (Eq k) => k -> [(k,v)] -> Maybe v
findKey key [] = Nothing
findKey key ((k,v):xs) = if key == k
                            then Just v
                            else findKey key xs
```

Look at the type declaration. It takes a key that can be equated, an
association list and then it maybe produces a value. Sounds about right.

This is a textbook recursive function that operates on a list. Edge
case, splitting a list into a head and a tail, recursive calls, they're
all there. This is the classic fold pattern, so let's see how this would
be implemented as a fold.

``` {.haskell:hs name="code"}
findKey :: (Eq k) => k -> [(k,v)] -> Maybe v
findKey key = foldr (\(k,v) acc -> if key == k then Just v else acc) Nothing
```

<div class="hintbox">

*Note:* It's usually better to use folds for this standard list
recursion pattern instead of explicitly writing the recursion because
they're easier to read and identify. Everyone knows it's a fold when
they see the <span class="fixed">foldr</span> call, but it takes some
more thinking to read explicit recursion.

</div>

``` {.haskell:ghci name="code"}
ghci> findKey "penny" phoneBook
Just "853-2492"
ghci> findKey "betty" phoneBook
Just "555-2938"
ghci> findKey "wilma" phoneBook
Nothing
```

![legomap](http://s3.amazonaws.com/lyah/legomap.png)
Works like a charm! If we have the girl's phone number, we <span
class="fixed">Just</span> get the number, otherwise we get <span
class="fixed">Nothing</span>.

We just implemented the <span class="fixed">lookup</span> function from
<span class="fixed">Data.List</span>. If we want to find the
corresponding value to a key, we have to traverse all the elements of
the list until we find it. The <span class="fixed">Data.Map</span>
module offers association lists that are much faster (because they're
internally implemented with trees) and also it provides a lot of utility
functions. From now on, we'll say we're working with maps instead of
association lists.

Because <span class="fixed">Data.Map</span> exports functions that clash
with the <span class="fixed">Prelude</span> and <span
class="fixed">Data.List</span> ones, we'll do a qualified import.

``` {.haskell:hs name="code"}
import qualified Data.Map as Map
```

Put this import statement into a script and then load the script via
GHCI.

Let's go ahead and see what <span class="fixed">Data.Map</span> has in
store for us! Here's the basic rundown of its functions.

The <span class="label function">fromList</span> function takes an
association list (in the form of a list) and returns a map with the same
associations.

``` {.haskell:ghci name="code"}
ghci> Map.fromList [("betty","555-2938"),("bonnie","452-2928"),("lucille","205-2928")]
fromList [("betty","555-2938"),("bonnie","452-2928"),("lucille","205-2928")]
ghci> Map.fromList [(1,2),(3,4),(3,2),(5,5)]
fromList [(1,2),(3,2),(5,5)]
```

If there are duplicate keys in the original association list, the
duplicates are just discarded. This is the type signature of <span
class="fixed">fromList</span>

``` {.haskell:hs name="code"}
Map.fromList :: (Ord k) => [(k, v)] -> Map.Map k v
```

It says that it takes a list of pairs of type <span
class="fixed">k</span> and <span class="fixed">v</span> and returns a
map that maps from keys of type <span class="fixed">k</span> to type
<span class="fixed">v</span>. Notice that when we were doing association
lists with normal lists, the keys only had to be equatable (their type
belonging to the <span class="fixed">Eq</span> typeclass) but now they
have to be orderable. That's an essential constraint in the <span
class="fixed">Data.Map</span> module. It needs the keys to be orderable
so it can arrange them in a tree.

You should always use <span class="fixed">Data.Map</span> for key-value
associations unless you have keys that aren't part of the <span
class="fixed">Ord</span> typeclass.

<span class="label function">empty</span> represents an empty map. It
takes no arguments, it just returns an empty map.

``` {.haskell:ghci name="code"}
ghci> Map.empty
fromList []
```

<span class="label function">insert</span> takes a key, a value and a
map and returns a new map that's just like the old one, only with the
key and value inserted.

``` {.haskell:ghci name="code"}
ghci> Map.empty
fromList []
ghci> Map.insert 3 100 Map.empty
fromList [(3,100)]
ghci> Map.insert 5 600 (Map.insert 4 200 ( Map.insert 3 100  Map.empty))
fromList [(3,100),(4,200),(5,600)]
ghci> Map.insert 5 600 . Map.insert 4 200 . Map.insert 3 100 $ Map.empty
fromList [(3,100),(4,200),(5,600)]
```

We can implement our own <span class="fixed">fromList</span> by using
the empty map, <span class="fixed">insert</span> and a fold. Watch:

``` {.haskell:ghci name="code"}
fromList' :: (Ord k) => [(k,v)] -> Map.Map k v
fromList' = foldr (\(k,v) acc -> Map.insert k v acc) Map.empty
```

It's a pretty straightforward fold. We start of with an empty map and we
fold it up from the right, inserting the key value pairs into the
accumulator as we go along.

<span class="label function">null</span> checks if a map is empty.

``` {.haskell:ghci name="code"}
ghci> Map.null Map.empty
True
ghci> Map.null $ Map.fromList [(2,3),(5,5)]
False
```

<span class="label function">size</span> reports the size of a map.

``` {.haskell:ghci name="code"}
ghci> Map.size Map.empty
0
ghci> Map.size $ Map.fromList [(2,4),(3,3),(4,2),(5,4),(6,4)]
5
```

<span class="label function">singleton</span> takes a key and a value
and creates a map that has exactly one mapping.

``` {.haskell:ghci name="code"}
ghci> Map.singleton 3 9
fromList [(3,9)]
ghci> Map.insert 5 9 $ Map.singleton 3 9
fromList [(3,9),(5,9)]
```

<span class="label function">lookup</span> works like the <span
class="fixed">Data.List</span> <span class="fixed">lookup</span>, only
it operates on maps. It returns <span class="fixed">Just
something</span> if it finds something for the key and <span
class="fixed">Nothing</span> if it doesn't.

<span class="label function">member</span> is a predicate takes a key
and a map and reports whether the key is in the map or not.

``` {.haskell:ghci name="code"}
ghci> Map.member 3 $ Map.fromList [(3,6),(4,3),(6,9)]
True
ghci> Map.member 3 $ Map.fromList [(2,5),(4,5)]
False
```

<span class="label function">map</span> and <span
class="label function">filter</span> work much like their list
equivalents.

``` {.haskell:ghci name="code"}
ghci> Map.map (*100) $ Map.fromList [(1,1),(2,4),(3,9)]
fromList [(1,100),(2,400),(3,900)]
ghci> Map.filter isUpper $ Map.fromList [(1,'a'),(2,'A'),(3,'b'),(4,'B')]
fromList [(2,'A'),(4,'B')]
```

<span class="label function">toList</span> is the inverse of <span
class="fixed">fromList</span>.

``` {.haskell:ghci name="code"}
ghci> Map.toList . Map.insert 9 2 $ Map.singleton 4 3
[(4,3),(9,2)]
```

<span class="label function">keys</span> and <span
class="label function">elems</span> return lists of keys and values
respectively. <span class="fixed">keys</span> is the equivalent of <span
class="fixed">map fst . Map.toList</span> and <span
class="fixed">elems</span> is the equivalent of <span class="fixed">map
snd . Map.toList</span>.

<span class="label function">fromListWith</span> is a cool little
function. It acts like <span class="fixed">fromList</span>, only it
doesn't discard duplicate keys but it uses a function supplied to it to
decide what to do with them. Let's say that a girl can have several
numbers and we have an association list set up like this.

``` {.haskell:hs name="code"}
phoneBook =
    [("betty","555-2938")
    ,("betty","342-2492")
    ,("bonnie","452-2928")
    ,("patsy","493-2928")
    ,("patsy","943-2929")
    ,("patsy","827-9162")
    ,("lucille","205-2928")
    ,("wendy","939-8282")
    ,("penny","853-2492")
    ,("penny","555-2111")
    ]
```

Now if we just use <span class="fixed">fromList</span> to put that into
a map, we'll lose a few numbers! So here's what we'll do:

``` {.haskell:hs name="code"}
phoneBookToMap :: (Ord k) => [(k, String)] -> Map.Map k String
phoneBookToMap xs = Map.fromListWith (\number1 number2 -> number1 ++ ", " ++ number2) xs
```

``` {.haskell:hs name="code"}
ghci> Map.lookup "patsy" $ phoneBookToMap phoneBook
"827-9162, 943-2929, 493-2928"
ghci> Map.lookup "wendy" $ phoneBookToMap phoneBook
"939-8282"
ghci> Map.lookup "betty" $ phoneBookToMap phoneBook
"342-2492, 555-2938"
```

If a duplicate key is found, the function we pass is used to combine the
values of those keys into some other value. We could also first make all
the values in the association list singleton lists and then we can use
<span class="fixed">++</span> to combine the numbers.

``` {.haskell:hs name="code"}
phoneBookToMap :: (Ord k) => [(k, a)] -> Map.Map k [a]
phoneBookToMap xs = Map.fromListWith (++) $ map (\(k,v) -> (k,[v])) xs
```

``` {.haskell:ghci name="code"}
ghci> Map.lookup "patsy" $ phoneBookToMap phoneBook
["827-9162","943-2929","493-2928"]
```

Pretty neat! Another use case is if we're making a map from an
association list of numbers and when a duplicate key is found, we want
the biggest value for the key to be kept.

``` {.haskell:ghci name="code"}
ghci> Map.fromListWith max [(2,3),(2,5),(2,100),(3,29),(3,22),(3,11),(4,22),(4,15)]
fromList [(2,100),(3,29),(4,22)]
```

Or we could choose to add together values on the same keys.

``` {.haskell:ghci name="code"}
ghci> Map.fromListWith (+) [(2,3),(2,5),(2,100),(3,29),(3,22),(3,11),(4,22),(4,15)]
fromList [(2,108),(3,62),(4,37)]
```

<span class="label function">insertWith</span> is to <span
class="fixed">insert</span> what <span class="fixed">fromListWith</span>
is to <span class="fixed">fromList</span>. It inserts a key-value pair
into a map, but if that map already contains the key, it uses the
function passed to it to determine what to do.

``` {.haskell:ghci name="code"}
ghci> Map.insertWith (+) 3 100 $ Map.fromList [(3,4),(5,103),(6,339)]
fromList [(3,104),(5,103),(6,339)]
```

These were just a few functions from <span
class="fixed">Data.Map</span>. You can see a complete list of functions
in the
[documentation](http://www.haskell.org/ghc/docs/latest/html/libraries/containers/Data-Map.html#v%3Aassocs).

Data.Set
--------

![legosets](http://s3.amazonaws.com/lyah/legosets.png)
The <span class="fixed">Data.Set</span> module offers us, well, sets.
Like sets from mathematics. Sets are kind of like a cross between lists
and maps. All the elements in a set are unique. And because they're
internally implemented with trees (much like maps in <span
class="fixed">Data.Map</span>), they're ordered. Checking for
membership, inserting, deleting, etc. is much faster than doing the same
thing with lists. The most common operation when dealing with sets are
inserting into a set, checking for membership and converting a set to a
list.

Because the names in <span class="fixed">Data.Set</span> clash with a
lot of <span class="fixed">Prelude</span> and <span
class="fixed">Data.List</span> names, we do a qualified import.

Put this import statement in a script:

``` {.haskell:ghci name="code"}
import qualified Data.Set as Set
```

And then load the script via GHCI.

Let's say we have two pieces of text. We want to find out which
characters were used in both of them.

``` {.haskell:ghci name="code"}
text1 = "I just had an anime dream. Anime... Reality... Are they so different?"
text2 = "The old man left his garbage can out and now his trash is all over my lawn!"
```

The <span class="label function">fromList</span> function works much
like you would expect. It takes a list and converts it into a set.

``` {.haskell:ghci name="code"}
ghci> let set1 = Set.fromList text1
ghci> let set2 = Set.fromList text2
ghci> set1
fromList " .?AIRadefhijlmnorstuy"
ghci> set2
fromList " !Tabcdefghilmnorstuvwy"
```

As you can see, the items are ordered and each element is unique. Now
let's use the <span class="label function">intersection</span> function
to see which elements they both share.

``` {.haskell:ghci name="code"}
ghci> Set.intersection set1 set2
fromList " adefhilmnorstuy"
```

We can use the <span class="label function">difference</span> function
to see which letters are in the first set but aren't in the second one
and vice versa.

``` {.haskell:ghci name="code"}
ghci> Set.difference set1 set2
fromList ".?AIRj"
ghci> Set.difference set2 set1
fromList "!Tbcgvw"
```

Or we can see all the unique letters used in both sentences by using
<span class="label function">union</span>.

``` {.haskell:ghci name="code"}
ghci> Set.union set1 set2
fromList " !.?AIRTabcdefghijlmnorstuvwy"
```

The <span class="label function">null</span>, <span
class="label function">size</span>, <span
class="label function">member</span>, <span
class="label function">empty</span>, <span
class="label function">singleton</span>, <span
class="label function">insert</span> and <span
class="label function">delete</span> functions all work like you'd
expect them to.

``` {.haskell:ghci name="code"}
ghci> Set.null Set.empty
True
ghci> Set.null $ Set.fromList [3,4,5,5,4,3]
False
ghci> Set.size $ Set.fromList [3,4,5,3,4,5]
3
ghci> Set.singleton 9
fromList [9]
ghci> Set.insert 4 $ Set.fromList [9,3,8,1]
fromList [1,3,4,8,9]
ghci> Set.insert 8 $ Set.fromList [5..10]
fromList [5,6,7,8,9,10]
ghci> Set.delete 4 $ Set.fromList [3,4,5,4,3,4,5]
fromList [3,5]
```

We can also check for subsets or proper subset. Set A is a subset of set
B if B contains all the elements that A does. Set A is a proper subset
of set B if B contains all the elements that A does but has more
elements.

``` {.haskell:ghci name="code"}
ghci> Set.fromList [2,3,4] `Set.isSubsetOf` Set.fromList [1,2,3,4,5]
True
ghci> Set.fromList [1,2,3,4,5] `Set.isSubsetOf` Set.fromList [1,2,3,4,5]
True
ghci> Set.fromList [1,2,3,4,5] `Set.isProperSubsetOf` Set.fromList [1,2,3,4,5]
False
ghci> Set.fromList [2,3,4,8] `Set.isSubsetOf` Set.fromList [1,2,3,4,5]
False
```

We can also <span class="label function">map</span> over sets and <span
class="label function">filter</span> them.

``` {.haskell:ghci name="code"}
ghci> Set.filter odd $ Set.fromList [3,4,5,6,7,2,3,4]
fromList [3,5,7]
ghci> Set.map (+1) $ Set.fromList [3,4,5,6,7,2,3,4]
fromList [3,4,5,6,7,8]
```

Sets are often used to weed a list of duplicates from a list by first
making it into a set with <span class="fixed">fromList</span> and then
converting it back to a list with <span
class="label function">toList</span>. The <span
class="fixed">Data.List</span> function <span class="fixed">nub</span>
already does that, but weeding out duplicates for large lists is much
faster if you cram them into a set and then convert them back to a list
than using <span class="fixed">nub</span>. But using <span
class="fixed">nub</span> only requires the type of the list's elements
to be part of the <span class="fixed">Eq</span> typeclass, whereas if
you want to cram elements into a set, the type of the list has to be in
<span class="fixed">Ord</span>.

``` {.haskell:ghci name="code"}
ghci> let setNub xs = Set.toList $ Set.fromList xs
ghci> setNub "HEY WHATS CRACKALACKIN"
" ACEHIKLNRSTWY"
ghci> nub "HEY WHATS CRACKALACKIN"
"HEY WATSCRKLIN"
```

<span class="fixed">setNub</span> is generally faster than <span
class="fixed">nub</span> on big lists but as you can see, <span
class="fixed">nub</span> preserves the ordering of the list's elements,
while <span class="fixed">setNub</span> does not.

Making our own modules
----------------------

![making modules](http://s3.amazonaws.com/lyah/making_modules.png)
We've looked at some cool modules so far, but how do we make our own
module? Almost every programming language enables you to split your code
up into several files and Haskell is no different. When making programs,
it's good practice to take functions and types that work towards a
similar purpose and put them in a module. That way, you can easily reuse
those functions in other programs by just importing your module.

Let's see how we can make our own modules by making a little module that
provides some functions for calculating the volume and area of a few
geometrical objects. We'll start by creating a file called <span
class="fixed">Geometry.hs</span>.

We say that a module *exports* functions. What that means is that when I
import a module, I can use the functions that it exports. It can define
functions that its functions call internally, but we can only see and
use the ones that it exports.

At the beginning of a module, we specify the module name. If we have a
file called <span class="fixed">Geometry.hs</span>, then we should name
our module <span class="fixed">Geometry</span>. Then, we specify the
functions that it exports and after that, we can start writing the
functions. So we'll start with this.

``` {.haskell:ghci name="code"}
module Geometry
( sphereVolume
, sphereArea
, cubeVolume
, cubeArea
, cuboidArea
, cuboidVolume
) where
```

As you can see, we'll be doing areas and volumes for spheres, cubes and
cuboids. Let's go ahead and define our functions then:

``` {.haskell:ghci name="code"}
module Geometry
( sphereVolume
, sphereArea
, cubeVolume
, cubeArea
, cuboidArea
, cuboidVolume
) where

sphereVolume :: Float -> Float
sphereVolume radius = (4.0 / 3.0) * pi * (radius ^ 3)

sphereArea :: Float -> Float
sphereArea radius = 4 * pi * (radius ^ 2)

cubeVolume :: Float -> Float
cubeVolume side = cuboidVolume side side side

cubeArea :: Float -> Float
cubeArea side = cuboidArea side side side

cuboidVolume :: Float -> Float -> Float -> Float
cuboidVolume a b c = rectangleArea a b * c

cuboidArea :: Float -> Float -> Float -> Float
cuboidArea a b c = rectangleArea a b * 2 + rectangleArea a c * 2 + rectangleArea c b * 2

rectangleArea :: Float -> Float -> Float
rectangleArea a b = a * b
```

Pretty standard geometry right here. There are a few things to take note
of though. Because a cube is only a special case of a cuboid, we defined
its area and volume by treating it as a cuboid whose sides are all of
the same length. We also defined a helper function called <span
class="fixed">rectangleArea</span>, which calculates a rectangle's area
based on the lenghts of its sides. It's rather trivial because it's just
multiplication. Notice that we used it in our functions in the module
(namely <span class="fixed">cuboidArea</span> and <span
class="fixed">cuboidVolume</span>) but we didn't export it! Because we
want our module to just present functions for dealing with three
dimensional objects, we used <span class="fixed">rectangleArea</span>
but we didn't export it.

When making a module, we usually export only those functions that act as
a sort of interface to our module so that the implementation is hidden.
If someone is using our <span class="fixed">Geometry</span> module, they
don't have to concern themselves with functions that we don't export. We
can decide to change those functions completely or delete them in a
newer version (we could delete <span class="fixed">rectangleArea</span>
and just use <span class="fixed">\*</span> instead) and no one will mind
because we weren't exporting them in the first place.

To use our module, we just do:

``` {.haskell:ghci name="code"}
import Geometry
```

<span class="fixed">Geometry.hs</span> has to be in the same folder that
the program that's importing it is in, though.

Modules can also be given a hierarchical structures. Each module can
have a number of sub-modules and they can have sub-modules of their own.
Let's section these functions off so that <span
class="fixed">Geometry</span> is a module that has three sub-modules,
one for each type of object.

First, we'll make a folder called <span class="fixed">Geometry</span>.
Mind the capital G. In it, we'll place three files: <span
class="fixed">Sphere.hs</span>, <span class="fixed">Cuboid.hs</span>,
and <span class="fixed">Cube.hs</span>. Here's what the files will
contain:

<span class="fixed">Sphere.hs</span>

``` {.haskell:ghci name="code"}
module Geometry.Sphere
( volume
, area
) where

volume :: Float -> Float
volume radius = (4.0 / 3.0) * pi * (radius ^ 3)

area :: Float -> Float
area radius = 4 * pi * (radius ^ 2)
```

<span class="fixed">Cuboid.hs</span>

``` {.haskell:ghci name="code"}
module Geometry.Cuboid
( volume
, area
) where

volume :: Float -> Float -> Float -> Float
volume a b c = rectangleArea a b * c

area :: Float -> Float -> Float -> Float
area a b c = rectangleArea a b * 2 + rectangleArea a c * 2 + rectangleArea c b * 2

rectangleArea :: Float -> Float -> Float
rectangleArea a b = a * b
```

<span class="fixed">Cube.hs</span>

``` {.haskell:ghci name="code"}
module Geometry.Cube
( volume
, area
) where

import qualified Geometry.Cuboid as Cuboid

volume :: Float -> Float
volume side = Cuboid.volume side side side

area :: Float -> Float
area side = Cuboid.area side side side
```

Alright! So first is <span class="fixed">Geometry.Sphere</span>. Notice
how we placed it in a folder called <span class="fixed">Geometry</span>
and then defined the module name as <span
class="fixed">Geometry.Sphere</span>. We did the same for the cuboid.
Also notice how in all three sub-modules, we defined functions with the
same names. We can do this because they're separate modules. We want to
use functions from <span class="fixed">Geometry.Cuboid</span> in <span
class="fixed">Geometry.Cube</span> but we can't just straight up do
<span class="fixed">import Geometry.Cuboid</span> because it exports
functions with the same names as <span
class="fixed">Geometry.Cube</span>. That's why we do a qualified import
and all is well.

So now if we're in a file that's on the same level as the <span
class="fixed">Geometry</span> folder, we can do, say:

``` {.haskell:ghci name="code"}
import Geometry.Sphere
```

And then we can call <span class="fixed">area</span> and <span
class="fixed">volume</span> and they'll give us the area and volume for
a sphere. And if we want to juggle two or more of these modules, we have
to do qualified imports because they export functions with the same
names. So we just do something like:

``` {.haskell:ghci name="code"}
import qualified Geometry.Sphere as Sphere
import qualified Geometry.Cuboid as Cuboid
import qualified Geometry.Cube as Cube
```

And then we can call <span class="fixed">Sphere.area</span>, <span
class="fixed">Sphere.volume</span>, <span
class="fixed">Cuboid.area</span>, etc. and each will calculate the area
or volume for their corresponding object.

The next time you find yourself writing a file that's really big and has
a lot of functions, try to see which functions serve some common purpose
and then see if you can put them in their own module. You'll be able to
just import your module the next time you're writing a program that
requires some of the same functionality.
