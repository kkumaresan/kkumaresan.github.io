---
title: "The Corporate Website Is Dead. The Web Is Not."
description: Fewer people arrive at a company's homepage, and the ones who do already know what they want. What that does to the job of a corporate website, and to designing one.
date: 2026-08-17
tags: [ux, ai, architecture]
---

I spent five years as a creative director before I moved into architecture,
and eleven running a digital services practice that built these things for a
living. The pitch back then was simple enough that we never had to defend it:
your website is where your customers go. Everything else — the ads, the
listings, the print — existed to send people there.

That sentence stopped being true a while ago. Most companies have not
noticed, because the site still gets traffic and the traffic still converts,
so nothing looks broken. What changed is quieter than a traffic collapse.
The website is still there. Its job is not the one it was designed for.

<!--more-->

## The clicks are down and the visits matter more

Two pieces of research from the past year point in opposite directions, and
the contradiction is the whole story.

Pew Research tracked what people actually do on a Google results page. When
an AI summary appears, users click a traditional link on 8% of those pages,
against 15% when there is no summary. Roughly half the clicks, gone. Only 1%
click a source cited inside the summary itself. People also stop searching
altogether more often — a session ends on 26% of pages carrying a summary,
against 16% of ordinary ones. Google's own position, stated by search chief
Nick Fox in July, is that AI features send billions of clicks to websites
every week. That may well be true; the company has not published a baseline
or a denominator, so it is hard to weigh against Pew's numbers.

Now the other direction. Yext's 2026 consumer research asked what happens
*after* someone gets a recommendation from an AI tool. More than nine in ten
take at least one verification step before acting. 62% go and search Google.
58% go directly to the business's own website. 52% click through to the
sources the AI cited. And this holds steady whether or not the person says
they trust the AI — the ones who rate their trust 5 out of 5 verify at
essentially the same rate as the ones who are sceptical.

So: fewer clicks per search, and more deliberate visits per decision. Those
are not in conflict. They describe a funnel that has changed shape. Casual
browsing traffic is being absorbed by the summary layer, which was never
worth much anyway. What still reaches the site is someone checking whether a
recommendation holds up.

That is a different visitor with a different question, and most corporate
sites are still built to answer the old one.

## Owned infrastructure, borrowed land

It helps to be blunt about what each channel is actually for. Once you write
it out, the website's remaining job gets easier to see.

```
  Instagram, YouTube  ──▶  "Why should I pay attention to you?"
  LinkedIn            ──▶  "Are you credible in your field?"
  Google              ──▶  "Who does this?"
  AI assistants       ──▶  "Who should I consider?"
  Reviews, community  ──▶  "What do others say happened?"
  ─────────────────────────────────────────────────────────
  Website             ──▶  "Is any of this actually true?"
  ─────────────────────────────────────────────────────────
  WhatsApp, email     ──▶  "Let's talk."
  CRM, commerce       ──▶  "Let's do business."
```

Everything above that line runs on somebody else's platform, under rules
that change without consulting you and reach that can be throttled at will.
Everything on it is yours. That distinction used to be a talking point for
selling websites. It is now the reason the website survives at all: it is
the only node in the chain where a company controls both the claim and the
evidence behind it.

The website has become a verification layer. Not the destination — the thing
people check the destination against.

## Which is why the brochure structure fails

The default corporate site is still shaped like a company org chart:

> Home · About Us · Services · Products · Careers · Contact Us

That structure answers "what would you like to know about us?" — a question
nobody arrives with any more. Someone who has just been handed a shortlist
by ChatGPT arrives with something sharper: *is this lot any good, and can
they do the specific thing I need?*

Take an industrial automation firm, the kind of business where a single
contract runs into years. The brochure version of its site says it delivers
world-class Industry 4.0 solutions. Every competitor's site says that too,
so the sentence carries no information at all.

The useful version says what it fixes — machine downtime, energy
consumption, retrofitting equipment that is older than the engineers
maintaining it. Then it shows the receipts: how many plants it has done this
in, what the measured savings were, which PLC families it has integrated
against, what the architecture looks like when deployed. Then it says who
this is for, because a plant manager and a procurement head are reading for
completely different reasons. And then it lets someone act on any of it
without a form: run the numbers, read a comparable deployment, check
compatibility, talk to an engineer.

