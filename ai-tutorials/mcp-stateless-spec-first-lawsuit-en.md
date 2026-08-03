---
title: "July 28 Was MCP's Coming of Age: The Protocol Dropped State, the Ecosystem Got Its First Lawsuit"
slug: mcp-stateless-spec-first-lawsuit-en
summary: "Two milestones in one day. The largest MCP spec revision ever shipped: the initialize handshake and session IDs are gone, the core goes stateless, and Roots, Sampling, Logging, and HTTP+SSE enter a 12-month deprecation countdown. The same day, gateway startup Runlayer sued Rippling, alleging a year-long 'customer trial' that ended in a cloned product — MCP's first commercial lawsuit. What breaks, what each kind of developer should do, and why these two events read as one story."
category: ai-tutorials
tags: [MCP, Model Context Protocol, stateless, agents, protocol spec, Runlayer, Rippling, MCP gateway, LLM]
coverImage: ""
status: published
locale: en
source: authored
translationSlug: mcp-stateless-spec-first-lawsuit
---

# July 28 Was MCP's Coming of Age: The Protocol Dropped State, the Ecosystem Got Its First Lawsuit

On July 28, 2026, two things happened to the MCP (Model Context Protocol) ecosystem on the same day.

The first was at the spec layer: the [MCP 2026-07-28 specification](https://modelcontextprotocol.io/specification/2026-07-28/changelog) shipped, described by its maintainers as "the largest revision of the protocol since launch." The core change fits in one sentence: **MCP no longer has state.** The initialize handshake is gone, session IDs are gone, and the protocol moves from stateful bidirectional connections to stateless request/response.

The second was at the business layer: MCP security-gateway startup Runlayer [sued Rippling](https://techcrunch.com/2026/07/28/mcp-startup-runlayer-accuses-rippling-of-stealing-its-product-idea/), alleging that the HR-tech company spent nearly a year in a "prospective customer" trial, obtained the product roadmap and source code, then turned around and built a competing MCP gateway of its own. Three claims: trade secret misappropriation, unfair competition, breach of contract. It is the MCP ecosystem's first commercial lawsuit.

In a single day, one protocol did two things: it bowed to operational reality, and it proved itself worth suing over. Either event alone would be significant. Read together, they are the full story — **a technology's rite of passage from experiment to infrastructure, compressed into 24 hours.**

Let's take them apart one at a time.

## The spec side: what got cut, and why

The new spec evolved from a Release Candidate locked on May 21, with a ten-week window for SDK maintainers across languages to validate the changes. The major changes cluster around a few SEPs (Specification Enhancement Proposals):

**SEP-2575 removes the initialize handshake.** Under the old protocol, client and server exchanged versions and capabilities before doing any work. Under the new one, that information travels with every request — protocol version and client capabilities ride in the `_meta` field (`io.modelcontextprotocol/protocolVersion`, `io.modelcontextprotocol/clientCapabilities`), alongside three HTTP headers: `MCP-Protocol-Version`, `Mcp-Method`, and `Mcp-Name`. Want to know what a server can do? A new `server/discover` endpoint answers at any time, no handshake required.

**SEP-2567 removes Mcp-Session-Id.** The protocol-level session concept is gone entirely. Tools that genuinely need cross-request state use explicit server-minted handles, passed back as ordinary arguments.

**MRTR (Multi Round-Trip Requests) replaces server-initiated requests.** Previously, a tool needing mid-execution user confirmation depended on a long-lived bidirectional stream. In the new design, the server simply returns `resultType: "input_required"` with its question; the client re-invokes the call with the user's answer attached. No persistent connection, same conversation.

Why the change? Because the old design hit a wall in production. The classic failure: multiple instances behind a load balancer, pod A issues a session ID, the next request routes to pod B, 404. Your options were sticky sessions or shared session stores, and serverless or edge deployment was effectively off the table. **Stateful protocols and horizontal scaling are natural enemies** — a lesson web backends learned twenty years ago, and MCP has now paid its tuition on.

The direction makes sense once you remember MCP's origins: a desktop-era design — stdio transport, one user, one process — where keeping state in memory was perfectly reasonable. But with SDK downloads [exceeding 400 million per month](https://blog.modelcontextprotocol.io/posts/2026-07-28/) (4x growth in a year) and servers running on other people's clouds serving tens of thousands of users, desktop-era assumptions became liabilities. Going stateless isn't a flex. It's the protocol formally acknowledging how it is actually used.

## The deprecation list: a stronger signal than statelessness

The revision also deprecates a batch of features, and I'd argue **the deprecations say more about MCP's direction than the stateless core does**:

- **Roots, Sampling, and Logging are deprecated together** (SEP-2577). The official replacements are blunt: pass directories as tool parameters, call LLM provider APIs directly, send logs to stderr or OpenTelemetry.
- **The HTTP+SSE transport is deprecated** (SEP-2596); Streamable HTTP becomes the sole recommendation.
- Deprecation policy: flagged features remain for at least 12 months, with removal no earlier than July 28, 2027. New clients auto-fall-back to the initialize handshake when they meet an old server, so old and new keep interoperating.

Note the subtext in Sampling's deprecation. Sampling let an MCP server borrow the client's model for inference — it was the ambitious part, the part where MCP aspired to be *the* universal agent-era protocol. Now the official guidance is "just call the LLM API directly," which amounts to **voluntarily ceding that territory.** Add Roots and Logging to the exit list and the revision's real theme surfaces: MCP is narrowing its ambition, retreating from "an agent protocol that manages everything" to "a protocol that does tool calling extremely well." In protocol design, the willingness to delete is usually a better maturity signal than the urge to add.

## Will your stuff break?

Three tiers, by audience:

1. **You just configure MCP servers in Claude Code / Cursor / VS Code: you'll barely notice.** Config file formats (`.mcp.json` and friends) are untouched; what changed is the wire protocol underneath. Incidentally, all four client formats covered by our [MCP Config Generator](/tools/mcp-config-generator) remain valid — and its SSE deprecation warning has just been upgraded from "heads-up" to "official."

2. **You build servers on an official SDK: upgrade and mostly relax.** Handshake fallback and header injection are handled for you. What you do need to audit is whether your business logic quietly depends on per-connection state — list results are now treated as cacheable and connection-independent, so dynamic per-connection tool lists need to migrate to explicit handles.

3. **You hand-rolled a transport layer, or you build gateways/proxies: real work ahead.** Handshake removal, required headers, `resultType`, new error codes — all of it lands in code you own. Twelve months sounds generous, but gateway products must speak both worlds simultaneously, which doubles the work.

## The lawsuit side: why the first case is about the gateway layer

Now the other event of the day. Runlayer's account: the parties signed a mutual NDA and a trial agreement prohibiting copying; Rippling ran a deep evaluation for nearly a year as a prospective customer, receiving everything up to and including source code; the deal died on price; and Runlayer says insiders then reported Rippling was using those materials to build a clone. Rippling confirms it is building its own MCP gateway but denies misappropriation, calling the suit a "panicked effort to avoid competition by fabricating claims." (One detail worth flagging: reports disagree on the filing venue; go by the court documents as they emerge.)

The courts will sort out who's right. For bystanders, two structural facts carry more information than the verdict will:

**First, MCP's first lawsuit is about the gateway layer, not the server layer.** Nobody has sued over any of the tens of thousands of MCP servers, because servers themselves aren't where the money is. The value sits at the control point where enterprises adopt MCP: authentication, auditing, permission governance. Where the money is, the lawsuits follow. This case effectively certifies that MCP gateways are a real business: **only things worth stealing are worth suing over.**

**Second, trial agreements don't stop customers with engineering muscle.** Every team selling AI infrastructure to large companies should pin this case to the wall: when your "prospective customer" employs hundreds of engineers, a year of deep evaluation is simultaneously a year of requirements research and competitive analysis. An NDA can stop documents from leaking. It cannot stop someone from having seen the right answer before building their own.

One layer of irony deserves its own line: **the spec released that very day chips away at part of the gateway vendors' moat.** With state gone, request-level routing becomes trivial — no sticky sessions, no session affinity — and proxies and gateways get dramatically easier to implement. The more mature the protocol, the thinner the purely technical barrier, and the more gateway vendors must stand on governance, compliance, and auditing — the hard, boring parts. The day Runlayer filed suit, the technical wall around its own market got a little lower.

## Reading the two events together

Line up July 28's two headlines and you see two classic markers of infrastructure maturity arriving at once:

**The protocol starts paying for operations** — cutting state, publishing a deprecation policy, guaranteeing a 12-month window, auto-falling-back for compatibility. These disciplines only grow on projects carrying serious production traffic. Experiments don't need deprecation policies, because nobody depends on experiments.

**The ecosystem starts fighting over money** — a first commercial lawsuit means companies have bet their futures on this protocol's surrounding market. Nobody litigates over a toy.

The action list for developers, in three items:

1. **Don't ride the window to its end.** The lesson of every 12-month deprecation period in tech history: procrastinators pile into the same quarter and hit the same potholes. If you're on an official SDK, upgrading now is as cheap as it will ever be.
2. **Inventory your dependence on Sampling, Roots, and protocol-level Logging.** All three are on death row with a stay of execution, and official replacement paths exist. Migration is labor, not difficulty — but only if you know what you're using.
3. **When evaluating AI infrastructure vendors, put "trial-period data boundaries" into contract review.** This cuts both ways, buyer and seller: whatever the Runlayer case's outcome, the industry habit of "deep trials that include source code access" is about to be repriced by this lawsuit.

## Coda

A protocol's maturity is never announced at a launch event. It is declared by two unglamorous documents: a deprecation schedule, and a legal complaint.

The first proves enough people depend on it that changes require commitments. The second proves enough money surrounds it to justify hiring Sullivan & Cromwell. On July 28, MCP collected both in a single day.

After that day, "emerging protocol" no longer fits. MCP is now receiving the full infrastructure treatment — including the parts that aren't fun.
