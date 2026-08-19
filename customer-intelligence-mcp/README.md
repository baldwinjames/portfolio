# Customer Intelligence MCP server

A company's first internal Model Context Protocol server. Python on AWS Bedrock AgentCore, behind Cognito OAuth with PKCE.

**Where:** a venture-backed SaaS company, 2026. Source is private.

## What it is

An MCP server that puts the company's customer record in front of anyone who asks a question in natural language. Underneath it are three ETL pipelines feeding a Postgres and pgvector store: thousands of documents, hundreds of thousands of emails, and thousands of deal outcomes.

It fed three different appetites from one substrate. Product got voice-of-the-customer, revenue got win-loss, and the executive team got trend search.

## Architecture

Retrieval is multi-stage. Semantic, lexical and structured signals get fused through reciprocal rank fusion rather than picking a winner, on 3,072-dimension embeddings.

Getting there took a six-configuration A/B across 140 queries, spread over 14 categories and 3 personas. **Precision@5 went from 29% to 62%**, a gain of 32.8 points. The single largest contributor was model selection, which I would not have guessed going in and which is exactly why the A/B existed.

The extraction layer routes by archetype, runs seven signal detectors before any model call, and gates every field against its own evidence across 27 fields. **False positives fell from 49.8% to 3.7%** across six gated runs, against a hard 5% ceiling that the change had to clear before shipping.

## Auth, and why it is the part I would talk about

Cognito OAuth with PKCE, which became the pattern later internal MCP servers were built on, including the data warehouse server.

This is the layer I would want to be asked about, because it is where the interesting failures live. OAuth against an MCP client is not the web flow. Token scope, refresh behaviour under a long-lived agent session, and what a client does when it gets a 401 mid-conversation are all things you find out by getting them wrong first.

## Testing and adoption

744 tests.

Adopted by 45% of the company inside 30 days, every one of them active. The CEO was the heaviest user by a wide margin. I came ninth on my own tool, which I would rather report than hide, because a tool whose author is its main user has not been adopted.

## Operations

The eval suite gating every retrieval and embedding change was promoted to a weekly scheduled run with dead-man alarms on it, so a silent failure cannot report green. That pattern came out of a habit worth keeping: an alarm that only fires on a bad result cannot tell you the difference between a good result and no result at all.

A separate triage agent watched the infrastructure with a four-tier action model, from fix-and-merge at the bottom up to paging a human at the top, behind a kill switch that defaults to off. It was gated at 100% tier-classification accuracy across 25 cases before it was allowed to run, because the tier decision was the thing standing between an automatic change and a human being woken up.

On its first day it corrected a cold-start misconfiguration and silenced a notification topic that had been firing more than 40 false pages a night.
