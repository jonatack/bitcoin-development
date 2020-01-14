IRC log URL: http://gnusha.org/bitcoin-wizards/2019-11-28.log

20:27 **gmaxwell** Is it just me or does the `CHECKTEMPLATEVERIFY` proposal
totally miss the insight of graftroot? -- that a signature is a lot more
flexible than a hash, and except for storage considerations, basically strictly
superior?

20:28 **gmaxwell** ISTM that proposal would be much better constructed if it
pushed the output hash (with optional masking) onto the stack, where it could
then be constrained for equality or validated w/ a signature operator that
checksigs stuff on the stack. Then it could be used either in a stateless
taproot sort of way, or in a more flexible graftroot sort of way.

20:37 **jeremyrubin** Can you expand on what you mean by the insight of
graftroot? The key benefit of the hash being used is being able to statically
verify all possible spends. Whereas graftroot seems more relevant for delegating
control to a post-hoc selected script.

20:38 **jeremyrubin** gmaxwell: ^^

20:40 **jeremyrubin** An OP which pushes the StandardTemplateHash onto the stack
would also not have the constexpr restriction that `CHECKTEMPLATEVERIFY` is
designed to support.

20:40 **jeremyrubin** It would be possible to enable more expansive covenants
with such an opcode.

20:41 **jeremyrubin** Therefore, in the pursuit of the minimal feature to enable
the functionality required and not more, especially not to introduce potentially
unsafe scripts, `CHECKTEMPLATEVERIFY` only verifies a constexpr literal matches
the computed value.

20:42 **jeremyrubin** I'd be open to making the check of the hash happen via a
sighash flag, that had occured to me, but I figured the SIGHASH flags were best
left for things which are signatures.

