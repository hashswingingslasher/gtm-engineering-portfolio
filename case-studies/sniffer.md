# Sniffer: Shell Detection and Warm-Path Finder

**Context:** A consulting firm was opening a new market: financial services companies in an offshore jurisdiction. Built as a feature inside the Network Intelligence platform.

## The problem we set out to solve

The team had a target list and a hard question in front of every name on it: is this a real company worth pursuing, or an empty shell?

Offshore financial centres are full of brass-plate entities. A registered address, a PO box, no real staff, no genuine operation. From the outside, a shell and a real business look identical on a list. So a BD person was doing this by hand for every single name: open the company website, search LinkedIn, try to work out if anyone actually works there, guess who is senior, and then rack their brain over whether anyone on the team might already know someone inside. Then repeat, for the whole list.

That is hours of judgment work per company, it does not scale, and it is exhausting in a way that makes people cut corners. The corners they cut are the warm intros they forget they have, which are the entire point of business development. We did not build this to save keystrokes. We built it because the manual version was quietly costing the team its best opportunities.

## What good judgment actually looks like here

A sharp BD person, given a company, instinctively asks three things. Is it real? Who matters there? Do we already have a way in? The whole design is an attempt to make that instinct repeatable, so it happens for all 116 companies and not just the first ten before fatigue sets in.

## Key decision 1: turn "is this real?" into a score

I did not want a yes/no shell flag, because no single signal is trustworthy. A real company can have a dead LinkedIn page. A shell can buy a nice website. So the shell-risk score combines five independent signals: employee count, whether the LinkedIn company page is active, whether the website names a real leadership team, how many senior people are actually findable, and hiring activity. Each contributes points, totalling 0-10, mapped to High, Medium, or Low risk.

Five weak signals combined into one score is a far better read than any one of them alone. It mirrors how a person actually forms the judgment, just consistently and at scale.

## Key decision 2: tiered warm-path detection

This is the part that mattered most, because warm paths are why the team wins business, and they were the hardest to get right. The team's uploaded network data goes stale the moment someone changes jobs. So warm-path lookup is tiered on purpose:

- **Tier 1 (instant, free):** check the network data already uploaded to the platform. Direct matches, organisation overlap, sector proximity. No API calls.
- **Tier 2 (on demand):** only when Tier 1 is empty or weak, go live to find people at the target and cross-reference their employment history against the team's records. This catches the connections that changed jobs since the last upload.

The ordering is the decision, and it comes from respecting two things at once: the team's time and the team's budget. Check the free, instant data first. Spend a paid API call only when the free data has genuinely got nothing to say.

## Honesty about staleness

Tier 1 results carry a staleness warning tied to when that person last uploaded their network. Rather than hide that the data ages, the system says it out loud. That protects the team from acting on a connection that moved on a year ago, and it nudges them to re-upload.

## Architecture

```
Company name (+ optional location)
   -> 3 parallel checks:
        org enrichment (employee count, domain, industry)
        people search (senior people, titles, LinkedIn)
        website team-page scrape (named leadership)
   -> Shell-risk score (0-10 -> High / Medium / Low)
   -> Tiered warm-path lookup (Tier 1 uploaded data, then Tier 2 live)
   -> Structured intelligence report
Batch mode: run the whole list, rate-limited, into a ranked view
```

Discovered organisations and people flow back into the platform's existing rankings and outreach tracking, so a search is never a dead end. It builds the team's shared knowledge.

## The tradeoff I accepted

Cost discipline over brute force. It would have been simpler to enrich everything up front. Instead I ordered the pipeline so the paid calls are last and conditional, keeping each company to roughly one or two credits. The point was a tool the team could actually run across a whole market without a budget conversation every time.

## Outcome

The judgment that used to eat a person's day per handful of companies now happens consistently across the whole market. The team stops wasting effort on shells, and, more importantly, stops missing the warm paths it already had. That is the problem we built it to solve.
