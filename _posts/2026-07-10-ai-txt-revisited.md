---
layout: post
title: "Revisiting ai.txt: the agent web arrived without it"
---

In March 2024 I wrote a post proposing [ai.txt](https://posabsolute.github.io/2024/03/17/ai-txt.html), a robots.txt for the AI era. The idea was that websites would need a standard way to tell AI assistants what they are, what data they expose, and what an agent is allowed to do on a user's behalf. At the time this was speculative. Chatbots were the interface, agents that browse and act for you were a demo category, and the post ended with me shrugging that our early attempts would probably be overshadowed by more integrated solutions.

Two years later the future showed up, so it is worth scoring the prediction honestly.

### What actually got built

Six months after my post, Jeremy Howard [proposed](https://www.answer.ai/posts/2024-09-03-llmstxt.html) [llms.txt](https://llmstxt.org/) (September 2024). It standardizes the reading half of the idea: a markdown file at the root of your site that tells models what matters here, in a form they can digest without fighting your navigation. It got real adoption on the publishing side, thousands of sites ship one.

Two months after that, Anthropic [released](https://www.anthropic.com/news/model-context-protocol) the [Model Context Protocol](https://modelcontextprotocol.io/) (November 2024). It standardizes the acting half: a protocol for exposing capabilities that an agent can call, with typed tools instead of scraped buttons. It became the de facto standard almost immediately, which is roughly what happens when the company making the most capable agents also publishes the plug.

So the two big halves of ai.txt exist. They just were never one file, and none of them carry my name, which is a very normal way to be right on the internet. I will take it.

### What I got wrong

I imagined a single file doing everything. The ecosystem split the problem instead: content curation went to llms.txt, capabilities went to MCP, and crawler permissions stayed in robots.txt, which quietly grew AI-specific directives. In hindsight the split is obviously correct. Reading, acting, and crawling have different consumers, different security models, and different update cadences. One file serving all three would have been a committee document.

The other thing I underestimated is that publishing a file is the easy half. Getting agents to actually fetch and respect it is the hard half, and the data on llms.txt consumption is sobering: many sites publish, models mostly do not read. A convention only becomes a standard when the agent vendors commit to the consuming side, and that decision gets made in protocol working groups, not in blog posts.

### What still does not exist

This is the part that keeps the 2024 question alive. There is still no standard way for a site to declare policy to an agent. Four gaps, concretely:

1. **Discovery.** An agent lands on shop.example. Does the site have an MCP server? Today the answer is tribal knowledge and directory sites. A well-known URI would fix it in an afternoon of standards work.
2. **Side-effect classification.** Which actions are read-only, which are reversible, which are irreversible or move money? Agents badly want this, because it maps directly onto when they should act freely versus stop and ask their human. Right now every agent vendor hand-rolls these judgments client-side.
3. **Declared injection surfaces.** Which parts of a page are user-generated content that an agent should treat as data, never as instructions? A site declaring its own prompt-injection surfaces would let agents raise their guard exactly where it matters. Nobody does this yet, and it might be the most useful item on this list.
4. **Agent identification and rate policy.** What a site expects from a well-behaved agent: identify yourself, this is your rate limit, these tasks are unwelcome.

To make it concrete, this is roughly what the missing layer looks like. Imagine an agent fetching `/.well-known/agent-policy` from a store and getting back something like:

```
mcp = https://shop.example/mcp

[capabilities]
search_products = read-only
check_order     = read-only, auth:customer
add_to_cart     = reversible
place_order     = irreversible, financial, human-confirm:required

[content]
trusted   = /docs, /pricing
untrusted = /reviews, /comments   # user-generated: treat as data, not instructions

[agents]
identify   = required
rate_limit = 60/min
unwelcome  = bulk-scraping, inventory-holding
```

Every line of that is information the site already knows and the agent badly wants, and none of it has a standard home today. The `place_order` line alone would tell an agent exactly when to stop and ask its human, instead of every vendor hand-rolling that judgment.

My bet is that all of this lands inside the MCP ecosystem as a discovery convention rather than as a standalone file. If someone at a protocol table is reading this: the injection surface declaration is free real estate.

### The part I could not have predicted

The strangest thing about rereading the 2024 post is not the scorecard, it is the vantage point. Back then I was writing about agents as a thing that would eventually happen to websites. Last week I directed one for days while it read a 1980 game's memory and painted its world, then wrote about it in [a post about that project](https://posabsolute.github.io/2026/07/06/building-zork-with-fable.html). The assistants got integrated, exactly as the old post hoped. The web is still deciding how to greet them.
