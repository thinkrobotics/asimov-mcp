---
name: find-parts
description: Find, compare and check electronics and robotics components from the ThinkRobotics store (thinkrobotics.com, India, prices in INR) using its catalogue tools. Use when the user is choosing, sourcing, pricing or checking availability of a hardware part for a project — microcontroller and single-board computer boards, sensors, motors and motor drivers, servos, batteries and power modules, displays, wireless modules, robot kits, 3D-printing parts, cables and connectors — or asks "which board/sensor/driver should I buy", "is this in stock", "how much does X cost", "does ThinkRobotics sell X", or wants a spec read from a product listing. Do not use for firmware or software questions that involve no purchase, for general electronics theory, or for other stores.
license: MIT
compatibility: Needs the ThinkRobotics public server (https://mcp.thinkrobotics.com/mcp) connected. No sign-in required.
---

# Find parts at ThinkRobotics

Use this skill to answer questions about what ThinkRobotics sells, what a part costs, whether it
is in stock, and what its listing says about it.

## The one rule

Every fact about a product — that it exists, its price, its stock, its specs — comes from a tool
call made in this conversation. Never answer these from memory, and never fill a gap with what a
part "usually" is. The catalogue changes daily, and many parts have near-identical clones with
different pinouts and voltages.

## Tools

These are on the ThinkRobotics public server, `thinkrobotics` (no sign-in). Your host may show them with a prefix.

| Tool | Use it for |
|---|---|
| `search_products` | Finding candidates. Takes a `query`, and optionally `max_price_inr`, `in_stock_only` and `limit` (1-10, default 5). Returns each product's handle, title, URL, lowest and highest price, summary and availability, and `has_more`. |
| `get_product` | One product's listing. `product` is its handle, its page URL or its product id. Returns the description as text, options, images, and the variants with SKU, price, list price, availability and buy link. |
| `check_stock` | Live price and availability for `items`: 1 to 20 SKUs and/or variant ids in one call. One line per item, in the order asked, each with `found`. |

## Workflow

1. **Work out what is needed before searching.** If the user is in a project, the constraints are
   usually already in it: logic voltage, interface (I2C, SPI, UART, PWM), current, connector,
   board family. Read them rather than asking. Ask only when a missing constraint would change
   which part is right.
2. **Search with short, concrete queries** — a part number or a plain name (`TB6612FNG`,
   `ESP32 devkit`, `NEMA 17 stepper`), not a sentence. If nothing useful comes back, try the
   part number alone, a common alternative name, or a broader category before concluding
   anything. Pass `max_price_inr` or `in_stock_only` when the user gave a budget or needs it now.
3. **Open the candidates with `get_product`** before recommending one. Search results are
   summaries; the listing text and the variant list are where the facts are.
4. **Confirm with `check_stock`** right before you give a final recommendation or a buy link, so
   the price and availability you state are live.
5. **Answer briefly**: the part, why it fits, price in rupees, availability, and the product link.
   When several parts fit, show two or three with the difference that matters, not ten.

## Reading specs

Listings are free-text descriptions written by the store, not structured data sheets.

- **Quote the listing** for any spec you rely on (voltage, current, interface, dimensions,
  pinout), and say that it is from the listing.
- **Flag gaps.** If the listing does not state something the user's design depends on, say
  "the listing does not state this". Do not infer it from the chip name, the photo, or a similar
  product. Suggest checking the manufacturer's data sheet, and say which value to look for.
- **Do not turn "probably compatible" into "compatible".** State what the listing confirms and
  what is still unverified.
- A product can have several versions (variants) with different specs and prices. Name the exact
  variant and its SKU.
- When `get_product` returns `all_variants_listed: false`, the product has more versions than
  the ones shown. Say so, and say nothing about the versions that are not listed.

## Price and stock

- Prices are in Indian rupees (INR). Show them as the tool returns them; do not convert
  currencies unless asked, and say so if you do.
- Availability is `in_stock`, `out_of_stock` or `unknown`. Report `unknown` as unknown — it is not
  "in stock".
- A `check_stock` line with `found: false` means the store has no variant for that SKU or id.
  That is not "out of stock": check the SKU for a typo, or search for the part by name.
- `has_more: true` on a search means there are further matches. Narrow the query rather than
  treating the first page as the whole range.
- A list price is shown only when the tool returns one. Do not call something discounted
  otherwise.
- Do not state delivery times, warranty or return terms from here — those come from the
  `store-policies` skill.

## Buy links

A buy link from `get_product` opens the store's checkout with that item, and **replaces whatever
is already in the shopper's cart**. Say this in the same message, before the link. If the user
only wants to look, give the product page URL instead.

## When things go wrong

- **No results:** say the store's search returned nothing for those terms, list what you tried,
  and offer a different search. "Not found" for one query is not proof the store does not sell it.
- **Tool error or timeout:** the tools return an error, never an empty result, when the store
  fails or cannot be read. Say the store's answer is unknown. Do not fall back to memory, and do
  not present an error as "out of stock" or "not sold".
- **"Rate limit reached for this tool. Retry after N seconds."** (or a store error that names a
  wait): this means wait, not "not found". The catalogue tools share a budget of about 30 calls
  a minute per client address, and each SKU in a `check_stock` call counts as one more. Tell the user the
  wait, then retry once after it; do not loop. Other users share the store's capacity, so a busy
  minute can refuse you before you reach your own budget.
- **Spend calls carefully:** check many SKUs in one `check_stock` call rather than one each, use
  variant ids from `get_product` when you have them, and do not repeat a search you already ran.
- **Server not connected:** tell the user the ThinkRobotics server needs to be connected in
  their host's MCP or connector settings.
