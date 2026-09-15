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
LaTeX internals, and the current maintainer is unable to invest the
necessary effort.)

I've here tried to implement changes in `memoir` directly that add
some tagging support while retaining the functionality of memoir. This
is a stop-gap only to allow me to keep using `memoir`'s many layout
features and still produced tagged PDFs. In the absence of a
compatible official `memoir` class the long-term solution is probably
to replace `memoir` with a compatible class and use compatible
packages to do the work `memoir` now does (for me).

`memoir-tagging.cls` is the [`memoir`
class](https://ctan.org/pkg/memoir) (version 3.8.4b 2025-11-04) with
minimal changes to improve tagging. The changes are marked by `#tag`
in comments.

## Changes and caveats:

- The code for `\part` and `\chapter` has been replaced with code that
  makes use of the new [heading
  templates](https://mirrors.ctan.org/macros/latex/required/latex-lab/latex-lab-sec-template.pdf).
  The replacement code edits the templates the plain LaTeX code uses
  and deactivates `memoir`'s own definitions. The edits try to mimic
  the effect of `memoir`'s configuration commands as much as I could
  figure (e.g., use `\chaptitlefont`) so that chapter styles etc.
  still more or less work. More sophisticated chapter formats will
  probably need their own templates.
- The code for `\book` is unchanged. Since `\book` is not a standard
  LaTeX command, no default command, templates, or tagging code are
  available to be edited; it would have to be done from scratch. This
  is done for `\part` and `\chapter` in [latex-lab-sec-template](https://github.com/latex3/latex2e/blob/develop/required/latex-lab/latex-lab-sec-template.dtx).
- Footnotes are not fixed, even though `memoir`'s footnotes are
  incompatible with tagging. Just like
  [latex-lab-footnotes](https://github.com/latex3/latex2e/blob/develop/required/latex-lab/latex-lab-footnotes.dtx),
  the `\footnote` code is circumvented by restoring the standard
  definition at the end of the class. (This is done by the code itself
  since the patch from `latex-lab-footnotes` doesn't apply to a
  renamed class.)
- `memoir`'s table of contents code is edited to include calls to the
  tagging sockets the same way that LaTeX's own ToC code in
  [latex-lab-toc-kernel-changes](https://github.com/latex3/latex2e/blob/develop/required/latex-lab/latex-lab-toc-kernel-changes.dtx)
  does.
- `memoir`'s `\tableofcontents` doesn't call `\chapter*` to produce a
  heading for the ToC. The way it does means that the entire ToC is
  not properly tagges/ I don't know why but also I don't know of a way
  to get the "Table of Contents" heading tagged properly other than
  by replacing that part with a call to `\chapter*`.
- `memoir` makes a change to `\@addamp` of the `array` package, which
  it loads. I honestly don't know what that change accomplishes but
  I've had it result in extra (empty, 0-width) table cells in some
  tables. These tables then had some rows with more cells than others,
  and that breaks PDF/UA-2.
- No attempt is made to discern if the class is loaded with tagging
  support on or off; it assumes it is on.
- `memoir` by default does not link the page numbers. However, the
  socket `contentsline/page/after` is the one that closes a `TOCI`
  tag corresponding to a contents line; without it the tags are
  unbalanced.
- `memoir`'s `\tableofcontents` directly formats a chapter heading for
  the TOC, 

`memoir-XX-BAD.tex` are test files from the [tagging project test file
collection](https://github.com/latex3/tagging-project/tree/main/tagging-status/testfiles-incompatible/memoir)

