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

## 2. `escalate` promises a review nobody performs

`escalate` is a real decision. The reviewer enforces it structurally: an
escalated pass cannot be resolved into an acceptance. But nothing downstream
acts on it, and by design Palomar has no appeals route and no human sign-off,
so an escalated submission is one that is simply not accepted.

The guide now says that plainly rather than promising specialist review. The
alternative is to drop the outcome and fold it into `revise` or `reject`. Worth
deciding before launch, because the word sets an expectation.

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
