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
someone hoping to get memoir to work before it's actually adapted to
deal with tagging. Whether and when that will happen is unclear,
since `memoir` is very complex and redefines a lot of LaTeX internals.

I've here tried to implement changes in `memoir` directly that add
some tagging support while retaining the functionality of memoir. This
is a stop-gap only to allow me to keep using `memoir`'s many layout
features and still produce tagged PDFs. In the absence of a compatible
official `memoir` class the long-term solution is probably to replace
`memoir` with a compatible class and use compatible packages to do the
work `memoir` now does (see below for ideas). But who knows, maybe
this is also the start of something. Feel free to propose fixes for
`memoir`'s other incompatibilities or improvements to mine.

`memoir-tagging.cls` is the [`memoir`
class](https://ctan.org/pkg/memoir) (version 3.8.4b 2025-11-04) with
minimal changes to improve tagging. The changes are marked by `#tag`
in comments.

## Changes and caveats:

- The (new) commands `\@memfirstaid` and `\@memnofirstaid` check if
  tagging is on. If it is, issue a warning and supress the code in its
  second argument (which otherwise is executed). In the case of
  `\@memfirstaid` it will try to load replacement code
  (memfirstaid-#1.sty). This can then be used while a solution is being
  tried out (avoiding having to redefine code memoir already defines),
  to avoid loading code for which a suitable replacement is available
  (e.g., a package that memoir emulates but where the package is
  compatible but not memoir's older code), or if a fallback to LaTeX's
  own definition provides at least some compatibility and avoids errors.
- The code for `\part` and `\chapter` is suppressed in this way.
  `memfirstaid-headings.tex` contains experimental code that makes use
  of the new [heading
  templates](https://ctan.org/tex-archive/macros/latex-dev/required/latex-lab/latex-lab-sec-template.pdf).
  The replacement code edits the templates the plain LaTeX code uses
  instead of `memoir`'s own definitions. The edits try to mimic
  the effect of `memoir`'s configuration commands as much as I could
  figure (e.g., use `\chaptitlefont`) so that chapter styles etc.
  still more or less work. More sophisticated chapter formats will
  probably need their own templates. **This requires the most recent
  versions of the heading templates, so `lualatex-dev`.**
- The code for `\book` is unchanged. Since `\book` is not a standard
  LaTeX command, no default command, templates, or tagging code are
  available to be edited; it would have to be done from scratch. This
  is done for `\part` and `\chapter` in
  [latex-lab-sec-template](https://github.com/latex3/latex2e/blob/develop/required/latex-lab/latex-lab-sec-template.dtx),
  which could be used as a model.
- The code for other sectioning commands is not touched: `memoir` uses
  `\@startsec` which should take care of tagging for those.
- `memoir`'s footnotes are incompatible with tagging. We make sure
  that the code from
  [latex-lab-footnotes](https://github.com/latex3/latex2e/blob/develop/required/latex-lab/latex-lab-footnotes.dtx)
  is available: we supress the redefinition in `memoir` and explicitly
  set them to the tagging-aware code. (`latex-lab-footnotes` tries to
  fix `memoir` too but the fix produces errors when `hyperref` is
  loaded.) With footnote code reverted to LaTeX's own, none of
  `memoir`'s footnote configuration options are available. (I believe
  much of that is copied from the `footins` package, and by removing
  that code and/or "disemulating" `footins` in `memoir` one could get
  that functionality back.)
- The same goes for `\title`, `\author`, `\date`, `\maketitle` and
  `\@maketitle`.
- `memoir` redefines `\@makecol` in LaTeX's output routine. We avoid
  this by supressing the definition of `\feetabovefloat` and
  `\feetbelowfloat`, and the call to `\feetabovefloat`.
- `memoir`'s table of contents code is directly edited in the source
  to include calls to the tagging sockets the same way that LaTeX's
  own ToC code in
  [latex-lab-toc-kernel-changes](https://github.com/latex3/latex2e/blob/develop/required/latex-lab/latex-lab-toc-kernel-changes.dtx)
  and
  [`tocloft`](https://github.com/LaTeX-Package-Repositories/herries-press/blob/main/tocloft/tocloft.dtx)
  do. (This includes the code for `book` even if there is no
  replacement for the `\book` command yet.)
- `memoir`'s `\tableofcontents` doesn't call `\chapter*` to produce a
  heading for the ToC. The way it does means that the entire ToC is
  not properly tagged. I don't know why but also I don't know of a way
  to get the "Table of Contents" heading tagged properly other than
  by replacing that part with a call to `\chapter*`. Without fixing
  this in the definition of `\newlistof`, this has to be done by
  redefining `\@Zmaketitle`, e.g.,
  ```
  \makeatletter
  \renewcommand{\@tocmaketitle}{\chapter*{\contentsname}}
  \makeatother
  ```
- `memoir` makes a change to `\@addamp` of the `array` package, which
  it loads. I honestly don't know what that change accomplishes but
  I've had it result in extra (empty, 0-width) table cells in some
  tables. These tables then had some rows with more cells than others,
  and that breaks PDF/UA-2.
- Since the class has been renamed, it won't load `memhfixc.sty` which
  makes `hyperref` work with `memoir`. E.g., it adds "book" to the
  `autoref` list of `hyperref` and also fixes some other links
  including footnotes.

`memoir-XX-BAD.tex` are (edited) test files from the [tagging project test file
collection](https://github.com/latex3/tagging-project/tree/main/tagging-status/testfiles-incompatible/memoir)

## Alternatives to `memoir` and its features

You may want to consider alternatives to `memoir` if you use `memoir`
for things that are not easily fixable to work with tagging and are unlikey to be fixed
soon. Here is a very incomplete list:

- The easiest and most reliable alternative is the stock `book` class.
- For setting page dimensions and layout, the `geometry` package can
  do a lot of the same things that `memoir`'s options can. It is not
  as convenient as `memoir` and doesn't provide as many presets (paper
  sizes, layouts). It uses different terminology than `memoir`'s
  (which uses traditional printer's and typesetter's terminology):
  e.g., `geometry`'s paper = `memoir`'s trim, layout/body = page,
  inner = spine, outer = edge, etc. However, `memoir` types out the
  layout dimensions on every run (on the console only, not in the log)
  and you could just use what `memoir` tells you there for `geometry`.
- `memoir`'s many footnote configuration methods are also provided by
  the [`footmisc`](https://ctan.org/pkg/footmisc) package, which is
  fully compatible with tagging.
- Changing the chapter and other heading styles can be done by editing
  LaTeX's "heading templates" as done in
  [`memfirstaid-headings.sty`](./memfirstaid-headings.sty). There is,
  as far as I know, no compatible alternative package that provides
  the functionalty of `memoir`'s `\makechapterstyle`.
- Changing the layout of the table of contents and adding other "List
  of ..." lists can be done with the (compatible)
  [`tocloft`](https://ctan.org/pkg/tocloft) package.
- Changing page styles (including heads and foots) or defining new
  page styles can be done with the (compatible)
  [`fancyhdr`](https://ctan.org/pkg/fancyhdr) package.
- `memoir` provides lots of hooks in various of its commands; these
  can largely be replaced by [LaTeX's own hook
  management](https://www.latex-project.org/help/documentation/lthooks-doc.pdf).
  E.g., instead of
  ```
  \renewcommand{\memendofchapterhook}{code}
  ```
  you can say
  ```
  \AddToHook{cmd/after/chapter}{code}
  ```
