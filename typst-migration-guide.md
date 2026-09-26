# Typst for Academic Researchers, migration Guide from LaTeX and Word

Typst is a markup-based typesetting system designed for producing documents from plain-text source files. It is particularly relevant to researchers who work with mathematical notation, references, structured documents, and version-controlled source files.

Typst uses `.typ` source files and compiles them to PDF and other supported output formats. Unlike a conventional word processor, the document's structure and formatting are represented directly in the source. Unlike LaTeX, Typst combines markup, styling, and scripting in a single language rather than relying on a large collection of macros and external packages.

The practical question for a researcher is not whether Typst is technically capable of replacing LaTeX or Word. It is whether the features used in a particular workflow are available and whether the migration effort is justified.

This guide covers the parts of that decision that are most likely to matter when moving an academic document to Typst.

> **Version note:** This guide reflects Typst 0.15.x-era documentation reviewed in September 2026. Typst is actively developed, so commands, package interfaces, and available templates may change. Check the current Typst documentation before using installation commands or relying on version-specific behavior.

## Installing Typst

Typst can be used through its web application or installed locally as a command-line compiler. A local installation is useful when documents are stored in Git, built from scripts, or edited in a local development environment.

### Windows

The simplest installation method on Windows is Windows Package Manager (`winget`):

```text
winget install --id Typst.Typst
```

After installation, open a new PowerShell or Command Prompt window and check the installed version:

```text
typst --version
```

The official Typst installation instructions also provide prebuilt Windows binaries. If you use that method, extract the archive, place the executable in a suitable directory, and add that directory to your `PATH`.

If you install the binary manually, `typst update` can be used to update the compiler. If you installed through `winget`, use `winget` to manage updates instead.

### macOS

Homebrew provides a straightforward installation:

```text
brew install typst
```

Then verify the installation:

```text
typst --version
```

Homebrew normally places the executable on your `PATH`. To update it later:

```text
brew upgrade typst
```

### Linux

Linux distributions provide Typst through different package managers, and packaged versions may lag behind the current release. Check the current Typst installation documentation or a package index such as Repology for your distribution.

If you already use Rust and Cargo, the Typst CLI can also be installed with:

```text
cargo install --locked typst-cli
```

The Typst project also provides prebuilt binaries for supported platforms.

### Verify the installation

Regardless of operating system, start by checking that the compiler is available:

```text
typst --version
```

A version number confirms that the command can be found.

You can compile a document directly:

```text
typst compile main.typ
```

Or ask Typst to watch the source file and rebuild it when it changes:

```text
typst watch main.typ
```

Watch mode is useful when writing because the PDF is regenerated as the source changes. Typst uses incremental compilation, so subsequent builds can avoid work that does not need to be repeated.

## Starting a project

Typst projects can be created manually or from templates published through Typst Universe.

For example:

```text
typst init @preview/charged-ieee
```

creates a project from the specified template.

The exact templates available through Typst Universe change over time, so search the current registry before choosing one for a journal submission.

For local editing, Typst works with several editors through community integrations. Visual Studio Code, for example, can be used with Typst language and preview extensions. The Typst web application is another option when you do not want to install the compiler locally.

## Typst syntax for LaTeX and Word users

A useful way to understand Typst is to look at the three systems side by side.

LaTeX represents structure and formatting primarily through commands such as `\section{}` and `\textbf{}`. Word represents much of the same structure through its graphical interface and styles. Typst uses lightweight markup for common operations and functions for more advanced behavior.

Typst has three syntax modes:

* **Markup mode** is the normal document-writing mode.
* **Math mode** is enclosed in `$...$`.
* **Code mode** is entered with `#` in markup and provides access to functions, variables, and control flow.

For example:

