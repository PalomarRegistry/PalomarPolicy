# Open policy questions

Raised while rewriting `CONTRIBUTING.md` against what the verifier and the
rubric actually do. Each is a decision for the registry, not a documentation
defect, so the guide describes current behaviour and these are recorded here.

## 1. `automation.methods` is required of work done by hand

The upstream `mathlib-initiative/formalization.yaml` v0.3 format marks
`automation` as required, with `manual` among the accepted `method` values, and
Palomar enforces it. Someone who proved a theorem by hand is therefore asked to
fill in an automation section to say that no automation was used, which reads
oddly and may deter exactly the submissions the registry most wants.

The fix belongs upstream: `automation` could become optional, with its absence
meaning `manual`. Until then Palomar either follows the upstream format or
diverges from it, and following it is the lesser cost.

## 2. What `review.status` should mean

Also inherited from upstream v0.3. Palomar checks only that it is a nonempty
string, while editorial review separately asks about AI involvement and the
human review performed, so the field carries no weight it is not already
carrying elsewhere. Three options: validate a defined vocabulary, accept free
text and document what it is for, or stop requiring it mechanically. The guide
currently lists the upstream values as examples without claiming they are
enforced.

## 3. Provenance minima are documented as editorial, enforced as warnings

`project.responsible_maintainers`, `provenance.result_origin`,
`repository.role`, and a substantively related source for source-based work were
described as mechanical minima. The verifier records a warning and substitutes
`unspecified` instead, deliberately, so that incomplete provenance does not stop
Lean and NanoDa verification. Editorial review still assesses them.

Either is defensible; the previous document simply described the wrong one. If
they should be hard requirements, the verifier and its tests need to change too.

## 4. Module uniqueness is claimed but not checked

The policy said the Challenge and Solution modules "must resolve uniquely" and
never resolve into `.lake`. `resolve_module_source` takes the first match in
Lake's ordered source path and checks only that it is a regular non-symlink file
inside the selected project. The Challenge dependency audit catches many unsafe
cases later, and there is no equivalent audit for the Solution. Either implement
the check or stop claiming it; the guide now describes what actually happens.

## 5. Undocumented rejection conditions

Repository, metadata, Lakefile and Comparator size caps, committed build output,
`packagesDir` ownership, YAML merge keys, and the exact Git URL rules can all
stop a submission and appeared only in verifier code. They are documented now.
Anything that can reject a submission belongs in the contributor guide.

## 6. A rejected review can be re-rolled, leaving no trace

A review returning `reject` leaves the submission at `review-ready`, which is not
terminal, so the submitter can withdraw from there, and withdrawal is designed to
leave no public trace of the review or the decision. Intake consults nothing about
earlier submissions: it checks that the repository is public and the commit
exists, and stops. There is no dedupe on repository and commit, no cooldown, and
no count of prior attempts recorded against anyone.

The same commit can therefore be submitted again for a fresh review. Because the
notability and literature passes are sampled model calls, repeated attempts
resample the decision rather than repeat it, and the effective floor becomes the
best of N attempts rather than the judgment the rubric describes. The submissions
with the most reason to retry are the ones the floor exists to catch.

What deters this today is cost, not policy. Each attempt spends a full
verification run and a full review, and the reviewer records what a review costs,
so the spend is at least visible afterwards. The admission cap helps less than it
looks: `MAX_INFLIGHT_PER_OWNER` is keyed on the repository owner, which a
submitter can multiply for free by pushing the same commit under another account
or organization. Keying it on the authenticated submitter would make it bite.

Issue-based intake needed no rule here, because repeated attempts were visible to
anyone reading the tracker. Private review removes that without replacing it.
Options: require a new commit for a new review, record attempts against the
submitter and show the count to the reviewer, apply a cooldown after a reject, or
accept the re-roll and say so plainly. The server already notes in code that
per-submitter quotas and backoff do not exist yet.
