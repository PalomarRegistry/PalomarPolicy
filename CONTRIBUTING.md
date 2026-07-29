# Submitting to Palomar

Palomar records a modest but meaningful claim: at a permanent Git commit, a
well-described Lean project contains proofs of the human-auditable statements in
`Challenge.lean`, as checked by Comparator, and an editorial review found the
mathematical description responsible enough to index.

It is a registry, not a journal. Acceptance does not assert novelty, importance,
correctness of an informal proof, or endorsement by a human expert.

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
- `comparator.json`, naming every theorem or definition to compare.

`lakefile.lean` and local/path dependencies are not accepted by the prototype.
The proof project may otherwise depend on arbitrary pinned Git repositories.
Palomar does not require the whole development to be “Palomar-shaped.”

The restriction applies to the **transitive import closure of
`Challenge.lean`**. Every non-core source file in that closure must come from:

- Mathlib or Tau Ceti (including their pinned infrastructure dependencies); or
- a repository version already indexed in Palomar.

Dependencies used only by `Solution.lean` may come from anywhere. The
mechanical report records the full project dependency set separately from the
smaller trusted challenge dependency set.

Comparator must accept the project using only `propext`, `Quot.sound`, and
`Classical.choice`. `sorryAx`, `Lean.ofReduceBool`, custom axioms, and unlisted
definition holes are not accepted.

## 2. The challenge surface

`Challenge.lean` is the part a mathematical reader is expected to audit.

- Prefer imports from Mathlib alone. Tau Ceti and Palomar-indexed imports are
  permitted, but are prominently recorded as a larger trust surface.
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

Definition holes are intrinsically easier to game. If `definition_names` is
nonempty, explain what values are intended and why the surrounding theorems
constrain them. Review may reject a formally comparator-valid but vacuous hole.

## 3. Informal account and provenance

Fill every required field in `formalization.yaml`, including:

- a plain-language account of each compared theorem;
- the source result and exact bibliographic references;
- what is original, translated, adapted, or still missing;
- authorship and AI involvement, including the human review performed;
- known fidelity gaps, extra assumptions, axioms, and scope limitations;
- the repository license.

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

The bar is intentionally a minimum. A useful formalization of a known result,
an independently checkable Lean companion to a paper, a careful counterexample,
or a reusable verified construction can all qualify without being a journal
article.

## 5. Review outcomes

Each review pass returns `pass`, `warn`, or `fail` with file-based evidence.

- `accept`: every mandatory question passes; warnings are recorded publicly.
- `revise`: the project may qualify after specific correctable changes.
- `reject`: a mechanical failure, fundamental statement mismatch, deceptive
  metadata, or failure of the editorial floor.
- `escalate`: the AI reviewer cannot responsibly resolve a material question.
  This is not acceptance; a human or specialist review is needed.

The final report, reviewer model identifiers, source commit, mechanical CI run,
and exact policy commit are public. Submitting grants permission to quote the
submitted metadata in that report and in the registry.

## 6. Versions

A new result receives `PALOMAR-NNNNNN`, derived from its submission issue.
Later corrections or dependency updates cite that identifier and become version
2, 3, and so on. Old entry files and source pins remain unchanged.
