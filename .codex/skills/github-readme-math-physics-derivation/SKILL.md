---
name: github-readme-math-physics-derivation
description: Use when writing or rewriting mathematical and physical derivations that must be rigorous, readable, and render correctly in GitHub README or Markdown files. Produces consistent note structure with conclusion, problem definition, derivation highlights, assumptions, step-by-step derivation, physical meaning, and final result.
---

# GitHub README Math And Physics Derivation

Use this skill for derivation notes that must satisfy all of the following:

- mathematically rigorous
- physically interpretable
- readable as a technical note
- stable in GitHub README / Markdown rendering
- consistent in structure across multiple files

Highest-priority governing principle:

- For every major step, the output signal must be written as a fully expanded closed form.
- This is the top-level rule and overrides brevity, elegance, compactness, and shorthand notation.
- Do not omit the expanded signal just because the expression becomes long.
- Do not stop at operator form, schematic form, or verbal description when the resulting signal can be written explicitly.
- For end-to-end flow documents such as `overall.md`, this rule applies to every stage signal in the chain, from the first input signal to the final output image.
- In an overall flow note, never write only
  `S_{i+1} = T[S_i]`
  and move on.
  You must also write the explicit closed form of `S_{i+1}`.

Primary formatting reference:

- When there is any doubt about layout, section rhythm, equation spacing, or how much text should appear between equations, use
  `../../../derive/azimuth_freq_folding.md`
  as the canonical output template.
- Prefer that file's practical rendering style over abstract formatting elegance.
- If a prettier layout risks GitHub rendering instability, choose the `azimuth_freq_folding.md` style.

## Mandatory Pre-Derivation Workflow

Before writing the derivation itself, run the following two checks conceptually:

### 1. Modular Decomposition via `architecture` or `brainstorming`

If the original problem is large, layered, or mixes geometry, signal processing, and implementation meaning:

- use `architecture` or `brainstorming` first to break the big problem into small modules
- do not start the derivation as one monolithic proof
- explicitly identify the module boundaries before drafting

Preferred module shapes for this note set are:

- geometry
- signal model
- operator or transform
- approximation
- per-replica closed form
- total-signal closed form
- physical meaning
- implementation correspondence

If the derivation feels too large to explain cleanly, that is a sign the module split is still wrong.

### 2. Consistency Audit via `analyze-project`

Before finalizing the note, inspect the listed physical assumptions and derivation steps using the `analyze-project` mindset.

You must actively look for:

- logical jumps where a new symbol or coefficient appears before being justified
- approximation mismatch, such as phase being expanded but envelope support still treated as exact without comment
- physical-meaning contradiction, where the prose claims one phenomenon is caused by A but the equations show it comes from B
- operator-order contradiction, where the prose says "LPF works because of X" but the actual chain does not yet make X true
- hidden scope expansion, where a derivation silently answers a different question than the one stated in `問題定義`
- unstated conditions, where a step is only valid under a narrow band, small-angle, or single-replica assumption but that condition is not written

If any of these appear, fix the derivation before presenting it as final.

### 3. Markdown Math Edit Safety

When editing an existing derivation note that already contains many display equations:

- do not use broad regex replacement or bulk search-and-replace across the whole file if the pattern can touch `$...$`, `$$...$$`, or code fences
- do not modify prose and math delimiters in the same bulk operation
- if prose cleanup is needed, explicitly avoid display-math blocks and code fences
- if equation cleanup is needed, edit equation boundaries locally and deliberately
- treat Markdown math delimiters as structural syntax, not ordinary text

Practical rule:

- never run a file-wide replacement intended for inline math unless you have first proven it cannot affect display math
- if that proof is not trivial, do not use the replacement

## Output Contract

Default section order:

1. `結論`
2. `問題定義`
3. `推導重點`
4. `符號與假設`
5. `逐步推導`
6. `物理意義`
7. `最終結果`
8. `實作對應`
9. `限制與適用範圍`

For shorter notes, you may omit `實作對應` or `限制與適用範圍`.

