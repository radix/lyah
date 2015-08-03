# LYAH

This is a fork of Learn You A Haskell For Great Good, which is licensed under
[CC-BY-NC-SA 3.0](http://creativecommons.org/licenses/by-nc-sa/3.0/)
by [Miran Lipovača](http://www.oreilly.com/pub/au/5027).

I've converted the HTML to markdown with Pandoc since I couldn't find any
source code for the original book.

# Why I'm doing this

LYAH isn't perfect, but it's got a fairly good/cute approach to learning. The
problem is, it's full of pretty offensive crap that can easily turn people off
of learning Haskell, and I personally want Haskell to grow a large and diverse
development community, and that requires being welcoming and friendly to
everyone.

Why not just write a new book from scratch? Well, I'm not qualified. And there
are other people who are. So why not just wait for them to finish, and start
advocating those new, better books? Because LYAH has a very entrenched position
in the Haskell community, and even if better books come along, it will probably
still be referenced a lot. So I would like to provide pretty much the same
experience as LYAH minus all the stupid crap. Maybe if this comes along enough
it will be able to effectively replace LYAH.

# What's so bad about LYAH?

Some of the stuff in LYAH is actually hurtful (like the BMI calculator example,
which literally insults people based on their weight. Seriously. What the
hell.), some is just stupid ("BOOBIES" on the calculator), and some just subtly
reinforces the non-diverse status quo (gendered language/assumptions). The goal
is to get this to be something that most people would actually be fine giving
to their small children, since that's the aesthetic (if not the content) of the
book.

I'm not going to justify every single change that's made in this fork. And if
you feel that something should be changed, please submit a PR and I'm very
likely to happily merge it, unless it makes the content _less_ respectful.

# Can I help?

Yes, please! I really don't have time to work on this project, but I got mad
enough that I started it. I have a TODO list below; please feel free to submit
a PR for anything you find!

# TODO:

- review for sexist/racist/demeaning stuff
- get rid of all sexist/racist/demeaning stuff
- get rid of the remainder of HTML cruft in the markdown files
- automate building to an HTML site
- automate building a *pretty* HTML site
- publish somewhere
- choose a new name so it's identifiably different

# Non-exhaustive list of stuff to replace

- bmi example (syntax-in-functions)
- "gayballs" (higher-order-functions)
- guys
- man
- girlfriend (input-and-output)
- "BOOBIES" calculator (functionally-solving-problems)
- "Writer? I hardly know her!"
- gang example (for-a-few-monads-more), along with pictures of villains, and gun stuff.

In addition to these things, it'd be great to be more proactive about diversity
and respect. Replacing or adding some pictures representing more diverse people
in a non-stereotyping way would be great.