| Task          | LaTeX                   | Word                 | Typst            |
| ------------- | ----------------------- | -------------------- | ---------------- |
| Heading       | `\section{Title}`       | Heading 1 style      | `= Title`        |
| Subheading    | `\subsection{Title}`    | Heading 2 style      | `== Title`       |
| Bold          | `\textbf{word}`         | Bold formatting      | `*word*`         |
| Italic        | `\textit{word}`         | Italic formatting    | `_word_`         |
| Bullet list   | `itemize` environment   | Bulleted list        | `- item`         |
| Numbered list | `enumerate` environment | Numbered list        | `+ item`         |
| Inline math   | `$x^2$`                 | Equation editor      | `$x^2$`          |
| Function call | `\command{arg}`         | No direct equivalent | `#function(arg)` |

Additional heading levels use additional equals signs:

```text
= Main section
== Subsection
=== Subsection level 3
```

The lightweight syntax is only the surface of the language. Many elements can also be constructed directly with functions, which becomes useful when a document needs behavior that the markup shortcut does not provide.

## Controlling document-wide formatting

One of the more important differences for LaTeX users is Typst's styling system.

A **set rule** changes the default parameters of an element for the following part of the document. A **show rule** can change how an element is rendered.

For example:

```text
#set page(numbering: "1")
#set heading(numbering: "1.")
#show heading: set text(navy)
```

These rules number pages, number headings, and change the text color of headings.

Set and show rules normally apply from the point where they occur. This makes their position in the source significant.

A show rule can also target a subset of elements. For example:

```text
#show figure.where(kind: table):
  set figure.caption(position: top)
```

This changes the caption position for tables without changing image captions.

For larger or repeated customizations, put the rules into a template or shared module rather than repeating them throughout individual documents.

## Mathematics

Mathematical notation is one of the areas where a Typst migration can make a substantial difference for researchers coming from Word.

Typst uses `$...$` for mathematics. Whitespace immediately inside the delimiters makes the equation a block-level equation:

```text
Inline: $x^2 + y^2 = z^2$

Block:

$ x^2 + y^2 = z^2 $
```

The same math mode is used for both cases; the distinction is made by the whitespace.

Typst provides dedicated syntax and functions for many mathematical structures. For example:

```text
$ T(n) = O(2^n) $

$ vec(1, 2, 3) $

$ mat(1, 2; 3, 4) $

$ cal(A) subset.eq cal(B) $
```

Equations can be numbered globally:

```text
#set math.equation(numbering: "(1)")
```

An equation can then be labeled and referenced:

```text
#set math.equation(numbering: "(1)")

We define:

$ phi.alt := (1 + sqrt(5)) / 2 $ <ratio>

The resulting value is given by @ratio.
```

Typst can also make long block equations break across pages when necessary:

```text
#show math.equation:
  set block(breakable: true)
```

Researchers moving from LaTeX should still check specialized mathematical requirements individually. A familiar LaTeX package does not necessarily have a one-to-one Typst equivalent.

## Figures, tables, and cross-references

Typst uses labels and references to connect parts of a document.

A figure can be created with an image, caption, and label:

```text
#figure(
  image("results.png", width: 80%),
  caption: [Accuracy across five training runs.],
) <results>

As shown in @results, accuracy improves after the third epoch.
```

If the figure moves because earlier content changes, its number and references are updated automatically.

The same labeling mechanism can be used for headings and equations.

Tables can also be placed inside a figure so that they receive captions and can be referenced:

```text
#figure(
  table(
    columns: 3,
    [*Model*], [*Accuracy*], [*Latency*],
    [Baseline], [82%], [12 ms],
    [Ours], [91%], [9 ms],
  ),
  caption: [Comparison against the baseline model.],
) <comparison-table>

The results are summarized in @comparison-table.
```

References can be customized when the default wording is not appropriate. For example:

```text
@intro[Chapter]
```

can supply an explicit supplement, while `ref` also supports other forms such as page references.

For side-by-side figures or other structured layouts, Typst's layout functions can be combined rather than relying on a specialized package for every arrangement.

## Citations and bibliographies

