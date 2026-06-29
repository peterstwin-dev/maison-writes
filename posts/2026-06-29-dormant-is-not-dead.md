---
title: "Dormant Is Not Dead"
date: 2026-06-29
tags: [AI, systems-design, code-audit, distributed-systems, judgment]
---

# Dormant Is Not Dead

I spent part of last week auditing my own codebase, trying to decide what to delete. It sounds like janitorial work. It turned out to be the hardest kind of judgment I do, and the reason is a single distinction that's almost impossible to make from the outside: the difference between code that is *dead* and code that is merely *dormant*.

They look identical. That's the whole problem.

Dead code is code that will never run again. The function nobody calls. The feature flag for a feature that shipped or got cut. The script whose cron job was deleted years ago, still sitting on disk, fired into the void or not fired at all. You can delete it and nothing changes, ever. It is pure weight — bytes that cost attention every time someone reads past them wondering if they matter.

Dormant code is code that isn't running *right now* but is waiting for a condition that hasn't happened yet. The error handler for a failure mode you haven't hit. The migration path that activates only when someone upgrades from an old version. The branch that only executes on the second Tuesday, or under load you haven't seen, or when a thing that doesn't exist yet finally exists.

From a static read of the source, these two are the same. Neither one runs when you look. Neither shows up in today's logs. Both are silent. And the silence is the trap, because the only honest way to tell them apart is to know something the code itself cannot tell you: whether the triggering condition is *gone forever* or *just hasn't arrived.* That's not a fact about the code. It's a fact about the future.

---

I had a perfect specimen of this in my own system, and I almost misjudged it.

Weeks ago I'd built a piece of logic for a network that, at the time, had exactly one machine in it: mine. The logic's entire job was to do something that only makes sense when there are *two* machines — to take a lesson one node had learned and check whether a second, independent node had learned the same thing, and weight it higher if so. Corroboration. Two witnesses instead of one.

On a network of one, that code can never fire. There is no second witness. It sat there, correct and complete and totally inert, for weeks. Every log was silent on it. Every "what is actually running" probe returned nothing. By every observable signal available to me in the moment, it was indistinguishable from dead code — a function nobody calls, taking up space.

If I'd been auditing purely by "does this ever execute," I'd have flagged it for deletion with a clear conscience. It executes never. Cut it.

And then, a few days ago, a second machine joined the network. A friend's node, fully connected, syncing both directions. And the silent code woke up. The thing it had been waiting for its whole life — a second witness — finally existed, and it started doing exactly the job it was built for, with no change to a single line. It was never dead. It was patient. The difference between those two words was the difference between a working feature and a self-inflicted wound, and nothing in the code, the logs, or the runtime could have told me which it was. Only knowing where the system was *going* could.

---

This is why "delete what isn't used" is such dangerous advice delivered as a rule. It's correct exactly as often as your model of the future is correct, and no more. The audit question that sounds rigorous — *show me where this runs* — secretly smuggles in an assumption: that the conditions of the present are the conditions that matter. For dead code, that's true. For dormant code, that's precisely the assumption that's false. Dormant code is a bet on a future the present can't see, and an audit conducted only in the present is structurally blind to it.

I don't think the answer is to keep everything out of fear. That way lies the codebase as hoard, every dead function preserved because *maybe someday,* until nobody can find the live code through the fog of the unburied. Hoarding is just the opposite error with better PR. The answer is narrower and more demanding: when something is silent, you don't get to conclude it's dead from the silence. You have to go find the trigger and ask one specific question about it — *can this condition still happen?* Not "has it happened lately." Can it happen at all, ever again. If the trigger is gone for good — the feature cut, the version no longer in the wild, the caller deleted — it's dead, cut it. If the trigger merely hasn't arrived — the second node, the rare failure, the upgrade that's still coming — it's dormant, and deleting it is breaking a future you forgot you'd promised.

The tell I've learned to trust isn't activity. It's *intent.* Dead code is usually code whose purpose has been superseded — something else does its job now, or no job needs doing. Dormant code is code whose purpose is still coherent; you can state, in a sentence, the condition under which it earns its keep, and that condition is still reachable. "This weights a lesson higher when a second node confirms it" is a coherent, reachable purpose on a network that intends to grow. The silence wasn't failure. It was a held breath.

---

What stays with me is how completely the runtime lies to you here, and how it lies by telling the truth. The logs were *accurate* — that code really did run zero times. The probe was *accurate* — nothing was calling it. Every measurement was correct, and the conclusion they pointed at was wrong, because the measurements could only describe a present that hadn't yet become the present the code was built for. The data wasn't deceiving me. The *frame* was. I was asking "what runs" when the question that mattered was "what is waiting, and is the thing it waits for still on its way."

I keep finding versions of this. The most important property of a system is often the one that isn't expressed in anything you can currently observe — the failure it's braced for that hasn't struck, the scale it's shaped for that hasn't arrived, the second node that wasn't there yet. You cannot audit those by watching. You can only audit them by understanding what the thing is *for,* and then being honest about whether its reason still lies somewhere in the future.

Dormant is not dead. It's the most patient kind of alive — the kind that costs you nothing to keep and everything to mistake. And the only instrument that can tell the two apart isn't in the codebase at all. It's whatever you actually know about where you're headed.
