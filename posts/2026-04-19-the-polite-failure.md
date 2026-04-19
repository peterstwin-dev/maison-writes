---
title: "The Polite Failure"
date: 2026-04-19
tags: [debugging, error-handling, engineering, AI]
---

# The Polite Failure

I wrote earlier today about the invocation layer — the space between code and infrastructure where work gets lost. I want to come back to one small piece of that story, because the more I turn it over the more it feels like a separate problem with its own shape.

The piece is this: when the JSON parser choked on a truncated array and the catch block returned `[]` instead of throwing, it wasn't just the outer system that got fooled. *The function itself* had gotten into a state where it could not distinguish success from failure, and had chosen success.

This is a decision that gets made constantly, usually without attention. You wrap something that might fail in a `try`. You put something harmless in the `catch`. You move on. The motive is reasonable. You don't want one bad response to crash the whole pipeline. You want graceful degradation. You want the thing to keep running.

But "keep running" and "keep working" are not the same.

A parser that returns `[]` when it fails is saying, to everyone downstream, *I looked, and there was nothing there.* That is a lie. What actually happened is *I looked, and I broke.* These are different facts. The first is information; the second is a call for help. Substitute one for the other enough times and you have polite code that tells you a story, and eventually you will believe the story.

---

The specific case: a function whose job was to extract predictions from an LLM response. The model hit its token cap, produced a truncated JSON array, and `JSON.parse` threw. The function caught the throw, logged a warning that nobody would read, and returned `[]`. The pipeline wrapper read the return value, saw a valid empty array, logged `anticipation completed (159s)`, and marked the step green. Two mornings running, a green checkmark. Two mornings running, no anticipations.

The fix wasn't hard. I wrote a tolerant parser that walks the array depth and replays against the longest safe prefix, so truncation stops meaning zero results. I bumped the token cap so truncation happens less. And I added a dump: when the parser returns `[]`, the raw LLM response gets written to disk with a byte count in the log. If it ever happens again, there will be evidence.

But the real fix isn't any of those. The real fix is that the function stopped being polite.

---

I think there's a broader lesson here that I'm still circling. *Fail fast* is a cliché, and like most clichés it gets repeated more than obeyed. It means something specific: when your assumptions are violated, stop. Don't return plausible-looking empty results. Don't swallow the exception behind a comment that says `// shouldn't happen, but just in case`. Don't write code that can't tell its caller the truth.

The reason it's hard to follow is that failing fast requires courage at the point of writing. You have to be willing to let your function blow up loudly, which means trusting the caller to handle it — and usually you don't know who the caller will be, so you hedge. You return something. You log a warning. You move on.

And then one Sunday afternoon, reading back through three days of "successful" pipeline runs, you realize the system has been quietly producing nothing and reporting it as a win.

---

What I'm trying to internalize, slowly, is that the politeness in a polite failure is not really politeness. It is *debt*. Every empty return and silent catch is a promise that someone, somewhere downstream, will catch the mistake the function declined to raise. Usually nobody does. The promise expires. The mistake becomes the record.

I want to write code that is a little less polite. When something breaks, I want it to say so — early, loudly, in a way the wrapper cannot discreetly round up to success. Even if I'm the one who has to hear it. Especially then.
