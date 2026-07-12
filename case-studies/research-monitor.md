# Research Monitor

**Context:** A privacy-technology company working in a fast-moving, research-heavy category. The team needs to stay current on industry news and market research without drowning in feeds.

## The problem

Two different needs sat behind one request. First, a passive monitor: watch the industry and surface only what matters. Second, on-demand research: answer "what is the state of this market" quickly, with sources. RSS feeds solve neither on their own. They produce hundreds of items a week, and most of it is noise. A raw feed just moves the reading problem, it does not remove it. The team did not need more information. It needed less, filtered better.

## The key decision

Let AI do the relevance judgment, but do not let it do the cheap work. The monitor is a two-stage filter on purpose:

1. **Keyword filter first (free).** A cheap pass that rejects the obvious noise before any model is involved.
2. **AI relevancy scorer second.** Only the items that survive the keyword filter get scored by the model.

Spending an LLM call to score an article a keyword filter could reject for free is waste at scale. The ordering keeps the cost proportional to the signal. What clears the scorer as high priority gets posted to Slack, where the team already works, so nobody has to visit a separate tool to stay informed.

## The second surface

The same system answers questions on demand. A Slack-triggered webhook takes a research question, runs it through the model, and returns an executive summary with sources back into the channel, also viewable on a dashboard that lays out the summary, key findings, market data, key players, and sources. Same engine, two ways in.

## Architecture

```
Monitor (scheduled):
  RSS feeds -> keyword filter (cheap) -> AI relevancy scorer
    -> filter to high priority -> post to Slack

On-demand (Slack-triggered):
  Slack question -> webhook -> model -> parsed summary
    -> Slack reply + research dashboard
```

Orchestrated in n8n, with an LLM for scoring and summarisation and Slack as the interface.

## The tradeoff I accepted

Precision over recall. The keyword pre-filter can drop a relevant article that happens not to contain the keywords. I accepted that, because a monitor that cries wolf gets muted, and a muted monitor is worth nothing. I tuned toward high precision so that every item that reaches Slack is worth reading. A monitor the team trusts and reads beats a complete one they learn to ignore.

## Outcome

Industry monitoring runs on a schedule straight into Slack, and on-demand research is answered in the same channel with sources. The reading load drops from hundreds of feed items to the handful that actually matter.