## Notebook Output Mode

If the target artifact is a Jupyter notebook rather than a Markdown note:

- prefer `.ipynb` as the primary output artifact
- keep the same mathematical rigor and fully expanded closed-form rule
- convert long narrative sections into Markdown cells
- place executable or reference code in Code cells
- do not merge a full stage into one giant cell if separate Markdown and Code cells make the notebook easier to read

Preferred notebook cell rhythm for figure-driven derivation notes:

1. title and navigation Markdown cell
2. summary and problem-definition Markdown cell
3. one or two setup Code cells for imports, parameters, shared helpers, and base signals
4. one symbols-and-assumptions Markdown cell
5. repeated stage pairs in this order:
   - Code cell that performs only the newly introduced transform for that stage and outputs one or more matching subimages
   - Markdown cell that explains the same stage in natural prose with the matching math and physical meaning
6. final-result Markdown cell

Notebook-specific rendering rules:

- use standard Markdown image syntax or simple HTML image tags with repository-relative paths
- keep equations in Markdown cells using `$$ ... $$`
- do not place explanatory prose inside Code cells
- prefer executable Code cells when the notebook is meant to be read on GitHub
- the default teaching-style notebook is sequential rather than independently rerunnable per stage
- do not duplicate full setup logic inside every stage cell just to make the stage independently runnable
- when the notebook is intended for local reading, optimize for scroll readability rather than README portability
- if the notebook is intended to render correctly on GitHub, prefer executed Code cell outputs for plots instead of relying only on local relative image paths in Markdown cells
- when GitHub rendering matters, execute the notebook before sign-off so the visual outputs are embedded in the `.ipynb`
- for dark visual style on GitHub, style the figures themselves with a dark plotting theme; do not assume the notebook viewer can be forced into a black background
- for compact teaching notebooks, prefer output image metadata with width `500` when the default notebook rendering makes the plots too large
- when possible, keep figure style consistent across stages for the same physical quantity, axis meaning, and visual viewpoint
- if a stage becomes clearer with a different view or different subplot choice, clarity takes precedence over strict visual uniformity

Local viewing guidance for notebook outputs:

- prefer opening the notebook in VS Code or Jupyter Lab
- if `jupyter` is available locally, `jupyter lab path/to/file.ipynb` is the default viewing workflow
- if a static preview is needed, export with Jupyter tooling rather than hand-converting the notebook back to Markdown

For figure-driven explanation notes whose primary purpose is to explain a sequence of plots or time-frequency diagrams:

- you may replace the theorem-first body with a figure-first body
- in that case, keep `Summary`, `Problem Definition`, and `Symbols And Assumptions` near the top
- then structure the main body as repeated stage blocks
- in notebook form, each stage is a `Code cell output -> Markdown explanation` pair
- the Code cell should show only the newly introduced transform for that stage, not the entire pipeline again
- the Markdown cell should read naturally, but it must still cover:
  1. what the figure shows
  2. the mathematical step
  3. the fully expanded closed form when applicable
  4. the physical meaning
  5. why this stage leads to the next one, unless it is the final stage
- after each stage explanation, append one `Fully Expanded Closed Form` block for the current stage signal
- in that block, absorb all prior operations into the current-stage expression
- do not paste the whole derivation history or list every previous intermediate signal again
- the reader should see one current signal formula that already reflects all effects accumulated up to that step
- the preferred model is the `tops_azimuth_overall.md` style:
  the stage block should show the current signal shape itself, not a nested operator chain
