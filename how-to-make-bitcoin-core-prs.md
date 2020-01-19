## How to Make Bitcoin Core PRs - A work-in-progress guide

Last updated: January 20, 2020

*Hi! This article now lives at
[https://jonatack.github.io/articles/how-to-open-bitcoin-core-prs](https://jonatack.github.io/articles/how-to-open-bitcoin-core-prs).
I continue to update it there. Cheers.*


### BEFORE YOU BEGIN

This guide builds on the foundation laid by [How to Review Bitcoin Core
PRs.](how-to-review-bitcoin-core-prs.md).

Remember that reviewing is the most effective way you can contribute as a new
contributor and will teach you more about the codebase than opening PRs.

Don't be in a hurry to add to the pile of open PRs.

A good rule of thumb is to review 5-15 PRs for each PR you make.


### HOW TO CONTRIBUTE PULL REQUESTS TO BITCOIN CORE

When you do start contributing PRs, here are some guidelines:

Aim for quality over quantity and a balance between deep work and quick wins.

Read and know the Bitcoin Core [developer
notes](https://github.com/bitcoin/bitcoin/blob/master/doc/developer-notes.md).

Choose your contribution carefully to be sure it is desired by the maintainers
and reviewers; without their approval, you risk squandering your time and social
capital.

The best place to look for good things to do as a new contributor is the ["good
first issues" page](https://github.com/bitcoin/bitcoin/contribute).

Good ideas may also be found while doing [PR
reviews](https://github.com/bitcoin/bitcoin/pulls) and in
[#bitcoin-core-dev](https://webchat.freenode.net/?channels=bitcoin-core-dev) IRC
discussions when maintainers or experienced contributors mention nice-to-haves.

Things to do and fix can also be found by searching for TODO and FIXME comments
in the codebase. An easy way to do this is to run `git grep "TODO\|FIXME"` on
the command line from root. Keep in mind that these comments can sometimes be
out of date.

I find it useful to keep a list of PR ideas in the form of [observed
todos](observed-todos.txt).

Focus on user problems, actual bugs, and "used, but untested" methods that
affect outcomes and need tests.

[Try to solve a clearly defined issue with a clear, minimal
change](https://github.com/bitcoin/bitcoin/pull/17728#issuecomment-565803646).

Avoid making trivial refactoring, style cleanup, or spelling PRs; these consume
valuable contributor and maintainer time, often for little gain. Code churn can
cause hidden new bugs and blurs the code history and git blame. Any code
refactoring needs substantial motivation, advantages, or simplifications.

Do not begin by trying to change consensus code; it is difficult and
dangerous territory. The goal of Bitcoin Core is to maintain the correct
consensus on the Bitcoin network -- all else is secondary.

Gather context about the change you have in mind. Has it already been proposed
in the past? Do issues or PRs about it already exist? (They probably do). Sleuth
a bit. Search the `git log -S` history. Look at the `git blame` of the code in
question and `git show` the commits. Check the GitHub file history. You'll see
who understands that part of the codebase and can be contacted for questions or
review.

It may save you a lot of time to consult with relevant people on IRC before
tackling a difficult project in potentially the wrong way, or one that has
already been tried and turned out to be a dead end for reasons that may not be
apparent to you.

Test coverage is essential; don't hesitate to write any missing
[unit](https://github.com/bitcoin/bitcoin/tree/master/src/test),
[functional](https://github.com/bitcoin/bitcoin/tree/master/test/functional), or
[property-based](https://github.com/bitcoin/bitcoin/blob/master/doc/rapidcheck.md)
tests, or improve the
[fuzzing tests](https://github.com/bitcoin/bitcoin/blob/master/doc/fuzzing.md)
or
[benchmarking](https://github.com/bitcoin/bitcoin/blob/master/doc/benchmarking.md).

A Gregory Maxwell warning to test contributors: "Overly exact tests have greatly
diminished value if their exactitude turns them into hyper-active *something
changed* detectors, when they would otherwise be more useful as *something isn't
right* detectors if constructed more in terms of invariants instead of strict
behaviour."

Documentation is important, e.g. high-level documentation of how things work and
interact, clear and accurate code docs, whether a function has a good
description and [Doxygen
documentation](https://github.com/bitcoin/bitcoin/blob/master/doc/developer-notes.md#coding-style-doxygen-compatible-comments),
test logging (both `info` and `debug`), and so on.

In general, PRs that intelligently improve documentation and tests in a well
thought-out way tend to be well-received.

Don't rush. Give your work time to mature, revisit it with fresh
perspective, and iterate on it a few times before putting it out there and
asking for others' time. It might be a good rule of thumb to write several WIP
PRs for each one that you do file.

Consider also the timing of what is going on in the project in terms of
priorities and releases as well as your current status. If you already have a
few PRs open it may be best to close them or concentrate on having them merged
before adding yet another PR to the review stack. The idea of limiting open PRs
to 5 per contributor is occasionally raised; consider staying well under that.

Put the time in to make your PRs as easy to understand and review as possible:
Code, tests, documentation, good commit messages, relevant references, concise
explanations.

Give some thought to PR size and scope. Many contributors err on the side of
making their PRs too large and time-consuming to review. It is often better to
keep your PRs short and focused to make them easier and safer to review. This
may require splitting your work into several consecutive PRs.

As described in the Bitcoin Core
[contributing guide](https://github.com/bitcoin/bitcoin/blob/master/CONTRIBUTING.md),
prefix your PR title with the relevant area (e.g. `doc`,
`test`, `rpc`, `wallet`), and become comfortable with frequently rebasing your
commits to squash them. Rebasing is also frequently done to keep your commits
logical, separate, and easier to review. Even better, hygienic -- each standing
on their own without introducing regressions.

A PR prefixed or labelled as changing UI/UX can attract excessive
bikeshedding. One tried-and-tested solution can be to make a minor UI fix or
change as a commit inside a non-UI/UX PR that changes related items or code. The
regular reviewers will see it, but few drive-by commentators will.

PR descriptions are important. They "sell" your PR yet are often either too
terse or too verbose. Write the essentials, then take the time to make your PR
description brief, clear, even
[pithy](https://www.collinsdictionary.com/dictionary/english/pithy).

It's a smart habit to give reviewers tips in your PR description on how to
review and test your changes: "Here's how to review this".

PR descriptions need to explain *why*.  Summarizing "what" can be good too, but
"why" is essential to review because if the why doesn't make sense then the what
probably doesn't matter. Sometimes things seem self-apparent, but when in doubt
no one was ever hurt by a little more concise explanation.

Don't rely on markdown in your PR description for it to make sense, especially
strike-through formatting. The Bitcoin Core merge script will copy your PR
description to git history in the merge commit but any formatting will be lost.

For simple edits of existing commits, don't hesitate to squash your commits
before pushing, without waiting for the maintainers to ask you to do so.

Set up Travis CI on your own GitHub Bitcoin repository so that when you push a
branch or commit, the linter and continuous integration tests run. It can be a
good idea to verify that they are all green on your GitHub repository before
filing a PR. Unfortunately, the Travis CI is also currently very slow and times
out frequently, signaling false negatives.

At the moment, https://bitcoinbuilds.org runs more quickly and reliably than the
Travis CI (but for the moment it tests fewer platforms), so don't hesitate to
consult it for early initial feedback when you push a PR or changes.

You can sign your commits using the [OpenTimestamps Git
Integration](https://github.com/opentimestamps/opentimestamps-client/blob/master/doc/git-integration.md).
It's easy to setup.

Remember that every time you comment on your PR after pushing, you're sending
notifications to the people who previously reviewed or commented on it. Respect
their time.

Learn how to get people to review your work when needed. It's a skill.

Persistence can pay off. Sometimes a PR succeeds on the second or
[third try](https://github.com/bitcoin/bitcoin/pull/16060) if the first ones
didn't see enough review and if there are valid reasons to continue.

In this case, [one idea seen in
practice](https://github.com/bitcoin/bitcoin/pull/16060#issuecomment-494142593)
is to recap various ACKs from the previous PRs, with GitHub usernames, to rope
in support for the new PR. If you do, be sure to do it in a comment -- not in a
commit message and not in the PR description.

Never put GitHub @-prefixed usernames in commits and PR descriptions; the latter
because usernames in the description are copied into the merge commit by the
merge script. This can cause endless annoying notifications for those
concerned. An
[update](https://github.com/bitcoin-core/bitcoin-maintainer-tools/pull/32) to
the merge script now warns maintainers if a merge message contains a @username,
but it's better not to add them to begin with.

Add
[release notes](https://github.com/bitcoin/bitcoin/blob/master/doc/developer-notes.md#release-notes)
when appropriate. RPC-related release notes should refer to the RPC help for
details instead of substituting for full documentation, for example:
"Please refer to the RPC help of `getbalances` for details."

Ask yourself how others will see your PR. Try to only file PRs that people will
be motivated to review and that are worth merging, for both Bitcoin and your
track record. Your PRs will receive more attention from the contributors and
maintainers and be merged more quickly.

Scale up your PR contributions gradually in line with your ability and
understanding of the project, process, and codebase.

Continue to do more reviewing and testing than adding PRs to the backlog.
Reviewing 5 to 15 PRs for every PR you open is a good range.

If you are a good reviewer and begin building a reputation and appreciation for
your help in moving the project forward, maintainers and other contributors
may reciprocate by reviewing your work more quickly as well!