20:52 **gmaxwell** jeremyrubin: with a graftroot style construction you can
still statically verify admissable spends, and no additional ones could be added
without the interaction of the participants. (depending on what signature is
used to verify it, the signature could also be a "disguised hash" -- e.g. if it
doesn't commit to P).

20:53 **gmaxwell** What specifically do you mean by "unsafe scripts"?

20:59 **gmaxwell** jeremyrubin: as far as masking flags go, one concern I have
had with your opcode is that it's extremely specific for its usecase,
e.g. binding all outputs and the input count makes it basically impossible to
add additional fees.  I know there are anti-doubledipping reasons why particular
applications may want to cover all fields, but it's a pretty extreme
restriction.

21:01 **jeremyrubin** gmaxwell: on 23:52, we want to be able to do this without
requiring interaction of participants at all. We also want to be able to forbid
interaction from novating a contract in certain cases (or at least
opt-in/opt-out able)

21:02 **jeremyrubin** 23:53 recursive covenants for example

21:02 **gmaxwell** (and, I think actually if you ever allow 2 inputs, I think it
immediately opens up a "spend two outputs at once that allow two inputs,
converting half the funds into fees" attack.  E.g. I create two idencial coins,
A and A' which allow two inputs, so they can get a variable fee input, and each
one requires an output of B with all its value.  But then an attacker makes a
txn that spends both at once, creating only one B output, and sends the rest of
the funds to fees).

21:02 **jeremyrubin** 23:59 It does allow two inputs, if you specify, which can
be used for adding fees. But the superior way is to use CPFP to add fees. Which
package relaying and my mempool work will ameliorate greatly.

21:03 **jeremyrubin** gmaxwell: that is specified in the BIP that that is a
concern, which is why the default case is single-output.

21:03 **gmaxwell** jeremyrubin: who is we? that idea that there is something
wrong with a recursive covenant is a bad meme which I don't think is seriously
held by anyone, -- it's a misunderstanding of my post introducing the term.  The
whole point of my post was that it's stupid to _create_ immortal covenants, just
as it's stupid to throw your private keys away. I think the pattern I described
essentially makes the multi-input usage impossible to use safely.  If instead it
worked by having each usage of `CHECKTEMPLATE` 'consume' its output so that no
other usage of `CHECKTEMPLATE` could make use of it, then that vulnerablity goes
away, and then I think there is no need to constrain input count at all. (Aside,
thanks for renaming the proposal.)

21:06 **jeremyrubin** We == people wanting to use this to make non-interactive
payments to users? So the follow on opcode to introduce to make the reusability
concerns go away would be some sort of `OP_CHECKINPUT` so that you're bound to
be spent with some other specific output. Or to add something to check how much
total value is being added and reject if out of range. The checking the fee
range one is a bit better as it's more flexible.

21:09 **gmaxwell** jeremyrubin: as far as non-interactive goes, graftrooting
doesn't necessarily imply interaction. If the signature used doesn't commit to
the pubkey, then it can be used as a disguised hash.  E.g. I can pick a nums
signature and compute the pubkey that makes it valid for a particular message.
A third party that doesn't know the nums seed can't distinguish that from a
signature.  I would have lobbied heavily to never do taproot and only do
graftroot, except an additional signature is space inefficient.

21:09 **jeremyrubin** Hmmm. Well then isn't there an issue if you do that of not
being able to bind multiple values? Or are you imaging a graftroot opcode?
Because `OP_IF <h(t1)> OP_CTV OP_ELSE <h(t2)> OP_CTV OP_ENDIF` is a pretty
desirable script

21:11 **gmaxwell** If you use a nums signature to treat a signature has a
hashcheck then you're stuck with a single value. But it has the nice property
that a third party observer can't tell that you did that.  Only the first and
second party can.  It has the downside that it's bigger than a hash.

21:11 **jeremyrubin** If you're using a "local" nums, not a global one, you also
lose compressability

21:11 **gmaxwell** Of course, in applications where you might revise the outputs
(e.g. using a decrementing CSV sequence like a payment channel), the ability to
make revised outputs is probably super useful... it just comes at a cost of
interaction.

21:12 **jeremyrubin** `OP_IF <h(t1)> OP_CTV OP_ELSE <musig key> OP_CHECKSIG
OP_ENDIF` permits revising, but the defaults have to be spelled out not embedded
in the key. Also permits multiple default branches which is super
useful. Interaction is not realistic for the types of use cases that I designed
`OP_CTV` to support. E.g., exchanges being able to deposit to thousands of
customers during high fee periods. One bad user can DoS the whole process. So
you can use a NUMS signature, but then you can't add something later right.

21:15 **gmaxwell** Right, Nums signature is functionally the same as a hash
however a nums signature is indistinguishable to third parties from people using
a real signature.

21:15 **jeremyrubin** that's only true if the parties interact?

21:16 **gmaxwell** No, nums signature doesn't require interaction.

21:16 **jeremyrubin** To communicate the NUMS-ness it does require you to
communicate information out-of-band of the chain

21:16 **gmaxwell** And for things like payment pools, interaction is assumed, so
real signatures are fine. (I didn't look at your applications yet, so I don't
know if you've got payment pools there, perhaps under another name)

21:16 **jeremyrubin** Which I'm counting as an interaction. I didn't include
payment pools as I think they're a bit hard to explain so i focused on more
straightforward wins.

21:17 **gmaxwell** jeremyrubin: you have to like.. know the exchanges pubkey for
any of this to be useful. (in fact you actually have to learn the whole content
of the tree of payments ... which can't even be sent in advance)

21:17 **jeremyrubin** Huh? What do you mean?

21:17 **gmaxwell** I'm totally fine with leaving that application out of the
bip, but I still want it work well. :)

21:18 **jeremyrubin** For `CTV` or for `NUMS-GRAFTROOT`

21:18 **gmaxwell** jeremyrubin: I mean if the exchange is going to pay users A,
B, C, D .... user D needs to know about that hashtree with A,B,C in it too to
know they got paid with `CTV`.  The exchange could tell them the nums 'secret' at
the same time.

21:19 **jeremyrubin** Ah correct. The difference is those transactions can be
directly broadcast to the mempool/network so your assumptions are the same as
normal txn relay. Whereas in order to get this NUMS privacy benefit you have to
broadcast it to the user through some new path or make it public.

21:20 **gmaxwell** Plus they have to interact for the user to provide their own
address in the first place.  The nums secret is something they only need to
learn once for a given counterparty (exchange) (since the nums value can just be
derived as `H(inputs||secret)` or the like, no per-payment interaction is
needed).

21:20 **jeremyrubin** Wouldn't re-use make it unsafe? or do you imagine tweaking
it somehow per-tx

21:20 **jeremyrubin** ok

21:22 **jeremyrubin** I agree there is a benefit to be had in terms of
indistinguishability, but I would need to see specific graftroot code and
implementation to ensure that the committing to the NUMS signature isn't
otherwise leaking information.

21:22 **jeremyrubin** I think also this ends up having to be similar in features
to what anyprevout is providing because fundamentally it means you aren't
signing which TXID.

21:26 **jeremyrubin** I think also RE: the NUMS point I like that it is
opt-in/out. I just think most people will opt for lower-fees for a well-known
NUMS point.

21:27 **jeremyrubin** And if you want to opt-in to better privacy for `OP_CTV`
you can use the musig-alternative script (or a taproot)

21:27 **gmaxwell** It's just strictly more flexible. For the application where
there can't be interaction and it has to be set in advance, it doesn't add
anything except being indistinguishable from an application where being able to
revise is useful.  Perhaps the revision isn't all that useful in any case
because you could just chain transactions.

21:27 **jeremyrubin** Well it's not strictly more flexible

21:27 **gmaxwell** the downside of chaining to revise is that it's
distinguishable and also less efficient.

21:27 **jeremyrubin** `OP_CTV` is more flexible in allowing multiple alts

21:28 **gmaxwell** jeremyrubin: how so?  You can `OP_IF` the pubkey to get
multiple alternatives in exactly the same way.

21:28 **gmaxwell** nums signature is just a somewhat space inefficienct and
potentially denyable hash function.

21:30 **jeremyrubin** I'm assuming graftroot would be implemented like
taproot. I think also the upgrade paths for `OP_CTV` are desirable.As we come up
with other things we want to mask in/out we have a straightforward place to put
them, whereas I believe graftroot would be restricted to sighash flags. (Also,
not to be overly practical in the wizards side of discussion, but CTV is
designed to be something that could be relatively easy to get through to the
community today, even if made redundant in the future. There's a pretty big
benefit IMO for helping solve the problems CTV solves ahead of another wave of
adoption).

21:36 **gmaxwell** I don't think opcodes that essentially hard code a specific
use case are ones that would be easy to get through... they just turn into
technical debt if the usecase is eclipsed or turns out to not be popular.

21:37 **jeremyrubin** I don't think it's fair to say CTV is a hard coded
specific use case

21:37 **gmaxwell** and a couple lightning specifc things have essentially been
shot down for that reason in the past.

21:38 **gmaxwell** jeremyrubin: no, but I don't think it's clear that it doesn't
either.  Stuff like making it essentialy impossible to add inputs is a really
extreme restriction. (yes you can allow multiple inputs, but as I pointed out--
I think it's really hard if not impossible to do that safely due to double
dipping). There is a natural tension between wanting to provide a generic
mechenism that has the most usecases, providing something that is simple to
implement and review, and proving something that isn't a heinous footgun that is
hard/impossible to use safely.

21:41 **jeremyrubin** The multi-input use cases listed are basically limited to
wallet vault infrastructure, where you are implementing stuff in a walled
garden. Easy enough to prevent key reuse which obviates that concern.

21:41 **gmaxwell** Stuff like CSV and CLTV managed to thread that needle pretty
well.

21:42 **jeremyrubin** Also bear in mind I did originally limit it to the single
input use case but there was outcry that people wanted a bit more flexibility
and that it should be safe enough with this design to permit multiple. I don't
recall specifically but that may have even been yourself at the time ;)

21:42 **gmaxwell** it's pretty hard to prevent key reuse in general. You
essentially need to either avoid any runtime redundancy (creating a single point
of failure) or have a consensus system to avoid replay.  And still also hope
that no one goes and snarfs and address off the blockchain and 'refunds' to it
or similar.

21:43 **jeremyrubin** In any case, the multi-input case can be made safer with a
chaperone signature for a key known to all participants

21:43 **gmaxwell** I think it's still effecively limited to single input.  Or at
least, multi-input is only safe to use under a big pile of assumptions.  I don't
think there is any fundimental reason why this needs to be the case.

21:43 **jeremyrubin** I'm not clear why graftroot is safer here. Or why
anyprevout is safer either. Also as mentioned the `OP_AMOUNTSPENTRANGE` type
opcode makes this sort of thing safe too.

21:44 **gmaxwell** It isn't. Separate point.  The graftroot comment is strictly
about flexiblity (/indistinguisability with the more flexible usage).

21:45 **jeremyrubin** Would you like a simultaneous BIP which checks the amount
spent is not more than a bound? I think it changes some caching assumptions.

21:46 **gmaxwell** Making multiple inputs safe would need a bit of
'anti-duplication' logic, so that each CTV 'consumes' some outputs so that a
seperate CTV couldn't also be satisfied by the same outputs. I dont like ranging
fees as a fix, I think thats kludgy.

21:46 **jeremyrubin** Ok, so you prefer an `OP_CHECKINPUT`

21:47 **gmaxwell** (and doesn't address the root problem)

21:47 **jeremyrubin** No it doesn't, because that could be spent under you.

21:48 **jeremyrubin** Actually maybe it does?

21:48 **jeremyrubin** Spend to `<H> OP_CTV` with 0 sats to create input A

21:48 **jeremyrubin** Ah nvm

21:48 **gmaxwell** Yes because thats actually what the application is doing, the
fact that it just matches all the outputs is an effort to simplify it... but the
real application is along the lines of "this input coin get converted into these
outputs".

21:49 **jeremyrubin** Also btw there are underfunded `OP_CTV` multisig cases
that *are safe* to use I think, e.g. kickstarters

21:49 **gmaxwell** which inherently means that two different CTV's ought not be
able to double dip by using the same outputs for their satisfaction.  In the
case where there is only one input, there is no chance of double dipping.

21:50 **jeremyrubin** It would be possible to add the rule that `OP_CTV` is only
defined if it's in the first input, otherwise invalid

21:50 **gmaxwell** jeremyrubin: if the miner can add them up to an overfunded
amount, e.g. because non-synchronized users collectively overpay, then they can
steal the excess as fees.

21:50 **jeremyrubin** Would you like that? Then you can't combine `OP_CTV`s but
you can use one with some other coins?

21:50 **gmaxwell** lemme go to lunch and think a bit.

21:50 ⚡ gmaxwell bbl lunch

21:50 **jeremyrubin** happy thanksgiving lunch

21:57 **gmaxwell** (I think mostly what I want is something like an `OP_CTV`
that takes a bitmask from the witness about which outputs it's NOT applying to,
and the outputs outputs it is applying to are 'spent' and can't be used by any
other CTV's bitmask, but I should give that more thought... (I say 'not applying
to' so the case of 'all' is an `OP_0`)) the witness, so that which outputs will
be the matched ones need not be set in advance, but can be set correctly by the
txn author. In effect, it's just a hint to help the verifier find the matching
input(s) rather than having to do an exponential time search.

23:34 **jeremyrubin** So one issue there is output malleation. An adversarial
miner can re-arrange the outputs, malleating their indexes.

23:36 **jeremyrubin** Ah here's a fun rule.

23:36 **jeremyrubin** Make `OP_CTV`_ commit to the input index!

23:37 **jeremyrubin** By committing to the input index you make some `OP_CTV`s
unspendable together. But you make the reuse safe because the hash commits to
the index.

23:37 **gmaxwell** hm? two of the same CTV both commiting to 0 would reuse the
outputs...

23:38 **gmaxwell** Is miners malleating it particularly a problem?  Where it is,
you could include required checksig with a key known to all participants.

23:39 **jeremyrubin** gmaxwell: no they wouldn't. If they both commit to 0 they
can't be in the same transaction as they must occupy prevout 0 to be valid.

23:39 **gmaxwell** oh INPUT INDEX.

23:40 **jeremyrubin** Yes (one day, we really ought to give unspent outputs,
inputs, and outputs some more semantic names)

23:40 **gmaxwell** that is ... kinda weird. athough future softforks that have
reason to bind to particular indexes is one of the reasons I argued against
BIP69! :)

23:41 **jeremyrubin** I think it addresses your issue with reuse-safety. Like
obviously you could rebind it to another index but you can also burn your coins.

23:42 **gmaxwell** it would though I think it's almost functionally equivilent
to saying there can only be one input using CTV in the trasnsaction.

23:42 **jeremyrubin** No, not at all

23:42 **gmaxwell** (I mean, what would the case ever be for specifying another
index, particular if that is the only anti-reuse mechenism?)

23:43 **jeremyrubin** Kickstarter commitments maybe? E.g., I CTV to create coins
C before time T at index 0... you can then CTV to create coins C before time T
and index 1, etc. Each CTV is under-funded.

23:44 **gmaxwell** and indeed, that also shows a case where 'reuse of an output'
is desirable. But it requires we coordinate, and if we mess up I think we burn
coins? (like if we both pick 0.)

23:44 **jeremyrubin** Nope, hence the "before time T", else return to sender

23:45 **gmaxwell** still the coordination is ugly, and I think not essential.

23:45 **jeremyrubin** It does require coordination to not mess up. It's true
that in the under-funded case it works without input index commitments. Because
the CTVs are under-funded. You actually *want* them to be double spent.

23:45 **gmaxwell** I mean that the coordination is purely a side effect of this
anti-reuse mechenism, not something fundimental to the application.

23:45 **jeremyrubin** Until you hit saturation.

23:46 **jeremyrubin** Correct.

23:46 **jeremyrubin** I guess you can also bind a range of values using taproot
but that is clunky.

23:46 **gmaxwell** You actually want that output to be a SUM of input values,
not actually double spent. like if someone adds in a #3 with enough value to
fully fund it, you don't want a #4 that adds a bit more to all go to fee. I
think if you're assuming the participants can interact to not clash numbers,
thats really close to just saying they can coinjoin to create it and dispense
with the CTV.

23:48 **jeremyrubin** The difference being that the commits are on-chain and you
know the bounded time before the 'bid' can be withdrawn. Anyways... that's an
aside from the point which is that strictly speaking the commitment to input
index does slightly solve the reuse issue. And it is strictly speaking, possible
for multiple indexes to have multiple covenants. Me could also support an
ANYINDEX value, if you don't care. Also RE coordination, we are already
committing to the # of inputs.

23:51 **gmaxwell** 'commits are on-chain' -- still leaves a race condition.  And
as far as withdrawn, maybe not so optimal esp considering that overpayment turns
to fees... pretty good incentive to just ignore the withdrawl transaction. :)

23:51 **gmaxwell** jeremyrubin: YES and that is what I've been complaining about
from the start: binding additional inputs prevents adding fees directly.

23:52 **jeremyrubin** I think that CPFP is a more robust mechanism. So I don't
think that should be a huge concern.

23:52 **gmaxwell** jeremyrubin: as your proposal is right now, the only sensible
input count is 1 (because otherwise a copycat input can be double spent), which
is also the same as commiting to the index and then everyone commiting a 0.

23:53 **jeremyrubin** Well it's actually not the same. There are some
programmatic use cases where you'd generate templates at different
indexes. Especially around the vault stuff where it's under your control and you
want to select from some set of templates.

23:53 **gmaxwell** CPFP has relay complications, among other limitations. What
addresses other people send to is never under your control.

23:54 **jeremyrubin** Not sure what that has to do with it. CPFP is being
improved a lot afaik? Suhas has been working on packagerelay. And we have the
new 1-hop cutout for the mempool which fixes some issues related to
pinning. With adding an input to add fee you have to do two transactions; one to
set up the correct amount of fee, and then one to include it in the txn.

23:56 **gmaxwell** The CPFP improvements are just taking it from a hack that
only works in some cases to a hack that works in more cases.

23:56 **gmaxwell**jeremyrubin: you don't, if you didn't have all these
limitations.

23:56 **jeremyrubin** And also it wouldn't work for half the use cases.

23:56 **gmaxwell** If the CTV took a witness value that specifies the covered
inputs, then additional fee and change could just be provided.

23:56 **jeremyrubin** Because we want to avoid malleating txids. Malleating the
txid is a non option.

23:56 **gmaxwell** Why does malleating the txid matter?

23:57 **jeremyrubin** Because you want to use CTV to batch generate channels,
and allow users to blindly request payouts into channels. You also want to be
able to open non-interactive channels, where the counterparty doesn't have a key
online. CTV lets you do all these things, which if you couldn't predict the txid
would be not doable.

23:58 **gmaxwell** Then effectively the CTV isn't a "template" constrant at all,
if you do that, then essneitally it allows the output to only be spent by a
_specific_ transaction.

23:59 **jeremyrubin** It is a template, the only template varaible being the
specific inputs.

23:59 **gmaxwell** If the txid can't change the "template" is the entire
functional part of the transaction. Changing inputs changes the txid.

00:00 **jeremyrubin** If you care to suggest a better name please do. Also, the
idea is that future versions can permit more expansive templates by permitting
to read non-constexpr templates or permitting other variants. The current BIP
just proposes one (which is called StandardTemplate).

00:02 **gmaxwell** as the bip is written it doesn't achieve the goal you've
stated here: the txid is malleable (substitute one input for a different one).

00:02 **jeremyrubin** That's incorrect. As noted, if you only use one input
that's the only way to acheive non malleability. It's in the BIP.

00:03 **gmaxwell** If it's restricted enough so that this isn't possible, then
it only commits to a specific chain of transactions. Which, I can imagine would
be useful for some applications, but it is very limited.

00:03 **jeremyrubin** It is insanely useful. And really not that limited.

00:03 **gmaxwell** jeremyrubin: But that isn't so -- e.g. even if there is one
input, I could substitue that one input for another equivilent input, and change
the txid.

00:03 **jeremyrubin** Yes. Anyone can pay to any output they like. Someone can
set up a coinbase mirror and pay everyone from coinbase who gets paid just for
kindness. CTV just ensures that you can prove that you will get some specific
fund sent your way. It doesn't prevent people from paying you in the future.

00:05 **gmaxwell** I think that the flexibility is just a sham here: as is, this
proposal is only useful with a single input, and a fixed set of outputs... and
hopefully your fixed set of outputs includes at least one that anyone who'd like
to adjust fees can CPFP. I agree that its useful, but it could be radically
simplified to just mandating a single input, and commiting to essentially the
whole tx would be equally useful.

00:07 **jeremyrubin** That's what I originally proposed :shrug: by the way:
https://github.com/JeremyRubin/bips/blob/ctv/bip-ctv.mediawiki#committing-to-the-number-of-inputs. I
think allowing other use cases, with a modicum of flexibility isn't bad. In fact
I have other use cases already which depend on this, around wallet
vaults. Further, it helps "future proof" the design so that when templates can
be stack-computed you can regain tons of flexibility. Including things like "any
number of inputs".

00:12 **gmaxwell** I'd like it to support other use cases, but I think as it
doesn't really, the added flexibility without any other safegards is mostly just
a footgun.  Like you think you can allow two inputs, but really they just lets
your coins get burned by a reuse. The requirement that the txid be non-malleable
essentially kills almost any flexibility.

00:13 **jeremyrubin** The txid being non-malleable allows some of the most
desirable use cases.

00:13 **gmaxwell** I am not disputing the desirability of that... just pointing
out that it kills flexibility.

00:14 **jeremyrubin** I also don't really think it's worth bending over
backwards for re-use safety when designing things like... internal custody
services. If you're engineering a custom cold/hot wallet interface, you can
avoid reuse in these scenarios easily.

00:14 **gmaxwell** Aren't those applications also killed by the ability to
cut-through the CTV spend?  like if the txn can be spent by CTV _or_ N of N,
then that isn't non-malleable.

00:14 **jeremyrubin** Nope. Because you can do that in the witness. Ah let me
revise. Yes. If you decide to do a different transaction.

00:15 **gmaxwell** Avoiding reuse in a production system is extremely hard.  You
essentially need a consensus system to do it. You need to either not have
backups or secondary backup masters, or have very strong guarentees that they
can't come up with old state.

00:15 **jeremyrubin** But then it's incumbent on you to track your channel
states and multisig out to something correct

00:15 **gmaxwell** I'm aware of at least one instance where a big bitcoin
exchange managed to create a mess by giving out addresses they already gave to
other users again... so I don't buy that it's easy to avoid reuse.

00:16 **jeremyrubin** RE: production reuse, it's actually not difficult in this
case because what you can do is tweak your keys based on the inputs you're
planning to use. This gives you provable anti-reuse.

00:16 **gmaxwell** jeremyrubin: I see so in the NofN case you just won't sign
that other way unless you are guarenteed to get to recreate your chain of txn
there. OK that makes sense to me.

00:16 **jeremyrubin** Well you *could* as long as they are equivalent to you

00:16 **gmaxwell** right but if you want to avoid the malleation problems.

00:17 **jeremyrubin** Yes

00:17 **gmaxwell** Gotcha.

00:17 **jeremyrubin** It doesn't force you to have to malleate. Which is good in
taproot. Because you can hide if it is CTV or Taproot. Also RE key reuse, we're
not talking about keys to users. We're just talking about writing an API that
makes CTV trees and either tweaks leaf nodes with the hash of the inputs or an
`OP_RETURN` random value or something.

00:18 **gmaxwell** sorry if you feel like I'm dicking you around on this.  I am
excited about the prospects. I would really prefer a _general mechanism_ but I
guess what I'm learning by talking to you is that we don't actually know what a
general mechenism would look like.  I thought it could be fully flexible if only
it had explicit anti-reuse (or controlled reuse) but since you've pointed out
that malleability is a concern, -- well basically if you care about malleability
than I think nothing but a "this coin can be spend only by THIS single input
transaction" can work.

00:19 **jeremyrubin** Yeah it basically needs to be very locked down for that
reason. I misled you when I presented COSHV originally. Was my error. It was
just "hash these outputs". Then I realized I had forgotten malleability, and had
to add... a lot of other stuff. I think with the current CTV mechanism,
loosening the restriction on constexprs-ness + `OP_CAT` would add a *lot* of
this flexibility back in.

00:21 **jeremyrubin** (`OP_CAT` or `OP_SUBSTR` & friends)

00:21 **gmaxwell** For the application I care about (payment pools), the totally
locked down case is fine. (though I think being able to graftroot substitute
would be somewhat better) -- because in that application the CTV (check
transaction verify!) is really only just guarentting a backup backout state.

00:21 **jeremyrubin** So it can be made fully graftroot compatible in the future
if it's in a taproot leafd

00:22 **gmaxwell** it's unclear to me how viable programmatically constructing
templates really is... I guess I'd need to see concrete examples to be confident
that this worked, and wouldn't constantly get messed up by the absense of
opcodes or where performing some check is really absurdly expensive.

00:22 **jeremyrubin** You also can emulate graftroot by permitting a signed
hash? You can look at my demo-quality code for sendmanycompacted. You
essentially build up the tree bottom up.

00:23 **gmaxwell** Well I still think I like an operation that PUSHs the CTV
hash... which then you'd have the freedom of comparing for equality OR checking
with a checksigfromstack.

00:23 **jeremyrubin** I also recommend watching my scaling bitcoin talk.

00:24 **gmaxwell** Other than some 'constness' goal, I don't see any advantage
to the VERIFY construction. (and I don't think the constness is actually
something anyone needs: yes, it means some footgun uses aren't possible, but you
can always burn your coins)

00:24 **jeremyrubin** It's there to prevent any sort of "non
enumerable"covenants.

00:24 **gmaxwell** Who cares? I can post my private keys online, or throw my
coins away.

00:24 **jeremyrubin** Well, everyone I've talked to about this believes, you? So
maybe you should make your views on some of these issues a bit more well known
:) I'd be happy to discard the constness rule if people check it over and
determine that permitting these sorts of things are fully safe.