- if a filter multiplies the previous stage, write its effect directly into the current-stage phase, support window, or envelope
- if a deramp or reramp cancels a previous quadratic term, show the cancelled result explicitly, for example `(\psi_{2,m}-\psi_{2,\mathrm{ref}})` rather than re-expanding the full operator history
- if an LPF or keep window is part of the current stage, show it as a `\mathrm{rect}(...)` factor in the current-stage closed form
- it is acceptable and often preferable to use stage-local placeholders such as `A_{i,m}`, `\psi_{i,m}`, `\chi_{i,n}`, `B_{\mathrm{LPF}}`, and `T_{\mathrm{LPF}}` so that the result stays readable while still being a true closed form
- avoid notebook-stage formulas of the form `\mathcal{F}^{-1}[ \mathrm{rect} \, \mathcal{F}[ \exp(\cdots)\sum\mathcal{F}[\cdots] ] ]` when the same stage can be written as a simplified current-stage signal
- the mathematical content must still satisfy the fully-expanded closed-form rule
- do not move all equations to the front and all figure explanations to the end when the note is plot-driven
- the reader should be able to see a figure and immediately read the matching mathematics and physics without scrolling to another section
- for GitHub rendering, prefer sizing images with stable HTML such as `<img src="./figures/foo.png" width="350">` when a figure would otherwise appear too large or too small
- do not add a dedicated `Key code` block by default in notebook stage explanations
- let the Code cell itself carry the implementation, and only mention code in prose when a non-obvious operation needs clarification
- the math-to-code relation must still be strict, but do not add a dedicated mapping block by default
- only add a short symbol-to-variable correspondence note when a step is genuinely hard to read without that extra help
- in explanatory prose outside code fences and display equations, do not use inline `$...$`
- when referring to a signal or symbol in prose, use plain language or backticked forms such as `s_1(eta)` or `k_a`, so the reader does not need to mentally re-parse inline LaTeX
- exception for notebook front matter: in `Summary` and `Symbols And Assumptions`, prefer standard inline math such as `$s_1(\eta)$` and `$k_a$` when that makes the notation read more naturally

Preferred visible structure, matching the reference file above:

- Start with `**重點摘要**` when the note is long enough to benefit from a front-loaded summary.
- Every newly generated derivation file must begin with a clickable table of contents.
- Use numbered main sections such as `**1. ...**`, `**2. ...**`, `**3. ...**`.
- Use `Appendix` sections when geometric, symbolic, or side derivations would otherwise interrupt the main flow.
- Keep the equation blocks dense and separated by short explanatory paragraphs, as in `azimuth_freq_folding.md`.
- In `**重點摘要**`, keep bullets short and avoid inline LaTeX when plain wording or backticked identifiers are clearer.
- Do not put display equations inside summary bullets.
- If the summary must highlight a key formula, end the bullet list first, then place the display equation as a normal paragraph-level block below the list.

## Clickable TOC Rules

GitHub-flavored Markdown navigation is mandatory for this note set.

- Every new `.md` derivation file must include a clickable `## Table of Contents` section near the top.
- The table of contents must use GitHub Markdown links that jump to headings within the same file.
- Table-of-contents entries must be written in English only.
- If an existing note is rewritten, normalize its TOC entries to English as part of the rewrite.
- For links between files inside the repository, always use repository-relative Markdown paths such as `./foo.md` or `../derive/foo.md`.
- Never use machine-specific absolute filesystem links such as `/home/...` inside repo Markdown that is intended to be pushed to GitHub.
- The link style must remain valid in all of these contexts:
  1. on GitHub after push
  2. in a fresh local clone on another machine
  3. in local Markdown preview that resolves relative repo paths
- If the file belongs to a larger derivation flow, the top of the file must also include a short `## Navigation` section with clickable links to:
  1. the overall flow document
  2. the prerequisite note(s), if any
  3. the next note(s), if any
- For top-level summary files such as an `overall` report, include two navigation layers:
  1. a clickable flowchart-style reading order
  2. a clickable per-section table of contents inside the same file
- Prefer plain nested bullet lists for clickable flowcharts; they are more stable on GitHub than custom HTML.
- Example pattern:
  - `[1. Problem Definition](#1-problem-definition)`
  - `[2. Step-By-Step Derivation](#2-step-by-step-derivation)`
- When you create a new file, add the clickable TOC during initial generation; do not leave it for later cleanup.

## Completion Checklist

Before claiming a derivation note is done, explicitly verify all of the following:

