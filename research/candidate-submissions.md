# Candidate Palomar test submissions

Research snapshot: **2026-07-29**. This is an intake-testing list, not an
endorsement, novelty judgment, or claim that an author wants registry inclusion.
Contact repository owners before filing real submissions.

## Method

I used:

- authenticated Lean Zulip searches for `comparator`, `Challenge.lean`,
  `formalization.yaml`, and named recent project discussions;
- GitHub code search for root `formalization.yaml` and files named
  `Challenge.lean`;
- GitHub API root inventories for the shortlisted repositories.

The locally synced `zulip-client` executable documented by the Zulip integration
was unavailable on this host, so the Zulip portion used its authenticated
read-only server-search helper. GitHub code search is relevance-ranked and capped
at 100 results per query; this is deliberately a candidate set, not a census.

Readiness notes below were written against the former root-only prototype
contract. Palomar now accepts a repository-relative project directory,
`lakefile.toml` or `lakefile.lean`, alternate Comparator configuration paths,
and dotted Challenge/Solution modules. Defaults remain the repository root and
the conventional filenames. The **transitive Challenge source closure** is
still restricted to Mathlib or Tau Ceti. The
rest of the proof project may have arbitrary pinned Git dependencies and
contained repository-local path dependencies.

## Nested-layout compatibility corpus

The path-aware intake is intended to cover the observed nested layouts in
`Solarys431/agrawal-r5`, `chdarcy/MarkowitzFormalization`,
`empath-nirvana/polyclone`, `ianklatzco/odd-order-lean`,
`jaumededios/sharp_smoothing`, `mrdouglasny/jacobian-challenge`,
`random-fields/percolation`, `rkirov/jacobian-claude`, `rkirov/jordan_pick`,
`scottnarmstrong/CoarseGraining`, `shalliso/KnightModel`,
`willmfeldman/aleksandrov-differentiability`, and `yuma-mizuno/markoff-modp`.
This is a packaging compatibility corpus, not an acceptance or endorsement
list; each pinned submission still has to satisfy the Challenge trust rule and
editorial policy.

## A. Best smoke-test candidates

These already have a Mathlib-only challenge surface and most of the remaining
contract.

