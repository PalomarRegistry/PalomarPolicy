# Submitting to Palomar

Palomar records a modest but meaningful claim: at a permanent Git commit, a
well-described Lean project contains proofs of the human-auditable statements in
`Challenge.lean`, as checked by Comparator, and an editorial review found the
mathematical description responsible enough to index.

It is a registry, not a journal. Acceptance is not publication, does not assert
novelty or correctness of an informal proof, and is not endorsement by a human
expert. It does assert that the submission cleared the substantive editorial
floor below.

## 1. Required repository shape

Submit a public GitHub repository at a full 40-character commit SHA. The
repository root must contain:

- `lean-toolchain`, pinned to a supported `leanprover/lean4:<version>`;
- `lakefile.toml` and, preferably, a committed `lake-manifest.json`;
- `formalization.yaml`, following the current
  [`mathlib-initiative/formalization.yaml`](https://github.com/mathlib-initiative/formalization.yaml)
  self-reporting standard;
- `Challenge.lean`, the small, human-auditable statement surface;
- `Solution.lean`, the proved version;
- `comparator.json`, naming every theorem or definition to compare;
- exactly one conventional root licence file: `LICENSE`, `LICENCE`, `COPYING`,
  `UNLICENSE`, or `OFL`, case-insensitively and optionally followed by `.md`,
  `.markdown`, or `.txt`.

`lakefile.lean` and local/path dependencies are not accepted by the prototype.
The proof project may otherwise depend on arbitrary pinned Git repositories.
Palomar does not require the whole development to be “Palomar-shaped.”

Intake parses `formalization.yaml` with a safe YAML loader, rejects duplicate
mapping keys, and requires one top-level mapping. As a mechanical minimum, the
following current self-reporting fields must be present and nonempty:

- `project.name`, `project.authors`, and `project.license`;
- `project.responsible_maintainers`, naming at least one person responsible for
  the submitted formalization;
- `provenance.result_origin`, either `original` or `source-based`, and
  `repository.role`, either `substantive-development` or `thin-wrapper`;
- `classification.arxiv`, containing one or two exact identifiers from the
  [arXiv category taxonomy](https://arxiv.org/category_taxonomy), and
  `classification.msc2020`, containing one to eight exact MSC2020 codes;
- when the result is `source-based`, at least one `sources` entry related by
  `formalizes`, `adapts`, or `independently-proves`;
- at least one `automation.methods` entry with `method` (use `manual` when
  appropriate);
- `review.status`.

The licence file must be an ordinary, nonempty UTF-8 file no larger than 1 MiB.
Intake requires one unambiguous standard SPDX match and exact agreement between
that identifier and `project.license`. Missing, multiple, custom, ambiguous, or
mismatched licence terms are mechanical failures and do not enter editorial
review.

Authors may be names or mappings containing a `name`. Unknown fields are
allowed. Passing this structural check says nothing about the quality or
accuracy of the metadata; the editorial review applies the fuller requirements
in section 3.

A source is not necessarily an arXiv paper. It may be a book, journal article,
web page, MathOverflow or other discussion, private communication, or a
folklore result. Give the most stable identifier or location available. Authors
and identifiers may be omitted when they genuinely do not exist or are not
known. A formalization that is the first presentation of a new result has
`result_origin: original` and may omit `sources` entirely; background sources
may still be listed with relationship `background` or `other`.

If the submitted repository exists only to expose another project through
Comparator, set `repository.role: thin-wrapper` and identify the substantive
formalization repository at an immutable full commit. This makes the registry
link to the mathematical work rather than treating its packaging as the work.

For example:

```yaml
classification:
  arxiv: [math.CO, math.NT]
  msc2020: [05C10, 11N13]
```

Choose categories for the mathematical result itself. A category does not
become apt merely because Lean or AI was used to formalize the result. The
classification review asks whether each choice is plausible, not whether it is
the best or most specific possible choice.

An update to an existing Palomar ID must come from the same source repository
as its current version. Repository transfers require explicit operator review
and are not accepted by the automated publication path.

The restriction applies to the **transitive import closure of
`Challenge.lean`**. Every non-core source file in that closure must come from
Mathlib or Tau Ceti, including their pinned infrastructure dependencies, or an
exact repository commit represented by a specific Palomar record version.
Indexed sources are reconstructed independently and each reached module is
compiled directly from a unique tracked source file by trusted Lean, without
using the indexed project's Lake plan as a source-to-object authority. Their
source bytes and versioned provenance are recorded, and their actual imported files are
included in definition-fidelity review. A recursively reached unindexed source
is not made acceptable merely because a parent project is indexed.

Dependencies used only by `Solution.lean` may use arbitrary pinned Git sources,
subject to the ordinary full-commit and confinement requirements. The
mechanical report records the full project dependency set separately from the
smaller trusted challenge dependency set.

Comparator must accept the project using only `propext`, `Quot.sound`, and
`Classical.choice`. `sorryAx`, `Lean.ofReduceBool`, custom axioms, and unlisted
definition holes are not accepted.

## 2. The challenge surface

`Challenge.lean` is the part a mathematical reader is expected to audit.

- Prefer imports from Mathlib alone. Tau Ceti imports are permitted but are
  prominently recorded as a larger trust surface. Palomar-indexed Challenge
  imports are supported and likewise recorded as `qualified`: indexing fixes a
  reviewable source snapshot, but does not make all present or future contents
  of that repository universally trusted.
- Prefer theorem statements to new definitions.
- Any definition needed to state a theorem must have a precise docstring and an
  ordinary mathematical meaning. Avoid encoding a desired answer into a
  definition.
- State every headline claim that the informal account attributes to the
  formalization. Do not hide material hypotheses, weaken quantifiers, swap a
  standard notion for a convenient surrogate, or market a bridge lemma as the
  advertised theorem.
- Keep the file small. The mechanical hard limit is 1,000 nonempty-or-comment
  lines and 100 KiB; review treats more than 300 lines or 32 KiB as a warning.
- The automated definition-fidelity packet carries up to 8 MiB of exact indexed
  Challenge-source text. A larger imported closure requires an expanded or
  operator-assisted audit and is escalated, not rejected or treated as a
  mechanical resource failure.

Definition holes are intrinsically easier to game. If `definition_names` is
nonempty, explain what values are intended and why the surrounding theorems
constrain them. Review may reject a formally comparator-valid but vacuous hole.

After acceptance, Palomar produces a Verso rendering of the pinned
`Challenge.lean` inside its confined publication pipeline. The canonical source
remains the commit-pinned GitHub file. Challenges with exactly one compared
declaration and no more than 100 lines or 32 KiB are shown inline; larger ones
open in a dedicated rendered view. Rendering failure delays publication but is
not an editorial rejection.

## 3. Informal account and provenance

Fill every required field in `formalization.yaml`, including:

- a plain-language account of each compared theorem;
- every mathematical source actually used, with the most precise available
  reference and its relationship to the formalization;
- what is original, translated, adapted, or still missing;
- authorship and AI involvement, including the human review performed;
- known fidelity gaps, extra assumptions, axioms, and scope limitations;
- the repository license.

The repository licence declaration covers the submitted repository snapshot at
the pinned commit. It does not relicense cited papers, reused formalizations,
mathematical sources, or dependencies, which retain their own terms. Palomar
records the declaration and detected licence but does not verify ownership or
provide legal advice.

Submitters must either be a responsible author or maintainer of the substantive
formalization, or have approval from one. For a thin Comparator wrapper this
means approval from someone responsible for the underlying formalization
repository, not merely the wrapper. The submission form records which basis
applies. A link or short note documenting approval is welcome but optional.

For each source, distinguish `formalizes`, `adapts`,
`independently-proves`, `background`, and `other`. Record prior formalizations
separately, including whether this work extends, reimplements, ports, compares
with, or otherwise relates to them. Source contact or endorsement is useful
context when known, but neither is required and neither substitutes for the
submitter-authorization rule above.

Write for a mathematically literate reader outside the immediate project. A
claim of novelty requires a credible literature search; otherwise say that
novelty is unknown.

An informal proof account is welcome but optional. If supplied, it must describe
the Lean proof that is actually present, not merely a plausible proof of the
same theorem.

## 4. Editorial floor

Mechanical correctness is necessary, not sufficient. Palomar does not index:

- trivialities presented as research contributions;
- a made-up theory whose definitions merely manufacture its conclusions;
- purported solutions of famous open problems without a serious statement
  audit, literature account, and honest treatment of the gap to the standard
  conjecture;
- duplicated or lightly repackaged work with no useful provenance;
- deceptive, materially incomplete, or promotional metadata;
- submissions whose mathematical content cannot be identified from the
  challenge and informal account.

The bar is a minimum, but it is a substantive research-interest minimum. The
actual mathematical result as stated must satisfy both of these tests:

1. it could plausibly warrant a research paper or serious research note; and
2. the reviewer can identify a credible research area and a plausible kind of
   mathematician in a research department who could reasonably find it
   interesting or relevant.

Both tests are required. Formal correctness, difficulty of the Lean work,
polished prose, novelty, or sheer size does not substitute for them. A niche
result may qualify when it has a credible specialist audience; a result so
isolated that no plausible research audience can be identified does not. A
confident failure of either test is a fundamental editorial failure and is
`reject`, not `revise`. Use `escalate` when responsible application of a test
genuinely needs specialist judgment.

## 5. Review outcomes

Each review pass returns `pass`, `warn`, `fail`, or `escalate` with file-based
evidence.

Scores use the following common anchors. Reviewers must judge the submitted
content, not the presence of keys, length of prose, or confidence of its tone.

- `1`: unusable, materially incorrect, or misleading;
- `2`: major errors or omissions prevent responsible reliance;
- `3`: minimally adequate, but with meaningful limitations or unverified claims;
- `4`: thorough, fair, evidence-supported, and correct apart from minor issues;
- `5`: exceptionally complete and independently checkable, with no meaningful
  gap found after a critical review.

A `4` or `5` requires concrete positive evidence for the relevant dimension.
It must not be awarded merely because Comparator passed, every field is
populated, or no contradiction was immediately noticed. Acceptance requires
every completed evidence score to meet the rubric minimum. Synthesis copies the
five registry scores exactly from the responsible evidence passes.

Notability uses a dimension-specific scale:

- `1`: incoherent, manufactured, materially deceptive, or crackpot-style work
  with no credible mathematical contribution;
- `2`: identifiable but trivial, routine, lightly repackaged, or lacking a
  plausible research audience;
- `3`: borderline mathematical interest, where paper-worthiness or a credible
  research audience is not affirmatively established;
- `4`: plausibly paper-worthy, with a specifically identified credible research
  audience;
- `5`: an unusually consequential result with clear interest beyond a narrow
  specialist audience.

Only a notability score at or above the minimum recorded in `rubric.json`
clears the floor. A reviewer should use plain, evidence-based descriptions such
as `trivial`, `confusing`, `unclear`,
`niche without an identifiable research audience`, or `crackpot-style framing`
when they accurately describe the submission. Frankness is not permission for
personal disparagement: assess the work and its framing, not the submitter.

- `accept`: every mandatory question passes.
- `revise`: the project may qualify after specific correctable changes.
- `reject`: a mechanical failure, fundamental statement mismatch, deceptive
  metadata, or failure of the editorial floor.
- `escalate`: the AI reviewer cannot responsibly resolve a material question.
  This is not acceptance; a human or specialist review is needed.

The summary, warnings, requested changes, and machine-readable report are
published for every decision. They must assess the work and its framing, never
disparage the submitter.

The final report, reviewer model identifiers, source commit, mechanical CI run,
and exact policy commit are public. Submitting grants permission to quote the
submitted metadata in that report and in the registry.

## 6. Versions

A new result receives `PALOMAR-YYYY-MM-DD-NNNNNN`, using the acceptance date
and its submission issue.
Later corrections or dependency updates cite that identifier and become version
2, 3, and so on. Old entry files and source pins remain unchanged.
