# memoir-tagging

The `memoir` class is currently
[incompatible](https://github.com/latex3/tagging-project/issues/910)
with the [tagging
functionality](https://latex3.github.io/tagging-project/) being
implemented in LaTeX to produce PDF-UA accessible PDFs. There is one
[example
document](https://github.com/latex3/tagging-project/tree/main/project-examples/ASV)
provided by the LaTeX tagging project that uses `memoir` and includes
band-aids to make the document compile and pass validation on
[VeraPDF](https://dev.verapdf-rest.duallab.com/) and the [LaTeX
project tag tool](https://texlive.net/showtags).

The problem with this bible example is that it does too much (e.g.,
define tagged bible book and verse sections) and too little (e.g.,
"fixes" the chapter commands in such a way that they only work for
that example, e.g., force everything to be two column and make
chapters unnumbered/starred). So it's not a good example to use for
someone hoping to get memoir to work before it's actually fixed. (This
will take time since `memoir` is very complex and redefines a lot of
LaTeX internals.)

I started with the bible example and tried to distill the changes
needed. It's very minimal; basically right now all it does is it
changes the TOC, part, and chapter commands. It produces a PDF that
validates, but has issues, e.g., sections in the TOC aren't
hyperlinked.

`memtag-20.tex` is the main file.

