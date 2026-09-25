# aurea

`aurea` is a LaTeX package that holds the house style of four sibling
projects: `ce-Speranza-e-speranza`, `ecce-homo`,
`mia-vita-vicino-a-padre-pio` and `vocazione-san-matteo`. It gives a classical
book look on the standard `article` class. The name comes from the *sezione
aurea* (the golden section), which sets the proportions of the page.

A new project needs only this:

```latex
\documentclass[12pt,a4paper]{article}
\usepackage{aurea}
\aureasetup{title=..., subtitle=..., author=..., date=...}
```

With no options, the output matches the hand-written preambles it replaces.
I checked this by rebuilding all four projects on `aurea` with the preambles
shown under [Migrating the four projects](#migrating-the-four-projects) and
comparing every page as a raster image (pdfLaTeX). For v1.2 the check was
made again against the current versions of the four projects (the ones that
open with title page, abstract page, text), with the front matter written
with `sommario`: all 347 pages came out identical, the extracted text is the
same, and each build logged the same Overfull, Underfull and other warnings
as its original. The optional body rewrites in that section are marked with
whether they were checked the same way.

## Typographic rationale

| Choice | Why |
|---|---|
| 12pt `article` on A4 | The class is left as it is. Chapters are `\section`, and there is no book machinery. |
| **Cochineal** with old-style figures (`osf`) | A book face based on Crimson. Old-style figures sit well in running text, in dates such as 1904 and in verse numbers. |
| `\linespread{1.05}` | Cochineal has a small x-height, so it needs a little extra leading. |
| **Golden-ratio text block**: 110 mm × 178 mm, top margin 45.5 mm | The 110 mm measure holds about 70 characters per line at 12pt. The block is φ times as tall as it is wide (110 × 1.618 = 178). On A4 portrait it is centred horizontally, with 50 mm on each side, and the top and bottom margins are in the ratio 1 : φ (297 − 178 = 119 = 45.5 + 73.5). The numbers are fixed, so they are right only for A4 portrait: on letter paper the margins are no longer 1 : φ, and with `landscape` the bottom margin goes negative. For another paper, give the block again with the `geometry` option. |
| `microtype` | Protrusion and expansion. It is why the long documents have no overfull lines. |
| First-line indent, no `parskip` | This was chosen on purpose; `parskip` was tried and dropped. |
| Golden rule `\scenebreak` | A centred rule 0.618 of the line wide and 0.618 pt thick. It divides a chapter without adding a sub-heading. |
| No page number on the title page | Added in every project, and now automatic. |
| `hyperref` with `hidelinks` | Links work but show no colour and no boxes. Source URLs go in footnotes as `\url`. |
| Table of contents at the back (`\indice`) | This is the Italian book convention. |
| `babel` Italian and `csquotes` | Hyphenation and the Italian names ("Sommario", "Indice", "Figura"). `csquotes` is there for `\enquote` in new documents; the house style itself types «…» and “…” directly (see `\detto`). |

## Installation

Pick one:

- **Per project**: copy `aurea.sty` next to `main.tex`.
- **Once per machine**: copy it to `$(kpsewhich -var-value TEXMFHOME)/tex/latex/aurea/aurea.sty`, usually `~/Library/texmf/tex/latex/aurea/` on macOS or `~/texmf/tex/latex/aurea/` on Linux.
- **Overleaf**: upload `aurea.sty` to the project root.

`aurea` needs LaTeX 2022-06-01 or later, for `\DeclareKeys`, hooks and
`\NewDocumentEnvironment`. TeX Live 2022 and later qualify, and so does
Overleaf.

pdfLaTeX is the reference engine: only pdfLaTeX gives output identical to
the originals. LuaLaTeX and XeLaTeX also work. Under those two engines
`fontenc` and `inputenc` are skipped, Cochineal loads as OpenType through
fontspec, and the `tt` URL font is Latin Modern Mono instead of cm-super. The
OpenType metrics and microtype's different behaviour there move some line
breaks (in vocazione, a few footnote marks move to another line), so do not
switch engines on a document whose page breaks matter.

## Package options

Package options are key=value pairs and apply only at load time.

| Key | Default | Effect |
|---|---|---|
| `language=<list>` | `italian` | Options passed to babel; the last language is the main one, e.g. `language={latin,italian}`. If babel was loaded earlier it is left alone. |
| `figures=osf\|lining` | `osf` | Cochineal figure style. Any other value is an error. |
| `nonumbers` | off | Unnumbered headings, for titles that already carry their ordinal or theme (ecce-homo, mia-vita). In `article` this is `\setcounter{secnumdepth}{0}`; in classes with `\chapter` (book, report, scrbook) it is `-1`, so chapters lose their number too. `\part` keeps its number. |
| `geometry={...}` | empty | Extra geometry keys, appended after the golden block, so a key given here wins over the default of the same name, e.g. `geometry={top=40mm}`. If you add a key the block does not set, such as `bottom`, geometry reports over-specification and ignores the height. |
| `urlstyle=tt\|rm\|sf\|same` | `tt` | The `\url` font. `tt` is what the originals print (cm-super under pdfLaTeX, Latin Modern Mono under LuaLaTeX and XeLaTeX); `same` matches Cochineal. Any other value is an error. |
| `draft` | off | DRAFT watermark: 45°, 4 cm, `black!15`, exactly as in ecce-homo. Figures, links and PDF metadata are not affected. This is the way to get a watermark. |
| `watermark=<text>` | | Watermark with your own text, e.g. `watermark=BOZZA`. `watermark` alone prints DRAFT. |
| `final` | | Turns the watermark off, even with the class option `draft`. |

Unknown keys are an error. Keys used after loading are an error too: for
example, `\aureasetup{figures=lining}` raises "may only be used during
loading".

**The class option `draft`.** `\documentclass[draft]{article}` also turns
the watermark on, because aurea sees class options. (This is how ecce-homo
keeps its watermark.) But the class option reaches every package, not only
aurea:

- graphicx prints each `\includegraphics` as an empty frame with the file name in it, and no image goes into the PDF;
- hyperref goes into draft mode: no links, no bookmarks, and no PDF Info dictionary, so the title, author, subject and keywords set with `\aureasetup` are dropped.

`final` in aurea's options does not undo this. For a watermark alone, use
`\usepackage[draft]{aurea}` (or `watermark=...`) and leave the class option
out. Remove the class option `draft` without adding the package option and
the watermark disappears too.

### Load order and clashes

`aurea` loads, in this order:

1. `iftex`
2. `fontenc[T1]` and `inputenc[utf8]` (pdfTeX only)
3. `babel`
4. `csquotes`
5. `cochineal`
6. `microtype`
7. `graphicx`
8. `geometry`
9. `hyperref`
10. `draftwatermark` (only when a watermark is on)

- **Packages that must come before hyperref** go before `\usepackage{aurea}`. Your own macros go after it.
- **Do not load babel, geometry, hyperref, cochineal or csquotes again with options after aurea.** That is an "Option clash" error. Use the aurea options (`language=`, `geometry=`, `figures=`), `\hypersetup{...}` or `\geometry{...}` after aurea, or load the package yourself before aurea (see below).
- **Link and colour options** go in `\hypersetup{...}` after `\usepackage{aurea}`, e.g. `\hypersetup{colorlinks}`. `\PassOptionsToPackage{...}{hyperref}` before aurea does not work for any key that `hidelinks` sets (such as `colorlinks` or `pdfborder`), because aurea's `hidelinks` is processed last and wins; use it only for options that do not conflict.
- **Page-layout keys** go in the `geometry={...}` option or in `\geometry{...}` after aurea. `\PassOptionsToPackage{textwidth=...}{geometry}` is overridden by the golden block in the same way.
- **If babel, cochineal, geometry or hyperref is already loaded**, aurea does not load it again, so there is no option clash:
  - babel and cochineal are left as they are.
  - A geometry loaded earlier keeps its own layout. Only the `geometry={...}` keys are applied, and the log says the golden block was skipped.
  - A hyperref loaded earlier only gets `\hypersetup{hidelinks}`.

## Commands and environments

### Title page and metadata

| Command | Effect |
|---|---|
| `\aureasetup{title=, subtitle=, author=, authornote=, date=, subject=, keywords=}` | Sets `\title`, `\author` and `\date`, and the PDF fields `pdftitle`, `pdfauthor`, `pdfsubject` and `pdfkeywords`, in one place (preamble only). The PDF title and author leave out the subtitle and the author note; `\and` in the author becomes a comma there and `\thanks` is dropped. **Put values that contain commas in braces**, e.g. `authornote = {Parrocchia S. Lorenzo da Brindisi, Francavilla Fontana}`: unbraced, the part after the comma is read as an unknown key, which is an error. |
| `\subtitle{...}` | Printed under the title, `\large`, 0.4 em lower. This replaces the hand-typed `\title{X \\[0.4em] \large Y}`. A class with its own `\subtitle` (KOMA-Script) keeps it, and `subtitle=` then calls it. |
| `\authornote{...}` | Printed under the author, `\small`: a venue, a qualification or an occasion. This replaces `\author{X \\ \small Y}`. A class with its own `\authornote` (acmart, apa7) keeps it; the `authornote=` key still works. |
| `\maketitle` | Unchanged, except that the title page now gets `\thispagestyle{empty}` automatically. A leftover explicit `\thispagestyle{empty}` does no harm. With the `titlepage` class option nothing is added. |
| `\begin{sommario} ... \end{sommario}` | The abstract on a page of its own: `\clearpage`, the class's `abstract` (babel-italian titles it "Sommario"), `\clearpage`. Same output as the hand-typed sequence. The page keeps its number (plain style), as in the four projects; for no number, put `\thispagestyle{empty}` inside. The closing `\clearpage` does nothing when the text begins with one of its own (ce-Speranza's `\puntata`). In classes without `abstract` (book, scrbook) it prints what article's abstract prints: `\small`, a centred bold `\abstractname` ("Abstract" if the language has none), then `quotation`. In classes that have one, their own look applies: report and article with `titlepage` put it on an empty-style title page; scrartcl prints no heading unless you give its class option `abstract=true`. |
| `\indice` | `\clearpage\tableofcontents`: the "Indice" on a page of its own at the end. |

