# Sincta

A privacy-first, fully on-device dictation app for macOS. Double-tap Fn to start talking, tap once to stop, and the transcript pastes into whatever app you were in. No audio and no transcript ever leaves the machine.

**Source:** private repository. Happy to walk through it.
**Started:** 27 July 2026. Still going.

## Why it exists

I wanted voice dictation and could not use a cloud tool. The obvious commercial answer sends your speech to a server, which was the one thing I could not accept. So the constraint came first and the architecture followed from it: if no audio may leave the machine, every cloud speech-to-text vendor is out no matter how fast or accurate, and the whole design collapses onto Apple's on-device Speech frameworks.

That single constraint is the most useful thing about this project as an engineering exercise. It removed the easy path on day one, and everything downstream had to be earned.

## What it does

It replaces a commercial cloud tool with one that has no network path at all. Speech capture, transcription and delivery all happen locally, and it is in daily use.

Electron for the shell, because the menu bar presence, onboarding and settings iterate faster there than in SwiftUI. Underneath it, two native Swift N-API addons reaching the frameworks Node cannot:

- **speech-bridge** wraps Apple's `DictationTranscriber` for on-device transcription
- **keys-bridge** handles global hotkey monitoring through Input Monitoring, plus an Accessibility text-insertion path

Both bridges are load-bearing. If either fails to build or package correctly, the app has no speech or no trigger.

## What is measured

Every number below came out of a bench script, with the raw JSON committed alongside the code that produced it.

| Gate | Result | Budget |
|---|---|---|
| End-to-end latency, median | **532ms** | 560ms |
| End-to-end latency, p95 | **541ms** | 700ms |
| Memory growth over three ten-minute sessions | **26MB** | 50MB |
| Memory slope | **−0.27 MB/min** | 1 MB/min |
| Word error rate, dictation mode | **18.3%** | no gate, baseline only |

Latency is measured from the stop-tap to the moment the paste keystroke is posted, across 50 real dictations.

The benches refuse to run while a routable network interface is up. A latency or accuracy number measured with a network path available proves nothing about an on-device claim, so overriding that refusal marks the result `network.overridden: true` in the JSON and disqualifies it from any gate.

## Decisions worth reading

The project carries nine architecture decision records. Three are worth pulling out.

**Choosing the speech engine.** I ported the speech code from an earlier personal project, and before porting anything I audited what that project had actually done with each path rather than assuming it worked. Several were broken, unmeasured, or dead. One module, `DictationTranscriber`, was documented as the right tool for dictation and had never been wired up. Apple's newer `SpeechAnalyzer` path was gated behind an environment flag and marked disabled until stable. The newer module was not the better module, and the only way to know was to go and look.

**Where the paste goes.** The transcript pastes into the app that was frontmost when dictation started, not whatever is frontmost when the paste fires. Those were the same thing until the recording indicator became clickable, at which point clicking it made the indicator the frontmost app and it quietly ate the paste. The fix was to capture the target at the start.

**Accepting a latency number that missed its target.** The original target was a median strictly under 500ms. 532 misses it. I accepted the value and stopped the tuning ladder at its first rung, and wrote down why: every gate other than the strict median line came back clean, and pushing lower would have spent real headroom out of the revision-tolerance budget chasing a threshold that was a judgment call in the first place. The ADR records the target as revised rather than met.

I would rather ship a documented miss than a met target nobody can reproduce.

## How the work gets done

Every phase sits behind a written gate with attested evidence, and every defect I find gets written up with its mechanism and a rule taken from it. There are 86 so far.

That log is not a bug list. The interesting part is almost never the defect. It is why the thing that was supposed to catch it did not. Four examples of what that turns up:

**A tuning session that measured nothing.** The task was to find the lowest safe flush window. The first sitting produced two clean populations of latency data, one at the default and one at the candidate value. Both passed. Both were worthless: the function constructing the native engine passed no arguments, so the addon's own constructor default decided the setting regardless of the config file. Both populations had measured the same pinned default. Nothing errored, nothing warned, and the numbers were plausible. The fix added a readback line logging what the bridge was actually configured with, because the failure was invisible by construction.

**A path resolver green through two reviews.** `resolveAddonDir()` shipped, was approved twice, and was touched by several later tasks without ever running inside a packaged bundle. Its first packaged execution threw immediately and blocked boot. In development the addons live at one path; packaged, they land at another. The function had exactly one code path and it happened to be right for the only environment anyone had run it in. Green code and reviewed code are not the same claim as exercised code.

**A fix I dispatched and then held.** The menu bar icon rendered blank on the first packaged launch, matching a defect class this project already knew: assets inside the app archive failing to load. I dispatched a fix on that premise. Before it landed I checked the mechanism, and the image loader returned a valid non-empty image from the real archive path. The glyph was drawn at 68 of 256 alpha and I simply could not see it. Had the fix landed it would have been a no-op sitting on top of an unrelated legibility problem, and it would have looked like it worked.

**A success code that could not see the failure.** Fourteen dictations, thirteen log lines reporting the paste succeeded, zero words delivered anywhere the user could see. Clicking the indicator's stop button activated the app, so the keystroke went into the app rather than the editor. The result code said `ok: true` because the keystroke genuinely was written and posted, and macOS gives no signal about where a synthetic keystroke lands. An honest result code is not a delivery check.

## Scale

978 commits over three weeks. Roughly 37,000 lines of application TypeScript, 7,400 lines of Swift across the two bridges, and 148 test files. Nine architecture decision records, 31 runbook entries, 30 phase and gate documents.

I wrote very little of that by hand. I architect and direct, the model writes the code, and I review for correctness and can explain every design decision. The gates, the benches, the network refusal and the learnings file all exist for the same reason: a model will hand you a passing result with real conviction, and you need instruments it cannot talk its way past.
