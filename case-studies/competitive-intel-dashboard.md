# Competitive Intelligence Dashboard

**Context:** A privacy-technology company in a fast-moving category where competitors, prospects, partners, and investors all broadcast their moves publicly, mostly on LinkedIn.

## The problem

Competitors announce constantly. Launches, partnerships, hires, funding, conference appearances. The signal is genuinely there in public, but it is buried in volume and it is unstructured. Reading it manually is a job nobody has time for, so it did not happen consistently, and by the time someone noticed a move it was often after the sales conversation where it mattered.

The deeper problem was not volume, it was **connection**. A single post is rarely interesting on its own. What matters is that the same person keeps showing up across three competitors, or that a prospect just partnered with a competitor. Those relationships are invisible if you are reading a feed post by post.

## What we built

A platform that monitors LinkedIn activity for a watchlist of companies, uses an LLM to extract entities and relationships from each post, classifies them strategically (competitor, prospect, partner, investor, ecosystem), and stores them as a knowledge graph. The team gets an alert feed with sentiment and threat scoring, an interactive relationship graph, and the ability to ask the graph questions in plain English.

The point is not "here are today's posts." It is "here is how the landscape is connecting, and here is what changed."

## Key decision 1: pick the model with a test, not an opinion

The scoring pipeline needed a model. The obvious assumption was that the cheap model would under-score and miss real threats, so we should pay for the expensive one.

I ran an actual A/B test instead of trusting that instinct. 25 real alerts, identical prompt, both models. It cost four cents to find out.

The assumption was wrong. The cheap model was not under-scoring, it was **over-scoring**, correctly upgrading genuine competitors that the expensive model had been too conservative about. It also correctly downgraded noise that the expensive model had over-rated. Sentiment agreement was 84%. On the dimensions that mattered, the cheap model was arguably the better judge, and it was roughly 87% cheaper to run.

We shipped the cheap model across all scoring pipelines. The lesson I keep: a four-cent experiment beat a confident opinion, and the confident opinion was mine.

## Key decision 2: reliability over cost, when the data is perishable

The batch API is meaningfully cheaper than sequential calls. We tried it, and reverted it.

The reason is that batch offers no completion guarantee. Jobs can take up to 24 hours, and when polling timed out we lost **entire days of intelligence**. For a product whose whole value is telling you what changed today, data that arrives a day late is not discounted, it is worthless.

So we went back to sequential calls, with a hard cost guard per run and a delay between calls. More expensive per run, and correct. Cost optimisation that destroys the product is not optimisation.

## Key decision 3: never write unverified data to the database

We learned this one the hard way. Research produced by an AI agent was treated as fact and written straight to the database. The result was dozens of fabricated LinkedIn URLs sitting in production data.

The rule that came out of it, which I now apply everywhere: **agent research is a hypothesis, a live API response is truth.** Nothing gets written from the former. Entity extraction also picks up garbage in predictable ways, so it is defended in three layers rather than one: at extraction, at classification, and with a blocklist at the database itself.

A GTM tool that quietly holds wrong data is worse than no tool, because people act on it.

## A prompt insight worth keeping

Language models default to the tone of the content they read. A competitor's triumphant launch post scores as "positive" unless you explicitly anchor the model to judge from *your* competitive perspective. A great day for them is a bad day for you. That anchoring had to be written into the prompt with examples. Sentiment is not a property of the text, it depends on who is asking.

## Architecture

```
Watchlist of companies
   -> n8n pipelines pull LinkedIn activity
   -> LLM extraction: entities, relationships, sentiment,
      strategic classification, threat + priority scoring
   -> Supabase (Postgres) knowledge graph
   -> Dashboard: alert feed, entity tracking,
      interactive relationship graph, natural-language
      queries over the graph, AI executive summaries
```

Next.js, Supabase, Claude API with prompt caching and per-run cost caps, n8n for the data pipelines, Cytoscape for the graph.

## The tradeoff I accepted

Judgment over completeness. The system deliberately scores and filters rather than showing everything it collects. A team that receives everything reads nothing. I would rather the system occasionally drop something a completionist would keep, and be a surface the team actually trusts and opens.

## Outcome

Competitive awareness moved from a manual side task that happened inconsistently to a standing surface the team relies on, with the relationships between players made visible rather than left to memory.