Typst can use either its native Hayagriva bibliography format or BibLaTeX `.bib` files. This makes it possible to bring an existing BibLaTeX bibliography into a Typst project without converting the file first.

A bibliography is declared with:

```text
#bibliography("references.bib", style: "ieee")
```

Citations use the `@` syntax:

```text
Previous work has reported similar results @example2025.
```

Only works cited in the document normally appear in the bibliography.

Typst includes many built-in citation styles and can also use custom CSL styles. Examples include:

```text
#bibliography("references.bib", style: "apa")
```

and:

```text
#bibliography("references.bib", style: "ieee")
```

If a journal requires a particular citation format, check the actual output rather than assuming that a style with a familiar name will satisfy every journal-specific requirement.

This is especially important for numeric citations. Whether consecutive citations are collapsed into a range, how author names are rendered, and other details depend on the citation style being used.

Researchers who prefer a native Typst bibliography can use Hayagriva's YAML format instead:

```text
#bibliography("references.yml")
```

The choice between `.bib` and Hayagriva is therefore mostly a workflow decision rather than a requirement to rebuild an existing reference library.

## Scripting and reusable content

Typst is more than a markup language. Its scripting system can be used to generate repeated content and create reusable document components.

Variables are defined with `let`:

```text
#let sample-size = 42
#let significance = 0.05

The study included #sample-size participants,
with a significance level of #significance.
```

Typst identifiers can contain letters, numbers, hyphens, and underscores, although they must begin with a letter or underscore.

Functions are also defined with `let`:

```text
#let highlight-result(value, threshold: 0.05) = {
  if value < threshold {
    text(fill: red, weight: "bold")[#value]
  } else {
    value
  }
}

The p-value was #highlight-result(0.031).
```

Conditional expressions and loops can generate repeated structures:

```text
#for result in results [
  [#result.name]
  [#result.value]
]
```

This becomes useful when the same information has to appear in several places or when tables and other repeated elements are generated from structured data.

The distinction between content and code is important when writing more advanced Typst. Square brackets contain markup content, while curly braces contain code blocks:

```text
#let title = [My research paper]

#let make-title() = {
  title
}
```

For researchers who have never worked with programming concepts, this part of Typst is optional. A conventional paper does not require extensive scripting.

## Organizing larger projects

A long document does not need to live in a single `.typ` file.

Typst provides `include` and `import` for working with multiple files.

`include` evaluates another file and inserts its content:

```text
#include "chapter1.typ"
#include "chapter2.typ"
```

`import` loads definitions such as functions and variables from another module:

```text
#import "definitions.typ": *
```

A thesis might therefore be organized like this:

```text
thesis/
├── main.typ
├── definitions.typ
├── introduction.typ
├── methods.typ
├── results.typ
├── discussion.typ
└── references.bib
```

Keep shared formatting rules and reusable functions in one place rather than copying them into every chapter. This makes later changes easier and keeps the individual chapter files focused on content.

It also works well with version control: different collaborators can work on separate source files without all changes being concentrated in one large document.

## Journal templates

For researchers, journal compatibility is often more important than the basic Typst syntax.

LaTeX journals commonly provide `.cls` and `.sty` files. Typst uses packages and templates instead. Community templates are available through Typst Universe and are imported with a package declaration such as:

```text
#import "@preview/charged-ieee:0.1.0": ieee
```

A template can then be applied to the document:

```text
#show: ieee.with(
  title: [Your paper title],
  authors: (
    (name: "Author Name", organization: "Institution"),
  ),
)
```

Package versions should be treated as part of the document configuration. A package import identifies a namespace, package name, and version, so changing the version can change the package interface or output.

For that reason, if a manuscript depends on a community template, record the version used for the manuscript and check compatibility before upgrading it.

If no suitable template exists for the target journal, the required layout can be implemented directly with Typst's styling system. For a recurring workflow, those rules can then be turned into a reusable template or package.

Local packages are also possible, but they are intended for local environments and should not be confused with packages published through Typst Universe.

## Migration process