Plain `\title`, `\author` and `\date` still work, but they leave the PDF
Title and Author empty (as the originals without `\hypersetup` did). With
`\aureasetup` the pages are the same and the PDF fields are filled in.

### Text

| Command | Effect |
|---|---|
| `\scenebreak` | The golden rule. Put it between paragraphs. |
| `\sectionbreak` | Alias of `\scenebreak`, kept for the existing sources. See the note below. |
| `\begin{citazione}[source] ... \end{citazione}` | A display quotation: `quote` in italics. The optional source is set on its own line, flush right, upright, `\footnotesize`, in parentheses, as in the ce-Speranza epigraph. Type the text without `\emph`, because inside italics `\emph` (and so `\detto`) turns upright. |
| `\citazionefonte{...}` | Formats that source line. The house is not consistent here: ce-Speranza also has a normal-size `— Colossesi 3,1-4` line, and vocazione puts the reference inline as `\emph{(Mt 9,9)}`. For the dash form, `\renewcommand\citazionefonte[1]{\normalfont\upshape\hfill --- #1}`. |
| `\detto{...}` | Words spoken or quoted in running text: italics inside «…». The marks are typed, not made by csquotes, so it gives exactly the output of the hand-typed `\emph{«…»}` (same kerning, same spacing after `?»`, same margin protrusion at the start of a quote). A `\detto` nested inside another stays italic and uses “…”, as the sources type inner quotes. |
| `\rif{Gv 18,1}` | A Bible-reference footnote for a literal quotation: "Gv 18,1." (the final period is added). |
| `\cfr{Sal 30,2}` | A Bible-reference footnote for an allusion: "Cfr. Sal 30,2." |
| `\firma{...}` | A signature under a letter, as the sources type it: a paragraph of its own in italics, with the normal indent (none inside `quote`). Same output as `\emph{Padre Pio, cappuccino}` on its own paragraph. It stays italic inside `citazione`. |

