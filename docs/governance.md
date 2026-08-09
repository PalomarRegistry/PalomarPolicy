# Palomar governance

Palomar launches with three named groups. Membership in one group does not
silently confer the powers of another, even when the initial memberships
overlap.

## Technical Maintainers

Technical Maintainers may maintain every Palomar repository, change the
software and its deployed behaviour, administer services and credentials, and
execute an approved moderation action through the validated private-database
workflow.

The initial Technical Maintainers are:

- Terence Tao
- Matthew Ballard
- Nestor Guillen
- Jaume de Dios

This is deliberately a high-trust operational role. It is intended for people
who can direct or review an urgent technical repair when another maintainer is
unavailable.

## Moderators

Moderators may authorize the retraction of one exact registered version from
the active public registry. A retraction preserves the canonical private
record and publishes only the minimal tombstone specified by the registry
contract. The authorizing moderator and the private reason are recorded in the
private Database; a Technical Maintainer executes the resulting validated
change. Restoration uses the same separation of authorization and execution.

The initial Moderators are:

- Jeremy Avigad
- Matthew Ballard
- Jaume de Dios
- Nestor Guillen
- Bryna Kra
- Kim Morrison
- Terence Tao
- Ravi Vakil
- Akshay Venkatesh

Moderation is exceptional public-data suppression, not an ordinary submitter
withdrawal and not deletion or rewriting of registry history.

## Scientific Advisory Board

The Scientific Advisory Board advises on Palomar's scientific direction,
standards, and policy. Board membership carries no operational duty and, by
itself, no repository or moderation authority. The board does not review,
approve, or endorse individual submissions.

The initial Scientific Advisory Board is:

- Jeremy Avigad
- Matthew Ballard
- Jaume de Dios
- Nestor Guillen
- Bryna Kra
- Kim Morrison
- Terence Tao
- Ravi Vakil
- Akshay Venkatesh

## GitHub trust model

Palomar uses the GitHub Free plan. It does not plan to buy GitHub Team.
Organization teams are still used as named access groups; that feature is
distinct from the paid plan.

The `technical-maintainers` organization team has Maintain access to every
Palomar repository. The `moderators` team has only the private Database access
needed for the moderation workflow. The `scientific-advisory-board` team has no
repository permission merely by virtue of board membership. Base organization
permission is None, so access comes only from an explicit team or repository
grant.

Required checks and branch protection are used on public repositories. GitHub
Free does not provide the same protected-branch enforcement for the private
Database and Submission State repositories. Palomar explicitly accepts that
limitation rather than purchasing GitHub Team: those repositories retain
fail-closed validation, pinned automation, pull-request review where another
maintainer is available, and manual maintainer discipline. This is an accepted
trust boundary, not a claim that the private repositories are structurally
protected.

The initial overlap between groups is intentional. Future membership changes
should be made explicitly for each role and reflected here and on the public
website.
