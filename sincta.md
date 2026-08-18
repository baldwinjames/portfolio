# Sincta

A privacy-first, fully on-device dictation app for macOS. Double-tap Fn to start talking, tap once to stop, and the transcript pastes into whatever app you were in. No audio and no transcript ever leaves the machine.

**Repository:** [github.com/baldwinjames/sincta](https://github.com/baldwinjames/sincta)
**Started:** 27 July 2026. Still going.
**Status:** pre-alpha, on its ninth phase gate.

## Why it exists

I wanted voice dictation and could not use a cloud tool. WisprFlow is the obvious commercial answer and it sends your speech to a server, which was the one thing I could not accept. So the constraint came first and the architecture followed from it: if no audio may leave the machine, every cloud speech-to-text vendor is out no matter how fast or accurate, and the whole design collapses onto Apple's on-device Speech frameworks.

That single constraint is the most useful thing about this project as an engineering exercise. It removed the easy path early, and everything downstream had to be earned.

## Shape of the system

Electron for the shell, because the menu bar presence, the onboarding flow and the settings surface iterate faster there than in SwiftUI.

Underneath it, two native Swift N-API addons doing the work Node cannot reach:

- **speech-bridge** wraps Apple's `DictationTranscriber` for on-device transcription
- **keys-bridge** handles global hotkey monitoring through Input Monitoring, and carries an Accessibility text-insertion path

Both bridges are load-bearing. If either fails to build or package correctly the app has no speech or no trigger, which turned out to matter more than I expected. See the packaging defect below.

## Decisions worth reading

The repo carries nine architecture decision records. Three are worth pulling out.

**Choosing the speech engine ([ADR 0001](https://github.com/baldwinjames/sincta/blob/main/wiki/decisions/0001-stt-engine-choice.md)).** I ported the speech code from an earlier personal project of mine, and before porting anything I audited what that project had actually done with each path rather than assuming it worked. Several were broken, unmeasured, or dead. One module, `DictationTranscriber`, was documented as the right tool for dictation and had never been wired up. Apple's newer `SpeechAnalyzer` path was gated behind an environment flag and marked disabled until stable. The newer module was not the better module, and the only way to know that was to go and look.

**Where the paste goes ([ADR 0003](https://github.com/baldwinjames/sincta/blob/main/wiki/decisions/0003-paste-target-capture.md)).** The transcript pastes into the app that was frontmost when the dictation started, not whatever is frontmost when the paste fires. Those were the same thing until the recording indicator became clickable, at which point clicking it made the indicator itself the frontmost app and it quietly ate the paste. The fix was to capture the target at the start.

**Accepting a latency number that missed its target ([ADR 0005](https://github.com/baldwinjames/sincta/blob/main/wiki/decisions/0005-stopflushms-360-acceptance.md)).** Covered below, because the way that session went is the point.

## What is measured

Nothing here is a claim I made up. Every number below came out of a bench script in the repo, and the raw JSON is committed alongside it.

| Gate | Result | Budget |
|---|---|---|
| End-to-end latency, median | **532ms** | 560ms |
| End-to-end latency, p95 | **541ms** | 700ms |
| Memory growth over three ten-minute sessions | **26MB** | 50MB |
| Memory slope | **−0.27 MB/min** | 1 MB/min |
| Word error rate, dictation mode | **18.3%** | no gate, baseline only |

Latency is measured from the stop-tap to the moment the paste keystroke is posted, across 50 real dictations. The benches refuse to run while a routable network interface is up, because a latency or accuracy number measured with a network path available proves nothing about an on-device claim. Overriding that refusal marks the result `network.overridden: true` in the JSON and disqualifies it from any gate.

## The tuning session that found a defect instead

This is the story I would tell if you only wanted one.

Phase 5 had a task to tune the flush window, which is the fixed wait between you stopping and the engine handing over its final text. The plan was a ladder: try 360ms, then 340, then 320, stop at the first value that passes every gate. I sat down to run it with a microphone.

The first sitting produced two clean populations of data. One at the shipped default of 400ms, one at a 360ms override. Both passed. Both looked fine.

Both were worthless. The function that constructs the native speech engine was calling it with no arguments, so the addon's own constructor default was silently deciding the flush window no matter what the config file said. Both populations had measured the same pinned 400ms default. The 360 number had never reached the bridge at all.

Nothing in the data looked wrong. The run at 360 did not report an error, did not warn, and produced latency figures in a plausible range. The only reason it surfaced is that I went looking for the mechanism rather than accepting a result that agreed with me.

The fix added the config wiring and a readback line that logs what the bridge was actually configured with, so the same failure cannot happen unseen again. Then I re-ran the ladder on the fixed build and got real numbers for the first time: median 532.3ms, p95 538.6ms, zero double pastes, twenty of twenty attested.

The original target had been a median strictly under 500ms. 532 misses it. I accepted the value anyway and stopped the ladder at its first rung, and wrote down why: every gate other than the strict median line came back clean, and pushing to 340 or 320 would have spent real headroom out of the revision-tolerance budget chasing a threshold that was a judgment call in the first place. The ADR records the target as revised rather than met.

I would rather ship a documented miss than a met target nobody can reproduce.

## A defect that had been green through two reviews

`resolveAddonDir()` is the function that finds the two native addons on disk. It shipped on 30 July, was read and approved through two separate review passes, and was touched by several later tasks. It had never once run inside a packaged application bundle.

The first time it did, it threw immediately and blocked boot. In development the addons live at one path, and in a packaged bundle they land at another. The function had exactly one code path, and that path happened to be correct for the only environment anyone had ever run it in. There was no packaged branch missing, because there was no branch at all.

The lesson I wrote down is that green code and reviewed code are not the same claim as exercised code. A function with no packaged-versus-development branch is not evidence that it does not need one. It may only mean nobody has run it packaged yet.

## A fix I dispatched and then held

The first attended launch of the packaged app showed the menu bar icon present and clickable and completely blank. This project already knew a defect class that fits that shape exactly: image files inside the app archive do not load for menu bar icons in this version of Electron, the same way the native addon files had needed unpacking. I dispatched a fix on that premise.

Before it landed, I checked the actual mechanism. The image loader returned a valid non-empty image from the real archive path, and a live re-check on hardware showed the icon rendering correctly and animating during a dictation. There was no load failure. The glyph was legible at 68 of 256 alpha, and I had not been able to see it.

The dispatched fix was held and converted into a documentation commit. Had it landed it would have been a no-op sitting on top of an unrelated legibility problem it could never have fixed, and it would have looked like it worked.

An obvious fix for a plausible defect is still a guess until you check the mechanism.

## The learnings file

There are 86 of these written up in [`wiki/learnings.md`](https://github.com/baldwinjames/sincta/blob/main/wiki/learnings.md). Each entry is what happened, the root cause, what I did, and the rule I took from it. A few of the recurring classes:

- A test that exercises the code without reaching the condition is the default outcome, not the exception
- A green test whose stated reason is false is worse than a missing test
- A fake that hardcodes the answer green-lights a seam no real object ever crossed
- A synthetic fixture that flatters the code under test is worse than no fixture
- A check that cannot tell success from the work not happening is not a check
- Every count written in prose in that file has been wrong at some point. Every count pinned by a test is right.

That last one is why the tables on this page cite committed JSON rather than my memory.

## Scale

978 commits over three weeks. Roughly 37,000 lines of application TypeScript, 7,400 lines of Swift across the two bridges, and 148 test files. Nine architecture decision records, 31 runbook entries, 30 phase and gate documents.

I wrote very little of that by hand. I direct the build, the model writes the code, and I review for correctness and can explain every design decision and why it was made. The gates, the benches, the attestation discipline and the learnings file all exist for the same reason: a model will hand you a passing result with real conviction, and you need instruments it cannot talk its way past.
