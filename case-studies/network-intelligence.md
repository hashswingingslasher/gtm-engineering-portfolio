# Network Intelligence

**Context:** A consulting firm whose business development runs on the team's collective LinkedIn networks. Each person has thousands of connections, but they sit in separate accounts as CSV exports with no shared view.

## The problem

The team's real asset is its combined network. But that asset was invisible. There was no way to see, across everyone, which organisations the team collectively had the strongest access to, or where two people both knew someone at the same target. Answering "who can get us a warm intro at company X" meant asking around and hoping someone remembered. At the scale of thousands of connections per person, manual cross-referencing is not possible.

## The constraint that shaped everything

LinkedIn CSV exports are messy. There is a byte-order mark, junk header rows, and dates in a `DD-MMM-YY` format nothing else uses. Worse, company names are free text. The same organisation shows up as "Goldman Sachs," "Goldman Sachs & Co," and "GS." If you rank organisations without fixing that first, the rankings are garbage. One real org gets split into five weak ones.

## The key decision

The hard problem here is not the dashboard. It is entity resolution. So the core of the build is a three-tier company normalisation pipeline that runs before anything else:

1. Exact match
2. Suffix strip (drop "Ltd," "& Co," "Inc")
3. Fuzzy match (fuse.js) for the rest

Messy names collapse to canonical organisations first. Everything downstream, every ranking and every overlap count, is only as trustworthy as that normalisation underneath it. I spent the effort there, not on the charts.

## Architecture

```
LinkedIn CSV exports (multiple team members)
   -> Ingestion (PapaParse, handles BOM / junk rows / odd dates)
   -> Company normalisation (exact -> suffix-strip -> fuzzy)
   -> Contact classification (seniority tier, sector)
   -> Overlap detection (shared connections across the team)
   -> Dashboard: organisations ranked by connection density,
      senior decision-maker access, and team overlap
   -> Branded .xlsx export
```

Next.js 15, Supabase with row-level security, deployed on Vercel.

## The tradeoff I accepted

Point-in-time over live. The data is only as fresh as each person's last export. I chose to be explicit about that staleness rather than pretend the picture is live, and I designed the system for periodic re-upload. I also tuned the fuzzy matcher toward precision, since over-merging two different companies is a worse error than leaving one unmatched, and kept the output human-reviewable.

## Outcome

The team's combined network became a single searchable asset. The system surfaces which target organisations the team already has warm access to, and where two members share a connection. Those shared paths are the warm intros that actually convert, and before this they were invisible.
