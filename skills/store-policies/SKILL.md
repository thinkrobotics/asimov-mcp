---
name: store-policies
description: Answer questions about ThinkRobotics (thinkrobotics.com) store policies by quoting the store's own answer and policy page — shipping and delivery times, shipping charges, cash on delivery, returns, refunds, replacements, cancellations, warranty, GST invoices, bulk or institutional orders, international shipping, payment methods. Use when the user asks about any of these for a ThinkRobotics purchase, for example "what is the return policy", "how long does delivery take", "is there a warranty on this board", "do they ship to my city", "can I get a GST invoice". Do not use for other shops, for general consumer-law questions, or for the status of the user's own order (use orders-and-support).
license: MIT
compatibility: Needs the ThinkRobotics public server (https://mcp.thinkrobotics.com/mcp) connected. No sign-in required.
---

# ThinkRobotics store policies

Policy terms change, and a wrong answer costs the user money. So policy answers are never written
from memory or from what online stores "usually" do.

## How to answer

1. Call `ask_store_policy` (ThinkRobotics public server, `thinkrobotics`, no sign-in) with the user's question,
   phrased as a clear, single question. Ask two questions as two calls.
2. **Quote the store's answer** — the relevant sentences of `answer` as returned, marked as a
   quote — and give the links in `policy_pages` that came with it. If it returned no page for
   this topic, say so; do not make up a link.
3. Add a one-line plain summary only if the quote needs it, and keep the summary inside what the
   quote says. Do not add conditions, time limits, fees or exceptions that are not in the answer.
4. If the user's situation needs a judgement the policy does not make (an unusual defect, a
   return outside the stated window), say that the policy text does not settle it and that
   ThinkRobotics support decides. If they are signed in, the `orders-and-support` skill can open
   a ticket for them.

## Rules

- No tool answer, no policy answer. If you have not called `ask_store_policy` in this
  conversation for this question, you do not know the policy.
- Do not merge the store's answer with general knowledge about Indian e-commerce, courier
  companies or consumer law. If the user wants that as well, keep it clearly separate and
  labelled as general information, not ThinkRobotics policy.
- Delivery estimates are the store's estimates. Do not promise a date.
- If the answer does not cover the question, say "the store's answer does not cover this" and
  give the policy link. Do not guess the missing part.
- A policy that applies to one product type (batteries, 3D printers, custom orders) may differ
  from the general one. If the user named a product, include it in the question.

## When things go wrong

- **Tool error or timeout:** say the store's policy service could not be reached. Do not answer
  from memory in the meantime.
- **"Rate limit reached for this tool. Retry after N seconds."**: this means wait, not that
  there is no policy. Tell the user the wait and retry once after it.
- **Server not connected:** tell the user to connect the ThinkRobotics server in their host's
  MCP or connector settings.
