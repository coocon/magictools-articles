# July 28 Was MCP's Coming of Age: The Protocol Dropped State, the Ecosystem Got Its First Lawsuit

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/mcp-stateless-spec-first-lawsuit-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/mcp-stateless-spec-first-lawsuit-en?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 Continue reading: July 28 Was MCP's Coming of Age: The Protocol Dropped State, the Ecosystem Got Its First Lawsuit](https://tools.cooconsbit.com/en/articles/mcp-stateless-spec-first-lawsuit-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
