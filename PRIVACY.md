# Privacy

This page describes what the Asimov by ThinkRobotics plugin and its two servers receive, return and
keep. It covers the plugin only. Purchases and your store account are covered by the privacy
policy on [thinkrobotics.com](https://thinkrobotics.com).

## Who is involved

| Party | Role |
|---|---|
| You | Run an AI agent (Claude Code, the Claude app, Cursor, Codex or another MCP client). |
| Your agent host and its AI provider | Run the model. They see everything the agent reads, including every tool result. Their own privacy terms apply. |
| ThinkRobotics | Runs the two servers at `mcp.thinkrobotics.com`, the store and the support desk. |

The plugin itself is a set of text files installed on your machine or account. It contains no
code, runs nothing in the background, and collects nothing.

## Public server — `https://mcp.thinkrobotics.com/mcp`

No sign-in. It does not know who you are.

| Tool | It receives | It returns |
|---|---|---|
| `search_products` | Your search terms and filters. | Public catalogue entries: title, link, price, summary, availability. |
| `get_product` | A product handle, URL or product id. | The public product listing. |
| `check_stock` | SKUs or variant ids. | Public price and availability. |
| `ask_store_policy` | Your question. | The store's answer and links to its policy pages. |
| `build_parts_list` | A description of your project, and optionally a budget and parts you own. | A parts list and a cart link. |

- **Your project files are not uploaded.** The `build-project` skill has your agent read them
  locally and send a short description of the build. What that description contains is up to
  your agent, so do not ask it to build a parts list from files that hold secrets.
- `build_parts_list` is answered with the help of an AI model operated for ThinkRobotics, so
  the project description, budget and list of parts you own are processed by that model's
  provider. Each call is a new anonymous session that is not linked to you or to earlier calls.
- Like any web server, it sees your IP address and your client's name (user agent). The address
  is used for rate limiting, and the server's request logs record it as a salted hash, not as
  the address itself. The
  user agent is used to tell which host (Claude Code, Cursor and so on) a call came from.
- Buy and cart links carry tags (`utm_source=asimov-mcp`, the host name as `utm_medium`, and
  a campaign name) so the store can count orders that came from the plugin. They carry nothing
  that identifies you.

## Account server — `https://mcp.thinkrobotics.com/account/mcp`

Sign-in required. Use it only if you want order or ticket tools.

**Sign-in.** You sign in on the store's own sign-in page with your store email and a one-time
code, then approve access for the host you are using on a ThinkRobotics page. The server learns
your store customer identity from that sign-in and nothing else: no tool accepts an email, phone
number or customer id as input. Your agent and its host receive tokens for this server only.
They never receive your one-time code or any store password.

**Permissions.** Reading orders, reading tickets and opening tickets are separate permissions,
each with its own tick box on the approval page. You can grant read access only.

| Tool | It receives | It returns |
|---|---|---|
| `list_my_orders` | How many orders to list. | Order number, date, status, payment status, total. |
| `get_my_order` | An order number. | Items, status, tracking, GST invoice link. |
| `list_my_tickets` | Nothing beyond your sign-in. | Your tickets: number, subject, status. |
| `get_my_ticket` | A ticket id from the list. | The ticket's opening text, status, priority and category. Not the later replies. |
| `reorder` | An order number. | A cart link for the in-stock items. |
| `create_ticket` | The subject and description you approved, and optionally an order number. | Ticket number and status. |

No tool replies to a ticket.

**Order data is kept to a minimum.** Tool results are read by your AI provider and stored in your
conversation history, so the server leaves out what the agent does not need. It does not return
postal addresses, phone numbers, email addresses or payment details. An order or ticket number that is not
yours returns "not found".

**Writes need your approval.** The skills tell the agent to show you the exact text and get a yes
before it opens a ticket, and hosts are told this tool changes data so that they can ask as
well. Tickets opened this way are marked as coming from Asimov by ThinkRobotics and name the tool that
sent them. A customer can open only a few tickets a day this way.

**No payments.** Neither server can place an order, charge you or change your account. Buying
always ends in a link that you open and check out yourself.

## What ThinkRobotics keeps

- **Request logs**: time, tool name, host name, user agent, a salted hash of the IP address,
  duration, outcome and a request id. What you send to a tool and what it returns, including
  order and ticket content, is not written to these logs.
- **Short-lived caches** of public catalogue and policy answers, to stay within the store's
  limits. They hold no personal data.
- **Sign-in records**: which hosts you approved, your store customer id and the permissions you
  gave, until access is revoked or expires. The tokens issued to your host are stored only as
  one-way digests. Your email, name and phone number are not stored in these records, and the
  store's own sign-in token is discarded as soon as it has identified you.
- **Tickets** you open or reply to are kept in the support desk like any other support request.

ThinkRobotics does not sell this data or use it for advertising.

## What stays with your host

Your conversations, including tool results that contain order or ticket details, are stored by
your agent host under its own terms. The skills tell the agent not to write order details into
files, commits or notes, but you should treat a conversation that contains them as personal
data.

## Revoking access

You can do any of these at any time:

1. **Disconnect in your host.** In Claude Code, run `/mcp`, choose `thinkrobotics-account` and
   select **Clear authentication**, or run `claude mcp logout thinkrobotics-account`. In the
   Claude app, Cursor and Codex, disconnect or remove the **thinkrobotics-account** server where
   you manage connectors or MCP servers. This removes the tokens from that host. A host that
   supports revocation also tells the ThinkRobotics server, and access for that host then stops
   at once.
2. **Uninstall the plugin.** This removes the skills and server entries. It does not by itself
   revoke access you already granted, so do step 1 first.

There is no ThinkRobotics page that lists or removes connected hosts; revoking is done from the
host. If a host removes its tokens without telling the server, nothing can use that access any
more, and the records expire on their own: an access token lasts about an hour and can be
renewed for at most 30 days after its last use. When ThinkRobotics erases your customer data at your
request, every approval is removed at once.

## Questions

For anything about this page or your data, contact ThinkRobotics through
[thinkrobotics.com](https://thinkrobotics.com), or open an issue on the plugin repository for
questions that contain no personal details.
