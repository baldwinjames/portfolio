# Portfolio

Case studies on systems I have designed and built. Most of the production work lives in private repositories owned by former employers, so what is here describes architecture, decisions and measured outcomes rather than source. Where a claim carries a number, the number came from an instrument, and I say so when it did not.

I ran global customer support organizations for thirteen years. In October 2025 I moved out of management to build, and everything here comes from the period since.

## Case studies

| System | What it is | Where the evidence is |
|---|---|---|
| [Nexus](./nexus/) | A voice-first AI chief of staff. Fifteen intelligence engines across six process-context layers, deciding what actually needs a person and suppressing the rest. Speech on OpenAI's Realtime API, reasoning on Claude. | ~3,982 tests, per-agent evaluation suites, 93 backend services. Personal R&D, single user. |
| [Sincta](./sincta/) | Privacy-first on-device dictation for macOS. Electron over two native Swift bridges. Replaces a commercial cloud tool with one that has no network path at all. | Nine decision records, committed benchmark JSON, ten gated phases, and every defect written up with its mechanism. Source is private. |
| [Investigation workflow](./investigation-workflow/) | An agentic workflow running the full Tier 2 support lifecycle: route, map the codebase, pull logs, check infrastructure, reach a root cause. | 91% routing match across a 50-ticket benchmark, zero misroutes. Validated by the VP of Engineering. Source is private. |
| [Customer Intelligence MCP](./customer-intelligence-mcp/) | A company's first internal MCP server. Python on AWS Bedrock AgentCore behind Cognito OAuth and PKCE. | 744 tests. 39 colleagues using it, every one active in the last 30 days. Source is private. |
| [Evaluation practice](./evaluation-practice/) | How I decide whether a change is allowed to ship, and the four times my own measurement told me no. | A 330-query deterministic suite with a frozen answer key. Suites and results are private. The method is here. |

