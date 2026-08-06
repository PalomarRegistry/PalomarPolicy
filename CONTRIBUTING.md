# Submitting to Palomar

Palomar is a registry of machine-checked formal proofs in Lean 4. It records an
exact commit of a public GitHub repository together with a small set of Lean
declarations that state the mathematical result.

The Lean evidence is organised as a Challenge/Solution pair. The Challenge
contains the declarations that state the result and is deliberately kept small
so that a mathematical reader can audit it. The Solution contains corresponding
declarations with the same types and supplies the proofs or definition values.

[Comparator](https://github.com/leanprover/comparator#comparator) checks that
the declarations in the Solution really are proofs, or definitions, of the
declarations stated in the Challenge, with the same types, using only the
permitted axioms. Palomar needs this comparison because the Challenge is the
small, human-auditable statement of record. Comparator makes it a mechanical
fact, rather than a claim, that the proof proves the stated thing.

A submission also has structured metadata and an informal mathematical account.
Palomar uses the upstream community
[formalization.yaml](https://github.com/mathlib-initiative/formalization.yaml)
self-reporting standard for formalisation projects; this is not a format
invented by Palomar. `formalization.yaml` records the required structured facts
about provenance, sources, licence, classification, authorship, automation,
review, repository role, scope, and known gaps. Narrative prose explaining what
the result says and why it matters may be in Challenge module documentation,
docstrings attached to the compared declarations, the project README,
`formalization.yaml`, or several of these locations. The formal statement alone
does not record all this context, and the editorial review reads the submission
as a whole against the Lean.

An accepted submission has passed two kinds of assessment:

1. Mechanical verification found that Comparator accepts the recorded Solution
   declarations as implementations or proofs of the recorded Challenge
   declarations, using only the permitted axioms. The exported proof is checked
   by Lean's kernel and by the independent NanoDa kernel checker.
2. An AI editorial review found that each recorded formal statement matches its
   informal description sufficiently closely, and judged that there is
   plausibly a mathematician who would find the result interesting or relevant
   as research.

Palomar is a registry, not a journal. Acceptance does not claim novelty,
validate an informal proof independently, or constitute endorsement by a human
expert. Acceptance is also not registration. The submitter sees the review
first and decides whether to register it with the registry record.

## 1. Decide whether the result is suitable

Mechanical correctness is necessary but not sufficient. The mathematical
result, as it is actually stated in the Challenge source, must satisfy both of
these tests:

1. It could plausibly warrant a research paper or a serious research note.
2. The reviewer can identify a credible research area and a plausible kind of
   mathematician in a research department who could reasonably find it
   interesting or relevant.

Both tests are required. A specialised result may qualify when it has a
credible specialist audience. Formal correctness, the difficulty or size of the
Lean development, polished prose, and novelty do not by themselves establish
research interest.

Palomar does not index:

- trivial results presented as research contributions;
- theories whose definitions have been designed merely to manufacture the
  advertised conclusions;
- purported solutions of famous open problems without a careful comparison with
  the standard conjecture, a serious literature account, and an honest
  statement of any gap;
- duplicated or lightly repackaged work without useful provenance;
- deceptive, materially incomplete, or promotional metadata;
- submissions whose mathematical content cannot be identified from the
  Challenge and the informal account.

A clear failure of either research-interest test leads to `reject`, not
`revise`.

## 2. Prepare an ordinary submission

The simplest submission is a public GitHub repository whose root is the Lean
project. In the ordinary layout, the Challenge, Solution, Comparator
configuration and formalisation metadata sit alongside the files that fix the
Lean toolchain and describe the Lake build and dependencies, plus a licence for
the submitted repository snapshot. Use these conventional files:

```text
lean-toolchain
lakefile.toml                 # or lakefile.lean
lake-manifest.json
formalization.yaml
Challenge.lean
Solution.lean
comparator.json
LICENSE                       # or another accepted licence filename
```

Submit the repository as `owner/name` together with the full 40-character SHA
of the commit to be reviewed. Branch names and tags are not accepted as
substitutes for a commit. The checked-out repository, excluding `.git` and
symbolic links, must be no larger than 500 MiB.

In this ordinary layout:

- `Challenge.lean` contains the declarations that state the result: the small
  statement file that a mathematical reader is expected to audit. Its module
  documentation and declaration docstrings may contain some or all of the
  narrative mathematical account.
- `Solution.lean` contains declarations with the same types and supplies the
  proofs or definition values.
- `comparator.json` tells Comparator which Challenge and Solution modules and
  declarations to compare.
- `formalization.yaml` gives the required structured provenance, sources,
  licence, authorship, process, classification, limitations, and review history,
  and may also contain some or all of the narrative mathematical account.
- the project README may contain some or all of the narrative mathematical
  account.
- the licence file states the licence for the submitted repository snapshot.

The filenames `Challenge.lean` and `Solution.lean` are conventions rather than
requirements. Alternative module names and less common layouts are described in
section 6, after the ordinary case.

### 2.1 Lean and Lake files

The Lean and Lake files fix the environment in which the submission is built.
The toolchain selects Lean, the Lakefile describes the project, and the
manifest records its exact dependencies. Together they allow the verifier to
reconstruct the submitted development at the pinned commit.

A TOML Lakefile is declarative and can be parsed without executing submitted
configuration. A `lakefile.lean` is executable Lean code, so without a
committed manifest the verifier cannot determine its exact dependencies before
running submitted code. This is why a Lean Lakefile always needs a committed
manifest.

#### Mechanical requirements

- The `lean-toolchain` file must name a Lean release, in the form
  `leanprover/lean4:<version>`, no older than the minimum recorded in
  `toolchains.json` in the [PalomarSubmission repository][submission-repo].
  There is no list of accepted versions to keep current: Palomar builds
  against the lean4export and Verso releases matching the toolchain, and
  records the exact revisions it used.
- The project root must contain exactly one of `lakefile.toml` and
  `lakefile.lean`. The Lakefile must be a regular file no larger than 1 MiB. A
  TOML Lakefile must be valid TOML.
- Commit `lake-manifest.json`. It is mandatory for a `lakefile.lean` project.
  The narrow exception for a TOML project without a manifest is in section 6.3.

### 2.2 Challenge and Solution modules

The Challenge is the small statement file that a mathematical reader is
expected to audit. The Solution supplies the proofs or definition values. A
reader should be able to identify the exact mathematical result from the
Challenge without having to disentangle the proof development.

The Challenge should be short and readable:

- Prefer imports from Mathlib alone. Tau Ceti is permitted, but Palomar records
  it as a qualified dependency and displays a warning about the larger body of
  code that must be trusted when reading the statement.
- Prefer theorem statements to new definitions.
- Give every definition needed by a compared theorem a precise docstring and
  its ordinary mathematical meaning.
- State every principal claim attributed to the formalisation. Do not hide
  material hypotheses, weaken quantifiers, replace a standard notion with a
  convenient surrogate, or present a merely supporting lemma as the advertised
  theorem.

Choose module names that remain unambiguous, and keep their source files in the
committed project source tree. Submitted `.lake` directories are discarded
before verification and must not be used to hold these files.

#### Mechanical requirements

- The Challenge and Solution modules must be distinct dotted Lean module
  names. Every component must match `[A-Za-z_][A-Za-z0-9_']*`.
- Palomar asks Lake for its ordered source paths and selects the first regular,
  non-symlink source file matching each module. Each selected file must lie
  inside the Lean project.
- The Challenge has a hard limit of 100 KiB and 1,000 lines. A Challenge over
  32 KiB or 300 lines receives a mechanical warning because it is harder to
  audit.

### 2.3 Comparator configuration

`comparator.json` identifies the Challenge and Solution modules, the
declarations to compare, and the axioms that the comparison may use. It
therefore fixes the exact formal claims tested by mechanical verification.

A name in `definition_names` identifies a definition whose value is left
unspecified in the Challenge and supplied by the Solution. If you use this
feature, explain the intended value and why the compared theorems constrain it.
Editorial review may reject a definition that makes the result vacuous even
when Comparator accepts it.

#### Mechanical requirements

`comparator.json` must be a regular JSON file containing one object and must be
no larger than 1 MiB. It has four required keys:

```json
{
  "challenge_module": "Challenge",
  "solution_module": "Solution",
  "theorem_names": ["MyProject.main_theorem"],
  "permitted_axioms": ["propext", "Quot.sound", "Classical.choice"]
}
```

- `theorem_names` must be a nonempty array. The optional `definition_names`
  array defaults to empty. Every entry in either array must be a nonempty
  string.
- No other keys are accepted, apart from the optional `definition_names` and
  `enable_nanoda` keys.
- `permitted_axioms` may contain only `propext`, `Quot.sound`, and
  `Classical.choice`.
- The optional `enable_nanoda` key is accepted for Comparator compatibility,
  but its submitted value is ignored. Palomar always enables NanoDa in a
  separate trusted configuration.
- Deliberate holes in Challenge declarations are allowed. Comparator must
  confirm that every proved Solution declaration does not depend on `sorryAx`,
  `Lean.ofReduceBool`, a custom axiom, or an unnamed missing definition.

### 2.4 Dependencies

The Challenge has a stricter dependency rule than the Solution because it is
the statement a reader must trust. Its **transitive import closure** means the
Challenge source and every Lean source file reached by following its imports
recursively. Every file in that closure must be one of:

- Lean core;
- Mathlib at a verified revision in its canonical repository, together with the
  exact dependencies pinned by Mathlib's manifest;
- Tau Ceti at a verified revision in its canonical repository, together with
  the exact dependencies pinned by Tau Ceti's manifest.

No other project-specific source may occur in the Challenge's transitive import
closure. Recursive imports are treated exactly like direct imports. Previous
acceptance by Palomar does not make a repository an approved Challenge
dependency.

Dependencies reached only from the Solution may come from any Git repository
that satisfies the requirements below. This permits a proof to use a broader
development without making that development part of the statement a reader
must audit. The mechanical report records the whole project dependency set
separately from the smaller set used by the Challenge.

#### Mechanical requirements

- The project may use Git dependencies for its proofs. Every Git package in
  `lake-manifest.json` must use a credential-free HTTPS URL without a query or
  fragment, and must be pinned to a full 40-character lowercase commit SHA.
- Do not commit compiled Lean or native build output outside `.lake`. The
  verifier rejects files with compiled-artifact suffixes including `.olean`,
  `.ilean`, `.a`, `.bc`, `.dll`, `.dylib`, `.o`, `.obj`, `.so`, and `.trace`,
  and it replaces submitted `.lake` state with fresh build directories.

### 2.5 Repository licence

The repository licence declares the licence for the submitted repository
snapshot at the pinned commit. It does not change the licence of cited papers,
mathematical sources, reused formalisations, or dependencies. Palomar records
the declared and detected identifier, but does not verify ownership or provide
legal advice.

The licence file and `project.license` in `formalization.yaml` must identify
the same unambiguous standard SPDX licence.

#### Mechanical requirements

The repository root must contain exactly one conventional licence file. Its
name is case-insensitive and must be one of:

- `LICENSE`, `LICENCE`, `COPYING`, `UNLICENSE`, or `OFL`;
- one of those names followed by `.md`, `.markdown`, or `.txt`.

The file must be a regular, non-symlink, nonempty UTF-8 text file no larger
than 1 MiB. Palomar's licence detector must find exactly one unambiguous
standard SPDX identifier, such as `Apache-2.0`, and that identifier must match
`project.license` in `formalization.yaml` exactly. A missing, multiple, custom,
ambiguous, or mismatched licence fails mechanical verification before
editorial review.

## 3. Write `formalization.yaml`

`formalization.yaml` is the required structured mathematical and editorial
record of the submission. It gives a reader the facts needed to identify the
exact claim, understand its provenance and limitations, and assess how the work
was produced and reviewed. Narrative prose may also appear there, but it need
not be duplicated there when it is supplied in an eligible Challenge
documentation location or README.

Palomar uses the [mathlib-initiative `formalization.yaml` v0.3
format](https://github.com/mathlib-initiative/formalization.yaml) as a base and
accepts unknown fields. Palomar also adds fields for provenance, repository
role, subject classification, and responsible maintainers. The file may contain
those additions at the same time as upstream fields such as `status`,
`fidelity`, and `alignment`.

#### File requirements

The file must be UTF-8 YAML no larger than 256 KiB. It must contain one
top-level mapping. Duplicate mapping keys and YAML merge keys are not accepted.

### 3.1 Fields checked mechanically

The mechanically checked fields identify the project, classify the mathematics,
and record how the formalisation was produced and reviewed. The values still
require mathematical and editorial judgement.

Classify the mathematical result itself, not the use of Lean or AI. Each code
need only be a plausible description; it need not be the unique or most
specific possible choice. For example:

```yaml
classification:
  arxiv: [math.CO, math.NT]
  msc2020: [05C10, 11N13]
```

The `automation.methods` and `review.status` fields come from the upstream v0.3
self-reporting format. For work performed without an automated system, use
`method: manual`. `review.status` describes the review completed before
submission, not the Palomar review that is about to occur. Upstream examples
include `unchecked`, `agent-reviewed`, `self-assessed`, `peer-reviewed`,
`author-verified`, and another accurately described free-form status. Use
`review.reviewers` and `review.notes` to explain who checked what.

#### Mechanical requirements

These fields are hard mechanical requirements:

- `project.name`: a nonempty string;
- `project.authors`: a nonempty list whose entries are names or mappings with a
  nonempty `name`;
- `project.license`: the exact SPDX identifier detected from the root licence
  file;
- `classification.arxiv`: one or two distinct codes from Palomar's checked-in
  arXiv taxonomy snapshot, `taxonomies/arxiv-categories.json`, in the
  [PalomarSubmission repository][submission-repo];
- `classification.msc2020`: one to eight distinct codes from Palomar's
  checked-in MSC2020 snapshot, `taxonomies/msc2020-codes.json`, in the
  [PalomarSubmission repository][submission-repo];
- `automation.methods`: a nonempty list of mappings, each with a nonempty
  `method`;
- `review.status`: a nonempty string.

### 3.2 Provenance required for editorial acceptance

Provenance tells the reader where the result came from, whether this repository
contains the substantive development, and who is responsible for the submitted
formalisation. These questions affect how the mathematical claim and its
relationship to earlier work should be assessed.

Use `provenance.result_origin: original` only when the formalisation is the
first presentation of the result. Such a submission may omit `sources`. It may
still list background material with `relationship: background` or `other`. Use
`provenance.result_origin: source-based` when the work formalises, adapts, or
independently proves a result presented elsewhere.

Palomar policy requires the following information to be accurate and
informative:

- `project.responsible_maintainers`: at least one person responsible for the
  submitted formalisation;
- `provenance.result_origin`: `original` or `source-based`;
- `repository.role`: `substantive-development` or `thin-wrapper`;
- for a source-based result, at least one source whose `relationship` is
  `formalizes`, `adapts`, or `independently-proves`.

#### Mechanical handling of missing provenance

The current verifier records a warning and substitutes `unspecified` when the
first three items are absent or unrecognised. It also warns when a source-based
result has no substantively related source. These warnings do not stop the
mechanical run, but the editorial metadata and provenance review still assesses
the missing information, and acceptance requires its score to meet the same
threshold as every other completed review dimension.

### 3.3 Mathematical sources and related formalisations

A mathematical source may be a paper, book, web page, MathOverflow or another
discussion, private communication, or a folklore result. It is not a Lean
software dependency. Record previous formalisations separately in
`related_formalizations` so that a reader can distinguish mathematical sources
from earlier Lean work.

Give every source a nonempty `title` and the most stable identifier or location
available. Choose the relationship that best describes how the submitted result
uses the source:

- `formalizes`: the Lean work formalises the source's result;
- `adapts`: it changes or extends the source's result;
- `independently-proves`: it proves the same result independently;
- `background`: the source supplies context rather than the recorded result;
- `other`: another relationship, which should be explained.

Source authors and identifiers may be omitted when they genuinely do not exist
or are unknown. Contact and endorsement are useful context but are not required
and do not replace submitter authorisation.

For each previous formalisation, use `note` to explain whether the present
work extends, reimplements, ports, compares with, or otherwise relates to the
earlier work.

#### Field constraints

- When contact information is supplied, `author_contacted` accepts `yes`, `no`,
  or `n/a`; `author_endorsement` accepts `participated`, `endorsed`,
  `no-response`, `not-contacted`, `declined`, or `n/a`.
- Each entry in `related_formalizations` must have an `id` and one of
  `builds-on`, `adapts`, `independent`, `supersedes`, or `other` as its
  `relationship`.

### 3.4 The informal account

Write for a mathematically literate reader outside the immediate project. The
narrative mathematical account may be in Lean module documentation in the
Challenge source, docstrings attached to the compared declarations, the
selected-project README or repository-root fallback, or
`formalization.yaml`. It may be in one of these locations or divided across
several, and need not be duplicated. Taken together, the supplied prose must
make it possible to identify and assess the exact claim being submitted.

Keep all required structured facts in `formalization.yaml`, including
provenance, sources and their relationships, licence, classification,
authorship, automation, review, repository role, scope, and known gaps.
Narrative elsewhere supplements those fields and does not replace them.

Across the eligible narrative locations, include:

- a plain-language account of every compared theorem;
- every known mismatch with the cited source, extra assumption, permitted
  axiom, scope restriction, degenerate case, and other limitation;
- an explanation of the mathematical sources used to choose, state, adapt, or
  justify the result, consistent with the precise references and relationships
  recorded in `formalization.yaml`;
- what is original, translated, adapted, proved, or still missing;
- the relation to previous formalisations;
- the authorship and production process, including AI involvement and human
  review;
- the repository licence.

Do not claim novelty without a credible literature search. If novelty has not
been established, say that it is unknown.

A source does not have to be archivable. Mathematics is communicated in
preprints, talks, social media posts, private correspondence and folklore, and
review judges the account you give, not the medium. Where a source cannot be
independently confirmed, say so, give the most stable identifier that exists,
and claim no more than it supports; you will not be asked to produce an archive
that does not exist.

Such a source is not evidence, though. Palomar records it as your account of
where the result came from, and it counts towards nothing else: novelty,
priority and notability have to stand on the result and on what can be checked.
What is marked down is a material citation that is wrong or
misattributed, a material claim resting on a source you do not identify, or
novelty claimed with no search behind it.

An informal account of the proof is optional and may be supplied in any of the
eligible narrative locations. If supplied, it must describe the architecture
and decisive steps of the Lean proof that is actually present, including
important assumptions and computational components. The reviewer compares it
with the Solution source; a plausible proof of the same theorem is not enough.

[submission-repo]: https://github.com/PalomarRegistry/PalomarSubmission

## 4. Confirm that you are authorised to submit

You must either:

- be a responsible author or maintainer of the substantive formalisation; or
- have approval from one.

The submission form records which basis applies. A link or short note
documenting approval is optional. For a thin wrapper, the relevant person is
responsible for the underlying formalisation repository, not merely the wrapper
repository.

Answering that you are a responsible author or maintainer is itself the basis,
and review will not ask you to document approval from yourself. Write access,
a shared owner, organisation membership, a fork, and a transferred repository
are none of them that answer: they say what you can do, not what the work is
or whose it is. Answering falsely is a material misrepresentation.

Source-author contact or endorsement does not replace this authorisation.

## 5. What mechanical verification establishes

The verifier checks out the exact commit and records hashes for the files it
uses. It validates the repository structure, metadata shape, licence,
dependency pins, Challenge dependency set, file sizes, and Comparator
configuration. It then:

1. discards submitted Lake build state and materialises the exact dependencies
   in the manifest;
2. compiles the Challenge separately against Lean core and the verified Mathlib
   or Tau Ceti dependencies;
3. records every Lean source file used by that compilation and rejects any
   source outside the permitted set;
4. protects that compiled Challenge module from replacement by project build
   output;
5. runs Comparator without network access or credentials;
6. forces the exported proofs through both Lean's kernel and NanoDa.

A passing mechanical report establishes that the recorded Solution satisfies
the recorded Challenge under those checks. It does not establish that the
Challenge says what the metadata claims, that a definition has its ordinary
mathematical meaning, that the metadata is accurate, that the result is novel,
or that the result is interesting. Those are editorial questions.

The verifier distinguishes a submission failure from a retryable infrastructure
or resource error. Editorial review normally begins only after the mechanical
report passes.

## 6. Less common cases

Use this section only if the ordinary root layout in section 2 does not fit the
repository.

### 6.1 A Lean project below the repository root

The submission form can select one repository-relative directory as the
**selected project**. That directory becomes the root used for its Lakefile,
default metadata path, default Comparator path, and module resolution.

If the selected project has its own `lean-toolchain`, that file is used.
Otherwise the repository-root `lean-toolchain` is used. When both exist, the
project-local file takes precedence. The single licence file always remains at
repository root.

### 6.2 Explicit metadata and Comparator paths

You may supply explicit paths in the form when the selected files do not use
their defaults:

- the metadata path may point anywhere inside the repository, but its basename
  must be exactly `formalization.yaml`;
- the Comparator configuration path must point inside the selected project and
  must end in `.json`.

Every supplied path is relative to the repository root and uses `/`. It must
not be absolute or contain an empty component, `.` or `..` component,
backslash, query or fragment character, control character, drive prefix, or
symbolic-link component. The resolved object must be a regular file, or a
regular directory for the selected project, inside the checked-out commit.

### 6.3 A TOML project without its own manifest

If a `lakefile.toml` project has no `lake-manifest.json`, the verifier can
construct a temporary manifest only when every direct Lake dependency is a
contained path dependency, each target has exactly one regular Lakefile and a
committed valid manifest, none of those target manifests contains another path
dependency, package names do not overlap, and all targets use the same
contained packages directory.

Commit the selected project's own manifest for every other layout. This
includes any direct Git dependency and any overlap among package names
contributed by the path-dependency manifests. A `lakefile.lean` project always
needs a committed manifest for the reason given in section 2.1.

### 6.4 Contained path dependencies

A local Lake dependency may point to another directory in the same repository
checkout. The target must be a distinct regular directory, must not lie below
`.lake`, and must not be reached through a symbolic link or escape the
checkout. Its manifest's `packagesDir` must identify a contained directory
named `.lake/packages` owned by the selected project or a contained path
dependency.

The registered record normalises each path dependency to a
repository-root-relative directory. `.` in that record means the repository
root, regardless of how the Lakefile spelled the relative path.

### 6.5 Thin wrappers

A **thin wrapper** is a repository that exists only to expose declarations from
another formalisation to Comparator. Set `repository.role: thin-wrapper` and
provide:

```yaml
repository:
  role: thin-wrapper
  substantive_formalization:
    id: owner/repository
    revision: 0000000000000000000000000000000000000000
```

The repository must be a public GitHub repository, and `revision` must be a
full lowercase commit SHA. Palomar records this underlying repository and
commit as the substantive formalisation. Submitter authorisation must also
concern that underlying project.

## 7. Editorial review

The review is performed by a language model working through a fixed sequence of
prompts. No person reads a submission before a decision, and no person signs
one. It is not peer review, and it is not evidence that a mathematician has
checked the result. It is a structured, recorded, automated reading, and it
should be weighed as that.

The review consists of required evidence passes followed by a synthesis step. A
pass examines one subject and returns a verdict, findings tied to files or
other evidence, and one or more scores. Synthesis combines those fixed pass
results into the final decision; it does not raise or average scores.

The required passes examine:

- whether every arXiv and MSC2020 classification is substantively plausible;
- the clarity, accuracy, and completeness of the required structured metadata,
  provenance, and narrative account across its supplied locations;
- alignment between every compared theorem and its informal account, including
  definitions, quantifiers, hypotheses, coercions, degenerate cases, and
  claimed scope;
- the fidelity and auditability of every material definition and imported
  concept used by the compared statements;
- the literature account and the result's research interest.

If an informal proof account is present in any eligible narrative location, an
additional pass compares it with the actual Solution proof and its imports.

Each pass uses one of three verdicts:

- `pass`: the pass found no material problem;
- `warn`: the pass found a specific warning but did not fail;
- `fail`: the pass found a material deficiency or contradiction, or could not
  affirmatively establish a mandatory criterion from the available evidence.

A failed pass does not by itself choose the final decision. Synthesis returns
`revise` when a specific, realistically correctable evidence or presentation
gap could make the submission qualify, and `reject` for a fundamental failure.
The report should describe uncertainty as an evidence limitation rather than
making a stronger negative claim than the evidence supports.

Scores run from 1 to 5:

- `1`: unusable, materially incorrect, or misleading;
- `2`: major errors or omissions;
- `3`: minimally adequate, but with meaningful limitations or unverified
  claims;
- `4`: thorough, fair, supported by evidence, and correct apart from minor
  issues;
- `5`: exceptionally complete and independently checkable, with no meaningful
  gap found after critical review.

The current minimum for acceptance is **4**. A score of 4 or 5 requires
concrete positive evidence, not merely successful compilation, populated
fields, familiar terminology, or the absence of an obvious contradiction. Every
score from every completed pass must reach 4. This includes classification,
provenance, auditability, and optional proof alignment as well as the five
scores registered in the registry: statement alignment, definition fidelity,
notability, literature, and clarity.

One exception, or these anchors would forbid what section 3 permits: a source
disclosed as unconfirmable, precisely stated, is not an "unverified claim" for
the `3` anchor, and does not by itself hold literature below 4. It cannot
support a `5`, which requires an account somebody else can check.

Notability has its own anchors:

- `1`: incoherent, manufactured, materially deceptive, or framed in a way
  associated with mathematically baseless claims, with no credible
  contribution;
- `2`: identifiable but trivial, routine, lightly repackaged, or without a
  plausible research audience;
- `3`: borderline interest, where paper-worthiness or a credible research
  audience has not been affirmatively established;
- `4`: plausibly paper-worthy, with a specifically identified credible research
  audience;
- `5`: unusually consequential, with clear interest beyond a narrow specialist
  audience.

A notability score below 4 is a fundamental editorial failure and leads to
`reject`, including when a credible research audience or plausible
paper-worthiness has not been affirmatively established. Findings may say
plainly that work is `trivial`, `confusing`, `unclear`, or `niche without an
identifiable research audience` where the evidence supports it. When the
evidence establishes only that the requirement was not demonstrated, the
finding should say that instead. Findings assess the work and its presentation,
never the submitter.

### Final decisions

- `accept`: the mechanical report passes, no completed pass has verdict `fail`,
  and every completed score is at least 4;
- `revise`: the result may qualify after specific, realistically correctable
  changes;
- `reject`: there is a fundamental semantic, provenance, or editorial failure,
  including failure to affirmatively establish the research-interest
  requirement.

Palomar has no appeals route and no human sign-off on decisions. A submitter
who believes the reading is wrong should correct or strengthen the submission
and submit the corrected commit.

Every decision includes a summary, warnings, requested changes, pass findings,
scores, and a machine-readable report.

## 8. Privacy, registration, and rendering

The review and the decision are not public unless the submitter chooses to
register them. They are not secret either: they may be audited and acted on by
the Palomar moderation team, they pass through GitHub and the model provider,
and Palomar retains them indefinitely so that any decision can be examined
later.

Mechanical verification runs in a public GitHub Actions workflow, so the
repository, the commit, and the fact that they were mechanically checked are
public from the moment of submission. That workflow does only the mechanical
check. It runs before any editorial review, contains none of the review text,
and shows no decision, so its public log reveals nothing about whether a review
happened or what it found. The submitter's identity, the review, and the
decision stay non-public unless the submitter registers.

On registration, Palomar archives the review beside the record. The reviewer
model identifiers, exact source commit, mechanical workflow run, and exact
policy commit also become public. Submitting grants Palomar permission to quote
the submitted metadata in the review report and registry record.

After acceptance and before registration, Palomar renders the pinned Challenge
source with Verso for display. Rendering compiles submitted Lean, so it runs
under the same restrictions as verification: no network access and no
credentials. The commit-pinned GitHub file remains the authoritative source. A
Challenge is eligible for inline display when exactly one declaration is
compared and the file is no more than 100 lines and 32 KiB. Larger
Challenges open in a dedicated rendered page. A rendering failure postpones
registration but does not reverse the editorial decision.

## 9. Updates and permanent identifiers

When a new accepted result is prepared for registration, Palomar assigns an
identifier of the form `PALOMAR-YYYY-MM-DD-NNNNNN`. The date is the acceptance
date. The six-digit serial is chosen randomly from `000001` through `999999`
and checked against identifiers already in the registry. Random allocation
avoids revealing the order and approximate number of accepted submissions that
have not been registered.

A later correction or dependency update cites the existing identifier and
becomes version 2, 3, and so on. Automated registration requires the same source
repository and the same selected project path as the current version. A
repository transfer needs explicit operator review. Earlier entry files and
their source commits remain unchanged.