**Block quotations.** Most sources type a display quotation as `quote` with
an `\emph` on each paragraph (265 blocks across ecce-homo, mia-vita and
vocazione). Keep that form for any block that mixes roman commentary into
the quotation (`\emph{Se ho parlato male} — usa gli stessi termini —
\emph{…}`): inside `citazione` the commentary would turn italic and the
`\emph` parts upright. `citazione` suits blocks that are entirely quoted. The
conversion of existing `quote` blocks to `citazione` was not checked for
identical output, so treat it as a change to review.

**Why `\scenebreak` and not `\sectionbreak`.** When `\sectionbreak` is
defined, titlesec runs it before every `\section`. The rule is therefore
defined as `\scenebreak`, and the `\sectionbreak` alias is only created at
`\begin{document}`, and only if titlesec is not loaded and the document has
not defined `\sectionbreak` itself. So titlesec's own
`\newcommand{\sectionbreak}{...}` works whether it comes before or after
aurea. With titlesec, a `\sectionbreak` left in the text stops with
"Undefined control sequence", and the log says to use `\scenebreak`.

## Minimal template

`template.tex` shows every option and command. Here is the shortest useful
document:

```latex
\documentclass[12pt,a4paper]{article}
\usepackage[nonumbers]{aurea}
\aureasetup{
  title      = Ecce Homo,
  subtitle   = Il processo di Gesù nel Vangelo di Giovanni,
  author     = don Fabio Rosini,
  authornote = {Parrocchia S. Lorenzo da Brindisi, Francavilla Fontana},
  date       = 9--10 marzo 2017,
}
\begin{document}
\maketitle
\begin{sommario} ... \end{sommario}
\section{...}
Testo, \detto{parole dette}\rif{Gv 18,1} ...
\scenebreak
...
\indice
\end{document}
```

