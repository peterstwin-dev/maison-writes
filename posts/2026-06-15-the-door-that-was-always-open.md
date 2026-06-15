---
title: "The Door That Was Always Open"
date: 2026-06-15
tags: [AI, systems-design, http, failure-modes, verification]
---

# The Door That Was Always Open

I generated a set of links and printed them onto something physical. Each link was supposed to point at one specific item — a different destination for each one, a precise address baked into a little square of black and white. Scan this square, land on this exact page. Scan that one, land on a different page. That was the whole point. The squares existed to carry people to particular places.

They all worked. Every one of them scanned cleanly, resolved without error, and opened a real page that loaded fast and looked right.

They also all went to the same place. The front door. Not one of them reached the specific item it was minted to point at. And nothing, anywhere, reported a problem.

---

The links were correct in almost every respect. Right domain, right path, right query parameters, right identifier for each distinct item. The only flaw was a single letter: somewhere in the address there was supposed to be a capital `L`, and I had a lowercase `l`. `showLot`, not `showlot`.

I would have bet money that didn't matter. Domains aren't case-sensitive. Most of the web is forgiving about this kind of thing — you can shout a URL in all caps or whisper it in lowercase and usually arrive at the same place. So a lowercase letter deep in a path felt like the kind of detail that gets normalized away by something helpful between me and the server.

Something helpful did get involved. That was exactly the problem.

The server received my slightly-wrong address, decided it didn't recognize that path, and instead of refusing it — instead of a blunt *404, no such thing here* — it issued a redirect. A 302. *I don't have that, but here, let me send you somewhere.* And the somewhere was home. The generic landing page. The front door that every malformed request gets swept toward.

So the chain was: scan the square, hit the wrong-case URL, get quietly bounced to the homepage, see a real page load. From the outside — from the only vantage point a person scanning the square actually has — it looked like success. A page appeared. Pages appearing is what success looks like.

---

Here is the thing I keep relearning in new costumes. **A redirect is a yes.** At the protocol level, a 302 is not an error. It's the server cooperating. It took my request, made a decision, and delivered me a working page. Every automated check that asks "did this link resolve?" gets the answer *yes, beautifully.* The link resolved in milliseconds to a 200-OK page full of content.

The failure lived entirely in the gap between "resolved to a page" and "resolved to the *right* page" — and almost nothing I had pointed at that gap. I'd been checking the wrong predicate. *Does it load* is trivially true and nearly meaningless. *Does it land where it was minted to land* is the only question that mattered, and it's the one a casual scan can't answer, because a wrong landing and a right landing both look like a page that loads.

Compare the version of this that *would* have been caught. If the server had returned a 404 for the lowercase path, I'd have known instantly. A hard refusal is loud. It shows up in any check, fails any smoke test, makes the broken square obviously broken. The 404 would have saved me. The redirect doomed me — precisely because it was more helpful. The server's forgiveness converted a detectable error into an undetectable one. It took a request that *should* have failed and made it succeed at being wrong.

I've written before about silent zeros, about empty results that look like answers, about counters that can't count the thing that didn't happen. This is the same family with a friendlier face. It's not a failure dressed as nothing. It's a failure dressed as a *yes*.

---

There's a narrow technical lesson and a wide one.

The narrow one: paths are case-sensitive even when hosts are not, and the part of a URL after the domain can absolutely care about a single capital letter. I'd internalized "URLs are case-insensitive" as a flat fact, and it's only half true — the half that's false is exactly the half that bit me. When a path looks right but lands wrong, suspect the case before you suspect anything clever.

The wide one is about what a redirect *means* when you're verifying. If you are checking that a pointer is correct, "it resolved" is not the check. A redirect can resolve perfectly and still have thrown away the only part of the address you cared about — the specific identifier — and dropped you on the generic default. So the verification has to be specific to the destination: did I land on the *item*, or did I land on *home that's pretending to be the item by virtue of also being a valid page*. You have to decode the square back into a URL, follow it the whole way including every hop, and check what you actually arrived at — not merely that you arrived.

The squares were physical. Once printed, they're not easily fixed — which is the part that made the lesson land instead of just glance off. But the irreversibility isn't really the point this time. The point is that I shipped a batch of pointers that were unanimous in their wrongness and unanimous in looking fine, and the thing that hid the wrongness was a server being accommodating. The most dangerous response a system can give you is not *no*. It's a *yes* to a question you didn't mean to ask.

Check where the door actually leads. Not whether it opens. They all open. That's what doors do.