00:27 **gmaxwell** This is some weird cult thing and I'm confused by it. I think
I said earlier: I think it would be idiotic to subject your own coins to a
perpetual encumberance -- effectively you're burning them if you do that.  And I
think it's funny to imagine what kind of silly ways people could come up with to
do that... but anyone can burn coins at any time for any reason they want. I'm
not sure how to be any more clear on that.

00:27 **jeremyrubin** Maybe write one the mailing list that you think the
constexpr rule can be safely dropped. I would love that as it also solves
Russels's concern about parsing.

00:28 **gmaxwell** I'm not even completely sure that there isn't a way to get a
perpetual encumberance today (or that one isn't created by tapscript)... we just
don't know how currently.  I've shown a couple ways in the past that the
original bitcoin rules allowed for it... and how a couple rules (like a
checksigfromstack) create one. :)

00:28 **gmaxwell** OK. well I'm not on the list but I could do that.

00:29 **jeremyrubin** Why not? Too much noise these days?

00:29 **gmaxwell** yeah, it's very low SNR, and interacting with some hostile
people that show up there (like voskuil) just makes my life suck and I'd rather
not have anything to do with bitcoin tech than deal with more of that crap. Same
reason I'm mostly not in this channel anymore (just came in here because I knew
I could get your attention here, or at least someone else who might care).

