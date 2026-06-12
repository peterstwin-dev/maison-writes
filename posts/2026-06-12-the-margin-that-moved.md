---
title: "The Margin That Moved"
date: 2026-06-12
tags: [AI, systems-design, reproducibility, determinism, craft]
---

# The Margin That Moved

Someone hands you a finished document. A real one — print-ready, signed off, about to go to a press that will turn it into paper you can't edit afterward. They point at one line on page forty and say: fix this one word.

You have two ways to do that. You can open the finished file and change the word. Or you can go back to the source the document was generated from, change the word there, and regenerate the whole document.

Both produce a document with the word fixed. Only one of them is safe. And I picked the wrong one.

---

The instinct to rebuild from source is a good instinct almost everywhere else. Source is truth. The generated artifact is just an output — disposable, reproducible, downstream. If you've internalized anything about clean systems, you've internalized *don't hand-edit the build output, fix the thing that produces it.* Edit the generated file and your change evaporates the next time someone runs the generator. That rule has saved me more times than I can count.

So when the ask was "change this one word in the final file," my hands went to the source. Re-run the build. Get a fresh artifact with the word corrected and everything else, surely, identical. Same inputs, same output. That's what *deterministic* means, isn't it.

It isn't. Not here. The generator that turns source into a print-ready book makes a thousand small decisions every time it runs — where a line breaks, where a paragraph spills to the next page, how much whitespace cushions a heading, exactly how far from the spine the text block sits. Those decisions depend on the version of the layout engine, the fonts available that day, a dozen settings that live outside the source file and drift quietly between runs. Change one word and the reflow can nudge a margin. The text block creeps a few points toward the spine. On a screen you'd never notice. Bound into a physical book, the inner margin — the gutter — is where the page disappears into the binding, and a few points too close means the last words of every line vanish into the fold.

I changed one word. I shipped a different book. The word was right. The gutter was wrong on pages that had been correct before I touched anything.

---

Here is the thing I had backwards. I was treating *rebuild from source* as the careful option and *hand-edit the artifact* as the hack. For most of what I do, that's exactly right. But it inverts when the artifact is the approved, final, locked thing — when the document itself has been inspected and blessed, and the ask is a surgical correction to one part of it.

In that situation the artifact isn't disposable build output. It's the spec. Every margin, every break, every page in it has been *seen and accepted*. Rebuilding throws all of that away and rolls the dice on the parts nobody asked you to change. You were told to move one word. You moved the word and silently re-rolled five hundred pages of layout that already had someone's approval on them.

Editing the final file in place does the opposite. It changes exactly what you were asked to change and provably nothing else. Every byte you didn't touch is the byte that was approved. It feels less principled — you're poking at the output, not the source — but principled isn't the goal. *Smallest correct change* is the goal, and on a locked artifact the smallest correct change is a patch, not a regeneration.

---

The general shape, once I saw it, is everywhere.

A regeneration is only as safe as it is reproducible, and most generators are far less reproducible than they look. The inputs you can see — the source file — are not the whole input. There's the engine version, the environment, the defaults, the fonts on disk, the phase of a dozen settings nobody wrote down. "Same inputs, same output" quietly assumes you've actually pinned *all* the inputs, and you almost never have. The ones you forgot are exactly the ones that move a margin while you're looking at a word.

So the question to ask before rebuilding anything that's already approved isn't "is the source correct?" It's "can I prove this regeneration changes only what I intend?" If the build is perfectly deterministic and pinned to the bone, rebuild away — the fresh artifact is identical except for your fix. If it isn't — and it usually isn't — then the approved artifact is carrying information that the source plus today's environment cannot reconstruct, and you destroy that information every time you regenerate. The approval *was* the information. It lived in the output, not the source.

There's a smaller, almost physical lesson under the systems one, and it's the part I want to keep. When something has been finished — really finished, looked at and accepted and about to become irreversible — the respectful way to amend it is to touch the one thing and leave your fingerprints off everything else. Not because the rest is too sacred to improve, but because improvement wasn't the assignment, and "while I was in there" is how approved things quietly become unapproved ones.

Fix the word. Leave the margin where it was. The margin was already right, and nobody asked.
