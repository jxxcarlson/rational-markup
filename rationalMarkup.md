# Rational Markup

This document is a first attempt at defining what we call *Rational Markup*.  It is to have the following characteristics:

1. The syntax is not burdensome as are xml and LaTeX

2. The syntax is not ambiguous and hard to parse, as is Markdown

3. There is only one way to do a given thing, e.g. sections, subsections etc.

In addition:

1. It compiles to Html, PDF, and LaTeX

2. It has scripting and macro facilities

3. It can be compiled interactively/on the fly by an IDE.

4. Amenities such as automatically generated section numbers, tables of contents, cross-references, etc. are carried out by manipulation of the AST.  

5. The parser is injective.  

## Syntax

Rational Markup has both block and inline elements.

### Blocks

In the discussion below, it is assumed for simplicity that leading white space is not significant.  An alternative is to make indentation significant, where indentation by a multiple of a fixed number of spaces indicates the *level* of a block, as in Markdown.  If a block of of level *n* is followed by blocks of level *n + 1*, then those blocks are children in the AST of the level *n* block.

Decision to make: is leading space significant, as it is in Markdown, Haskell, and Python.

Blocks are either *plain* or *marked*.  Marked blocks begin with the leading character |.  Some examples:

```
|section Intro

This is a sample Rational Markup document.

|subsection Stuff

This is a test.
Fee fie fo fum!

|q 
Four score and seven
years ago ...

|items Groceries
- apples
- potatoes
- stawberry jam

|math 
\int_0^1 x^n dx = \frac{1}{n+1}

|numbered To do
- Groceries
- Go to the gym
- Take out the trash

|env theorem 
There are infinitely many primes.

|code elm
type Term
    = T
    | F
    | Zero
    | Succ Term
    | Pred Term
    | IsZero Term
    | IfExpr Term Term Term
    
|verbatim
This is
  well, this is,
    like I said,
      well, this is
        like I said,
          Somethat recursive, ha ha!
          
|image AST
https://cdn-images-1.medium.com/max/1200/1*5wwVRWaTe7t_w0LhmLQAzQ.jpeg
```

A marked block begin with the character `|` in leading position, followed immediately by the *block name*, followed by zero or more arguments separated by spaces.  Thus, we could have

```
|q (Abraham Lincoln)
Four score and seven
years ago ...
``` 

In this case the text `(Abraham Lincoln)` is a positional argument
giving the author.  One can also have named arguments:

```
|q author: Abraham Lincoln, title: Gettysburg address
Four score and seven
years ago ...
```

Named arguments have the advantage that the are delimited by commas,
parentheses are unnecessary. In addition, one can extend the language without breaking changes if one uses named arguments.

The blocks in the example above are *ordinary*, meaning that they 
are terminated by one or more blank lines. Blocks can also be terminated
by the string `.end` in leading position:

```
|q+ 
Four score and seven
years ago ...

etc., etc.
.end
```	

Such blocks are called *extended* 

## Inline elements

Rational markup has a small number of frequently used inline elements with a special syntax, as well as a general syntax for the other inline elements.  Here are the special elements:

1. `*italic*`

2. `**bold**`

3. Backticks for inline code

4. `~~strikethough~~`

5. `$a^2 + b^2 = c^2$` — or maybe something like `($a^2 + b^2 = c^2$)` so as to eliminate begin-end ambiguities and to allow authors to say "this costs $10.

The general syntax (ATM) is as in the examples below. 

```
This is a |red warning|, and this is |font-size 48 Really Important|
```

This approach means that | is a reserved character and cannot be used in leading position, so those issues must be dealt with.  Note that the 
text `A | B` enclosed in backticks is OK.  The syntax for inline elements is much like that of blocks:

- The leading character `|`

- Immediately followed by the element name

- Followed by zero or more arguments

- Followed by content

- Terminated by the character `|`


## Parsing

Text is first parsed into blocks, obtaining a `T: Tree Block`. Then an inline parser is mapped over `T` to obtain a `T': Tree Markup`.  An immediate advantage of this strategy is that it facilitates incremental parsing.  Suppose that a change to the text is made that affects only the content of a certain block.  Then only that content needs to be parsed by the inline parser, and its result substituted in `T'`. Other changes, of course, require manipulation of `T'`, though in most cases only  the text corresponding to a proper subtree must be parsed.