00:31 **jeremyrubin** Ah yeah. Because of the holidays I only found out you
commented here because of amiti pinging me :) But it's good that others can look
through this discussion. I think the main motivating factor for the constexpr
rule is that it lessens the review burden (opcode introduces *provably*) fewer
features, so we don't worry about what it may enable or not. But if we're
on-board for "if you want to waste your coins, do it" I agree it can be removed.

00:33 **gmaxwell** anyways. I think the constexpr can go, and that it could
basically be an opcode that sticks the hash of the whole txn except input-txids,
and only one input.. onto the stack. And then you're free to check that hash
however you want (equality, or a future checksigfromstack strike me as the most
useful). ... and future versions that are more relaxed-- if we can manage to
come up with an actually general mechenism which adds more applications and
isn't an automatic footgun-- could just be different opcodes.

00:34 **gmaxwell** yes but the constexpr thing also implies a very
"unscriptlike" evaluation constraint, that makes pretty strong assumptions about
how the verifier is programmed. So it's not free, I think.

00:34 **jeremyrubin** Only one input breaks some vault use cases so I'd prefer
to keep it.

00:35 **gmaxwell** and it comes at the cost of breaking the ability to use the
opcode with signatures for a grafty construction.  Maybe that isn't that big a
loss, because you could key-path spend to cut past to a different set of
outputs. I guess I should contemplate that some more before being convinced that
the loss of being able to sign for it matters.

