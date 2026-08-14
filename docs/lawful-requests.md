# Palomar lawful requests

This is the route for data-protection requests, including requests under the
UK and EU General Data Protection Regulation, and for copyright complaints
about material Palomar publishes or preserves. It says where to write, what a
request needs to contain, who decides it, what outcomes are actually available,
and which ones are not.

It is not the route for changing your mind about a submission. A submitter
withdraws from any non-terminal state before registration, and a registered
version is corrected by registering a further version rather than by removing
the earlier one. The [protocol specification](specification.md) describes both.
The [privacy policy](https://palomar-registry.org/privacy.html) says what
Palomar holds and why; this document says what you can ask Palomar to do about
it.

## Where to write

privacy@palomar-registry.org. The address forwards to the operator. There is no
web form, no ticket number, and no other address for this; a request sent
somewhere else may be redirected here rather than answered where it arrived.

Write in whatever language and format suits you. The list below is not a form.
It is what makes a request answerable without a round trip.

## What to put in a request

- What you are asking for, in one sentence, before the reasoning.
- What it concerns: the permanent identifier and version, such as
  `PALOMAR-2026-08-14-000001` version 2, or the exact URL, or the repository
  and commit if the material is not a registered record.
- Where in it the material is. A record carries submitted metadata, cited
  sources, a declared authorization relationship and its optional free-text
  evidence, an archived redacted review, a rendered Challenge, and a preserved
  copy of the source repository and its pinned dependencies. Naming the field
  or the file saves the most time.
- Your relationship to the material: the person the data is about, someone
  acting for them, the rights holder, or an agent for the rights holder.
- For a copyright complaint, what the protected work is and where it can be
  seen, so that the copy and the original can be compared, and a statement that
  you believe in good faith that the use is not authorized and that you are
  entitled to make the complaint.
- For a data-protection request, enough for the operator to be satisfied that
  the request is yours to make. Palomar will ask for more only where the
  material is not already public and the answer would otherwise disclose
  something to whoever wrote in.

A request that names no record and no material can only be answered by asking
which one you mean.

## What can be asked about

Personal data reaches Palomar's public surfaces in a small number of places.
Palomar publishes no submitter: no registry record names the account that
proved push access, and no schema has a field for it. What is published about a
person is the authorization relationship a submitter declared, the free-text
evidence beside it, which identifies whoever the submitter wrote into it, the
authors and identifiers of cited sources, the quoted submission metadata, and
whatever appears in the preserved source repository itself, including the names
and addresses in its Git history.

Two surfaces exist before any registration. Mechanical verification runs in a
public GitHub Actions workflow, so the repository, the commit, and the fact
that they were mechanically checked are public from verification onward,
whether or not the submission is ever registered. The submission state and the
delivered review are private and are retained indefinitely so that a decision
can be audited later. Private means access-controlled, not confidential: the
operators, GitHub, and the model provider can see the material relevant to
their roles.

A request may therefore concern personal data in a registered record and its
published artifacts, personal data in private submission state, or content in
a repository Palomar has preserved. Palomar does not control the original
GitHub repository a submission was made from. That is the submitter's, and a
request about the original belongs to whoever owns it, or to GitHub.

## Who decides

One of the named [Moderators](governance.md#moderators) decides, under the same
authority they exercise over any other public-data suppression, and a Technical
Maintainer executes the resulting change through the validated private-database
workflow. The authorizing Moderator and the private reason are recorded in the
private Database. Restoration, if a request is later shown to have been wrong,
uses the same separation of authorization and execution.

The operator receives requests, establishes the facts, and answers you. The
operator does not decide a suppression alone.

## What the outcomes are

Suppression of the public projection is the substantive outcome for most upheld
requests, including most erasure requests. One exact registered version is
withdrawn from the active public registry. The public data service then omits
that entry and its artifacts and serves only a minimal date-only tombstone, so
that a citation of the identifier does not silently become a wrong answer. What
was published stops being published.

Correction happens by a new version. A registered entry is immutable, so
nothing is edited in place. Where the complaint is that recorded metadata is
inaccurate, the fix is a further version of the same identifier, registered
through the ordinary route by someone with write access to the source
repository. Where the person asking is not in a position to register that
version, the outcome available is suppression rather than correction.

Where the material is in submission state rather than in a published record,
there is nothing public to suppress, and the request is answered by correcting
or annotating the private record.

Where a copyright complaint is upheld against content in a repository Palomar
preserved, the removal reaches Palomar's copy of that material as well as the
public projection of the record that binds to it. This costs the preservation
promise the specification makes for that record, which is why it is decided by
a Moderator and not by whoever happens to hold the credentials.

A request may also be declined. If it is, you are told why, in terms of what
Palomar holds and what it is for, and not by silence.

## What is not available

The canonical entry is not deleted, and registry history is not rewritten. A
suppression records that a version was suppressed; it does not remove the
version from the ledger. The canonical bytes stay in the private,
access-controlled repository, along with the private submission state and the
delivered review.

The reason is the thing the registry is for. Palomar's claim is that an
accepted result was accepted under a stated contract, by a review recorded at
an exact policy commit, on an exact source commit. A registry that can quietly
lose an entry cannot support that claim about the entries it still has, because
nothing distinguishes a record that was never made from one that was removed.
Retaining the canonical bytes in an access-controlled repository, rather than
serving them, is how suppression and audit integrity are both had at once. It
is also why the tombstone carries a date and nothing else: enough to show that
something was there, not enough to republish it.

Palomar will not treat a lawful request as a way to reverse an ordinary
editorial outcome, and will not use one to remove a record because someone
would prefer it were not there.

## The request itself

Your correspondence is retained with the moderation record, because the reason
a version was suppressed has to be reconstructible years later by someone who
was not there. The private reason recorded in the Database is not published,
and the tombstone names no requester.

## How long it takes

Every request is read by a person, and a person answers it. Palomar is run by a
small group of volunteers with no rota and no support desk, so no response time
is promised here that the operation could actually keep. What can be said is
that requests are not queued behind an automated triage that never reaches a
human, that a request naming a record and an outcome is answered faster than
one that does not, and that urgency stated in the first message is treated as
urgency.

Data-protection law gives you rights to a response within statutory periods and
to complain to your supervisory authority if a response does not come. Nothing
here narrows that. This section describes how the operation behaves, not a
deadline offered in place of the one the law sets.
