# Evaluation practice

How I decide whether a change is allowed to ship, and the four times my own measurement told me no.

## The rule

A gate that cannot go red is not a gate.

Everything below follows from that. It sounds obvious written down and it is violated constantly, usually not by anyone cutting corners but by a suite that was never capable of failing in the first place.

## Why the LLM judge got banned

I was gating retrieval quality with a model as judge, which is the default answer and it is a good answer for things with no ground truth.

Then I ran the same inputs twice and the score moved twelve points.

Not twelve points between two versions of the system. Twelve points on identical inputs, run to run. Every conclusion I had drawn from that gate was inside its own noise floor. I banned model-judged quality gates by written decision record and rebuilt on a deterministic suite: 330 queries and 101 hand-labelled records, with the ground truth hash-frozen so a change to the answer key cannot pass unnoticed as a change to the system.

The suite is slower to build and it can go red for a reason you can point at.

## The four changes I killed

Each of these was a change I expected to work, built, measured, and threw away.

**Hybrid search that reported a 28.8-point gain.** Full measurement put it two points net negative. This is the one I tell people about, because the initial number was not fabricated and was not a bug. It was real, on a slice that did not represent the query distribution. A number can be honest and still be wrong about the thing you are asking it.

**Multi-vector retrieval, 74% worse.** Fast to kill, at least.

**Time decay weighting.** The intuition is strong. Recent documents matter more. The measurement disagreed.

**A cheaper reranker model, 8.2 points down.** Cost saving that was not.

Four changes, all of which I believed in enough to build. That ratio is roughly what I expect now, and it is the argument for the suite rather than an argument against my judgment.

## Holding a change back for five months

I built a redesign I believed in and it did not clear the eval gate. I held it for five months rather than override the gate I had written, and shipped it when it met the bar honestly.

I would rather have that on the record than a faster ship date. The whole apparatus is worthless the first time its author waves it through.

## The instrument needs the same scrutiny as the code

The failure I keep meeting is not a bad result. It is a good result from a broken instrument, and it never announces itself.

A tuning session on a personal project ran two clean populations of latency data before I noticed the configuration had never reached the component being configured. Both populations had measured the same pinned default. Nothing errored. Nothing warned. The numbers were plausible. ([Full account here.](./sincta.md#the-tuning-session-that-found-a-defect-instead))

From the same project, written down after meeting each one:

- A test that exercises the code without reaching the condition is the default outcome
- A green test whose stated reason is false is worse than a missing test
- A fake that hardcodes the answer green-lights a seam no real object ever crossed
- A synthetic fixture that flatters the code under test is worse than no fixture
- A check that cannot tell success from the work not happening is not a check
- A negative assertion fails silently when the value under test moves

Six ways to hold a passing test that proves nothing. I have shipped every one of them.

## Teaching it

I built a 16-unit hands-on curriculum out of this, because the reflex is the transferable part rather than any particular suite. The thing worth teaching is not how to write an eval. It is how to catch one that is subtly wrong, which is a harder skill and the one that everything else depends on.
