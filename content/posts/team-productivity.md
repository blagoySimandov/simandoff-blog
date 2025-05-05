+++
date = "2025-05-05T00:17:26+01:00"
draft = false
title = "Developer Productivity: Why Moving Work Across the Board Matters More Than Perfect Code"
summary = "A look at the PR process and how being overscrupulous can harm a team."
author = "Blagoy Simandoff"
canonicalURL = "https://simandoff.blog/posts/gcp-signed-urls-browser-oddity/"
tags = ["Developer practices", "Team Productivity"]
keywords = ["Developer Productivity", "PR", "Code Review", "Developers", "Team productivity"]
showReadingTime = true
showBreadCrumbs = true
showPostNavLinks = true
showWordCount = true
showRssButtonInSectionTermList = true
useHugoToc = true

[cover]
image = "/cables.jpg"
alt = "A group of cables held together by a hand"
caption = "Photo by Mika Baumeister taken from Unsplash"
relative = false
+++

---

In software engineering, we are often pressured by deadlines. As programmers, we
are tasked with solving complex problems quickly. And although computers don’t
make mistakes, **we as programmers are still human**, and a bug or two
inevitably slips into our code. But as all great teams know, there's a process
designed to catch and **correct** these issues — **Code Review**.

Code Review is an integral process for any software project, and it's the
**steady flow of pull requests** keeping a codebase alive and evolving.

When working on a project dear to us, we can become overly scrupulous or
emotionally attached to a codebase. That can lead us to extremes, like blocking
PRs over minor issues—renaming a variable, extracting a function, or refactoring
code for style. While these concerns may be valid in the long term, they are
rarely good enough to block a pull request.

PRs are like blood flowing through a codebase. If you block them unnecessarily,
the entire system slows down and stagnates. And just like in a living organism,
it's far healthier to keep the blood flowing continuously—even if it's not
_perfectly_ clean—than to hold everything up waiting for perfection.

## A team is like a living CPU

Just like a CPU is composed of multiple threads working in parallel, a team is
an **organic system** made up of individual engineers, each contributing to a
common mission: improving the codebase and moving the product forward.

And just like how CPUs can suffer from race conditions and deadlocks due to poor
sync, **teams, too, can fall into productivity traps** when members become
blocked.

In a team of 5 people, the moment a member is blocked waiting for a PR review,
the impact is bigger than just a 20% slow down (which by itself is a lot). The
blocked work might delay integration, slow testing, or create merge conflicts
that snowball into bigger issues and even more possible blockages.

**This is why velocity matters.**

Unblocking a teammate with a quick PR review is often a far better use of the
next 15 minutes than continuing to bang your head against a stubborn bug.

## Reviews Help, Pedantic Reviews Hurt

Reviews help teams move forward. But pedantic reviews are unhelpful and actually
harm a team’s performance. Pedantic reviews spin up a chain reaction which can
lead to a cycle of synchronization problems, each time having to ping the other
engineer, each time having to context switch and lose a little bit of energy and
focus. This leads to frustration and slows the process for no reason. In most
cases, your 80% good-enough PR reviewed quickly contributes more than a 100%
polished one merged two days later.

Let’s walk through what happens when you leave a “minor” comment on a pull
request—say, something like _“Please rename this variable to something more
descriptive.”_

On the surface, it feels harmless. It might even seem helpful

But under the hood, you’ve just set off a chain of events that will slow down
the entire team:

1. **The developer needs to see your comment.**

   Maybe they get the notification right away. Maybe they’re deep in a different
   task and won’t check their notifications for hours. Sometimes, it’s the next
   day.

2. **They have to switch context.**

   Even if it’s just a small change, the dev now has to pause whatever they’re
   working on, reload the mental state of the original PR, and figure out how to
   address your comment.

   Best-case scenario: they’re indifferent to the change and just rename the
   variable.

   Worst case: they disagree, feel frustrated, and now a simple review turns
   into a pointless debate over subjective naming preferences that might not
   even last until the next quarter.

3. **They make the change and ping you again.**

   Now the dev has to re-request review or tag you directly. Depending on your
   availability, you might not reply immediately. That adds even more idle time.

4. **The PR is blocked until you approve.**

   The PR is blocked, and the team is not moving. That means feature work, bug
   fixes, or integrations, depending on this PR, also stay blocked. Merge
   conflicts (the bane of my existence) might get created as a result of that

A simple comment can cause an avalanche of problems.

## Taking a look at the PR process

A PR would have requested changes of only two types:

1. The PR includes changes that would either not work correctly, or even if they
   do, the code being merged would result in the overall health of the codebase
   degrading.
2. Changes that try to bring up a PR's quality unnecessarily.

The second scenario is the one we need to be wary of.

If a codebase is good enough so that merging it will positively impact the
codebase, we should have no reason to block it.

If you have some considerations around the implementation that can better clean
up the code or even refactor certain parts, create a ticket.

The ticket can then be triaged and prioritized appropriately by the team, and
**now the team holds the power** to decide whether refactoring is worth their
time in this current moment. If it is, great, pick up the ticket and do the
work; otherwise, shove the ticket into the backlog until it's time comes.

And if you feel like you cannot be bothered to cut a ticket for small changes
like renaming a variable, ask yourself - If a change is not worth the trouble to
create a ticket for, is it really worth blocking a PR over it?

## Conclusion

Move the work across. Merge the PR. File a ticket for improvements if needed.
Don’t let perfect be the enemy of shipped
