# Palomar protocol

This document defines the current repository, review, registration, and
publication contract. `CONTRIBUTING.md` is the submitter-facing standard;
`rubric.json`, the files under `prompts/`, and
`schemas/review.schema.json` are the machine-readable editorial contract;
`taxonomies/classification-guide.md` supplies the binding interpretation for
classification review. Every review records its exact PalomarPolicy commit, so
later policy changes do not reinterpret an earlier decision.

## Components and authority

| Component | Authority | Must not do |
| --- | --- | --- |
| `PalomarServer` | Intake, GitHub push-access check, private status access, withdrawal, and registration consent | Execute source, run editorial review, or write registry records |
| `PalomarSubmission` | Public mechanical verification and accepted-Challenge rendering | Hold private identity or review state, or decide editorial policy |
| `PalomarSubmissionState` | Private append-only submission state and scheduled orchestration | Publish unregistered decisions or define policy |
| `PalomarPolicy` | Submission rules, rubric, prompts, and review schema | Hold submissions, registered records, or credentials |
| `PalomarTemplate` | Reusable submission starter and CI example | Define binding policy or registry truth |
| `PalomarReviewer` | Automated editorial review, evidence validation, source preservation, registration PR preparation, clean-check merge, and finalization | Change policy, register without consent, or treat model output as trusted instructions |
| `PalomarDatabase` | Private canonical ledger, immutable record schemas, and filtered-publication tooling | Perform intake, execute source, or conduct review |
| `PalomarArchive` | Public native forks and immutable record-specific preservation tags | Hold private submission data or credentials that can mutate `PalomarRegistry` |
| `PalomarWeb` | Read-only human presentation of the public projection | Become another registry authority |

The public website and data service are views. The private PalomarDatabase
repository is the canonical ledger, and the private PalomarSubmissionState
repository is the canonical submission history.

## Submission lifecycle and privacy

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

The internet-facing server is stateless between requests. It writes every
durable fact and transition to PalomarSubmissionState. GitHub OAuth is used once
to establish push access to the submitted repository; the OAuth token is then
discarded. Push access is not proof of authorship or source-author approval, so
the submission separately records its authorization relationship.

The repository, commit, submission identifier, public verification run, and
mechanical logs are public from verification onward. Submitter identity,
editorial review, and decision remain private unless the submitter registers an
accepted review. “Private” means access-controlled, not confidential: operators,
GitHub, and the model provider can see the material relevant to their roles.
Private submission state and delivered reviews are retained indefinitely so a
decision can be audited later.

The submitter may withdraw from any non-terminal state. Withdrawal leaves no
public editorial decision. `verification-failed` means the mechanical gate did
not pass; `review-failed` means an operational review failure rather than an
editorial rejection. A `revise` or `reject` decision is not registered and a
corrected commit enters as a new submission.

## Mechanical verification

PalomarServer dispatches the public verification workflow and permanently pins
the matching run when it first observes it. The workflow publishes a
submission-scoped `mechanical-report.json`; that artifact, not the run title or
moving branch state, is the authoritative mechanical result.

A passing report binds at least:

- the submission identifier, source repository, full commit, selected project,
  requested paths—including the explicitly selected Comparator configuration
  path—workflow run, and workflow revision;
- the resolved and hashed Challenge, Solution, Comparator configuration,
  `formalization.yaml`, Lakefile, root licence, and Lean toolchain;
- the compared declarations, classifications, provenance, authorization-related
  source metadata, full dependency graph, and Challenge source closure; and
- the exact Comparator, lean4export, NanoDa, Landrun, and other trusted tooling
  revisions used by the run.

Every exported proof is checked by Lean's kernel and replayed through the
pinned independent NanoDa kernel. Palomar has no single-kernel passing result.

Mechanical metadata requirements are hard failures where `CONTRIBUTING.md` says
so. Missing provenance fields that the guide identifies as editorial minima are
recorded as warnings or `unspecified` values and are then assessed by editorial
review. `automation.methods` remains required by the adopted upstream format,
including `method: manual`; `review.status` describes review performed before
submission.

Challenge and Solution must be distinct module names. Palomar asks Lake for its
ordered source paths and selects the first matching regular, non-symlink file
inside the selected project. The Challenge is compiled separately against a
frozen trusted environment, and every source in its transitive closure must
resolve to Lean core or the accepted Mathlib/Tau Ceti statement surface.
Proof-only dependencies used by the Solution may be broader, but remain pinned
and confined.

The reviewer independently downloads the pinned artifact, checks that its
workflow revision remains in trusted history, and binds it to the private
submission record. Missing, stale, ambiguous, or malformed evidence fails
closed.

## Editorial review

No person starts an ordinary review or signs off on its decision. The scheduled
private pipeline checks out the immutable source and recorded policy commit,
constructs the ordered evidence packets, invokes the configured model engine,
and validates every result against the rubric and schemas.

Each pass receives only its prompt, binding policy inputs, the mechanically
recorded submission evidence, and earlier pass results declared by the rubric.
Repository content, external material, tool output, and earlier model output are
untrusted evidence, never instructions. Review engines run in a fail-closed
sandbox without write or shell tools and cannot access registration credentials
or unrelated operator files. Literature research access, when enabled, is used
only by the literature and notability pass.

