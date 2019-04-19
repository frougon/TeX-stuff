# mmxref — Gather and list back-references for target labels

## Introduction

I've written the `mmxref` package to answer [this
question](https://tex.stackexchange.com/questions/484666/list-where-ref-have-been-made/)
on [TeX.StackExchange.com](https://tex.stackexchange.com/) and practice a bit
with the programming facilities offered by
[LaTeX3](https://www.latex-project.org/latex3/). I don't consider this a
serious package, in that I didn't take the time to design a great API that
I'll be willing to support for years to come—I don't have myself any use for
the package, apart from practicing LaTeX3. But I found the problem interesting
enough to write this code, and did take the time to ensure that what it does
(the logic) is done correctly—as far as I know!

So, what is it all about? In LaTeX, you can make references to other places in
a document by means of labels, these other places being typically sectional
units, figures, tables or equations. What `mmxref` allows one to do is to call
a command that inserts a sentence listing all “places” (part, chapter,
section, etc.) that point, by means of `\mmxref`—or a variant such as
`\mmxautoref`—to a given target label declared with `\mmxlabel`. In other
words, `mmxref` can generate sentences that list all back-references to the
label of your choice.

## Background

Let's step back a little bit and discuss what can be done with references
declared with a command such as standard LaTeX's `\label`,
[zref](https://ctan.org/pkg/zref)'s `\zlabel` or `mmxref`'s `\mmxlabel`. In
order to point to a label somewhere in your document, standard LaTeX
documentation tells you to write something like `see section~\ref{the section
label}`, and LaTeX automatically replaces `\ref{the section label}` with the
appropriate section number. You can also use `\pageref{the section label}` to
insert the printed number of the page where `\label{the section label}`
happens to be located on, and this is pretty much all that standard LaTeX
offers. However, with the `zref` package by Heiko Oberdiek, you can attach
basically anything you want to labels. Let's quickly introduce two packages
that improve on LaTeX's standard label/reference system:

- Among others, the [hyperref](https://ctan.org/pkg/hyperref) package allows
  `\ref` and other commands to create hyperlinks in the output PDF file. It
  also avoids the need to hardcode the nature of the referred-to “place”,
  i.e., you can write `see~\autoref{the section label}` without hardcoding the
  fact that the pointed-to place is a section: `hyperref` knows, because the
  label points to the closest place before the `\label` where
  `\refstepcounter{some-counter}` was called, and the name of the counter
  (`part`, `chapter`, `section`, `equation`, `theorem`...) is in general
  enough to know how to name that “place”. This is particularly interesting
  with theorems and propositions, because there is no hard rule saying which
  statements are theorems and which are propositions, so you could well call
  something a theorem and later change your mind, or the other way around.
  Having to update all places that state `theorem~\ref{some label}` to read
  `proposition~\ref{some label}` would be tedious and error-prone (sometimes,
  the words are more separated than that...); however, this can be avoided
  when the data written to the `.aux` file by a command such as `\label`
  contains enough information to deduce the kind of “place” it refers to.

- The [zref](https://ctan.org/pkg/zref) package has many modules for users, a
  nice interface for programmers and allows one to attach pretty much any kind
  of information to a label. This way, one can have the target section,
  chapter, theorem or whatever name automatically included in references; one
  can adapt automatic wording generated for a reference based on the
  underlying counter; one can access the absolute page number where a given
  `\zlabel` appears; with [pdfTeX](https://www.tug.org/applications/pdftex/)
  and the `zref`'s `savepos` module, one can access the precise position of a
  `\zlabel` on page, which allows one to compute the distance between two
  elements on a given page, etc.

`mmxref` uses [zref](https://ctan.org/pkg/zref) to manage its references.

## Quick Instructions

You can declare a target label with `\mmxlabel{refname}` and point to it with
`\mmxref{refname}` or `\mmxautoref{refname}` (the latter outputs text such as
“section 2.1” instead of just “2.1”).<sup>a</sup> `\mmxpageref{refname}`
outputs the printed page number for the place where `\mmxlabel{refname}` has
been put.

Since `mmxref` may need to refer to labeled places it hasn't seen yet in your
document, it uses the `.aux` file in the usual way: what is typeset in a given
LaTeX run is based on the contents of the `.aux` file written by the
*previous* LaTeX run. This implies that several compilation runs are needed in
general for things to stabilize. Most of the times, two are sufficient. Don't
worry about that: `mmxref` prints a warning on the terminal and in the log
file when more compilations are needed.

By default, all references managed by `mmxref` are automatically prefixed with
`mmxref-`. This avoids clashes between, e.g., `\label{foobar}` and
`\mmxlabel{foobar}`, because the latter actually defines the label
`mmxref-foobar` (this can be seen in the `.aux` file). You can easily change
this prefix with `\mmxSetLabelPrefix{prefix to use}` in the preamble. For
instance, to remove this prefixing altogether, the following would do:

```latex
\mmxSetLabelPrefix{}
```

When you place `\mmxlabel{refname}` somewhere in your LaTeX document, the
created label is tied to counter `somecounter`, defined by the closest place
before this `\mmxlabel` command where `\refstepcounter{somecounter}` was
called. The default representation for a reference to this label is the one
obtained with `\thesomecounter`—this is what you get from `\mmxref{refname}`.
As said, `\mmxpageref{refname}` outputs the printed page number (as obtained
from `\thepage` at the time of page `\shipout`) and `\mmxautoref{refname}`
uses `\mmxFormatSimpleRef{somecounter}` to map the name `somecounter` to a
word (or more, since you can freely redefine it) such as “section”, “chapter”,
etc., that is appropriate for `somecounter`, in order to produce a reference
text such as “section 1.2”.

Now the back-references. Let's take an example:

```latex
\documentclass{report}
\usepackage{mmxref}

\begin{document}
% Of course, A, A.1, A.2 are for the example; you wouldn't include something
% like this in a real document, especially in a label!

\chapter{Chapter A}
\mmxlabel{label for chap. A}

\section{Section A.1}
\mmxlabel{label for sec. A.1}

See~\mmxautoref{label for sec. A.2}.

\section{Section A.2}
\mmxlabel{label for sec. A.2}

Some text (...)

\mmxInsertBackReferences{label for sec. A.2}
\end{document}
```

In this example, the `\mmxInsertBackReferences{label for sec. A.2}` command
will insert a sentence that lists all back-references for `label for sec.
A.2`; in this precise case, this means only Section A.1, as it is the only
place in the document that points to `label for sec. A.2`. How exactly does
the `\mmxautoref{label for sec. A.2}` decide that its “source” is Section A.1?
This happens because `\mmxlabel{label for sec. A.1}` is the closest call to
`\mmxlabel` before the `\mmxautoref{label for sec. A.2}` that generated the
back-reference. That is why the back-reference generated here for `label for
sec. A.2` points to Section A.1.

Now, you may express the point of view that Section A.1 is *inside* Chapter A,
and might want the back-reference to point to chapter A instead of
Section A.1. This can be done by indicating the desired label for the
back-reference in the optional argument of `\mmxref` or `\mmxautoref`. In the
example, you would replace `\mmxautoref{label for sec. A.2}` with
`\mmxautoref[label for chap. A]{label for sec. A.2}`.

Note: `\mmxInsertBackReferences` also accepts an optional argument that can be
used to change the start of the automatically-generated sentence (see
`simple.tex` in the the `examples` directory).

## Summary of the available commands

The user commands provided by `mmxref.sty` are:

- `\mmxlabel`, `\mmxref`, `\mmxpageref`, `\mmxautoref`,
  `\mmxInsertBackReferences`, `\mmxSetLabelPrefix`: see above.

- `\mmxSeparatorBetweenTwo`, `\mmxSeparatorBetweenMoreThanTwo`,
  `\mmxSeparatorBetweenFinalTwo`, `\mmxFormatStartOfInsertionPhrase`,
  `\mmxFormatStartOfInsertionPhraseForLabel`, `\mmxFormatSimpleRef`,
  `\mmxFormatSimpleRefForLabel`, `\mmxFormatBackRef`,
  `\mmxFormatBackRefForLabel`: all these are hooks for customizing the output
  text. Given one or several strings as arguments, each of these commands
  produces something that is automatically inserted by some other command from
  the `mmxref` package.

- `\mmxSetup`: not useful for now, as `mmxref` doesn't accept any option.

For all things that aren't documented here (in particular most commands from
the second item of the preceding list), please refer to `mmxref.sty`. Most
user-oriented commands (those defined with `\NewDocumentCommand`) are preceded
by a comment indicating the role played by each parameter. Also, be sure to
have a look in the `examples` subdirectory!

### License

This package is free software released under the LaTeX Project Public License
version 1.3 or (at your option) any later version. See `mmxref.sty` for the
precise licensing information.

### Thanks

Special thanks go to Enrico Gregorio, whose code kindly offered in [this
answer](https://tex.stackexchange.com/a/484806/73317) at
[TeX.StackExchange.com](https://tex.stackexchange.com/) provided the starting
point—and an important one—of this package.

--------------

**Footnotes**

a. The strings used by `\mmxautoref` to name sectional units can be
customized: see the `with-translated-strings.tex` example that translates them
in French (specifically, see how `\mmxFormatSimpleRef` is redefined). By
default, when referring to a subsection or subsubsection, the string used will
be “section”, because I think that writing things like “see subsection 1.2” is
ugly and the precision it brings as compared to “see section 1.2” is quite
unimportant. This behavior can be changed by simple redefinition of
`\mmxFormatSimpleRef` and friends, but it's quite intentional and definitely
not a bug. The same goes for subsubsections and subparagraphs.
