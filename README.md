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

I've here tried to implement changes in `memoir` directly that add
tagging support while retaining the functionality of memoir, so do the
kind of thing that the package maintainer will eventually do.


`memtag-tagging.cls` is the [`memoir`
class](https://ctan.org/pkg/memoir) (version 3.8.4b 2025-11-04) with
minimal changes to improve tagging. The changes are marked by `#tag`
in comments.

`memoir-XX-BAD.tex` are test files from the [tagging project](https://github.com/latex3/tagging-project/tree/main/tagging-status/testfiles-incompatible/memoir)

Caveats:

- No attempt is made to discern if the class is loaded with tagging
  support on or off; it assumes it is on.
- TOC handling inserts tagging sockets
- `memoir` by default does not link the page numbers. However, the
  socket `contentsline/page/after` is the one that closes a `TOCI`
  tag corresponding to a contents line; without it the tags are
  unbalanced.
- `\chapter` uses the heading template `chapter`. This breaks the
  configurability of chapter headings in `memoir`. The template uses
  the memoir configuration commands as much as I could figure (e.g.,
  use `\chaptitlefont`) but more sophisticated chapter formats will
  probably need their own templates.
- Changes haven't been made for `\book` and `\part`.
- `memoir`'s `\tableofcontents` directly formats a chapter heading for
  the TOC, and doesn't call `\chapter*` to do this. I don't know why
  but also I don't know of a way to get the "Table of Contents" tagged
  properly otherwise.
- Getting an error I can't get rid of:
  ```
  ! Undefined control sequence.
  <argument> \ERROR 
                  \cs_set_eq:NN \__fnote_tmp:w \exp_stop_f: 
  l.24 \begin{document}
  ```

## Sources

- TOC code: required/latex-lab/latex-lab-toc-kernel-changes.dtx
- Sectioning: https://ctan.mirror.globo.tech/macros/latex-dev/required/latex-lab/latex-lab-sec-template.pdf

