# Portfolio

Case studies on systems I have designed and built. Most of the production work lives in private repositories owned by former employers, so what is here describes architecture, decisions and measured outcomes rather than source. Where a claim carries a number, the number came from an instrument, and I say so when it did not.

I ran global customer support organizations for thirteen years. In October 2025 I moved out of management to build, and everything below except the earliest work comes from the period since.

## Case studies

| System | What it is | Where the evidence is |
|---|---|---|
| [Sincta](./sincta.md) | Privacy-first on-device dictation for macOS. Electron over two native Swift bridges. Replaces a commercial cloud tool with one that has no network path. | [Public repository](https://github.com/baldwinjames/sincta): commit history, nine architecture decision records, committed benchmark JSON, 86 written-up defects |
| [Nexus](./nexus.md) | A voice-first AI chief of staff. Layered agents over a 93-module backend, with speech on OpenAI's Realtime API and reasoning on Claude. | ~3,982 tests and per-agent eval suites. Personal R&D, single user. |
| [Investigation workflow](./investigation-workflow.md) | An agentic workflow running the full Tier 2 support lifecycle: route, map the codebase, pull logs, check infrastructure, reach a root cause. | Benchmarked and engineering-validated. Source is private. |
| [Customer Intelligence MCP](./customer-intelligence-mcp.md) | A company's first internal MCP server. Python on AWS Bedrock AgentCore behind Cognito OAuth and PKCE. | 744 tests, adopted by 45% of the company inside 30 days. Source is private. |
| [Evaluation practice](./evaluation-practice.md) | How I decide whether a change is allowed to ship, and the four times my own measurement told me no. | Suites and results are private. The method is here. |

## A note on numbers

I keep my claims in tiers, and I will tell you which tier a number is in if you ask.

**Measured** means an instrument produced it, I know the method, and I can tell you the date. **Reported** means it is real and attested but nothing was instrumented. Anything I cannot put in one of those two buckets does not go in a document with my name on it.

The reason this matters is in [evaluation-practice.md](./evaluation-practice.md). I have built a retrieval change that reported a 28.8-point improvement and measured two points net negative. The number was not lying. The measurement was.
