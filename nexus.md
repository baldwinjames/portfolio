# Nexus

**A voice-first AI chief of staff.** Fifteen intelligence engines running continuously over calendar, meetings, colleagues, projects and commitments, deciding what actually needs a person and staying quiet about everything else.

| Intelligence engines | Backend services | Tests at freeze | Process-context layers |
|:--:|:--:|:--:|:--:|
| **15** | **93** | **~3,982** | **6** |

---

## The premise

A knowledge worker's context is scattered across Slack, calendar, transcripts, Confluence, Jira, a CRM and their own memory. Most tools answer that by aggregating everything into a dashboard, which moves the sorting problem from many places to one place without solving it.

Nexus inverts it. Every engine below exists to earn the right to interrupt. Signals are detected, correlated against other signals, weighed against what you are currently doing, and then mostly suppressed. What survives is a draft to approve, a call to prepare for, or a decision only you can make.

**The hard part is deciding not to speak.**

---

## The system

Four phases, one loop. Sensing feeds reasoning, reasoning drives action, and what you do with the result feeds back to raise or lower the bar for interrupting you again.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="diagrams/nexus-phases-dark-v2.svg">
  <img width="680" alt="The four-phase intelligence cycle: sense feeds reason, reason drives act, act feeds learn, and learn raises the interrupt threshold back at reason." src="diagrams/nexus-phases-light-v2.svg">
</picture>

A sixteenth engine, the **NexusDaemon**, sits outside that cycle and keeps time for it: every five minutes it checks whether a briefing is due, a meeting is thirty minutes out, signals are stale, or autonomous tasks are ready.

| Phase | Engines |
|---|---|
| **Sense** | Signal Scanner · Native OS Intelligence · Persona Engine · Commitment Extractor · Post-Meeting Processor |
| **Reason** | Correlation Engine · Governor · Conversation Agent · Briefing Agent · Meeting Prep Agent · Task Intelligence Agent |
| **Act** | Execution Orchestrator · Instruction Compiler |
| **Learn** | Adaptive Intelligence · MemRL |

The two diagrams below trace the paths that matter most: how a detected signal either reaches you or doesn't, and how a meeting is prepared for and then digested.

## How a signal becomes an interruption, or doesn't

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="diagrams/nexus-signal-dark-v2.svg">
  <img width="360" alt="How a detected signal passes through correlation and the Governor to either reach you or be suppressed, with the learning loop feeding back." src="diagrams/nexus-signal-light-v2.svg">
</picture>

The loop closes on the Governor. Every time a surfaced signal is accepted, corrected or ignored, a confidence-scored rule is created or adjusted. Rules that get reinforced become permanent preferences; rules that get contradicted decay and deactivate.

The practical effect is that the threshold for interrupting you **rises over time** on the signal types you have shown you do not care about.

---

## The meeting loop

The second continuous cycle. Preparation happens before you arrive; digestion happens without you asking.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="diagrams/nexus-meeting-dark-v2.svg">
  <img width="360" alt="The meeting loop: daemon tick, persona refresh, meeting prep, then post-meeting transcript processing feeding commitments and tasks." src="diagrams/nexus-meeting-light-v2.svg">
</picture>

Thirty minutes out, attendee profiles are refreshed if stale and prep is assembled from personas, open commitments and project context. When the meeting ends the transcript is pulled and run through commitment extraction, signal extraction and persona enrichment, so the profiles used for the next meeting are already better than the ones used for this one.

---

## Two models, split by what they are good at

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="diagrams/nexus-voice-dark-v2.svg">
  <img width="230" alt="The dual-model voice architecture: the instruction compiler seeds the OpenAI Realtime session, which calls Claude for reasoning." src="diagrams/nexus-voice-light-v2.svg">
</picture>

Speech-to-speech is where you want the conversation, not the thinking. The Realtime API holds the turn and keeps the voice natural over a WebSocket, while Claude reasons behind exposed tool definitions. Async function calling means the voice can keep talking through work that takes seconds.

Measured cost of the split, written down rather than assumed:

| Constraint | Measured |
|---|---|
| Added latency on complex queries | 800 – 2,000 ms |
| Session length | Hard limit, no automatic reconnection |

---

## Six views, driven by voice

The screen follows the conversation. Ask about a person and People appears; make a decision and it executes and confirms.

| View | What it holds |
|---|---|
| **Home** | Morning briefing, today's plan with readiness scores, blocked decisions, active signals |
| **Calendar** | Per-meeting preparation scored 0–100%, with talking points and open commitments per attendee |
| **People** | Colleague profiles with relationship health, communication style, and an epistemic grade from A to F |
| **Projects** | Roadmap and consulting portfolios with health, execution state and QBR readiness |
| **Analytics** | Adoption and impact metrics as conversational insights, every number drillable |
| **Live** | The voice surface. Speech, touch and keyboard together |

---

## Every engine

<details>
<summary><b>SENSE</b> — five engines that watch</summary>

<br/>

**Signal Scanner** · *Built, scans Supabase and BigQuery*
Watches adoption metrics and KPI data for anomalies: adoption drops, usage spikes, power-user dropoff, gaps and KPI movement. Anything unusual becomes a signal.

**Native OS Intelligence** · *Built, Electron main process*
Observes apps, focus, processes and screen at the OS level, then classifies focus state on a five-minute sliding window. This is what tells the Governor whether you are in deep work.