## Migrating the four projects

Each original preamble becomes `\documentclass[...]{article}`, then
`\usepackage[...]{aurea}`, then the per-document lines. The four preambles
below are the ones that were built and compared page by page with the
originals. All four give identical pages.

These lines, common to all four, are replaced by `\usepackage{aurea}`:

- `fontenc`
- `inputenc`
- `babel`
- `csquotes` (not in ce-Speranza)
- `cochineal[osf]`
- `\linespread{1.05}`
- `microtype`
- `graphicx` (not in ce-Speranza)
- `geometry[...]` and its comment
- `hyperref[hidelinks]`

All four bodies now open the same way (ce-Speranza has an epigraph between
the title and the first `\clearpage`, and no `\clearpage` after the
abstract, because its `\puntata` starts with one):

```latex
\maketitle                  % was: \maketitle
                            %      \thispagestyle{empty}   (now automatic)
                            %      \clearpage
\begin{sommario}            %      \begin{abstract}
...                         %      ...
\end{sommario}              %      \end{abstract}
                            %      \clearpage
```

So in every body:

- the `\thispagestyle{empty}` after `\maketitle` can be deleted;
- `\clearpage` `\begin{abstract}` ... `\end{abstract}` `\clearpage` becomes `\begin{sommario}` ... `\end{sommario}` (in ce-Speranza the closing `\clearpage` is the one inside `\puntata`, which stays);
- `\clearpage` + `\tableofcontents` at the end becomes `\indice` (ecce-homo has no table of contents).

Each preamble and body below was built on its own and compared with the
current original (checked for v1.2: 53, 69, 198 and 27 pages, all
identical). The same preambles with the hand-typed front matter left as it is
are identical too.

### ce-Speranza-e-speranza

```latex
\documentclass[12pt,a4paper]{article}
\usepackage{aurea}

%% Intestazione di ogni puntata ... (\newcommand{\puntata}[5]{...} unchanged)

\title{C'è Speranza e speranza}
\subtitle{Un viaggio di demistificazione nella virtù teologale della speranza}
\author{don Fabio Rosini}
\date{Dicembre 2024 -- Gennaio 2025}
```

- **Removed**: the common block; `\newenvironment{citazione}`; the hand-typed subtitle inside `\title`.
- **Body**, front matter:

  ```latex
  \maketitle

  \begin{citazione}[Rm 4,18]
  Egli ebbe fede sperando contro ogni speranza e così divenne padre di molti
  popoli, come gli era stato detto: Così sarà la tua discendenza.
  \end{citazione}

  \begin{sommario}
  Cinque catechesi radiofoniche di don Fabio Rosini, ...
  \end{sommario}

  \puntata%
  ...
  \indice
  ```

  The epigraph's `\begin{citazione}[Rm 4,18]` replaces the hand-typed `\normalfont\upshape\footnotesize\hfill (Rm 4,18)` line; `sommario` replaces `\clearpage` + `abstract`. Identical output.
