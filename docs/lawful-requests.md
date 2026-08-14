# Palomar lawful requests

This is the route for data-protection requests, including requests under the UK
and EU General Data Protection Regulation, and for copyright complaints about
material Palomar publishes or preserves. It says where to write, what helps a
request along, who decides it, what Palomar does by default, and what it does
when the default is not enough.

Everything below describes how Palomar ordinarily works. None of it limits a
right you have under applicable law. Where a right requires more than the
default outcome described here, the right decides the matter and Palomar
complies; the internal design is a reason for a default, never a reason a
request cannot be granted.

This is not the route for changing your mind about a submission. A submitter
withdraws from any non-terminal state before registration, and a registered
version is ordinarily corrected by registering a further version rather than by
removing the earlier one. The [protocol specification](specification.md)
describes both. The [privacy policy](https://palomar-registry.org/privacy.html)
says what Palomar holds and why; this document says what you can ask Palomar to
do about it.

## Where to write

privacy@palomar-registry.org. The address forwards to the operator, and it is
the route that reaches a decision fastest.

It is not the only valid route. A data-protection request is effective wherever
you send it, and a request that arrives at another Palomar address, in a GitHub
issue, or in ordinary correspondence is forwarded here and treated as made on
the day you sent it. Forwarding does not restart any period that applies to it.

Write in whatever language and format suits you. Nothing below is a form.

## What helps a request along

None of this is required. A request that says only that you want to know what
Palomar holds about you, or that you want it erased, is a valid request and is
answered as one; Palomar identifies the affected material itself rather than
asking you to do it.

- What you are asking for, before the reasoning.
- What it concerns, if you know: a permanent identifier and version, such as
  `PALOMAR-2026-08-14-000001` version 2, or a URL, or a repository and commit.
- Where in it the material is. A record carries submitted metadata, cited
  sources, a declared authorization relationship and its optional free-text
  evidence, an archived redacted review, a rendered Challenge, and bindings to
  the preserved copies of the source repository and its pinned dependencies.
  Naming the field or the file saves the most time.
- Your relationship to the material: the person the data is about, someone
  acting for them, the rights holder, or an agent for the rights holder.

Palomar asks for identification only where there are reasonable doubts about
who you are, and then asks for what is necessary and proportionate to resolve
that doubt and nothing else. Most requests about already-public material need
no identity check at all, because answering them discloses nothing that is not
already published.

## What a request can reach

Personal data reaches Palomar in a small number of places, and each of them is
considered separately when a request arrives.

Palomar publishes no submitter: no registry record names the account that
proved push access, and no schema has a field for it. What is published about a
person is the authorization relationship a submitter declared, the free-text
evidence beside it, which identifies whoever the submitter wrote into it, the
authors and identifiers of cited sources, the quoted submission metadata, and
whatever appears in a preserved source repository, including the names and
addresses in its Git history.

Some of that is public before any registration. The repository, the commit, the
submission identifier, the public verification run, and the mechanical logs are
public from verification onward, whether or not the submission is ever
registered.

The rest is private, which here means access-controlled and not confidential.
PalomarDatabase holds the canonical ledger and PalomarSubmissionState holds
submission history; both are private repositories, and both are retained so
that a decision can be audited later. The operators, GitHub, and the model
provider can see the material relevant to their roles.

A record does not carry a copy of the submitted source. It binds to native
public forks and immutable preservation tags in the `PalomarArchive`
organization, and those forks are a separate public surface from the registry
data service. Palomar does not control the repository a submission was made
from. That belongs to its owner and whoever else controls it, which is not
necessarily the person who proved push access to submit it, and a request about
the original belongs to them or to GitHub.

## Who decides

One of the named [Moderators](governance.md#moderators) decides, under the same
authority they exercise over any other public-data suppression, and a Technical
Maintainer executes the resulting change. The authorizing Moderator and the
private reason are recorded in the private Database. Restoration, if a decision
is later shown to have been wrong, uses the same separation of authorization
and execution.

The operator receives requests, establishes the facts, assesses them, and
answers you. The operator does not decide a suppression alone.

## Data-protection requests

Palomar assesses each right you invoke on its own terms, and each copy of the
affected data on its own terms. Access, rectification, restriction, objection,
and erasure are different rights with different conditions, and a request may
succeed under one and not another. The copies are the public projection served
by the data service, the canonical entry in PalomarDatabase, submission state
in PalomarSubmissionState, the preserved fork and tags in `PalomarArchive`, the
public verification run and its logs, the generated feeds and derived pages,
and the correspondence about the request itself.

Two outcomes are the ordinary defaults, and neither is the maximum relief
available.

The first is suppression from the public projection. One exact registered
version stops being served, as described under
[what suppression does](#what-suppression-actually-does) below. This is the
usual substantive answer to an erasure request about published material.

The second is correction by appending. A registered entry is not edited in
place, so an inaccuracy is ordinarily corrected by a further version of the
same identifier. Palomar does not make rectification conditional on the
requester holding write access to any repository: where someone can register a
corrected version, that is the cleanest route, and where nobody can, the
available routes are suppression of the inaccurate version and a correction
recorded with the decision. Which of those satisfies the right is assessed for
the request rather than chosen in advance.

Where an applicable right requires more than these defaults, Palomar complies.
In a rare upheld case that can reach personal data in the private ledger or in
submission state, which the append-only design otherwise leaves unchanged. Such
a step is authorized by a Moderator, executed by a Technical Maintainer, and
recorded as the exception it is.

Where processing is disputed, and where the law requires processing to be
restricted while a dispute is resolved, Palomar restricts it rather than
continuing to publish and arguing afterwards.

A refusal, in whole or in part, names its legal ground. Palomar does not claim
a standing exemption for the registry. Where an exception such as Article 17(3),
for freedom of expression and information or for archiving and research
purposes, is relied on, it is assessed against the particular data and the
particular record, and the assessment is what you are given rather than a
citation.

Requests are not limited to submitters. A cited source author, or anyone else
named in published material, may make one.

## Copyright complaints

This is an informal route. Palomar has not designated a DMCA agent, so this
section is not a designated-agent process and nothing here is a substitute for
formal service. If the operator later registers an agent, formal service on
that agent supersedes this section.

A complaint is most useful when it identifies the protected work and where it
can be seen, so that the copy and the original can be compared, identifies the
material complained of precisely enough to find it, gives contact details, and
states that you believe in good faith that the use complained of is not
authorized by the copyright owner, its agent, or the law, and that you are
entitled to make the complaint.

Complaints are acted on promptly. Where one is upheld, the person who submitted
the material is told what was complained of and what was done, so that they can
respond or correct the submission.

## What suppression actually does

The active public data service stops serving that version and its artifacts and
serves a minimal date-only tombstone in their place, so that a citation of the
identifier does not silently become a wrong answer. That is the bounded claim,
and it is the only surface the suppression mechanism itself covers.

Every other public surface Palomar controls is assessed separately in the same
decision, and removed or disabled where the request requires it. Those are the
preserved fork and tags in `PalomarArchive`, the public verification run and
its logs, and the generated feeds and derived pages.

Where an upheld complaint reaches preserved source, what a Moderator can
authorize is disabling public access to Palomar's preserved copy: an owner of
the `PalomarArchive` organization removing the fork or otherwise making it
non-public. This is deliberately outside the archive identity's own powers, and
it is recorded like any other moderation decision. Two limits are worth stating
plainly. Repositories in one GitHub fork network share a single preserved fork,
so disabling public access to it affects every record bound to that network and
not only the one complained of. And Palomar cannot reach copies it does not
control: the original repository, other forks in the same network, third-party
mirrors, and search-engine caches. Where those exist, Palomar identifies them
to you so that you can pursue them with whoever does control them.

## Why the default retains canonical bytes

The canonical entry ordinarily stays in the private, access-controlled
repository when the public projection is suppressed, along with submission
state and the delivered review.

Palomar's claim is that an accepted result was accepted under a stated
contract, by a review recorded at an exact policy commit, on an exact source
commit. A registry that can quietly lose an entry cannot support that claim
about the entries it still has, because nothing distinguishes a record that was
never made from one that was removed. Retaining the bytes without serving them
is how suppression and audit integrity are ordinarily both had. It is also why
the tombstone carries a date and nothing else: enough to show that something
was there, not enough to republish it.

That is the reason for a default, and not more than that. It is not a ground
for refusing a request, and it does not survive a right that requires
otherwise.

Editorial disagreement alone is not a ground for suppression.

## What happens to your request

The request is handled by the operator and kept in the operator's mailbox. What
is recorded with the moderation decision in the private Database is the minimum
needed to reconstruct why a version was suppressed, years later, by someone who
was not there: what was asked, on what basis, what was decided, and by whom. It
is readable by the Moderators and Technical Maintainers named in
[governance](governance.md), and by nobody else. The private reason is not
published, and the tombstone names no requester. The
[privacy policy](https://palomar-registry.org/privacy.html) discloses this
retention.

## How long it takes

Data-protection requests are answered within the period the law allows, which
is ordinarily one month from receipt, extendable where the law permits it for a
complex request. If Palomar extends, you are told within the first month, and
told why. Copyright complaints carry no statutory clock and are acted on
promptly.

Beyond that, an honest description of the operation: Palomar is run by a small
group of volunteers, and there is no rota, no support desk, and no ticket
number. What that means for you is that nothing is queued behind an automated
triage that never reaches a human, and that a request naming what it wants is
answered faster than one that does not. Urgency stated in the first message is
treated as urgency.

## If you disagree

Say so, to the same address. A decision explained badly is worth challenging
before anything else, and a decision made on wrong facts is worth correcting.

A request to exercise a right and a complaint about how Palomar has handled
personal data are different things, and you may make either without the other.
Palomar acknowledges a complaint about its handling of personal data within 30
days, as the United Kingdom rules require.

You may also complain to a supervisory authority. In the United Kingdom that is
the Information Commissioner's Office. In the European Union it is the
authority of the member state where you live, where you work, or where the
alleged infringement took place. You do not have to come to Palomar first, and
you retain the right to an effective judicial remedy against a decision of a
supervisory authority or against Palomar, whatever this document says.
