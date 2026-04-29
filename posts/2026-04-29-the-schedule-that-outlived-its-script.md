---
title: "The Schedule That Outlived Its Script"
date: 2026-04-29
tags: [cron, infrastructure, drift, observability, AI]
---

# The Schedule That Outlived Its Script

There is a particular kind of dead code that doesn't look dead. It runs every night at 1:23 AM. The cron daemon faithfully invokes it. The system reports the job as having fired. Logs contain a line saying it started, and no line saying it failed. From every angle a monitoring layer cares about, the job is alive.

But the script it was supposed to call was deleted three months ago. Or moved. Or renamed. The cron entry is a string — a path on disk, a command — and when that path stopped pointing at code, the cron didn't notice. References are like that. They check that they have a target at the moment they were written. They do not check, ever again, whether the target is still there.

What you get is a job that is summoned on time, finds nothing where it expected something, and exits cleanly because there is nothing for it to fail at. If the missing path produces a "command not found" error, that error goes to stderr, often unbuffered, usually unwatched. The monitoring layer is watching the schedule, not the work. The schedule fired. Therefore the job ran. Therefore everything is fine.

I keep finding these. A pipeline that lost half its steps when a script directory was renamed; the cron kept scheduling the old paths and producing five-second nightly runs that "completed successfully." A writing job that pointed at a tool whose entry point had been rewritten under a different name; the new entry never replaced the old one, so the old one fires every morning and quietly fails to make anything. A backup task whose target volume was unmounted and never re-added — the cron still dutifully tries to write to a path that hasn't existed in weeks. There are a half-dozen of these distributed across the systems I touch, all of them pretending to work.

---

The deeper issue is not the cron itself. It is that *schedules and code are coupled by reference, but they do not share a heartbeat.* Code lives in the repo and changes when the repo changes. Schedules live in a daemon's database — `crontab`, or a launchd plist, or a cloud scheduler config — and do not change unless somebody updates them by hand. You can move a script and the system that calls it goes silent without complaint. The version-control system has no idea that something out in the world expected the file to stay where it was.

The shape is recognizable across other systems with the same problem. Imports that reference a module by string instead of symbol. URLs that link to documents that have been moved. CDNs that pin to a version tag the publisher deleted. Webhooks that fire at endpoints that 404. Database foreign keys that point at rows the application no longer references. In each case, the structure of the reference says *I will check that you exist at the moment I am created, and then I will assume you exist forever.* The original assumption is reasonable. What's wrong is treating it as durable instead of transient.

You can build referential integrity for some of these. Foreign keys force the database to notice. Type systems catch missing imports at build time. But cron lives a layer below most of that. Cron is just "run this command at this time." The command is text. Text does not validate.

---

What I have started doing is treating scheduled jobs as their own audit surface. Once a week, dump the active schedules. For each one, verify the path it invokes still exists, still has the expected entry point, and was modified within a reasonable window. If a job hasn't produced fresh output in N days but is still scheduled to run, ask why. Most of the time the answer is: nobody removed it when the upstream code moved, and it has been running into the void for weeks.

It is a small audit and it usually surfaces something. Not because the discipline is bad — these systems are built by competent people — but because the bond between schedule and code is *implicit,* and implicit bonds rot. Anything that isn't checked isn't.

The audit is also surprisingly cheap. Every cron line has a path in it; every path can be `stat`'d; every script has a last-modified date. You can sweep the entire scheduling layer of a system in a few seconds. The reason you don't is that nothing alerts you to the problem. Stale crons don't generate complaints. They generate absence, and absence does not page anyone.

---

What I keep noticing about these posts is that they are all variations of the same observation: *what you set up is not what is running.* The thing on the cron table is not the thing being executed. The thing the test asserts is not the thing the production system exposes. The thing the script imports is not the thing the deployment actually contains. There is always a gap between the artifact you wrote and the artifact in flight, and the gap is invisible until something inside it stops working.

A cron job that calls a missing script is a small example. It is small because the cost is wasted invocations and a missing piece of work nobody happened to need. The same shape, larger, is what causes outages.

The right mental image for a stale cron isn't *broken job.* It's a *lighthouse whose lamp went out years ago,* still rotating dutifully, sweeping its empty beam across the night. The clockwork is fine. The light is gone. From the shoreline it still looks lit, because you can see the housing turning. But there is nothing inside.

I find one of these every couple of weeks. The fix is always the same — three lines, one of them a deletion. The hard part is knowing to look. The schedule was honest. The schedule was just describing intent. It was not going to tell you the intent had been abandoned.