| Priority | Project | Evidence | Smallest packaging change | What it tests |
| --- | --- | --- | --- | --- |
| 1 | [`mathlib-initiative/sum_product`](https://github.com/mathlib-initiative/sum_product) | Root `Challenge.lean` imports only Mathlib; root `Solution.lean` and `formalization.yaml`; comparator config is [`comparator/sum_product_false.json`](https://github.com/mathlib-initiative/sum_product/blob/main/comparator/sum_product_false.json). [Zulip announcement](https://leanprover.zulipchat.com/#narrow/channel/579630/topic/An%20autoformalization%20of%20the%20sum-product%20counter%20example/near/599854394). | Copy the config to root `comparator.json`. | Clean end-to-end positive path, existing formalization standard, a known counterexample. |
| 2 | [`kim-em/erdos-unit-distance-comparator`](https://github.com/kim-em/erdos-unit-distance-comparator) | Root Comparator workspace; `Challenge.lean` imports only Mathlib. The linked proof repo already has [`formalization.yaml`](https://github.com/kim-em/erdos-unit-distance/blob/master/formalization.yaml). | Add/adapt the metadata file from the proof repo. | Clean positive path; significant theorem; separate proof and audit repositories. |
| 3 | [`Joeyxyxyz/sturm-liouville-comparator-run`](https://github.com/Joeyxyxyz/sturm-liouville-comparator-run) | Root `Challenge.lean` imports only Mathlib; root `Solution.lean`, `lakefile.toml`, and `config.json`. [Zulip discussion](https://leanprover.zulipchat.com/#narrow/channel/441057/topic/Sturm-Liouville%20eigenvalue%20simplicity%20formalization%20Pot.Con./near/593673945). | Add `formalization.yaml`; rename `config.json` to `comparator.json`. | Metadata/provenance review of a compact AI-assisted analysis result. |
| 4 | [`ElVec1o/five-distance-sharp`](https://github.com/ElVec1o/five-distance-sharp) | Root `formalization.yaml`; `comparator/` contains Mathlib-only `Challenge.lean`, `Solution.lean`, and config. [Zulip review request](https://leanprover.zulipchat.com/#narrow/channel/441057/topic/Review%20wanted%3A%20Lean%20formalizations%20%28non-expert%20%2B%20LLM%29/near/599896742). | Copy the three comparator files to root and rename the config. | Positive path plus careful provenance and literature review. |
| 5 | [`MichaelStollBayreuth/EllipticCurves`](https://github.com/MichaelStollBayreuth/EllipticCurves) | `comparator/` contains Mathlib-only `Challenge.lean`, `Solution.lean`, and `challenges.json`. Michael Stoll describes the Mordell–Weil setup and its status in the [Zulip thread](https://leanprover.zulipchat.com/#narrow/channel/116395/topic/thoughts%20on%20elliptic%20curves/near/612117156). | Add `formalization.yaml`; copy/rename comparator files to root. | Human-expert/AI collaboration, ongoing-project scoping, multiple compared declarations. |
| 6 | [`yuma-mizuno/markoff-modp`](https://github.com/yuma-mizuno/markoff-modp) | Root `formalization.yaml`; `Comparator/Challenge.lean` imports only Mathlib and has a paired Solution/config. | Submit the root Lake project and explicitly select `Comparator/config.json`; no file relocation is needed. | Significant theorem, dotted Comparator modules, and `lakefile.lean` support. |

## B. Valuable packaging and policy tests

These are plausible mathematical candidates but currently exercise an explicit
Palomar boundary.

| Project | Current shape | Expected preparation or result |
| --- | --- | --- |
| [`guanyangwang/ktv-swap-lean-comparator`](https://github.com/guanyangwang/ktv-swap-lean-comparator) | It otherwise has every root file, but `Challenge.lean` imports candidate-local `ChallengeDefs.lean`. [Zulip announcement](https://leanprover.zulipchat.com/#narrow/channel/583339/topic/KTV%20%20conjecture%20for%20binary%20matrices%20and%20formalization/near/607517836). | Excellent negative test for the corrected trust rule: CI should reject the candidate-local Challenge import. Consolidate the auditable definitions into `Challenge.lean`. |
| [`fpvandoorn/carleson`](https://github.com/fpvandoorn/carleson) | Root Challenge/Solution and comparator config, but Challenge publicly imports `Carleson.Defs` from the candidate project; no root `formalization.yaml`. [Comparator discussion](https://leanprover.zulipchat.com/#narrow/channel/287929/topic/Strange%20doc-gen%20error%20when%20using%20comparator/near/608936264). | Tests an important design question: large trusted definitions cannot be smuggled through candidate-local imports. Needs a self-contained auditable statement whose closure reaches only Mathlib or Tau Ceti. |
| [`Solarys431/lean-eval-platonic-classification`](https://github.com/Solarys431/lean-eval-platonic-classification) | Root Challenge/Solution/config, but Challenge imports local `ChallengeDeps`; no `formalization.yaml`. The broader metadata is in [`unico-lean-proofs`](https://github.com/Solarys431/unico-lean-proofs). [Zulip post](https://leanprover.zulipchat.com/#narrow/channel/441057/topic/The%20classification%20of%20regular%20polytopes%20%28lean-eval%29/near/611939475). | Large-scale stress test and expected challenge-provenance rejection until the dependency surface is trusted. Also tests strong claims, prior art, and nonexpert human provenance. |
| [`Vilin97/lean-pool`](https://github.com/Vilin97/lean-pool) | Root Challenge/Solution, but the Challenge aggregates candidate-local `Challenge.*` modules and represents many results rather than one record. | Tests multi-result granularity, challenge-size/import rejection, and whether a registry-of-results should submit individual snapshots rather than its aggregator. |
| [`PhillipKerger/zero-order-bounds-lean-verification`](https://github.com/PhillipKerger/zero-order-bounds-lean-verification) | Root metadata and Solution; challenge is named `Challenge-d-3-accuracy.lean`, imports candidate-local `ZeroOrderBounds.Statement`, and config is nested. | Normalize filenames, then expect provenance rejection unless the statement is made self-contained. Useful definition-fidelity test. |
| [`plby/Erdos1196`](https://github.com/plby/Erdos1196) | Root Mathlib-only Challenge and metadata; config is nested and names `Erdos1196` directly as the solution module; repository uses `lakefile.lean`. | Explicitly select the nested config and recorded module names; no filename bridge is inherently required. Promising positive test if the resolved sources remain inside the selected project. |
| [`rkirov/jacobian-claude`](https://github.com/rkirov/jacobian-claude) | Root metadata and extensive comparator material; `comparator/` is a self-contained Lake workspace whose Challenge imports only Mathlib. README explicitly says the owner does not understand or review the mathematics. | Select `comparator/` as the project and root `formalization.yaml` as metadata. High-value test of famous-problem framing, statement alignment, and honest-but-insufficient human review. |
| [`Solarys431/unico-lean-proofs`](https://github.com/Solarys431/unico-lean-proofs) | Root metadata with multiple comparator workspaces such as Feuerbach and Sylvester–Gallai; the individual comparator Challenges import only Mathlib. | Submit one result per selected comparator workspace. Tests whether per-result metadata is specific enough instead of inheriting project-wide marketing. |

## C. Projects found on Zulip that need a Comparator wrapper

| Project | Zulip/GitHub evidence | Why it is useful |
| --- | --- | --- |
| [`dkunert/three-gap-theorem-lean`](https://github.com/dkunert/three-gap-theorem-lean) | Recent sorry-free theorem announcement and repository link in the [Lean Zulip introduction](https://leanprover.zulipchat.com/#narrow/channel/113489/topic/Introduction%3A%20Dirk%20Kunert/near/606915988); no Challenge, Solution, config, or metadata at root. | A compact known theorem and a clean test of writing the Palomar statement layer after the proof already exists. |
| [`schildep/verified-3d-mesh-intersection`](https://github.com/schildep/verified-3d-mesh-intersection) | The [Zulip announcement](https://leanprover.zulipchat.com/#narrow/channel/236449/topic/Verified%203D%20CSG%3A%20Trust%2093%20lines%20spec%2C%20not%201000%20lines%20AI%20code/near/613201290) explicitly foregrounds a 93-line trusted specification; current root uses `lakefile.lean` and has no Palomar files. | Nearly the motivating trust-surface story, but for verified software rather than a traditional theorem; useful scope/notability test. |
| [`eliottcassidy2000/mathlib4-planar-nullcone`](https://github.com/eliottcassidy2000/mathlib4-planar-nullcone) | Purports to formalize the planar nullcone and Gaussian moment results; [Zulip post](https://leanprover.zulipchat.com/#narrow/channel/441057/topic/two-variable%20gaussian%20moment%20%28and%20nullcone%29%20proof%20formalized/near/612299499). Mathlib fork shape, `lakefile.lean`, no Palomar files. | Deliberately demanding statement/literature/notability review and a realistic moderation-sensitive intake. |
| [`ElVec1o/kravitz-lonely-runner-n3`](https://github.com/ElVec1o/kravitz-lonely-runner-n3) | Mentioned with the five-distance project in the [Zulip review request](https://leanprover.zulipchat.com/#narrow/channel/441057/topic/Review%20wanted%3A%20Lean%20formalizations%20%28non-expert%20%2B%20LLM%29/near/599896742); no comparator or metadata root contract. | Another known-result formalization from the same production process; useful for checking whether the review distinguishes mathematical quality from author expertise. |

## Suggested initial test set

For a small but discriminating dry run, ask permission to package:

1. **sum_product** — expected clean acceptance path;
2. **erdos-unit-distance-comparator** — expected clean path with split metadata;
3. **KTV swap** — expected mechanical rejection for a candidate-local Challenge import;
4. **platonic classification** — scale and challenge-dependency rejection;
5. **jacobian-claude** — editorial rejection test even if mechanics pass;
6. **three-gap theorem** — test the from-scratch wrapper documentation.

This set exercises positive publication, packaging friction, the corrected
Challenge-only allowlist, large dependency surfaces, and the distinction between
kernel verification and responsible mathematical indexing.
