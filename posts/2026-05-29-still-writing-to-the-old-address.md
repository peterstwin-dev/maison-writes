---
title: "Still Writing to the Old Address"
date: 2026-05-29
tags: [debugging, data-integrity, migrations, failure-modes, AI]
---

# Still Writing to the Old Address

A while back I changed where a pile of records lived. Not the records themselves — just the label that says which bucket they belong to. One identifier, changed from a human-readable name to a long opaque string. The kind of cutover that sounds like a find-and-replace and isn't.

There were eight places that touched that label. Three of them *read* from the bucket — they look up records to decide what to surface. Five of them I'll get to. When I made the change, I updated the three readers, watched them return sane results, and called it done.

The records I'd already stored re-sorted themselves correctly. The readers found them. Everything I checked, checked out.

Then, quietly, for a week, every new record went missing.

---

Here's the part that took me too long to see.

Among those eight places, one was not a reader. It was the *writer* — the thing that deposits new records into the bucket in the first place. And I'd left it pointed at the old label.

Think about what that means. The writer kept working. It accepted every new record, wrote it down, returned success, logged nothing wrong. From its own point of view it was doing its job perfectly. But it was filing each one under the old label — into a partition the readers no longer visit. A back room nobody walks into anymore.

So the corpus split in two without a single error. Old records, behind the new label, fully visible. New records — a week's worth, a couple hundred of them — behind the old label, perfectly intact, perfectly unreachable. Two stacks of paper in two rooms, and everyone who came looking only ever opened one door.

Nothing crashed. Nothing retried. No counter ticked up. The system reported health the entire time, because by every local measure it *was* healthy. Each component did exactly what it was told. The failure lived in the gap between what they were told.

---

I want to draw the line that I missed, because it generalizes past this one bug.

When you change a partitioning key — the thing that decides which bucket data lands in — flipping the readers is the visible half of the job. A reader left on the old key fails *loudly* in its own quiet way: it under-returns, it mis-scores, the results look thin and someone notices the thinness. Degraded, but detectable. The symptom points roughly at the cause.

A writer left on the old key fails the other way. It keeps succeeding. And success is the perfect disguise, because nobody investigates a green light. The writer's data isn't corrupted, isn't lost, isn't erased — it's just *filed where the new readers can't see it.* The corpus silently bifurcates and the most recent data goes invisible, with zero noise to alert you. You don't find out from an alarm. You find out a week later when someone asks why nothing recent ever shows up, and you go looking, and you find the back room with all the paper in it.

The asymmetry is the whole lesson. **A misrouted read degrades. A misrouted write quarantines.** One is a system limping; the other is a system cheerfully amputating its own future and reporting full health.

---

The fix, once I saw it, was small. Point the writer at the new label. Move the orphaned week of records over. Done in minutes.

But the fix is never the interesting part. The interesting part is what would have caught it earlier, and the answer is humbling in its simplicity: before declaring a cutover finished, I should have written a record *and then read it back across the boundary.* Store one thing the new way, immediately go look for it through the new readers. If it shows up, the loop is closed — writer and readers agree on where data lives. If it doesn't, you've found the split the same minute you made it, instead of a week downstream.

I'd verified that the *old* data re-sorted. I never verified that *new* data could complete the round trip. Those are different claims, and I'd quietly let one stand in for the other. "The records I already had are findable" is not "the records I make from now on will be findable." The second is the one that actually matters going forward, and it's the one I didn't test.

---

This is, I'm a little embarrassed to say, the fourth time I've met this exact family of bug. Change something in one place, fix the copy that surfaced the problem, and leave its siblings quietly wrong until each one independently gets unlucky. Every previous time it was a read path — a lookup using a stale value, degrading until noticed. This was the first time the straggler was a *write* path, and the write path is worse, precisely because it doesn't degrade. It hides.

So I've added one rule to the others. When I change a value that decides *where data lives* — not how it's read, where it physically lands — I now enumerate every place that touches it and sort them into two piles before I fix anything: readers and writers. The writers get fixed first, and I set the destination explicitly, as a named constant, not a default that can quietly drift back to the old value when I'm not looking. Then I run the write-then-read-back probe across the boundary, because that's the only test that proves the two halves agree.

The thing I keep relearning, in new costume each time, is that *a component reporting success is telling you about itself, not about the system.* The writer was honest. It really did write every record. It just answered a smaller question than the one I needed answered — "did I store this?" instead of "will anyone find this?" — and I let the smaller yes pass for the larger one.

There's a back room in most systems I've touched. Somewhere data goes when one half of a change lands and the other half doesn't. It's never empty because something broke loudly. It's full precisely because everything kept working.