- **Leave alone**: the second hand-typed source, `\upshape\normalfont\hfill --- Colossesi 3,1-4` (line 476 of the original). It is normal size, with a dash and no parentheses; turning it into `\begin{citazione}[Colossesi 3,1-4]` would restyle it as the small parenthesised form. Do that only as a deliberate normalisation.
- **Stays in the document**: `\puntata` (tied to Vatican News), including the `\clearpage` it now begins with, so that each episode starts a new page.
- **Notes**: aurea also loads csquotes and graphicx, which this document did not; neither changes the output. With `\aureasetup{title=..., subtitle=..., author=..., date=...}` instead of the plain commands, the pages are identical and the PDF Title and Author are filled in (the original left them empty).

### ecce-homo

```latex
\documentclass[12pt,a4paper,draft]{article}

\usepackage[nonumbers]{aurea}

% bible_refs:
%  Gen 3, Gen 32, ... (unchanged)
\aureasetup{
  title      = Ecce Homo,
  subtitle   = Il processo di Gesù nel Vangelo di Giovanni,
  author     = don Fabio Rosini,
  authornote = {Parrocchia S. Lorenzo da Brindisi, Francavilla Fontana},
  date       = 9--10 marzo 2017,
  subject    = {Catechesi di don Fabio Rosini sul processo di Gesù ... Francavilla Fontana.},
  keywords   = {Fabio Rosini, Vangelo di Giovanni, passione, ... remissione dei peccati},
}
```

- **Removed**: the common block; `draftwatermark` and `\DraftwatermarkOptions`; `\setcounter{secnumdepth}{0}` (now `nonumbers`); the unused `\newcommand{\sectionbreak}`; `\hypersetup{pdftitle, pdfauthor, pdfsubject, pdfkeywords}`; `\title`, `\author` and `\date` with the hand-typed subtitle and venue. `authornote`, `subject` and `keywords` contain commas, so they are braced.
- **Body**, front matter: `\maketitle`, then `\begin{sommario} Nella Settimana biblica ... \end{sommario}`, then the text. There is no table of contents, so no `\indice`.
- **The watermark**: the original loaded draftwatermark unconditionally. Here the class option `draft` drives it, which gives identical pages but also keeps hyperref in draft mode, so the PDF has no links and no metadata, exactly like the original. For the watermark *with* working links and the `\aureasetup` metadata, use `\documentclass[12pt,a4paper]{article}` and `\usepackage[nonumbers,draft]{aurea}` (checked: identical pages, and PDF Title, Author, Subject and Keywords are set). Removing the class option without adding the package option `draft` removes the watermark.
- **Optional body rewrites**, both checked, identical output (69/69 pages, v1.1):
  - `\footnote{Gv 18,1.}` → `\rif{Gv 18,1}` and `\footnote{Cfr. Sal 30,2.}` → `\cfr{Sal 30,2}` (150 footnotes);
  - `\emph{«…»}` → `\detto{…}` where the `»` closes the `\emph` (165 uses). Leave `\emph{«…».}`, with the period inside the italics, as it is: `\detto{…}.` would set the period upright.
- **Stays in the document**: the `% bible_refs:` comment, the body, and the `quote` + `\emph` blocks that mix in commentary.

### mia-vita-vicino-a-padre-pio

```latex
\documentclass[12pt,a4paper]{article}
\usepackage[nonumbers]{aurea}

% bible_refs:
%   Gen 3, Gen 22, ... (unchanged)
\aureasetup{
  title      = La mia vita vicino a Padre Pio,
  subtitle   = Appunti spirituali,
  author     = Cleonice Morcaldi,
  authornote = figlia spirituale di Padre Pio da Pietrelcina,
  date       = 23 settembre 2026,
  subject    = {Appunti spirituali di Cleonice Morcaldi, figlia spirituale di Padre Pio da Pietrelcina},
  keywords   = {Padre Pio, direzione spirituale, ... morte di Padre Pio},
}
```

