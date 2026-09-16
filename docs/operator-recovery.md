# Operator recovery and execution profiles

The default execution profile remains `palomar-standard-v1` on GitHub's standard
Ubuntu runner. Verification retains its 19,800-second execution budget and
350-minute job timeout. Source authorization, dependency provenance, confinement,
and evidence requirements apply to every execution profile.

A Technical Maintainer may request `palomar-namespace-16x32-v1` for an admitted
failed submission. It provides 16 effective CPUs and a nominal 32 GiB allocation
(with a 28 GiB minimum usable memory check). Selection is explicit; neither
admission nor verification automatically escalates a submission. Rendering
inherits the selected profile and retains its existing deadlines.

Namespace must remain disabled until the dedicated qualification workflow passes
on the actual runner configuration. Qualification exercises the existing
systemd/Landlock supervisor, filesystem and network denial, child-process cleanup,
wall-clock enforcement, and trusted OOM telemetry. A privileged container label
alone is insufficient. Record the successful run and environment configuration
before enabling `PALOMAR_NAMESPACE_ENABLED=true` in PalomarSubmission. If the
existing supervisor cannot work there, leave the profile disabled and propose a
separate supervisor design; do not reduce confinement to get a green run.

Operator recovery reserves the ordinary owner and submitter concurrency slots.
It reuses the admitted source commit, paths, and authorization. It records the
operator's numeric GitHub identity, reason, profile and a fresh attempt ID, and
preserves earlier failures, runs, and alert origins. It does not increment the
ordinary admission backoff. That backoff policy remains unchanged; a submitter
blocked by a Palomar incident should be directed to a maintainer for recovery.
Withdrawn submissions are never restarted by this command.

Verification retries are limited to individual transient network operations and
report delivery. They remain inside the job's original time allowance. Checksum,
authorization, missing revision, and compilation failures do not authorize an
automatic repeat of the verifier.

Operator alerts retain the original diagnosis. A validated successful verification
after the alert, of the same commit and configuration, marks the alert recovered. A subsequently
verified descendant commit for the same configuration marks it superseded, with an explicit
statement that the original commit was not proved to work. Withdrawal is a
separate disposition. Recovery evidence is a public workflow URL and source
commit; private review notes and bearer status URLs never enter an alert.

Disposition delivery edits the original Zulip message with a content-hash
precondition. Human edits are conflicts for an operator to resolve. Missing
messages and insufficient edit permissions are recorded as delivery failures;
no replacement message is posted. A lost acknowledgment is safe to retry.

## Future dedicated large worker

A donated or externally hosted machine, including a potential large FLT worker,
needs a separately approved profile. Palomar must control its immutable image,
software updates, access policy, credentials, job dispatch, confinement, resource
supervision, cleanup, and evidence production. The provider may supply hardware;
it may not supply unverified success evidence. Require the same qualification
suite, measure effective cgroup capacity, define scheduling and cancellation,
and perform a reproducible end-to-end verification before accepting production
work. This change does not provision or enable such a machine.
