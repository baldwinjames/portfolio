# Portfolio

Case studies on systems I have designed and built. Most of the production work lives in private repositories owned by former employers, so what is here describes architecture, decisions and measured outcomes rather than source. Where a claim carries a number, the number came from an instrument, and I say so when it did not.

I ran global customer support organizations for thirteen years. In October 2025 I moved out of management to build, and everything here comes from the period since.

## Case studies

| System | What it is | Where the evidence is |
|---|---|---|
| [Nexus](./nexus/) | A voice-first AI chief of staff. Fifteen intelligence engines across six process-context layers, deciding what actually needs a person and suppressing the rest. Speech on OpenAI's Realtime API, reasoning on Claude. | ~3,982 tests, per-agent evaluation suites, 93 backend services. Personal R&D, single user. |
| [Sincta](./sincta/) | Privacy-first on-device dictation for macOS. Electron over two native Swift bridges. Replaces a commercial cloud tool with one that has no network path at all. | Nine architecture decision records, committed benchmark JSON, 86 defects written up with their mechanisms. Source is private. |
| [Investigation workflow](./investigation-workflow/) | An agentic workflow running the full Tier 2 support lifecycle: route, map the codebase, pull logs, check infrastructure, reach a root cause. | Benchmarked and engineering-validated. Source is private. |
| [Customer Intelligence MCP](./customer-intelligence-mcp/) | A company's first internal MCP server. Python on AWS Bedrock AgentCore behind Cognito OAuth and PKCE. | 744 tests, adopted by 45% of the company inside 30 days. Source is private. |
| [Evaluation practice](./evaluation-practice/) | How I decide whether a change is allowed to ship, and the four times my own measurement told me no. | Suites and results are private. The method is here. |

## A note on numbers

I keep my claims in tiers, and I will tell you which tier a number is in if you ask.

**Measured** means an instrument produced it, I know the method, and I can tell you the date. **Reported** means it is real and attested but nothing was instrumented. Anything I cannot put in one of those two buckets does not go in a document with my name on it.

That distinction is the whole point rather than a disclaimer. [Evaluation practice](./evaluation-practice/) covers the four retrieval changes my own gate caught before they shipped, and why a number can be honest and still be wrong about the thing you are asking it.