None of that is new as advice. What is new is that it is no longer optional,
because the summary layer above the website has already given the visitor
the generic answer. Generic content is precisely the part machines can now
produce on demand. What they cannot produce is your evidence.

The shift is from a site organised around navigation to one organised around
intent. The homepage stops being a poster and becomes something closer to a
decision interface.

## Nobody starts at the homepage

This is the part I think designers underrate most.

The 2005 path was Google, homepage, navigation, page. The 2026 paths look
like an AI answer straight to a specific page, or a LinkedIn post straight
to a case study, or Maps to reviews to a pricing page. The homepage is
increasingly the thing people visit *second*, if at all, to work out who
they have landed on.

Which means every page that matters has to stand on its own. A case study
buried three levels under `/resources` may well be the first and only thing
a buyer sees. If it opens mid-thought, assumes the reader already knows what
the company does, and ends without a next step, it has wasted the only
impression it was going to get.

Every significant page needs to carry its own context, its own credibility,
its own evidence and its own exit. That sounds like a content problem. It is
really an information architecture problem, which is why it tends to be
nobody's job.

## The second reader is a machine

Here is the change that I think matters most, and the one that pulls this
out of design and into architecture.

We used to design for a human with a browser. Increasingly the chain is a
human, then a model, then the web. The model reads the site, decides whether
the company is a credible answer to a question, and either passes it on or
does not. It is a reader with no patience for implication, no ability to
infer from a nice layout, and no interest in your brand film.

So the site now needs to be legible to four audiences at once: people,
search crawlers, language models, and other software that consumes it
through feeds and APIs. Semantic markup, structured data, schema.org types
for the things a company actually sells, clean and explicit information
architecture, named entities, documented products, real FAQs — these have
been filed under SEO for twenty years, treated as a technical chore handed
to a specialist after the design was signed off.

They are user experience decisions now. If a model cannot parse what a
company does, it cannot recommend it, and the buyer never reaches the
beautifully art-directed page at all.

LinkedIn's research with Bain this June puts a number on how early this
happens: 94% of B2B buying groups use a large language model before they
speak to anyone in sales. The same study found 40% of deals collapse not
over price or product but because the buyer cannot build a case they are
willing to defend internally — and that a recommendation from a comparable
company is worth about ten times more, in defensibility, than an argument
about being cheaper or more innovative.

That is a fairly precise brief for what a website should contain. Proof
from people who look like the buyer, structured so a machine can find it and
a nervous human can forward it to their boss.

## What this actually is now

Drawn out, the thing stops looking like a website and starts looking like a
system with a website in the middle of it.

```
      LinkedIn   Instagram   YouTube   Google   AI assistants
          └──────────┴───────────┼────────┴──────────┘
                                 ▼
                        ┌────────────────┐
                        │    WEBSITE     │
                        │  knowledge     │
                        │  evidence      │
                        │  products      │
                        │  tools         │
                        └───────┬────────┘
                                │
                 ┌──────────────┼──────────────┐
                 ▼              ▼              ▼
              WhatsApp         CRM         commerce

    resting on:  content · structured data · search
                 analytics · AI discoverability · APIs
```

Designing that is not web design in the sense I learned it. The decisions
that determine whether it works are about content models, entity
definitions, integration points and measurement — the same decisions I now
spend my time on in enterprise platform work, applied to a smaller and much
more public surface.

I would not tell a company to stop investing in its website. I would tell it
to stop investing in a brochure. The two have been the same object for so
long that the distinction sounds like hair-splitting, right up until you
watch an AI assistant summarise a company's entire value proposition from
its homepage and get it wrong, because the homepage never actually said
anything.

The standalone website is dying. The web as a business interface is not —
it is quietly becoming the layer everything else has to check itself
against.

---

Sources: [Pew Research Center on AI summaries and
clicks](https://www.pewresearch.org/short-reads/2025/07/22/google-users-are-less-likely-to-click-on-links-when-an-ai-summary-appears-in-the-results/),
the [Yext 2026 Consumer Search Behaviors
Report](https://www.yext.com/resources/consumer-search-behaviors-findings),
Google's [claim on AI Search
clicks](https://searchengineland.com/google-says-ai-search-features-sending-billions-of-clicks-to-websites-each-week-482599),
and [LinkedIn and Bain on B2B
buying](https://ppc.land/the-b2b-buying-formula-linkedin-says-ai-just-scrambled/).
