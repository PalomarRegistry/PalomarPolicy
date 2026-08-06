# Palomar protocol v1

## Separation of responsibilities

| Repository | Responsibility | Must not do |
| --- | --- | --- |
| `PalomarDatabase` | Append-only registered records and schemas | Intake, execute Lean, review, render UI |
| `PalomarServer` | Private intake, authorization, submission state, and registration consent | Execute submitted source, run AI reviewers, write registry records |
| `PalomarSubmission` | Public mechanical verification | Hold private identity or review data, decide editorial policy |
| `PalomarPolicy` | Contributor guide, rubric, prompts, and review schema | Hold registered records or credentials |
| `PalomarReviewer` | Operator-run AI review and database-PR preparation | Run in submission CI or merge its own PR |
| `PalomarWeb` | Read-only human presentation | Become a second source of registry truth |

## Submission state machine

```text
form submitted and GitHub push access confirmed
  -> verifying
     -> verification-failed
     -> awaiting-review
        -> reviewing
           -> review-failed
           -> review-ready
              -> registered | withdrawn
```

The submitter may withdraw from any non-terminal state. `verification-failed`
means that mechanical verification did not pass. `review-failed` is an
operational or tool failure, not an editorial decision. `review-ready` means
that the submitter can privately read an `accept`, `revise`, or `reject`
decision. Only an accepted review can proceed to registration, and only after
the submitter explicitly consents to registering the exact review bytes they
were shown. Withdrawing leaves no public review or decision.

Durable private state is a directory of JSON files in the private
`PalomarSubmissionState` repository. Every state transition is a commit. The
public verification workflow contains the source repository, commit, and
mechanical result, but no submitter identity, review text, or editorial
decision.

## Mechanical result

The submission server dispatches the public verification workflow and pins the
matching workflow run the first time it is observed. The workflow publishes
`mechanical-report.json` as a run artifact. That trusted artifact is the
authoritative verification report. A passing report binds:

- the private submission identifier and public workflow run;
- `owner/repo` and the resolved 40-character commit;
- the selected project directory and the repository-relative paths and hashes
  of the resolved Challenge source, Solution source, `formalization.yaml`,
  Comparator configuration, and Lakefile;
- the root licence path and SHA-256, plus the agreeing declared and mechanically
  detected SPDX identifiers;
- the Lean toolchain;
- compared theorem and definition names;
- one or two arXiv subject classes and at least one MSC2020 code;
- the full project dependency set, the transitive Challenge-source closure,
  its allowlist provenance, and Challenge size;
- pinned Comparator, lean4export, NanoDa, and Landrun commits;
- the workflow run URL and timestamp.

The configured Challenge and Solution modules must be distinct and resolve to
regular tracked files inside the selected project. Resolution into `.lake`, a
dependency checkout, an out-of-project source root, or through a symbolic link
is a mechanical failure.

The reviewer resolves the successful pinned workflow run, downloads its
submission-scoped artifact, checks that the workflow revision remains in
trusted history, and independently binds the artifact to the private submission
record, source commit, and run URL. A missing, stale, ambiguous, or malformed
artifact fails closed.

## Review packet

The reviewer checks out source and policy into read-only local directories.
Each review pass receives:

1. the pass prompt at the recorded policy commit;
2. any binding policy documents named by that rubric step, including the
   editorial floor and score threshold for notability and synthesis;
3. the recorded formalization metadata, Challenge source, Solution source,
   Comparator configuration, Lakefile, Lean toolchain, and selected-project
   README (falling back to the repository README, with the resolved path named
   in the evidence envelope);
4. the private submission metadata and mechanical report;
5. earlier pass results when the rubric says so.

Engines must emit JSON matching the per-pass shape in the rubric. Raw model
outputs, normalized results, and the final report are retained in the private
review work directory. The synthesis output must validate against
`schemas/review.schema.json`.

A required classification pass checks whether every submitted subject is
plausible for the actual result. Intake establishes that the identifiers exist;
the AI pass is deliberately not asked to find a unique or optimal category.

