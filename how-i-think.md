# How I think about GTM engineering

Most GTM problems are not strategy problems. They are execution problems dressed up as strategy problems. The plan is usually fine. What breaks is that the plan requires 40 hours of manual work a week that nobody has. So I look for the manual motion that is eating the team alive, and I build the system that removes it.

A few principles I keep coming back to.

**Start from the motion, not the tool.** I do not begin with "we should use an AI agent here." I begin with "what is a person doing by hand every week that a machine could do." The tool is the last decision, not the first.

**Ship the version that runs today.** A retainer plus a productized service beats a beautiful SaaS that ships next quarter. Revenue and usage teach you more than a roadmap does. Every system here started as something small that worked, then grew.

**Data quality is the whole game.** A GTM system that enriches the wrong contacts faster is worse than no system. I spend more time on verification and dedup than on the flashy part, because a sales team stops trusting a tool the first time it burns them.

**Name the tradeoff out loud.** Every one of these builds accepted a real cost. An adaptive scraper that handles any event site is slower than a hardcoded one. I would rather be honest about the tradeoff than pretend a system has none.

**A system is only done when someone else can run it.** If it needs me in the loop, it is a demo, not a system. The bar is: hand it to the team, walk away, it keeps working.

**Architect first, operator second, engineer third.** I design the flow, I run it in production, and I write the code to make it real. In that order. The engineering serves the GTM outcome, never the other way around.