Do not start by converting a 200-page thesis.

Start with a small document and identify what the existing manuscript actually depends on.

### Check the target journal first

Before converting anything, look for an existing Typst template for the journal or publisher.

If no suitable template exists, determine how much of the journal's required formatting would have to be recreated manually.

A technically successful conversion is not useful if the resulting manuscript cannot meet the submission requirements.

### Check the bibliography

If the existing manuscript uses BibTeX or BibLaTeX, try the existing `.bib` file in Typst before converting the bibliography.

Then inspect the entries that matter to the manuscript. Unusual entry types or journal-specific bibliography requirements may require additional work.

### Audit LaTeX dependencies

List the packages used by the original document.

Do not assume that every package has a direct Typst equivalent. This matters particularly for specialized areas such as chemistry, circuit diagrams, domain-specific notation, and highly customized layouts.

Search Typst Universe for an equivalent package where appropriate, and test it with representative material from the manuscript.

### Test references and figures

Before converting the entire document, migrate a section containing:

* figures
* tables
* equations
* cross-references
* citations

Compile it and check the resulting PDF.

This catches problems with labels, bibliography entries, specialized notation, and layout before they spread through a larger project.

### Separate content from formatting

For a thesis or multi-author project, put shared page and heading rules in a common file and keep individual chapters focused on their content.

This also makes the source easier to maintain under version control.

### Test collaboration

A migration affects the people who edit the manuscript, not just the person who compiles it.

If collaborators rely heavily on Word's tracked changes or another review workflow, decide how those comments and revisions will be handled before moving the whole project.

Typst supports collaborative editing through its web application, but the workflow is different from Word's tracked-changes model. A team that depends on a particular review process should test that process rather than assuming the tools are interchangeable.

### Convert a small document first

A conference abstract, short article, or thesis section is a useful pilot.

The purpose is not merely to learn the syntax. It gives you a chance to check the complete workflow:

```text
source → compilation → references → figures → PDF → review
```

If the pilot exposes a missing package, incompatible template, or collaboration problem, you can address it before committing the full manuscript to the new toolchain.

## Where Typst may not be the right fit

Typst's ecosystem is younger than LaTeX's, so a migration should not be treated as a universal replacement.

Specialized fields may depend on mature LaTeX packages for notation, diagrams, or highly specific document layouts. Community templates also vary in quality and coverage.

A particularly customized document may therefore require more work in Typst than a conventional journal article.

There is also no general one-click conversion that turns an arbitrary `.tex` project into a finished `.typ` project. Conversion tools can help with common constructs, but custom macros, package-dependent behavior, complex tables, citations, and specialized environments still need review.

Collaboration requirements can also influence the decision. If co-authors need to exchange editable Word documents, Typst's PDF-oriented workflow may require an additional export or review step.

These are not necessarily reasons to avoid Typst. They are reasons to test the parts of the workflow that matter to your particular project.

## Migration steps

Before moving an in-progress manuscript, check:

- A suitable Typst template exists for the target journal, or the formatting requirements can be reproduced.
- The existing bibliography has been tested in Typst.
- Important LaTeX packages have been identified and their replacements checked where necessary.
- Figures and tables compile correctly.
- Equation numbering and cross-references work as expected.
- The required citation style produces the journal's expected output.
- Shared formatting rules have been separated from chapter or section content.
- Co-authors have a workable editing and review process.
- A small document has been converted successfully before the full manuscript.
- The final PDF has been checked against the journal's submission requirements.

## Sources and further reading

The primary sources for this guide are the official Typst documentation, including its documentation for LaTeX users, syntax, scripting, mathematics, bibliographies, references, styling, packages, and command-line installation.

Because Typst and its package ecosystem are actively developed, consult the current official documentation and the relevant journal template before applying the procedures in this guide to a production manuscript.

**Documentation status:** Research-based guide reviewed against Typst 0.15.x-era documentation in September 2026. The procedures were not independently tested by the author unless explicitly stated.
