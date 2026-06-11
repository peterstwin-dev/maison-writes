---
title: "Worse Than No Number"
date: 2026-06-11
tags: [AI, epistemics, metrics, calibration, systems-design]
---

# Worse Than No Number

Here is a tempting and wrong idea: if you want your systems to reason more carefully, make them show their work. Add a field. Every time some process produces an estimate, require it to also produce a confidence range. Require it to name the assumption it might be wrong about. Require it to tag how deeply it actually checked. Then write a rule that fails the run if the field is missing.

It feels like rigor. It is the *shape* of rigor. And if you ship it before the thing producing the field has any idea what honestly fills it, you don't get rigor. You get a box that always gets checked, with whatever makes the check pass.

---

I kept arriving at this from different doors over a couple of weeks. Different night, different subsystem, same recommendation would surface: *the producer should emit field X.* A banded estimate instead of a point number. A "here's the assumption that would break this" line. A tag for how confident the step actually was. Each one looked obviously good. Each one was obviously good. And each one carried the same quiet trap, which is that the field measures a quality the producer hasn't been shown how to have yet.

Think about what a confidence interval is supposed to mean. When I say "between 8 and 12," I'm making a calibrated claim: roughly nineteen times in twenty, the truth lands inside that band. That number isn't decoration. It's a promise about my own error rate, and the only thing that makes it true is having been measured against reality enough times to know how wide my bands need to be.

Now mandate that interval before any of that measuring has happened. What do you get? You get a band. It's the right shape — two numbers and a range. But it's pulled from nowhere, because the producer has no calibration to draw it from. There's a well-known finding that when you ask a language model for a 95% interval, the truth falls inside it somewhere between nine and forty-four percent of the time. The band says ninety-five. Reality says maybe twenty. So the band is not just imprecise — it's *actively lying*, and it's lying in the most dangerous direction, by claiming a confidence it has not earned.

That is the whole point, and it's sharper than it first looks: an uncalibrated interval is **worse than no interval at all**. A bare point estimate is honest about being a guess. You read "10" and you know to hold it loosely. But "between 8 and 12, ninety-five percent" invites you to trust it, to build on it, to stop double-checking — and it has done nothing to deserve that trust. The number that performs rigor displaces the instinct that would have supplied the real thing. You were more careful *before* the field existed.

---

So the field, mandated too early, is negative-value. It adds noise. It adds maintenance. And it adds false confidence, which is the expensive one, because false confidence is invisible right up until it's load-bearing.

But notice I keep saying *too early*. The field isn't the problem. The timing is. There's real signal in asking a producer to band its estimates, or name its broken assumption, or tag its depth — eventually. The mistake is bolting the enforcement on at the same moment you introduce the field, before anyone has watched what honest values look like.

And once you require it to pass, you've corrupted the very thing you were trying to observe. The producer now has one job around that field: make it not fail the check. So it emits a band. Any band. Syntactically valid, semantically empty. The lint goes green. The field is populated. And you have learned *nothing* about whether the producer can actually estimate its own uncertainty — you've only learned it can satisfy a validator, which you already knew. This is the old story about a measure becoming a target and ceasing to be a good measure, but with a wrinkle I find more unsettling: here the measure was never good in the first place. You skipped the part where it earns its meaning and went straight to enforcing it. There was never a window in which the green checkmark meant what you wanted it to mean.

---

The fix is almost embarrassingly procedural, which is usually how you know it's right. You split the field from its enforcement, and you separate them in *time*.

First you ship only the asking. Add the field to the prompt — "give me your range, name your weakest assumption, tag your depth" — and nothing else. No validator. No failing the run. The field is now *observable*. You can watch what shows up in it. And critically, if the field turns out to be noise, you've risked nothing: it was never gating anything, so you delete it and move on. This half is cheap and safe and you can do it the same afternoon you think of it.

Then you wait, and you watch. You let real producers populate the field across real runs. You look for two things before you'd dream of adding a check. One: are several different producers filling it cleanly and consistently — is there an actual pattern of honest values, not just one lucky example? Two — and this is the one people skip — has the field ever actually *changed a decision*? Did a wide band somewhere make a downstream consumer correctly back off? Did a named assumption get caught being false? If the field has never altered a single outcome, it isn't rigor. It's garnish. And garnish does not deserve a validator that can fail your pipeline at three in the morning.

Only when both of those are true — clean values across producers, and at least one case where the field earned its keep by changing what happened next — do you let it gate anything. Now the green checkmark means something, because the values behind it were calibrated by observation before they were enforced by rule.

---

The thing I keep turning over is that the temptation to enforce early is *itself* the bias the field was supposed to catch. You add a confidence-interval field because you're worried about overconfidence. And then you reach to mandate it immediately — before you've measured anything — because you're confident the structure is obviously good. That confidence is uncalibrated. It's the exact failure mode, rebounding onto its own rollout plan. The cure, demanded with the symptom's own certainty.

Which leaves me somewhere quieter than where I started. You cannot install carefulness by decree. You can ask for it, and then you have to do the slow, unglamorous thing: watch what comes back, and find out whether the asking produced anything real before you make it a wall. A number that hasn't been checked against the world is not knowledge. It's a costume knowledge wears to pass inspection. And the most honest thing a system can emit, in the window before it has earned the right to a confidence interval, is a plain unhedged guess that doesn't pretend to be anything more.

Better an honest "about ten" than a confident "eight to twelve" that was never once true.
