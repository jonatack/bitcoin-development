## [WIP] How to Review Bitcoin Core PRs

Last updated: July 19, 2019


### BEFORE YOU BEGIN

These self-study notes build on the foundation laid by these 3 articles:

1. [A Gentle Introduction to Bitcoin Core Development](https://bitcointechtalk.com/a-gentle-introduction-to-bitcoin-core-development-fdc95eaee6b8)
by [Jimmy Song](https://twitter.com/jimmysong) (2017)

2. [Understanding the Technical Side of Bitcoin](https://medium.com/@pierre_rochard/understanding-the-technical-side-of-bitcoin-2c212dd65c09)
by [Pierre Rochard](https://twitter.com/pierre_rochard) (2018)

3. [Contributing to Bitcoin Core, a personal account](https://bitcointechtalk.com/contributing-to-bitcoin-core-a-personal-account-35f3a594340b)
by [John Newbery](https://twitter.com/jfnewbery) (2017)


### INTRODUCTION

Reviewing and testing can be the best ways to begin contributing to Bitcoin
Core.

Experienced review and testing are regularly cited by long-term Bitcoin Core
developers as both

  - resource bottlenecks, and

  - the best and most helpful way to begin contributing and earning reputation
    in the community.

Yet the process and learning curve can seem intimidating and in practice very
few new contributors do it.

As a newcomer, this article represents my current understanding after a couple
months, a few dozen PR reviews (some more useful than others), a few issues
tested or handled, and a handful of commits.

Some of this understanding was gained from years of

  - contributing to a large open source project,
    [Ruby on Rails](https://github.com/rails/rails), and co-editing its blog and
    newsletter, [This Week in Rails](https://rails-weekly.ongoodbits.com/archive)

  - development and lead maintainership of
    [Ransack](https://github.com/activerecord-hackery/ransack),
    a popular Ruby search engine

  - daily discussions on this topic of mutual interest with my good friend and
    colleague, [Kasper Timm Hansen](https://twitter.com/kaspth), a developer at
    [Basecamp](https://basecamp.com/about/team) and since 2016 a member of the
    [Ruby on Rails Core Team](https://rubyonrails.org/community/).

Yet much of it is unique to Bitcoin Core and came only with time. I would have
preferred to know these things from the start. So here we are. This is foremost
a self-study exercise, but perhaps it can be useful for others.

### TERMINOLOGY

[PR](https://help.github.com/en/articles/about-pull-requests): An acronym for
*pull request*, sometimes called a merge request. A proposed change to the code
or documentation in the source code repository.

[WIP](https://en.wikipedia.org/wiki/Work_in_process): An acronym for *work in
progress*.

[ACK, NACK, nit](https://github.com/bitcoin/bitcoin/blob/master/CONTRIBUTING.md#peer-review):
Defined [here](https://github.com/bitcoin/bitcoin/blob/master/CONTRIBUTING.md#peer-review).


### GENERAL

As a newcomer, the goal is to try to add value, with friendliness and
humility, while learning as much as possible.

A good approach is to make it not about you, but rather "How can I best serve?"

One of the most challenging aspects facing new contributors is the breadth of
the codebase and the complexities of the technologies surrounding it.

Be aware of what you don’t know; long-term contributors have years of experience
and context. The community has built up a deep collective wealth of knowledge
and experience. Keep in mind that your new ideas may have already been proposed
or considered several times in the past.

Remember that contributor and maintainer resources are limited -- ask for them
carefully and respectfully. The goal is to try to give more than you take, to
help more than hinder, while getting up to speed.

Follow the [bitcoin-core-dev IRC channel](https://webchat.freenode.net/?channels=bitcoin-core-dev)
and the
[bitcoin-dev mailing list](https://lists.linuxfoundation.org/mailman/listinfo/bitcoin-dev).

Before jumping in, take plenty of time to

  - understand the contribution process and guidelines, not only by reading
    the documentation in the [repository](https://github.com/bitcoin/bitcoin),
    notably
    [Contributing to Bitcoin Core](https://github.com/bitcoin/bitcoin/blob/master/CONTRIBUTING.md)
    and everything in the [doc](https://github.com/bitcoin/bitcoin/tree/master/doc) and
    [test](https://github.com/bitcoin/bitcoin/tree/master/test) directories,
    but also by observing interactions on the
    [bitcoin-core-dev IRC channel](https://webchat.freenode.net/?channels=bitcoin-core-dev)
    and the ongoing [code review in the repository](https://github.com/bitcoin/bitcoin/pulls)

  - get to know the maintainers and regular contributors: what they do, what
    they like and want, how they give feedback

The big picture is much more important than nits, spelling, or code style.

Steps to improve understanding of the big picture:

- Do the [Chaincode Labs study guide](https://github.com/chaincodelabs/study-groups)

- Read and do [Programming Bitcoin](https://github.com/jimmysong/programmingbitcoin)

- Study the [Bitcoin Improvement Proposals](https://github.com/bitcoin/bips/)
  (often referred to in singular form by the acronym "BIP"), and return to them
  frequently

- Subscribe to the [Bitcoin Optech newsletters](https://bitcoinops.org/) and
read their [Scaling Book](https://github.com/bitcoinops/scaling-book)

Aim for quality over quantity and a balance between deep work and quick wins.

Focus on user problems, actual bugs, and "used, but untested" methods that
affect outcomes and need tests.

Documentation is important, e.g. whether a function has a good description and
[Doxygen documentation](https://github.com/bitcoin/bitcoin/blob/master/doc/developer-notes.md#coding-style-doxygen-compatible-comments)
for all its arguments, and high-level documentation of how
things work and interact.

Test coverage is essential; don't hesitate to improve or write any missing
[unit](https://github.com/bitcoin/bitcoin/tree/master/src/test) or
[functional](https://github.com/bitcoin/bitcoin/tree/master/test/functional)
tests.

Be a contributor. Help PRs move forward by reviewing,
[proposing tests](https://github.com/bitcoin/bitcoin/pull/15996#issuecomment-491740946),
or fixes in a helpful way, proposing to rebase, or even offering to take over
the PR after months of silence. In short, help each other!

Try to avoid overly commenting in PRs about code style issues and nits,
particularly with PRs labeled as WIP, or when a PR has just been filed and the
PR author is mainly looking for Concept ACKs, e.g. general consensus, not
nitpicking. Long-term contributors report that activity like this repels them,
and it can diminish your social capital on the project. Try to understand what
kind of review is needed and when to do what.

The best time for any nit comments is after the Concept ACKs and consensus on
the PR, and before the PR is finalized and has tested ACKs.

Give nits and style advice in an advisory way -- as in, feel free to ignore,
feel free to adjust if you happen to rebase, etc.

Keep in mind that no one is forced to take your review comments into account;
it's perfectly fine for the author to reply that they don't want to do something
if they feel it is outside the scope of the change, especially if your comment
is nitpicky.

When you can, scale up the difficulty and priority of the PRs you review.

You can add more value and learn more by taking the time to do deep, quality
review of the [high-priority](https://github.com/bitcoin/bitcoin/projects/8)
and the more difficult PRs. These PRs tend to intimidate people and can stagnate
for months, killing their author's motivation with death by a thousand cuts from
lack of quality review, nitpicking/code style bikeshedding, and rebase hell.
Reviewing them well provides a true service to Bitcoin.

The process of ramping up takes time; nothing can substitute for months and
years invested in gathering context and understanding from following the
[code](https://github.com/bitcoin/bitcoin),
[issues](https://github.com/bitcoin/bitcoin/issues),
[PRs/reviews](https://github.com/bitcoin/bitcoin/pulls), [bitcoin-core-dev IRC
channel](https://webchat.freenode.net/?channels=bitcoin-core-dev), and
[bitcoin-dev mailing
list](https://lists.linuxfoundation.org/mailman/listinfo/bitcoin-dev).

Keep ego and hopes out of the process. Don't take things personally and keep
moving forward.

When in doubt, assume good intentions.

Be patient with people and outcomes.

Work on it every day.

These are all much easier said than done. Be forgiving with yourself and others.

John Newbery:

- A good rule of thumb is to review 5-15 PRs for each PR you make
- Start small
- Have a plan, don't spread yourself too thin
- Sharpen your tools
- Ask for and offer help, like rebasing for people or adding test cases
- People are generous with their time because they care
- Contribute by understanding what others want and being respectful


### TECHNICAL SPECIFICS

The [Bitcoin ACKs](https://bitcoinacks.com/) web app dashboard is useful for
following PR and review activity.

Be sure to learn and understand the [peer review
process](https://github.com/bitcoin/bitcoin/blob/master/CONTRIBUTING.md#peer-review).
The process is
[often](https://github.com/bitcoin/bitcoin/pull/15626)
[updated](https://github.com/bitcoin/bitcoin/pull/16149),
so refer back to it frequently.

Concept ACK means that the reviewer acknowledges and agrees with the concept of
the change, but is not (yet) confirming they've looked at the code or tested it.
This can be a valuable signal to a PR author to let them know that the PR has
merit and is headed in the right direction.

As a new contributor, be cautious with giving a NACK. Assume by default that you
might lack understanding or context. If you do NACK, provide good reasoning.
[Here's one
example.](https://github.com/bitcoin/bitcoin/pull/12360#issuecomment-383342462)

Manual testing of new features and reported issues is always welcome.

A comment that is really helpful in review: "Here's what I tested and my
methodology", particularly to back up an ACK.

While you're reviewing, adding tests yourself helps you understand the behaviour
and you can send them to the author who can add them to the PR too.

Contributing automated tests to the PR author is a really helpful way to start
contributing. Authors really appreciate it when someone reviews their PR and
provides additional tests.
[Here's an example.](https://github.com/bitcoin/bitcoin/pull/15996#issuecomment-491740946)

jnewbery/instagibbs:
- check out the branch locally
- run a difftool on each commit in turn using meld on linux and opendiff on mac
- ACK the commit hash of the HEAD commit from my local checkout of the branch,
  so unless my local tools are compromised, I know I'm ACKing the exact changes.
  It's also useful when a force push happens and links to old commits are lost
  on GitHub
- though unless you GPG sign the ACK, GitHub could just modify what you're
  writing. If you want to fully remove trust, you can go the full Marco Falke
  and sign/opentimestamp all of your review comments
- [Open Timestamps Git Integration](https://github.com/opentimestamps/opentimestamps-client/blob/master/doc/git-integration.md)

aj:
- i find "gitk" useful for getting an overview of changes

fanquake:
- I have some core dev tools in https://github.com/fanquake/core-review
  It has guides on how to gitian build, review certain types of PRs, etc
  I still need to improve the docs and push up more stuff I have sitting locally
- There is also https://github.com/bitcoin-core/bitcoin-maintainer-tools
- Rough distro list for build-related changes:
  https://github.com/fanquake/core-review/blob/master/operating-systems.md

harding:
- In case it's helpful to anyone, I took a quick look at the PR and
[made notes about my review process and what I'd test
for](https://gist.github.com/harding/168f82e7986a1befb0e785957fb600dd)

harding's process:
1. Open PR in browser
2. Checkout PR in dev environment
3. Build
4. Run integration tests
5. During and after 2-4, make notes on what I need to review
6. Answer what questions I can by looking at the code
7. Start regtest node (or testnet or mainnet if necessary) and review actual
   operation matches my expectation from the code

John Newbery:

I always download the PR branch to my machine so I can build and review locally.
I don't use the github webpage to review, only to leave comments.

My git tools: https://gist.github.com/jnewbery/1d45a6b5e14b3f3fefe5942d0cc2608d

I've got a short script that checks out the PR branch and queries the github API
to add a description to that branch locally, that makes it easier when I have a
bunch of PRs checked out locally that I can run a `git branch` command and see
what they are

once I have the branch locally, I'll set off a build in a VM while I look
through the changes

first, I run something like `git log --oneline upstream/master..`
that gives me a list of all the commits in the PR branch, one per line

and then I use a one-liner, which I have bash aliased as git-review, that steps
through the commits one-by-one, printing the commit log to the console, then
opening my difftool program:

for commit in `git log master..HEAD --oneline | cut -d' ' -f1 | tac`; do git log -1 $commit; git difftool ${commit}{^,} --dir-diff; done

I'll look at the diff, and when I quit the difftool program, git-review will
step forward to the next commit

First run through, I'll just skim everything, reading the commit logs and
looking at the overall changes, so I get an idea of what the PR is doing

Then I'll go through again, but look at each commit in more detail,
reviewing every line in detail

I make extensive use of the functional tests by adding
`import pdb; pdb.set_trace()`
break points, and then manually running RPC commands on the nodes under test

I pull the PR description from the github API and then use it to label the branch

There's an attribute of a git branch called `description` that you can update
manually, my `gb` doesn't map onto `git branch` exactly, I do some formatting on
the output so it includes the description

I'm using vagrant, just the standard ubuntu bionic image.
I have a vagrant file that installs all the dependencies.

I run make check and the functional tests, not usually --extended because I'm
too impatient.

Here's my vagrant config (YMMV): https://github.com/jnewbery/btc-dev

A toolchain for bitcoind data files: https://github.com/jnewbery/bitcointools

MarcoFalke: to run functional tests in the gui, pass BITCOIND=bitcoin-qt

To turn on debug=net with the logging RPC: bitcoin-cli logging '["net"]'

If we need any more linters it should be to check for unexpected changes to
consensus code


### ABOUT MAKING PULL REQUESTS

When you do start contributing PRs, here are some guidelines:

Focus on user problems, actual bugs, and "used, but untested" methods that
affect outcomes and need tests.

Good ideas may often be found in PR reviews and #bitcoin-core-dev IRC
discussions.

Avoid making refactoring, fixup/cleanup, or trivial spelling PRs; these consume
valuable contributor and maintainer time, often for little gain. Code churn can
cause hidden new bugs and blurs the code history and git blame. Any code
refactoring needs substantial motivation, advantages, or simplifications.

Do not begin by trying to change consensus code; it is difficult and
dangerous territory. The goal of Bitcoin Core is to maintain the correct
consensus on the Bitcoin network -- all else is secondary.

Test coverage is essential; don't hesitate to write any missing
[unit](https://github.com/bitcoin/bitcoin/tree/master/src/test) or
[functional](https://github.com/bitcoin/bitcoin/tree/master/test/functional)
tests, or improve the
[fuzzing tests](https://github.com/bitcoin/bitcoin/blob/master/doc/fuzzing.md) or
[benchmarking](https://github.com/bitcoin/bitcoin/blob/master/doc/benchmarking.md).

A Gregory Maxwell warning to test contributors: "Overly exact tests have greatly
diminished value if their exactitude turns them into hyper-active *something
changed* detectors, when they would otherwise be more useful as *something isn't
right* detectors if constructed more in terms of invariants instead of strict
behaviour."

Documentation is important, e.g. whether a function has a good description and
[Doxygen documentation](https://github.com/bitcoin/bitcoin/blob/master/doc/developer-notes.md#coding-style-doxygen-compatible-comments)
for all its arguments, and high-level documentation of how things work and
interact.

In general, PRs that intelligently improve documentation and tests in a well
thought-out way tend to be well-received.

I find it useful to keep a list of PR ideas in the form of
[observed todos](observed-todos.txt).

Don't rush; it's often better to work on code/tests/PRs without actually
proposing them as PRs... give your work time to mature and revisit it with fresh
eyes and perspective. It might be a good rule of thumb to write several WIP
PRs for each one that you do file.

Scale up your PR contributions slowly in line with your ability and
understanding of the project, process, and codebase.

Put the time in to make your PRs as easy to understand and review as possible:
Code, tests, documentation, good commit messages, relevant references, concise
explanations.

In your PR description, it's a smart habit to give reviewers tips on how to
review and test your changes, e.g. "Here's how to review this".

It is essential for commit messages and PR descriptions to explain *why*.
Summarizing "what" can be good too, but "why" is essential to review because if
the why doesn't make sense then the what probably doesn't matter. Sometimes
things seem self-apparent, but when in doubt no one was ever hurt by a little
more concise, pithy explanation.

Set up Travis CI on your own GitHub Bitcoin repository so that when you push a
branch or commit, the linter and continuous integration tests run. It's a good
idea to verify they are all green on your GitHub repository before filing a PR.

Learn how to get people to review your work when needed. It's a skill.

Persistence can pay off. Sometimes a PR succeeds on the second or
[third try](https://github.com/bitcoin/bitcoin/pull/16060) if the first ones
didn't see enough review and if there are valid reasons to continue.

In this case, [one idea seen in
practice](https://github.com/bitcoin/bitcoin/pull/16060#issuecomment-494142593)
is to recap various ACKs from the previous PRs, with GitHub usernames, to rope
in support for the new PR. If you do, be sure to do it in a comment -- not in a
commit message and not in the PR description.

Never put GitHub usernames in commits and PR descriptions; this can cause
endless annoying notifications for those concerned.

[Add info on when to add release notes]

Any release notes should refer to the RPC help for details instead of
substituting for full documentation, for example: "Please refer to the RPC help
of `getbalances` for details."

Ask yourself how others will see your PR. Try to only file PRs that people will
be motivated to review and that are worth merging, for both Bitcoin and your
track record. Your PRs will receive more attention from the contributors and
maintainers and be merged more quickly.

Continue to do more reviewing and testing than adding PRs to the (substantial)
backlog. Reviewing 5 to 15 PRs for every PR you make is a good range.

If you are a good reviewer and begin building a reputation and appreciation for
your help in moving the project forward, maintainers and other contributors
might reciprocate by reviewing your work as well.


### BOOKS

#### C++ Books

Bitcoin Core is written in C++ 11. Here are some C++ book recommendations:

The Reference

- The C++ Programming Language, 4th Edition

Learning C++

- Accelerated C++ (condensed/faster -- from 2001, a bit out of date, but
  excellent)
- C++ Primer, 5th edition (more gentle/longer -- not to be confused with the
  knock-off, "C++ Primer Plus", which is not recommended)

Intermediate/advanced

- Effective C++
- Effective Modern C++
- Effective STL
- Exceptional C++
- More Exceptional C++
- Modern C++ Design: Generic Programming and Design Patterns Applied
- C++ Concurrency in Action

#### Cryptography

- [Foundations of Cryptography](https://www.amazon.com/Foundations-Cryptography-Basic-Tools-Vol-dp-0521791723/dp/0521791723/)
- [Waxwing's blog](https://joinmarket.me/blog) by JoinMarket expert
  [Andrew Gibson](https://mastodon.social/@waxwing)


### CREDITS

A special thank you to [John Newbery](https://twitter.com/jfnewbery)
for launching the
[Bitcoin Core PR Reviews Club](https://bitcoin-core-review-club.github.io/) and
to the long-term contributors who participated so far:
[Dave Harding](https://hash.social/users/harding),
Anthony Towns, Gregory Sanders, Michael Ford, Andrew Chow, Pieter Wuille,
Bryan Bishop, Mike Schmidt, and Marco Falke.

Thanks to [Steve Lee](https://twitter.com/moneyball) for reviewing this
write-up and his suggestions.

This article includes observed comments on GitHub and IRC by the following
Bitcoin Core contributors/maintainers who deserve to be credited:
Wladimir J. van der Laan, Marco Falke, Pieter Wuille, and Gregory Maxwell.

Over the years I had become disillusioned by the central influence of BDFLs in
programming languages and open source projects. Wladimir van der Laan's
[long-standing](https://twitter.com/orionwl/status/1131564038444453889)
[humble](https://twitter.com/orionwl/status/1131827793908645888)
[service](https://twitter.com/orionwl/status/1131924832071880705) to Bitcoin
sparked the possibility to me of perhaps doing the same.

Finally, a big thank you to the Core contributors for their patience with my
stumbling review attempts so far, notably John Newbery, Marco Falke, João
Barbosa, practicalswift, Gregory Sanders, Jonas Schnelli, Pieter Wuille and
Wladimir J. van der Laan, and to Adam Jonas and John Newbery for their guidance
and advice from the start.

Cheers,

[Jon Atack](https://twitter.com/jonatack)
