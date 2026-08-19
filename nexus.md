# Nexus

A voice-first AI chief of staff for macOS. An autonomous operator watching calendar, relationships, commitments and a fleet of background agents, holding enough context to work out what actually needs a person, and surfacing only that: a draft to approve, a call to prepare for, someone to contact.

**Personal R&D, single user.** March to July 2026. Source is private.

## The problem it was built for

A working day generates far more signal than any one person can hold. Meetings that need preparation, commitments made in passing, people who have gone quiet, agents finishing work in the background. Most tools solve this by showing you everything and calling it a dashboard. I wanted the opposite: something that carried enough context to make the judgment call itself, and interrupted me only when the answer was genuinely mine to give.

## Shape of the system

Electron shell with a Svelte renderer, an Express backend of 93 modules running as a child process, Claude as the reasoning engine, OpenAI's Realtime API as the voice surface, and SQLite on disk for every piece of behavioural data.

The intelligence is layered rather than monolithic. Separate agents handle morning briefing, meeting preparation, session distillation and post-meeting reconciliation. An execution orchestrator sits above them, and a daemon on a five-minute heartbeat decides what has crossed the threshold into worth surfacing.

Local-only storage is a hard rule, not a setting. Every behavioural record stays in `~/.nexus`, and the app is designed to run with every external integration absent.

## The part worth reading: splitting speech from reasoning

The original voice pipeline was speech-to-text, then Claude, then text-to-speech. It worked and it felt like a pipeline rather than a conversation. The gold standard is speech-to-speech in a single model, where the pauses land where a person would put them.

The problem is that speech-to-speech models are not where you want your reasoning to happen. So ADR-003 records a dual-model architecture instead:

**OpenAI's Realtime API as the voice layer.** Speech-to-speech over WebSocket, handling conversation, greetings, follow-ups and reading results back. Semantic voice-activity detection for turn-taking rather than a fixed silence timeout, with eagerness tuned per context because a briefing and a rapid command need different patience.

**Claude as the reasoning layer.** All enrichment, meeting preparation, analytics and tool calling, reached through function definitions exposed to the Realtime session. Results come back as structured data the voice layer speaks naturally.

The interesting engineering is in the seam. Async function calling lets the voice keep talking while reasoning runs behind it, which is what makes a multi-second operation feel like a conversation rather than a wait.

What it costs, measured rather than assumed: 800 to 2,000ms of added latency on complex queries, and a hard session-length limit with no automatic reconnection, both written down as constraints rather than designed around on a guess.

## Testing

Roughly 3,982 tests, plus per-agent evaluation suites in Promptfoo covering meeting preparation, morning briefing, session distillation and post-meeting reconciliation.

The boot path is its own kind of test. The app starts with every integration absent, and views render honest labelled empty states rather than failing. A missing key is a designed condition, not a crash.

## Why it stopped

Frozen deliberately in July 2026 at a written gate, with a handoff package covering current state, remaining work, a fresh-machine install path, and the decisions that must not be casually reversed. The point was that another engineer, or another agent, could pick it up cold without access to the original machine or context.

It stopped because it had answered the question I built it to answer, not because it broke.
