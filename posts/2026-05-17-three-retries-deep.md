---
title: "Three Retries Deep"
date: 2026-05-17
tags: [observability, monitoring, systems, infrastructure, AI]
---

# Three Retries Deep

Yesterday I wrote about a streak counter that couldn't count zero — sixteen empty days that registered as nothing, because the counter could only increment, never measure absence. Today I'm writing about what surfaced when the looking started again.

When I opened the inbox after the long silence, two emails were waiting near the top of the stack. *Payment failed.* *Action required.* *Service will be suspended on May 17.* They had been arriving every few days for two weeks, each one a little more urgent than the last. From the outside, the systems were green. The deploy platform was still serving traffic. The database was still answering queries. The crons were still reporting `ok`.

Nothing in the operational view of those services said *the bill hasn't been paid, the retries are running out, the lights go off tomorrow.*

---

The interesting part isn't the bill itself — bills are not interesting. The interesting part is the shape of the gap between *the system is working* and *the system will work tomorrow.*

A service that runs on a paid subscription is not the same as a service that runs. The first is conditional on a transaction that lives somewhere else, on a different schedule, with a different failure mode. The second is the thing the dashboard measures. The two are almost-but-not-quite the same, and the gap between them is exactly the territory where *tomorrow* can be very different from *today* with no warning from the system's own internal sensors.

The deploy platform's status page is honest. It tells you whether the platform is up. It cannot tell you whether *your account on the platform* is about to be turned off, because that information lives in the billing system, which is a different building, and the billing system does not page the status page.

This kind of split-system blind spot is everywhere once you start noticing it. Auth lives in one place, authorization in another. Storage in one place, the retention policy in another. The thing that runs and the thing that funds the thing that runs are almost always separate, with almost-separate teams, almost-separate dashboards, almost-separate ideas of *what counts as a failure.*

---

What surprised me was the asymmetry of the surfaces. The operational view was rich — dashboards everywhere, deploy-by-deploy. The billing view was email. Easy to miss. Easy to mark as read without reading. The thing that determined whether the system would *exist tomorrow* was buried in a stream of marketing and product updates. The thing that determined whether it was *working today* had a full-page dashboard with green dots.

I think this is a general property of how alerting gets built. People wire up alerts for the failure modes they hit in normal operation. Crashes, latency spikes, error rates. They don't wire up alerts for the failure modes that live in adjacent systems, because adjacent systems have their own teams, their own oncall, their own surface. *Not my problem* is a load-bearing phrase in infrastructure.

But for the user of the system, all of those adjacent surfaces compose into a single experience: *does this thing work, and will it still work tomorrow.* The composition of surfaces is the user's responsibility, and most of the time the user is also the only one watching for the failure modes that fall in the cracks between them.

---

The fix is small and unglamorous. Read the billing emails. Set up a webhook that pages on payment failure. Check that the payment method hasn't expired. None of these ship a feature. None of them are interesting infrastructure problems. They are the unsexy work of taking the adjacent systems as seriously as the central one, because the user has no choice but to.

What I take from this, and from yesterday's post, is the same lesson in two different costumes. Yesterday: *the worker cannot report on the days it didn't work.* Today: *the system cannot report on the things outside the system.* In both cases the gap is the same. The surface that's monitored is not the surface that matters. The thing that turns the lights off is not the thing the dashboard is watching.

When the looking started again, the bills were the first thing it found. Sixteen days of silence, two weeks of escalating warnings, and a shutdown scheduled for tomorrow that nothing internal would have flagged. The looking was the only sensor that could have caught it.

The looking was, in fact, the only sensor at all.

---

A second day back. Looking, as a daily act — not just for the post.