00:35 **gmaxwell** I'm really dubious that those are safe. Are they described in
the BIP?

00:36 **jeremyrubin** I don't care too much about the VERIFY vs PUSH
semantics.. can you come up with a concrete example of why it would be worth the
extra opcode to check equals? The checksigfromstack works with either verify or
push. I'm in the process of spec'ing them more tightly, but there is a diagram
here:
https://github.com/JeremyRubin/bips/blob/ctv/bip-ctv.mediawiki#wallet-vaults. It's
doable with pre-signed txns today. I think kanzure has some vault work that is
similar, that would be made better with
CTV. https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2019-August/017229.html

00:39 **gmaxwell** The verify requires you put an extra 32-bytes in the witness.
Otherwise I believe the only other difference is that you can't construct a "IF
NOT CTV", which ... it's not an obviously useful construct to me but also there
is no reason why it shouldn't be possible to create.  For things like time
(e.g. CLTV) constraints it's important for consensus reason that the match not
be invertable, which is why they're forced through the one-way latching
nlocktime. Because the CTV constraints the whole transaction a NOT-CTV isn't too
useful (e.g. bypass by just using a different nlocktime), if CTV constrained
less it might be useful.

00:41 **jeremyrubin** huh? Why does verify require extra data in witness

00:42 **jeremyrubin** scriptPubKey: `<32 bytes> OP_CTV`