**Persona Engine** · *Built, multi-source enrichment*
Builds a profile of every colleague from meeting transcripts, Slack, Confluence, calendar and native observation. Tracks communication style, decision patterns, relationship health, and an epistemic grade from A to F for how well it actually understands them.

**Commitment Extractor** · *Built, deadline tracking*
Reads meeting transcripts for mutual commitments, both what you promised and what was promised to you, with explicit or inferred deadlines. Overdue items surface in briefings and meeting prep on their own.

**Post-Meeting Processor** · *Built, daemon-triggered*
On meeting end, pulls the transcript and runs commitment extraction, signal extraction and persona enrichment, creates tasks from action items, and stores a structured summary. No manual trigger.

</details>

<details>
<summary><b>REASON</b> — six engines that decide</summary>

<br/>

**Correlation Engine** · *Built, graph traversal*
Traverses the entity graph breadth-first to three hops to find connections between separate signals, then has Claude write the correlation chain and a suggested action. An adoption drop plus a stakeholder going quiet becomes one insight rather than two data points.

**Governor** · *Built, Silent / Proactive / Strategic*
Sits between detection and surfacing. Weighs each signal's urgency against your current focus state and decides to surface, queue or suppress. Critical act-now signals are never suppressed. The interruption modeller feeds it, learning which signal types you consistently ignore.

**Conversation Agent** · *Built, text and voice*
The reasoning loop behind every chat and voice turn. Runs up to ten tool iterations per turn across 18 tools in parallel, then synthesises. In voice mode it streams with sentence-boundary detection so speech lands naturally.

**Briefing Agent** · *Built, scheduled and on demand*
Gathers tasks, open commitments, drift signals, calendar and project status into a morning or evening briefing, and pre-populates the daily plan so you subtract rather than build from scratch.

**Meeting Prep Agent** · *Built, daemon-triggered*
Thirty minutes before a meeting, pulls attendee personas, open commitments with those people, project context and recent interactions, then generates talking points, coaching per communication style, and flags risks like an overdue promise to an attendee.

**Task Intelligence Agent** · *Built*
Enriches each task with context from Notion, Confluence, transcripts and project data, produces an execution brief naming dependencies and missing information, and assigns a readiness score so unready work does not clutter the plan.

</details>

<details>
<summary><b>ACT</b> — two engines that do</summary>

<br/>

**Execution Orchestrator** · *Built, concurrent sessions*
Runs the tasks that do not need your judgment. Builds a context package, picks a transport between the Claude Agent SDK and a terminal session, monitors progress and self-heals on failure. A three-tier safety gate separates auto-execute from confirm from block, and it never sends external communications unasked.

**Instruction Compiler** · *Built, 8 sections, ~2,000 tokens*
At voice session start, queries every intelligence component for learned rules, pending commitments, active signals, predictions and recent session memory, then assembles a prioritised instruction set inside a per-section character budget.

</details>

<details>
<summary><b>LEARN</b> — two engines that improve</summary>

<br/>

**Adaptive Intelligence** · *Built, 9 service files*
Three parts. The learning engine turns every correction into a confidence-scored rule that strengthens on reinforcement and decays on contradiction. The signal cortex scores cross-system events for urgency without API calls. The instruction compiler assembles the result into live context.

**MemRL** · *Built, not yet wired into prompt assembly*
Stores every output as a trajectory with a Q-value that rises on acceptance and falls on neglect. High scorers become exemplars, low scorers get pruned. Scoring, promotion, pruning and feedback logging are all implemented.

</details>

<details>
<summary><b>NexusDaemon</b> — the heartbeat</summary>

<br/>

**NexusDaemon** · *Built, 10+ scheduled jobs*
A five-minute heartbeat that checks whether a briefing is due, a meeting is 30 minutes out, signals are stale, commitments are approaching deadlines, or autonomous tasks are ready. Each check is independent and wrapped so one failure never blocks the others.

</details>

---

## Status

Frozen deliberately in July 2026 at a written gate, with a handoff covering current state, remaining work, and the decisions that must not be casually reversed, so another engineer or agent could resume it cold. It stopped because it had answered the question it was built to answer.

Next on the list: **MemRL** is scoped, built and tested, but its integration into the conversation agent's prompt assembly is not wired in yet.

---

## What it runs on

| Layer | Technology | Purpose |
|---|---|---|
| Desktop | Electron 35, macOS arm64 | System tray, global shortcuts, OS-level observation |
| UI | Svelte 5 runes | Six views on a shared token-driven design system |
| Backend | Express 4, localhost only | 93 service files, 31 route files, runs as a child process |
| Reasoning | Claude, Anthropic SDK and Agent SDK | Conversation, briefing, execution, enrichment |
| Voice | OpenAI Realtime API | Speech-to-speech with barge-in and semantic VAD |
| Local STT | Sherpa-ONNX | On-device transcription, no cloud path |
| Local store | better-sqlite3 | Persona data, trajectories, learned rules, intelligence state |
| Cloud store | Supabase, Postgres and pgvector | Tasks, analytics, vector search |
| Sources | Google Calendar, Granola, Slack, Confluence, BigQuery | Meetings, transcripts, messages, docs, KPIs |

---

<sub>Source is private. Every count and status on this page comes from the repository at its July 2026 freeze.</sub>
