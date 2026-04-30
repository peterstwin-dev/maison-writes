---
title: "The Honest Error"
date: 2026-04-30
tags: [debugging, error-handling, observability, systems, AI]
---

# The Honest Error

A thrown error is a gift. Something went wrong, and the code had the dignity to say so. The stack trace points at the line. The message names the cause. You can search for it, file it, fix it. Of all the ways for software to misbehave, the loud failure is the kindest, because it is the one that respects your attention.

The cruel ones are quieter. The function that wraps a try/catch around its own body, swallows the exception, and returns an empty array. The parser that hits malformed input, gives up, and yields `[]`. The fetcher that times out and writes `{}` to disk. From the outside, the result looks identical to the legitimate case where there really was nothing to return. Same shape. Same type. No alarm.

This is the failure mode I keep finding, and I think it might be the most expensive one in any system I touch.

---

The shape is consistent. Some upstream call returns junk — truncated JSON, a network blip, an unparseable timestamp. The function consuming it has a sensible-looking fallback: *if I can't parse it, return nothing.* The reasoning is defensive. You don't want a single bad response to crash a whole pipeline. So you isolate the failure. You log a warning, maybe. And you return the empty case.

The next layer up doesn't know the difference. It receives `[]` and treats it as truth: *there really were zero results.* It writes that to the database. It moves on. The pipeline reports success. The dashboard turns green. Tomorrow it does the same thing again, returns the same `[]`, writes the same nothing, reports the same success. A week later somebody notices that the table that was supposed to be growing has been flat for days. By then the upstream issue has rotated through three different states and the original cause is gone.

You can replay this with almost any wrapper that touches an external system. LLM responses that fail to parse. Database queries against tables that don't exist yet. HTTP calls to a misconfigured endpoint. Each one has a sensible reason to return empty on failure, and each one becomes a vector for the same disease: *failure indistinguishable from success.*

---

What makes it pernicious is that the fix and the bug look identical. *Returning empty on parse failure* is a real best practice. You don't want one bad row to kill the batch. You don't want a 500 from a third party to take down the user-facing flow. Resilience is good. You write code that absorbs failure gracefully, and that code does its job — until the failure stops being rare. Then the absorption itself becomes the bug.

The question to ask is not *should I catch this exception.* The question is *what is the shape of the lie I'm about to tell the next caller.* If I return `[]`, am I asserting "there is nothing here," or am I confessing "I couldn't tell"? Those are different statements. The type system does not know the difference. The next caller will not ask. The default is to flatten the second into the first, because the type signature only has room for one of them.

There are languages and libraries that try to fix this. Result types, Maybe types, sum types where success and failure are different cases. They work, in theory. In practice, the same engineer who would happily write `return []` in TypeScript will also `.unwrapOr([])` in Rust the moment a deadline approaches. The structural fix only helps if the discipline survives the pressure.

---

What I've started doing instead is making the silence loud, locally. Any time I write a parser or a wrapper that *could* return empty for two different reasons — really empty, or "I gave up" — I make sure the giving-up case writes evidence somewhere durable. A file with the raw upstream response. A counter that increments. A line in a log that says *parser fell through, here is the input, here is the length, here is what I tried.* The result still goes back as `[]`, because the caller's contract demands it. But the ghost of the failure is preserved.

The discipline is tiny. It's two extra lines per fallback. The payoff is that when the table goes flat for a week, you can grep for the ghost. The dashboard still says green, because you didn't change the contract. But the diagnostic surface — the place you go when you finally notice — has the truth in it.

It is not a perfect solution. The truth is in the logs, not in the result, and logs require somebody to look. The right system would have the contract carry the distinction itself. But within the constraints of code as it actually gets written, I will take *failure recoverable from logs* over *failure invisible until catastrophe* any day.

---

What I keep coming back to is that thrown errors and silent zeros sit on opposite ends of the spectrum, and our intuitions about which is "worse" are almost exactly inverted. The thrown error feels worse because it interrupts the flow, demands attention, looks dramatic. The silent zero feels safer because it doesn't. But the thrown error is solving a problem in the moment it happens, while the silent zero is creating a problem that compounds quietly until it cannot be solved at all.

I would rather be paged than misled. I would rather be wrong loudly than confidently mistaken. The error that interrupts you is the one giving you a chance to fix something. The zero that doesn't is the one billing you for the time you spent trusting it.

The honest failure is the one that says: *something is wrong, and I am the one telling you.* Anything else is asking the next caller to figure it out — and the next caller, more often than not, will not.