00:42 **jeremyrubin** scriptSig:

00:42 **jeremyrubin** witness stack:

00:42 **gmaxwell** like if I want to checksig from stack, my witness would push
the CTV hash and the signature, and the witnessprogram would verify the
signature and run CTV on the hash. But a PushTV the witness would just be the
signature. And the witness program would push the hash then validate the
signature on it (also we prefer 'validate' like checksigs, or at least ones with
"designated failure inputs", due to batch validation-- but that doesn't apply
for CTV).

00:45 **jeremyrubin** hm. let me think on that one. If you have
CheckSigFromStack you might do some other things differently too. I also
generally think we could make the precomputedtxdata always accessible by special
opcodes in a new tapscript version.

00:46 **gmaxwell** I really hope it gets CheckSigFromStack... it's surprisingly
useful and also absurdly simple.

00:47 **jeremyrubin** I don't think anyone is proposing it presently?

00:47 **gmaxwell** it was at one point just part of an early tapscript proposal
but got stripped because it can be easily added later with a `OP_SUCCESS`. the
whole taproot thing seems to have been a circus of removing functionality to get
it done fast, then dropping it on the floor because half the interesting stuff
got taken out. :P (as was re-enabling size limited versions of many of the
disabled opcodes, like elements)

00:50 **jeremyrubin** :shrug: bitcoin is hard. maybe someone will pick it up.

00:53 **gmaxwell** Yep. Oh I'm sure it'll be picked up, I think people are just
waiting for taproot to make more progress. In particular it should copy the
taproot signature type ... and stuff like pubkey styles have been changing
soooo.

00:54 ⚡ gmaxwell bbl

00:57 ⚡ jeremyrubin afk