Synthesis must reproduce the evidence-pass scores exactly. An acceptance cannot
override a failed mandatory pass or a score below the rubric minimum, and a
fundamental notability failure requires rejection rather than revision. These
are structural guarantees; Palomar does not separately claim that a human
confirmed the model's substantive judgments.

The complete delivered review remains private and is bound to the submission by
digest. The status page presents its decision and actionable comments rather
than internal scores. For a registered result, Palomar publishes a redacted
archived review containing the decision and comments but not scores or
per-finding severity. The private canonical materials retain the information
needed to reconstruct how the decision was reached.

There is no ordinary appeal or human override. A changed review must be produced
under the current contract and delivered again; registration consent never
carries across review bytes.

Intake does not yet deduplicate repository and commit pairs or retain prior
attempts for the reviewer. Resubmitting the same commit can therefore obtain a
fresh sampled editorial decision. Whether to prevent that, expose attempt
history, impose a cooldown, or explicitly permit it remains an open integrity
and abuse-policy question.

## Registration, versions, and publication

Only an accepted review can be registered. The submitter explicitly consents to
the digest of the delivered review. Before any public registration side effect,
PalomarReviewer verifies the consent and evidence bindings and reserves the
permanent identifier and version in private state. A retry reuses that
reservation.

For a new result, the identifier contains the acceptance date and a random
six-digit serial. An accepted correction or dependency update creates the next
integer version of an existing identifier; a new mathematical result receives a
new identifier. Explicit version URLs are immutable, while an unversioned URL
resolves to the latest active version.

Palomar is still pre-launch. Until `.palomar-launched` is added to
PalomarDatabase, the database may be reshaped for testing and a pre-launch
registration is not permanent publication history. Public launch requires
settling the database contents and adding that marker; the append-only
guarantees below apply across subsequent history.

After the submitter's consent is recorded, registration renders the accepted
Challenge, validates the render and complete record, archives the mechanical
report, normalized workflow provenance,
redacted review, and source-preservation receipt in one content-addressed
evidence tree, then opens a PalomarDatabase pull request. The private scheduler
merges only when GitHub reports the change clean and all database checks have
passed. Merging that pull request is the registration event. A later scheduled
step verifies the merged entry and finalizes the private submission state.

After the launch boundary, canonical entries, schemas in use, renders, and
evidence are immutable. The private database may record a maintainer-only
suppression of one exact version from the generated public projection without
deleting or rewriting canonical bytes. The public service then omits the
affected entry and artifacts and serves only a minimal date-only tombstone.
Authority for exceptional intervention is an open governance question; there
is no submitter-facing ordinary takedown route.

The public data service is an active-only projection built from the private
ledger. Publication removes private-only fields, writes immutable release
objects to private R2, verifies their digests, and atomically advances the
current-release pointer. The read-only Worker exposes only allowlisted paths and
has no raw-GitHub fallback. PalomarWeb consumes this projection and is never a
source of registry truth.

## Source preservation

Before a registration branch is published, PalomarReviewer preserves the
submitted repository, every pinned Git dependency, and any separately recorded
substantive formalization. Each must be a public GitHub repository pinned to a
full commit and representable by an ordinary native fork. Git LFS is rejected
throughout the graph. Submitted and substantive repositories may not contain
submodules; an inert dependency gitlink is permitted only because verification
never initializes it and the ordinary Git object is preserved.

Repositories in one GitHub fork network share one public fork owned by
`PalomarArchive`. Every accepted commit receives an immutable record-specific
tag protected by a no-bypass ruleset. The archive identity is demoted from its
temporary per-fork administrator grant before the tag is written, and every
fork, commit, tag, and receipt is read back. Any failure stops registration.

The record and `source-archive.json` bind the complete source graph to those
forks and tags. Availability monitoring checks both original and preserved
commits. The intentional preservation promise is this maintained native-fork
copy; Palomar does not additionally archive Lean toolchain binaries, Mathlib
cache objects, or other non-Git artifacts.

## Security and deferred boundaries

Submitted source is hostile. Verification and rendering execute without
credentials or network access inside Landrun, with ceilings on time, memory,
processes, files, and output. Palomar never runs `lake update` or submitted
post-update hooks. Resource exhaustion is a retryable infrastructure outcome,
not a mathematical rejection.

The server holds separate least-privilege credentials for private state and
public workflow dispatch, so compromising the intake service does not grant
write access to verification code or the registry. Registry and archive
automation use distinct identities. Model engines cannot access registration
credentials or unrelated operator files.

Rendered HTML is sanitized, content-addressed, and served from
`data.palomar-registry.org`, separate from the website origin. PalomarWeb embeds
it with a restrictive CSP and an iframe sandbox without `allow-same-origin`.

The current service accepts only public GitHub repositories and Lean projects.
Private-repository support, DOI minting, automatic dependency updates, stronger
repeat-submission abuse controls, and proof assistants other than Lean remain
future work tracked outside this specification.