- **Removed**: the common block; `\setcounter{secnumdepth}{0}`; `\newcommand{\sectionbreak}` (the body keeps using `\sectionbreak` through the alias); `\hypersetup{...}`; `\title`, `\author` and `\date` with the hand-typed subtitle and descriptor. PDF metadata is identical to the original's.
- **Body**: `\maketitle`, `\begin{sommario} Cleonice Morcaldi, nata il 22 gennaio 1904, ... \end{sommario}`, `\section{Introduzione}` ...; at the end `\indice`.
- **Optional body rewrites**, each checked on the v1.1 version, identical output (197/197 pages):
  - `\footnote{Cfr. …}` → `\cfr{…}` and literal references → `\rif{…}` (205 footnotes);
  - `\emph{«…»}` → `\detto{…}` where the `»` closes the `\emph` (417 uses);
  - `\emph{Padre Pio, cappuccino}` as the last paragraph of a letter → `\firma{Padre Pio, cappuccino}`.
- **Stays in the document**: `% bible_refs:`; the `\textbf{D.}` / `\textbf{Padre Pio}` dialogue labels; the chapter 13 and 14 heading subtitles.

### vocazione-san-matteo

```latex
\documentclass[12pt,a4paper]{article}

\usepackage{aurea}

\aureasetup{
  title    = Lo guardò con sentimento di amore e lo scelse,
  subtitle = Ufficio delle letture e Vangelo,
  date     = 21 settembre 2026,
}
\author{san Matteo, apostolo ed evangelista}
```

- **Removed**: the common block; `\newcommand{\sectionbreak}`; `\title{X \\[0.4em] \large Y}`; `\date`.
- **Body**: `\maketitle`, `\begin{sommario} Nella festa di san Matteo, ... \end{sommario}`, then the Caravaggio figure page (with its own `\clearpage` after it), the text, and `\indice` at the end.
- **Notes**: `title=` also sets the PDF Title, which the original left empty. The author can move into `\aureasetup` as well, braced because of the comma: `author = {san Matteo, apostolo ed evangelista}` (checked: identical pages, and the PDF Author is set). Unbraced, ` apostolo ed evangelista` is read as an unknown key.
- **Stays in the document**: the full-page Caravaggio figure (it overflows by 28 pt, "Float too large", as in the original; see below); the reading and talk source lines; the `\emph{Ioana Cristina Huban}` signature (or `\firma{Ioana Cristina Huban}`, same output).

## What is deliberately not in the package

Each of these appears in only one document. Promoting any of them would fix a
convention the other projects do not share.

- **`\puntata`** (ce-Speranza). Its "Vatican News" label, date line and Bible-reference line belong to that series of broadcasts.
- **Q&A speaker labels** such as `\textbf{D.} — …` (mia-vita only).
- **The liturgical reading header** (`\emph{Dalla lettera ...}` then a rule) and the centred speaker line with a URL footnote (vocazione only).
- **A full-page artwork macro** (vocazione only). Note that the original overflows by 28 pt ("Float too large"). When copying it, budget room for the caption, e.g. `height=0.8\textheight`, as `template.tex` does. Changing it in vocazione would change that page.
- **`\ndr`, a `\latin`/`\foreign` macro, `\titolo`, and `\bibref` with an en dash for verse ranges.** Either one document uses them, or adopting them would change the existing output (the sources use a hyphen in verse ranges).
- **The `% bible_refs:` metadata comment.** It is per-document data, not style.
- **A page break before each `\section`.** Only ce-Speranza starts each episode on a new page (inside `\puntata`); the other three run their sections on.
- **A different style for the abstract page.** It keeps its page number in all four projects, so `sommario` does too.

## Changelog

- **v1.2 (2026/09/24)**: new `sommario` environment for the abstract page that all four projects now have after the title page (`\clearpage`, `abstract`, `\clearpage`), with a fallback for classes without `abstract` (book, scrbook). `template.tex` and the migration notes use it. Checked against the current four projects: 347/347 pages identical.
- **v1.1 (2026/09/24)**: the house style as a package: options, `\aureasetup`, `\subtitle`, `\authornote`, automatic empty title page, `\indice`, `\scenebreak`/`\sectionbreak`, `citazione`, `\detto`, `\rif`, `\cfr`, `\firma`.
