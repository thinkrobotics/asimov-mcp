# Asimov by ThinkRobotics

Search the [ThinkRobotics](https://thinkrobotics.com) electronics and robotics catalogue from
inside your project. With this plugin your coding agent can find parts, read their listings, check
live price and stock in Indian rupees, quote store policies, and turn a project into a parts list
with a buy link. If you sign in with your store email, it can also look up your orders and support
tickets.

It works in Claude Code, the Claude app (chat and Cowork), Cursor and OpenAI Codex, and its two
servers work in any other MCP client.

## What is in it

The plugin is Markdown and JSON only: four skills and pointers to two remote MCP servers. It has
no hooks, no executables and no local server, and it needs no API key.

| Skill | When the agent uses it |
|---|---|
| `find-parts` | You are choosing, pricing or checking stock of a component. |
| `build-project` | You want a parts list for a whole build. It reads your project files (`platformio.ini`, sketches, `requirements.txt`, wiring notes, README) to work out what is needed. |
| `store-policies` | You ask about shipping, returns, warranty or similar. Answers are quoted from the store with a link. |
| `orders-and-support` | You ask about your own orders or support tickets (needs sign-in). |

### Public server — no sign-in

`https://mcp.thinkrobotics.com/mcp` (Streamable HTTP). All tools are read-only.

| Tool | What it does |
|---|---|
| `search_products` | Search the catalogue; optional maximum price in INR and in-stock filter. 5 results by default, 10 at most. |
| `get_product` | One product's listing by handle, page URL or product id: description, options, images, and its variants with SKU, price, stock and buy link. It says when a product has more variants than it lists. |
| `check_stock` | Live price and availability for up to 20 SKUs or variant ids. A SKU the store does not have is reported as not found, which is different from out of stock. |
| `ask_store_policy` | The store's own answer to a policy question, with links to the policy pages on that topic. |
| `build_parts_list` | A parts list for a described project, with a reason per part, live price and stock, a total, candidate alternatives for some out-of-stock items, and one cart link for the parts that are in stock. Can take a couple of minutes. |

The server limits calls per client address: about 30 catalogue calls a minute (each SKU
in a stock check counts as one), and about 5 parts-list builds an hour. Past that a tool answers
"Rate limit reached for this tool. Retry after N seconds." That means wait; it does not mean the
store has nothing.

### Account server — sign-in required

`https://mcp.thinkrobotics.com/account/mcp` (Streamable HTTP, OAuth).

| Tool | Kind | What it does |
|---|---|---|
| `list_my_orders` | read | Your recent orders: number, date, status, payment status, total. |
| `get_my_order` | read | One order's items, status, tracking and GST invoice link. |
| `list_my_tickets` | read | Your support tickets: number, subject, status. |
| `get_my_ticket` | read | One ticket's status and its opening text. It does not show later replies. |
| `reorder` | read | A cart link for a past order's in-stock items. |
| `create_ticket` | write | Opens a support ticket. The agent shows you the exact text and asks first. A few per day at most. |

The plugin cannot reply to a ticket or show support's replies. Answer support by replying to the
email they sent you, or through [thinkrobotics.com](https://thinkrobotics.com).

No tool pays for anything. Buying always ends in a link that you open and check out yourself.
**A buy or cart link replaces whatever is already in your store cart.**

## Install

### Claude Code

```
/plugin marketplace add thinkrobotics/asimov-mcp
/plugin install asimov-mcp@asimov-mcp
```

Then run `/mcp` and choose **thinkrobotics-account** to sign in. The public server needs no
sign-in.

### Claude app (chat and Cowork)

1. Open **Customize > Plugins**, choose **Add marketplace**, and enter
   `thinkrobotics/asimov-mcp`.
2. Install **Asimov by ThinkRobotics**.
3. On the plugin's **Connectors** tab, connect the two servers. The account server asks you to
   sign in.

Without the plugin, you can add either server URL above as a custom connector. You get the tools
but not the skills.

### Cursor

Install **Asimov by ThinkRobotics** from **Customize**, or from the Cursor Marketplace once it is listed.

To add only the MCP servers, without the skills, use an install link:

- [Add the public server to Cursor](https://cursor.com/link/mcp/install?name=thinkrobotics&config=eyJ1cmwiOiJodHRwczovL21jcC50aGlua3JvYm90aWNzLmNvbS9tY3AifQ==)
- [Add the account server to Cursor](https://cursor.com/link/mcp/install?name=thinkrobotics-account&config=eyJ1cmwiOiJodHRwczovL21jcC50aGlua3JvYm90aWNzLmNvbS9hY2NvdW50L21jcCJ9)

The same links in Cursor's own scheme, for pages that allow it:

```
cursor://anysphere.cursor-deeplink/mcp/install?name=thinkrobotics&config=eyJ1cmwiOiJodHRwczovL21jcC50aGlua3JvYm90aWNzLmNvbS9tY3AifQ==
cursor://anysphere.cursor-deeplink/mcp/install?name=thinkrobotics-account&config=eyJ1cmwiOiJodHRwczovL21jcC50aGlua3JvYm90aWNzLmNvbS9hY2NvdW50L21jcCJ9
```

Or add them to `~/.cursor/mcp.json` by hand:

```json
{
  "mcpServers": {
    "thinkrobotics": { "url": "https://mcp.thinkrobotics.com/mcp" },
    "thinkrobotics-account": { "url": "https://mcp.thinkrobotics.com/account/mcp" }
  }
}
```

### OpenAI Codex

```
codex plugin marketplace add thinkrobotics/asimov-mcp
```

Then open `/plugins` and install **Asimov by ThinkRobotics**. To sign in to the account server:

```
codex mcp login thinkrobotics-account
```

### Any other MCP client

In VS Code, Gemini CLI, Windsurf, ChatGPT and other MCP clients, add the server URLs above as
remote (Streamable HTTP) servers. To add the skills on their own:

```
npx skills add thinkrobotics/asimov-mcp
```

## Signing in

Only the account server needs it, and only if you want order or ticket tools.

- Your host opens a ThinkRobotics page in your browser. You enter the email you use at
  thinkrobotics.com and the one-time code the store sends to it. There is no password, and
  neither the agent nor the host sees the code.
- You then approve access for that host. Reading orders, reading tickets and opening tickets are
  separate permissions, each with its own tick box.
- The agent never asks for your email or a customer id, and no tool accepts one. It can only see
  the account you signed in with.
- To sign out and revoke access, disconnect the server in your host's MCP or connector settings.
  There is no separate ThinkRobotics page for this. See
  [PRIVACY.md](PRIVACY.md#revoking-access).

## What data it sends

Your search terms, policy questions and project descriptions go to the ThinkRobotics servers,
along with your IP address. The `build-project` skill reads your project files locally and sends
a short description of the build, not the files. With sign-in, the account server returns your
order and ticket details to your agent, which means they pass through the AI provider you use.
The full description is in [PRIVACY.md](PRIVACY.md).

## Things to know

- Prices are in Indian rupees and the store ships from India.
- Product specs come from free-text listings. The skills tell the agent to quote them and to say
  when a listing does not state a voltage, pinout or rating, instead of guessing. Check the
  manufacturer's data sheet for anything your design depends on.
- If the store cannot be reached or is rate-limiting, the tools return an error, and the agent
  should tell you so rather than answer from memory.

## Repository layout

```
skills/                           the four skills (Agent Skills format)
.mcp.json                         MCP servers for Claude Code and the Claude app
mcp.json                          MCP servers for Cursor, Codex and other Agent Plugins hosts
plugin.json                       Agent Plugins 1.0 manifest (Codex, Cursor)
.claude-plugin/plugin.json        Claude manifest
.claude-plugin/marketplace.json   Claude marketplace entry
.cursor-plugin/plugin.json        Cursor manifest
.agents/plugins/marketplace.json  Codex marketplace entry
```

## Support

For a problem with the plugin, open an issue on this repository. For an order or a product, sign
in and ask your agent to open a support ticket, or contact ThinkRobotics through
[thinkrobotics.com](https://thinkrobotics.com).

## Licence

[MIT](LICENSE)