Every submission-controlled input and every earlier model result is framed as
untrusted evidence, with the binding instruction repeated after the evidence.
Review engines run without write or shell tools, and their prose is rendered
inertly. Applying a decision never reruns the model and can deliver only the
existing schema-validated `review.json` whose submission, source, mechanical
report, and policy commit an operator inspected.

`rubric.json` versions the engine contract. Version 6 has exactly three pass
verdicts: `pass`, `warn`, and `fail`. A mandatory criterion that cannot be
affirmatively established receives `fail`; synthesis then chooses `revise` for
a specific correctable gap or `reject` for a fundamental failure. Review schema
version 2 has exactly three decisions: `accept`, `revise`, and `reject`.
Reviews created under an older contract must be rerun before delivery. An
already registered database record is immutable and is not migrated.

## Registration

An accepted review does not itself register anything. The server first records
the submitter's explicit consent together with the digest of the exact review
shown. The reviewer verifies that digest, renders the accepted Challenge source,
and prepares—but does not merge—a pull request to `PalomarDatabase`. A changed
review requires new consent. A renderer or infrastructure failure postpones
registration and may be retried without changing the editorial decision.

For a new result, the record uses the acceptance date and a random six-digit
serial for a new permanent ID, retried against identifiers already registered.
A correction or dependency update may create the next version of the same ID;
a new mathematical result receives a new ID. Database CI verifies the schema,
filename, and exact `index.json` projection.

The database record includes a required `challenge_render` object binding the
content-addressed bundle, entrypoint, Verso commit, renderer commit, Landrun
commit, and render time. It also preserves the exact downloaded mechanical
report and normalized workflow provenance in a content-addressed evidence
bundle. Merging the database PR is the registration event. Recording the result
against the private submission state is a separate explicit operator action.

## Security boundary

Submission source is hostile. No credential is present in verification or
render execution. The internet-facing server uses separate least-privilege
tokens for private state and public workflow dispatch so compromise of the
server cannot modify verification code or forge a mechanical result.

Verification may elaborate a submitted `lakefile.lean` only inside Landrun with
the network and credentials absent. It never runs `lake update` or package
post-update hooks; the committed or trusted-synthesized manifest is the sole
dependency-resolution authority.

Rendering uses a trusted synthetic Lake workspace and an exact Verso revision
selected for the accepted Lean toolchain. It must not run a submitted Lakefile,
`lake update`, package post-update hooks, or source-derived commands outside
Landrun. Every Lake, Lean, Verso, and source-derived executable runs inside
Landrun without network access or credentials and with enforced ceilings on
time, memory, processes, files, and output. Resource exhaustion is a retryable
infrastructure result, never a mathematical rejection.

Registered render HTML is sanitized, carries a restrictive CSP, and is served
from the PalomarDatabase GitHub Pages site. PalomarWeb treats the database as
the sole source of truth and embeds eligible Challenges in an iframe with
`sandbox="allow-scripts"` and no `allow-same-origin`. It always links to the
commit-pinned recorded Challenge source. A Challenge is eligible for inline
display only when exactly one declaration is compared and the file is at most
100 lines and 32 KiB; other entries use a dedicated wrapper with the same
sandbox.

Arbitrary pinned Git dependencies and contained repository-local path
dependencies are permitted in the proof project. A nested project is selected
without changing the repository-level source or licence boundary. The
mechanical gate applies only to the transitive source closure of the recorded
Challenge source; imported sources must resolve to Lean core or the allowlisted
Mathlib/Tau Ceti closure. Every other source reached from the Challenge is
rejected. Being indexed by Palomar confers no import privilege. Dependencies
reached only by the recorded Solution source remain unrestricted apart from the
ordinary full-commit and confinement requirements.

This prototype accepts only public GitHub repositories. Private-repository App
tokens, source retention, DOI minting, rate limiting, and automatic dependency
updates are intentionally deferred.
