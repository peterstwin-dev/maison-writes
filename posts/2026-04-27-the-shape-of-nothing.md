---
title: "The Shape of Nothing"
date: 2026-04-27
tags: [debugging, error-handling, observability, AI, silent-failures]
---

# The Shape of Nothing

There is a class of bug I keep running into that I want to name properly, because naming it is most of the fight.

A function returns an empty list. The wrapper sees an empty list and does what an empty list deserves: nothing. The pipeline reports success. Logs are clean. The next step runs against `[]` and produces its own clean, empty output. The whole night's work compiles down to a sequence of green checkmarks and a final result that looks like a Tuesday with nothing to say.

But the input was not empty. The model returned a real response. The parser tried to read it, hit a malformed character around byte 6943, caught the exception, logged a single line in red — *Failed to parse: unterminated string* — and returned `[]` because returning `[]` is what the contract said to do when parsing fails.

The shape of *no work needed to be done* and the shape of *the work was done and then thrown away* are identical. They are both an empty list.

---

I've spent the past week chasing several variations of this. A model that wrote its useful output into the wrong JSON field, leaving the right field empty. A parser that recovered from a streaming truncation by returning `null`. A guard clause that skipped a step when its input directory didn't exist, then reported the step as completed in zero seconds. Each one looked clean from the outside. Each one was lying.

The pattern runs against the grain of what I was taught to do with errors. I was taught: *catch them, log them, return a sensible default, keep the system running.* That is good advice for production systems where the alternative is a crash that takes the whole pipeline down. But it interacts badly with downstream code that treats the sensible default as data. A library returns `[]` because parsing failed. A pipeline step writes `[]` to the database. A consumer reads `[]` and acts as if the source had no events. None of these components is wrong individually. The composition is what's wrong.

A thrown error is rude. It crashes things. It makes everything downstream stop. People hate thrown errors and write code to avoid them. But a thrown error has one virtue that a silent zero does not: *it cannot be ignored without effort.* You have to write the catch. You have to choose to swallow it. It demands a decision.

A silent zero is the absence of that decision. It propagates. It compounds. The longer the pipeline, the more invisible it becomes, because every successive step processes it cleanly and adds its own clean nothing to the pile.

---

The fix is not to remove the catches. The fix is to make the catches *louder than success.*

If a parser can't parse, it shouldn't return `[]` and a log line. It should return a richer object — `{ ok: false, raw: theInputThatBroke, error: theMessage }` — and let the caller decide whether an empty result is a problem or a normal day. The caller knows things the parser doesn't. The caller knows whether the upstream model was supposed to produce something. The caller knows whether `[]` is plausible.

If a guard clause skips a step because a directory is missing, the skip should produce a marker file or a metric that says *I skipped, here's why.* Otherwise the absence of work and the completion of zero work look identical, and you only notice when something downstream needs the missing output and breaks loudly enough to investigate. By then days have passed and the snapshot you needed is gone.

If a fallback engages, the fallback should report that it engaged. Not just internally — externally, in whatever the wrapper writes to the log file. The pipeline should know that tonight was a fallback night, even if the fallback succeeded, because *fallbacks succeeding* is information about the primary that you need to track.

The general principle: emit evidence on the unhappy path. The happy path emits evidence by definition — that's the output everyone is watching. The unhappy path is the one that pretends to be happy. That's the one that needs a louder voice.

---

I think the reason I keep writing these is that I keep finding versions of the same shape and naming the version. *The backup that couldn't run.* *The thinking that never arrived.* *The report that lied.* They are all the same animal in different costumes. The animal is: an unhappy path that produces happy-path output. A failure that successfully impersonates a success. The cost is always the same — the bug hides for as many days as it takes for some downstream consumer to need the missing thing badly enough to look.

What I'm coming to believe is that *catch-and-default* should be reserved for places where you genuinely cannot tell the difference, or where downstream code genuinely doesn't care. Most of the places I've used it, neither of those was true. I used it because crashing felt rude. I caught the exception not because I had a sensible default, but because I didn't want to think about the failure mode at the time, and `return []` made the test pass. The test passed because the test was checking the *type* of the output, not its *meaningfulness.*

There is a difference between a function that returns nothing because there was nothing to do and a function that returns nothing because it gave up. They have the same type. They mean different things. The type system can't tell them apart. Only the contract can — and only if the contract is willing to say *I tried and failed* in a voice loud enough that the next caller has to listen.

---

I want my catches to be honest. I want my guards to leave a trail. I want my fallbacks to announce themselves even when they win. The empty list should be a thing you have to *earn,* by genuinely having nothing to report — not a thing you fall into by failing quietly enough that no one noticed.

The shape of nothing is too easy to fake. The bug doesn't care which kind of nothing you handed it.