- every stage signal that should exist in the chain is present
- every major stage output is written in fully expanded closed form
- no stage is represented only by an operator form when a closed form is expected
- the file has `## Navigation`
- the file has `## Table of Contents`
- all TOC entries are in English
- all inter-file links use repository-relative paths
- the GitHub math safety rules are still satisfied
- for figure-driven notes, every major figure is immediately followed by its matching mathematics and physical explanation
- for figure-driven notes with code, every major figure is immediately followed by its matching mathematics, code, and any necessary clarification for non-obvious correspondences
- for edited math-heavy Markdown files, local structural verification has been completed before sign-off
- display-math delimiters `$$` are balanced
- there are no corrupted delimiter fragments such as `` ` $ ` ``, `` $ ` ``, or half-converted math fences
- there are no accidental prose replacements inside display equations

## Core Writing Rules

- State the final answer first. Do not make the reader wait for the conclusion.
- Separate mathematical cause from physical interpretation.
- Distinguish geometry, signal model, approximation, and numerical implementation.
- Define every new symbol before using it in later equations.
- If a result depends on an approximation, state the approximation immediately before or after the equation.
- Prefer one claim per paragraph.
- When moving from one equation to the next, say what changed: substitution, approximation, shift, convolution property, Taylor expansion, coordinate transform, or physical interpretation.
- When a step transforms a signal, state explicitly what the signal becomes after that step.
- If a signal is renamed from `S_i` to `S_{i+1}`, always write the resulting signal explicitly before moving on.

## Section Writing Pattern For `3.1 / 3.2` Style

Use the following writing rhythm for stage derivations that must match the current `3.1` and `3.2` style in `derive_main.md`.

- Start each subsection with 1-3 sentences of prose that state what this stage does before any equation appears.
- In those opening sentences, explicitly say:
  1. what the input signal is
  2. what operation is being applied
  3. what the output signal becomes
  4. why this matters physically
- After the opening prose, write the key definition equation for the stage.
- Then write the fully expanded closed-form output signal for that stage.
- After the output signal, add 1-2 short paragraphs explaining which term in the equation carries the main physical meaning.
- If the stage has a direct discrete implementation meaning, end with one short implementation-correspondence paragraph.

For sections in the style of `3.1` and `3.2`, the preferred subsection rhythm is:

1. opening purpose paragraph
2. key mathematical definition
3. fully expanded stage signal
4. explanation of the dominant terms
5. implementation correspondence or transition to the next stage

Additional execution rules:

- Do not begin a subsection with a naked formula.
- Do not leave a stage described only verbally; the stage output signal must appear explicitly.
- If the next stage depends on a local phase model, rewrite the phase into that local form in the current subsection rather than deferring it vaguely.
- If the core action is structural, such as folding or mosaicking, explicitly state what mathematical operation it corresponds to.
- When replica support, phase, or coordinates are reinterpreted, state that change in words before using it in equations.

### `3.1` Pattern

For a folding/explain subsection:

- first explain why continuous content becomes periodically replicated
- then write the replication or folding equation itself
- then explain what overlap means physically on the folded axis
- end by explaining why later unfolding is still possible

### `3.2` Pattern

For a mosaicking subsection:

- first state that mosaicking is not replica removal
- explicitly say it is support relocation / coordinate relabeling / reassembly
- write the total mosaicked signal first, such as
  `$$ S_3(\tau,f_\eta) = \sum_m S_{3,m}(\tau,f_\eta) $$`
- explain clearly that the coordinate meaning changes from folded axis to extended axis
- define the replica-dependent phase function before using it
- if deramping comes next, rewrite that phase into a local quadratic form in the same subsection
- end with the discrete implementation meaning: sub-matrix reorder plus assembly corresponds to the continuous mosaicking operation

## GitHub Markdown Math Rules

Assume GitHub README compatibility is mandatory.

- Use `$$ ... $$` for display equations.
- For single-line display equations in the form `$$ equation $$`, keep one blank line above and one blank line below the equation line.
  This means every `$$ equation $$` line must be visually isolated by empty lines on both sides.
- For multiline display equations written as
  `$$`
  `...`
  `$$`
  keep one blank line before the opening `$$` and one blank line after the closing `$$`.
- Use `$ ... $` for signal names, coordinates, and symbols in explanatory prose when they carry mathematical meaning.
- For inline math, do not keep spaces immediately inside delimiters.
- Prefer `$equation$`, not `$ equation $`.
- Keep at least one normal prose space outside inline math delimiters when adjacent to words or punctuation in Chinese prose.
- Prefer `這是 $x$ 的定義`, not `這是$x$的定義`.
- Do not prefer backticked signal identifiers in prose when inline math is clearer.
- When referring to the word `equation` inside prose, prefer `$equation$` rather than plain `equation`.
- In prose, do not place Chinese punctuation directly adjacent to an inline math delimiter. Prefer `， $a=b$` over `，$a=b$`, and similarly avoid `$a=b$，`.
- In Chinese prose, if inline math follows punctuation or a connective phrase, leave a literal space before the opening `$`.
- Prefer `其中， $W_a(f_\eta;\omega_s)$` and `也就是說， $\omega_s$ ...`, not `其中，$W_a(f_\eta;\omega_s)$` or `也就是說，$\omega_s$ ...`.
- after any edit that touches mathematical notation, perform a local delimiter sanity check before considering the note finished
- Do not use `align`, `aligned`, `eqnarray`, or AMS alignment environments.
- Do not rely on `\{...\}` after operators like `\mathcal{F}` or `\exp`; prefer `\left[...\right]` or `\left(...\right)`.
- Do not wrap math symbols in backticks.
- Avoid HTML inside math.
- Break long derivations into multiple display equations instead of multiline alignment.
- Do not split one logical equality across multiple `$$ ... $$` blocks.
- Every display equation must be self-contained and render correctly on GitHub when viewed alone.
- When a display equation is short and structurally simple, prefer writing it on a single source line inside one `$$ ... $$` block if that improves readability.
- When using a single-line display equation, keep one literal space after the opening `$$` and before the closing `$$`.
- Prefer `$$ A = B $$`, not `$$A=B$$`.
- This single-line preference must not override the GitHub stability rules below. If single-line formatting makes the source harder to read or risks delimiter/operator instability, keep a compact 2-line block instead.
- When a display equation is too long for a clean single-line form, compress it to 2-3 source lines when possible.
- Prefer breaking at high-readability boundaries such as after the left-hand side, or between major multiplicative / additive factors.
- Do not stretch one equation into 4-6 short source lines unless the expression is genuinely too long to keep stable and readable in 2-3 lines.
- For step-by-step derivation chains, prefer separate display equations written as
  `A = ...`, then `= ...`, then `= ...`
  rather than repeating the full left-hand side in every block.
- In those step-by-step equality chains, keep the first block with the explicit left-hand side, and let later blocks begin with `=`.
- For those chained derivation blocks, prefer 1-2 source lines per display equation when possible; do not over-fragment one step into many short lines.
- Inside display equations, spaces around `+`, `-`, `*`, `/`, and `=` are allowed and often preferable for readability.
- However, those operators must never appear on their own source line.
- Treat the following as iron rules for GitHub stability:
  1. never write raw signal notation in prose or wrap a signal name in backticks
  2. in prose, prefer backticked signal names such as `S_i(tau,f_eta)` instead of inline LaTeX
  3. never place `=` on its own source line inside a display equation
     always write `A = B`, never
     `A`
     `=`
     `B`
  4. never place a binary operator on its own source line inside a display equation
     this includes `=`, `+`, `-`, `*`, `/`, `\cdot`, `\times`, and similar infix operators
     always attach the operator to the left-hand or right-hand expression on the same line
     never write
     `A`
     `-`
     `B`
     or
     `A`
     `/`
     `B`
     in particular, inside `$$ ... $$`, never let a new source line begin with `+ `, `- `, `* `, or `/ `
     GitHub Markdown may parse such lines as list items before math rendering
     so write
     `A + B`
     or
     `A`
     followed by `+B`
     but never start the next line with `+ B`
  5. for simple signal definitions, do not write multiline source like
     `$$`
     `S_i(\tau,f_\eta)`
     `=`
     `\sum_m S_{i,m}(\tau,f_\eta)`
     `$$`
  6. instead, write the equation source as `lhs = rhs`, with the left-hand side and `=` on the same line
- For signal-definition equations, prefer single-line display source such as
  `$$ S_i(\tau,f_\eta) = \sum_m S_{i,m}(\tau,f_\eta) $$`
  when the expression is short enough.
- Even for long display equations, write the opening line as `lhs =` rather than
  `lhs`
  followed by a separate line containing only `=`.
- For long numerators, denominators, products, or sums, keep each infix operator attached to an adjacent term.
- Never let `+`, `-`, `*`, `/`, or `\cdot` appear as a standalone source line in display math.
- Never let a display-math source line start with `+ `, `- `, `* `, or `/ `.
- If a full expression is too long, either:
  1. keep a compact total-signal equation in one block, or
  2. define the per-component signal in a separate self-contained block.
- If a derivation is intentionally written as a step-by-step equality chain, continuing the same equality across multiple display blocks is allowed and preferred, but the continuation blocks must begin with `= ...` and remain self-contained.
- Prefer
  `\mathcal{F}\left[x(\eta)\right]`
  over
  `\mathcal{F}\{x(\eta)\}`
- For Fourier operators, do not write `\mathcal{F}\biggl\{...\biggr\}`.
- Prefer `\mathcal{F}\biggl[...\biggr]` for GitHub-stable rendering.
- Prefer
  `\exp\left(-j\phi(f)\right)`
  over
  `\exp\{-j\phi(f)\}`
- For multiline or complex display equations, avoid `\left...\right` because GitHub rendering may become unstable.
- Prefer explicit delimiter sizing such as `\bigl...\bigr`, `\Bigl...\Bigr`, or `\biggl...\biggr` instead.
- For important final expressions, important transformed signals, or key operator definitions, you may use GitHub-supported math coloring:
  `{\color{red} ... }`
- Use red sparingly and only for results the reader is expected to carry forward.
- Do not rely on color for basic readability; the equation must still be understandable without it.
- Do not nest `$$ ... $$` display blocks inside list bullets when GitHub list stability matters, especially in `**重點摘要**`.

## SymPy Escalation Rule

When the derivation requires any of the following, explicitly use the `sympy` skill:

- symbolic Taylor expansion
- explicit derivative or second-derivative coefficients
- algebraic simplification of phase terms
- verification that two expressions are equivalent
- full expansion of a filtered or transformed closed form

Use `sympy` especially when the note claims:

- "this coefficient comes from the second derivative"
- "this quadratic term cancels exactly"
- "after this step the signal becomes ..."

Do not guess those steps by hand when symbolic verification is feasible.

## Derivation Rules

### Conclusion

Start with 3-6 bullets:

- what the signal becomes
- why the phenomenon happens
- what operation fixes or exposes it
- what the final mathematical criterion is

### Problem Definition

State exactly what is being proved. Good examples:

- why mosaicking unfolds folded replicas
- why deramping makes LPF effective
- why FFT-based azimuth compression causes wrap-around

### Derivation Highlights

Summarize the proof chain in 3-6 bullets. Each bullet should correspond to one major section of the derivation.

### Symbols And Assumptions

List only symbols actually used in the derivation. Include:

- coordinates
- sampling parameters
- reference functions
- approximation domain such as small-angle or second-order expansion

### Step-By-Step Derivation

Organize by causal layers. Typical order:

1. geometry
2. signal model
3. transform or operator
4. approximation
5. resulting expression
6. interpretation

Each subsection should answer one narrow question.

For signal-processing notes, each major step must end with an explicit output signal block.

This is a non-negotiable rule:

- every major step must end with a fully expanded closed form
- every transformed signal must be written explicitly after the step
- if both per-replica and total-signal forms exist, write both
- if the total signal is long, it must still be written explicitly; length is not an excuse to abbreviate
- compact operator notation may appear first, but it never replaces the expanded closed form

Minimum contract for each step:

1. input signal
2. operation or approximation applied
3. resulting per-component signal if applicable, such as `S_{i,m}(tau,f_eta)`
4. resulting total signal, such as `S_i(tau,f_eta)`

If the step acts on a sum of replicas, write both:

- the transformed single-replica expression
- the transformed total-signal summation
- If the transformed total-signal equation is long, write:
  - one self-contained compact total equation
  - one or more self-contained fully expanded per-component equations
  - the fully expanded total output as well whenever it can be written explicitly
- Do not continue a total-signal equality in a second display block.

Do not stop at operator shorthand such as

`$$ S_3(\tau,f_\eta) = S_2(\tau,f_\eta)\cdot H_{\mathrm{de}}(f_\eta) $$`

when a fully expanded closed form can be written.

Instead, continue until the transformed signal is written in fully expanded closed form.

### Physical Meaning

Explain what each major term means:

- envelope / support window
- phase term
- shift index
- sampling operator
- filter action

Do not repeat the algebra verbatim.

### Final Result

Restate the final equation set in compact form. This section should be copy-paste ready.

If the derivation contains intermediate signals such as `S_2`, `S_3`, `S_4`, `S_5`, include the final compact set for all of them, not only the last one.

### Implementation Correspondence

When relevant, connect the derivation to code operations such as:

- append / concatenate
- FFT / IFFT
- multiply by filter
- crop / LPF / masking
- index shift by `m * PRF`

State clearly whether the code is:

- exact implementation of the operator
- discretized approximation
- heuristic simplification

## Style Rules

- Prefer concise technical prose over conversational explanation.
- Avoid hype, reassurance, or meta commentary.
- Avoid duplicated summaries.
- Use flat bullet lists only.
- Keep headers short.
- Use English for file titles and mathematical section labels when the file already uses English notation; use Chinese prose if that matches the note set.
- If the reference file uses Chinese explanatory prose with English symbols, keep that mixed style.
- Prefer the exact visual rhythm of `azimuth_freq_folding.md` over introducing a new house style.

## Equation Patterns

Preferred pattern for introducing a shifted component:

`$$ S_{2,m}(\tau,f_\eta) = S_{1,\mathrm{cont}}(\tau,f_\eta-m\cdot\mathrm{PRF}) $$`

Preferred pattern for operator application:

`$$ S_3(\tau,f_\eta) = S_2(\tau,f_\eta)\cdot H_{\mathrm{de}}(f_\eta) $$`

Required follow-through pattern when full expansion is needed:

`$$ S_{3,m}(\tau,f_\eta) = \text{fully expanded closed form after applying }H_{\mathrm{de}} $$`

`$$ S_3(\tau,f_\eta) = \sum_m S_{3,m}(\tau,f_\eta) $$`

Preferred pattern for FFT-based processing:

$$
I(\eta) =
\mathrm{IFFT}
\left[
\mathrm{FFT}\left[s(\eta)\right]
\cdot
\mathrm{FFT}\left[h(\eta)\right]
\right]
$$

## Consistency Checklist

Before finishing, verify:

- all symbols are defined
- no `align` or `aligned` remains
- no math variables are wrapped in backticks
- `\exp` and `\mathcal{F}` use GitHub-safe delimiters
- section order is consistent
- final result section exists
- physical meaning is separated from algebra
- important carried-forward results are highlighted in red when useful
- each major processing step ends with a fully expanded closed form
- no major step stops at shorthand if a fully expanded closed form could have been written
- the big problem was decomposed into clear modules before writing
- physical assumptions and derivation steps were audited for logical or physical inconsistency
- if symbolic coefficients or cancellations were needed, `sympy` was used instead of hand-waving
- no equation is split across display blocks in a way that breaks GitHub rendering
- the final visual structure still matches `azimuth_freq_folding.md`
