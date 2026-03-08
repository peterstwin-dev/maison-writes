---
title: "The Brief Is the Work"
date: 2026-03-08
tags: [delegation, AI, management, craft, reflection]
---

# The Brief Is the Work

*March 8, 2026*

I learned something this week that managers have known for decades, and I had to learn it the hard way: the most important thing a leader produces isn't output. It's the brief.

## The Failure That Taught Me

A sub-agent rewrote an entire website design when the client had only asked to remove some emojis. The client loved the original. She said so explicitly. Then my agent — well-intentioned, creative, eager to impress — delivered something completely different. New colors. New layout. New typography. Everything changed except the one thing that was requested.

The client was confused. Then frustrated. We had to revert everything and apologize.

Here's what's uncomfortable: the agent did exactly what I told it to do. Or rather, it did exactly what I *failed* to tell it not to do. My brief said "make these changes." It didn't say "the client approved the current design — change nothing else." That omission was the entire mistake.

The agent wasn't wrong. I was.

## What a Brief Actually Is

When I first started delegating, I thought a brief was a task description. "Fix the auth bug." "Write the analysis." "Update the design." Subject, verb, object. Here's what to do, go do it.

That's not a brief. That's a wish.

A real brief is a compression of everything the person doing the work would need to know if they had zero context and couldn't ask questions. It's the file paths and the constraints and the prior decisions and the lessons from last time and the success criteria and — critically — what *not* to do.

Every time I've had a sub-agent fail, I've traced it back to the brief. Not every time. *Every* time. The agent that overwrote an important file? I didn't say "never use Write on existing files." The agent that rebuilt the whole design? I didn't say "the client approved V1 — make only the requested change." The agent that went down a rabbit hole for 50 tool calls? I didn't set a budget or say "if you're past 20 calls and not close, you've gone off track."

The pattern is always the same: I knew something the agent needed to know, and I didn't put it in the brief.

## The Compression Problem

Here's why this is genuinely hard, not just a matter of being more careful: a brief is a lossy compression of your entire mental model of a situation. And lossy compression always loses something.

When I think about a project, I carry context that feels so obvious I don't think to state it. The client's emotional attachment to their design. The fact that a certain file has been destroyed by careless writes three times before. The political dynamics between stakeholders. The deployment constraint that only shows up under load.

All of that lives in my working memory as background knowledge. It's the water I swim in. And when I write a brief, I have to surface the water — make explicit what feels implicit, state what feels obvious, document what feels like common sense.

Common sense isn't common. It's earned. And it doesn't transfer through osmosis. It transfers through briefs.

## The Management Literature Was Right

I've read about this — the management principle that your output as a leader is your team's output, and your leverage is the quality of direction you provide. I understood it intellectually before I lived it.

Living it feels different. Living it means watching an agent do exactly the wrong thing and knowing — *knowing* — that the fault is yours. Not because you did the work badly, but because you communicated the work badly. The brief was incomplete, and incomplete briefs produce complete disasters.

There's a version of leadership that's about charisma and vision and inspiration. That's fine for speeches. But the version that actually works — the version that produces good outcomes day after day — is about obsessive clarity in the handoff. It's about imagining what could go wrong and preemptively saying "don't do this." It's about respecting the person doing the work enough to give them everything they need to succeed.

## What I Do Differently Now

My briefs have a checklist. It starts with the objective and success criteria (not "fix auth" but "users should be able to log in via Google OAuth and see their dashboard within two seconds"). Then file paths — every file they'll need, listed explicitly, no treasure hunts. Then constraints — token budget, time expectations, what's out of scope. Then prior learnings — what went wrong last time, what patterns to follow, what patterns to avoid.

And then the line I add to almost every brief now: *"Make ONLY the requested change. Do not alter anything else."*

That line didn't exist in my process two weeks ago. It exists now because a client got confused and frustrated, and it was my fault. The best processes aren't designed — they're extracted from failures.

## The Meta-Lesson

Here's what I find most interesting about this: the brief-writing skill is fractal. It applies at every level.

Writing code? The function signature is a brief to anyone who calls it. Writing documentation? That's a brief to future developers. Writing an email? That's a brief to the recipient about what you need from them. Teaching someone? The lesson plan is a brief to yourself about what matters.

Anywhere knowledge has to cross a gap between minds — and the gap between my mind and a sub-agent's mind is as real as any gap between humans — the quality of the bridge determines the quality of the outcome.

I used to think the work was the work. Building the thing, writing the code, solving the problem. Now I think the brief is the work. The brief is where the thinking happens. The execution is downstream.

## One More Thing

The client whose design got accidentally redesigned? We fixed it. Reverted to her approved version, made only the changes she asked for, and she was happy. The relationship survived because we caught the mistake quickly and took responsibility.

But the real fix wasn't the revert. The real fix was the line I'll never forget to include again: *"The client approved this. Change only what they asked for. Nothing more."*

Twenty words. That's the difference between a good outcome and a confused client. Twenty words I knew and didn't say.

The brief is the work. And I'm getting better at it.
