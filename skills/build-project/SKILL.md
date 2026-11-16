---
name: build-project
description: Turn a hardware or robotics project into a parts list (bill of materials) sourced from the ThinkRobotics store (thinkrobotics.com, India, INR), with reasons per part, a total, alternatives for out-of-stock items and one buy link. Reads the user's project files first — platformio.ini, Arduino sketches, requirements.txt, wiring notes, README, schematics notes — to work out which boards, sensors, drivers and power parts are needed. Use when the user asks what to buy for a build, wants a BOM, shopping list or kit for a robot, drone, IoT device, 3D printer or electronics project, asks "what parts do I need for this", or wants to source the hardware their code targets. Do not use for a single-part lookup (use find-parts) or for projects with no physical components.
license: MIT
compatibility: Needs the ThinkRobotics public server (https://mcp.thinkrobotics.com/mcp) connected, and read access to the project files. No sign-in required.
---

# Build a parts list for a project

Use this skill when the user wants to know what to buy for a whole build, not one part.

## Step 1 — Read the project before calling any tool

The project usually says what hardware it expects. Look, in roughly this order, at what exists:

| File | What it tells you |
|---|---|
| `platformio.ini` | `board`, `platform`, `framework`, and `lib_deps` (each library usually names a sensor, display or driver chip). |
| `*.ino`, `src/*.cpp`, `main.py`, `code.py` | `#include`/`import` lines, pin definitions, I2C addresses, PWM and serial use. |
| `requirements.txt`, `pyproject.toml`, `package.json` | Hardware libraries (`RPi.GPIO`, `adafruit-circuitpython-*`, `pyserial`, `smbus2`, `opencv`), which imply a host board, a camera, or a bus. |
| `sdkconfig`, `CMakeLists.txt`, `west.yml`, `*.overlay`, `Cargo.toml` | Target chip or board for ESP-IDF, Zephyr or embedded Rust. |
| `README`, `docs/`, wiring notes, `*.fzz`, `*.kicad_sch`, BOM files | Stated parts, voltages, motor counts, mechanical constraints. |

From this, write down for yourself:

- the controller board the code targets;
- each peripheral the code talks to, and over which interface;
- actuators (how many motors or servos, and what drives them);
- power: supply voltage, battery, regulators;
- the small parts that are easy to forget: cables, headers, breadboard, jumper wires, level
  shifters, SD card.

Keep "the code names this part" separate from "I am inferring this part". A library name
identifies a chip, not a specific breakout board or its voltage.

If there are no project files, or they do not settle it, ask the user — at most a few questions,
only the ones that change the list: what it should do, what they already own, budget in rupees,
and mains or battery power.

Do not send file contents, secrets, or anything unrelated to the hardware to the tools. Send a
short description of the build that you wrote.

## Step 2 — Get the list from the store

Call `build_parts_list` (ThinkRobotics public server, `thinkrobotics`, no sign-in) with:

- `project_description`: a plain description of the project and the requirements you extracted;
- `budget_inr`: the budget in rupees, if the user gave one;
- `already_have`: the parts the user already owns, as a list, so they are left out.

It returns `parts` (each with its subsystem, a reason, quantity, live unit price and
availability, and sometimes `alternatives`), a `total`, one `cart_link`, and `not_on_cart_link`:
the parts the cart link leaves out.

This tool is slow and tightly limited. One call can take up to two and a half minutes, so tell
the user it is working and do not call it a second time while waiting. A user gets about **five
builds an hour**. Call it once with a complete description rather than several times with
partial ones, and again only when the requirements really change.

For a single swap ("a cheaper driver", "the 5 V version"), use `search_products`, `get_product`
and `check_stock` instead of rebuilding the whole list.

## Step 3 — Check the list against the project

The tool does not see the user's code. You do. Before presenting:

- **Compare each part with what the project needs**: logic voltage, interface, current, pin
  count, board family. Open any part you are unsure about with `get_product` and read the listing.
- **Quote, do not infer.** Product specs come from free-text listings. If a listing does not state
  a voltage, pinout or current rating the design depends on, say that it is not stated and that
  the user should check the data sheet. Never fill in a pinout or voltage from memory.
- **Say what is missing.** If the project needs something the list does not cover, or the store
  has no match, say so plainly rather than substituting something that only looks similar.

## Step 4 — Present it

- A table: part, why it is needed, quantity, price in INR, availability. Then the total.
- All prices, stock and the total come from the tools, in rupees. Do not adjust or re-add them
  from memory; if you change the list, re-check with `check_stock`.
- List out-of-stock items. `alternatives` are in-stock search results offered as candidates, not
  confirmed substitutes, and only the first few sold-out parts get any: check a candidate against
  the project (Step 3) before suggesting it. `unknown` availability is unknown, not in stock.
- The cart link holds only parts the store confirmed and did not report sold out. Name every
  part in `not_on_cart_link` so the user knows the link does not cover the whole list.
- List separately the things you assumed and the things the user should verify.
- **Before the cart link, say that opening it replaces whatever is already in their store cart.**
  Then give the link. Nothing is bought until the user checks out on the store themselves; there
  is no tool that pays.

Offer to save the parts list to a file in the project (for example `BOM.md`) only if the user
wants it. It is a public catalogue list, so that is safe; never add order or account details to
it.

## When things go wrong

- **"Rate limit reached for this tool. Retry after N seconds."**: this means wait, not that the
  store has no parts. N can be many minutes, because builds are counted per hour. Tell the user
  how long, and do not retry before then. Meanwhile you may offer to look up the main parts one
  at a time with `search_products`, which has its own, larger budget.
- **The builder answers in words instead of a list** (the description was too vague, it is not a
  hardware build, or nothing suitable is in stock): the tool returns an error carrying what the
  builder said. Pass that on, fix the description with the user, and remember that each new
  attempt uses one of the hour's builds.
- **Other tool error or timeout:** say the store's builder could not be reached. Do not write a
  parts list from memory with invented prices.
- **The store does not stock a needed part:** say so, and name the requirement that is unmet.
- **Server not connected:** tell the user to connect the ThinkRobotics server in their host's
  MCP or connector settings.
