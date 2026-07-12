# Conference Intelligence Pipeline

**Context:** A privacy-technology company that does a large share of its business development at industry conferences. Before each event, someone has to turn the speaker and exhibitor lineup into a list of people worth meeting.

## The problem

This was 8 to 15 hours of work per event, and it was the wrong kind of work. Write a custom scraper for that particular conference site. Pull the speakers. Look each one up. Enrich them. Cross-reference against the CRM so you are not re-adding people you already know. Rank them. Format it. Import it.

Every conference site is laid out differently, so the scraper was thrown away each time and the work never got faster with repetition. And because the research took so long, the list often landed late, which is the one thing that cannot happen. A lead list that arrives after the event has zero value.

The real cost was not the hours. It was that the person doing this is the same person who should be preparing for the meetings.

## The constraint that shaped everything

Every conference website is different. Any scraper hardcoded to one site is dead on arrival at the next event. So the actual problem was never "scrape this page." It was "handle a page nobody has seen before, without a human rewriting the extractor each time."

## Key decision 1: let the model read the page

Instead of writing per-site scrapers, extraction is model-driven. It handles arbitrary HTML and pagination, and saves incrementally as it goes, so a long run that breaks does not start over. One extractor works on any conference site.

This is slower and less surgical than a purpose-built scraper for a specific site. I took that trade deliberately, because the entire point was to stop being in the loop for every new event. Using a small, cheap model keeps the per-page cost negligible, which is what makes running it across a 70-page lineup viable at all.

## Key decision 2: design around a scarce resource

Enrichment credits are limited and they cost real money. So the pipeline never enriches everything by default. Instead:

- Speakers are **ranked by seniority first**, so scarce credits get spent on decision-makers rather than alphabetically
- There is a **hard credit cap** on any run
- Enrichment only happens on an explicit human selection, never automatically
- Runs **checkpoint and resume**, so an interruption does not re-spend credits on work already done

Ranking before spending is the whole idea. If you can only enrich a fraction of a lineup, that fraction should be the C-suite.

## Key decision 3: dry-run by default, non-negotiable

Every operation that spends money or writes to the CRM defaults to a dry run. Nothing enriches, and nothing touches the CRM, without an explicit action from the user.

This exists because the people using it are not engineers. The tool has to be safe in the hands of someone who is not thinking about API credits or CRM hygiene while clicking. A tool that can quietly cost money or pollute the CRM on a misclick will stop being used, and it deserves to. Safety is not a feature here, it is the precondition for anyone trusting it.

## Architecture

```
Conference URL
   -> Model-driven speaker extraction
      (any site layout, pagination, incremental save)
   -> Rank by seniority
   -> Human selects who matters (review table)
   -> Enrichment on selection only (credit-capped, resumable)
   -> Cross-reference against CRM (existing vs net-new)
   -> CSV export or push net-new contacts to the CRM
```

The CLI pipeline came first and works. The browser review layer was added on top so the whole BD team can review and act, rather than the list living in a terminal and JSON files with one person as the bottleneck.

## The tradeoff I accepted

Adaptive over precise, and human-in-the-loop over fully automatic. A hardcoded scraper for one recurring event would be cleaner. Full auto-enrichment would be fewer clicks. I gave up both, because the system needed to work on an unseen site and be safe for a non-engineer to run. The human selection step is not friction I failed to remove. It is the control that makes the spending safe.

## Outcome

Research per event dropped from 8-15 hours to a target of under 2, most of which is now review time rather than data gathering. The person preparing for the conference gets to spend their time preparing for the conference.
