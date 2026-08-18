# Debugging notes

Selected failure signatures from [Sincta](./sincta.md). Each one is a defect I found, the mechanism behind it, and the rule I took away. There are 86 of these in [`wiki/learnings.md`](https://github.com/baldwinjames/sincta/blob/main/wiki/learnings.md) and these are the six that generalize furthest.

I keep this file because pattern recognition is the part of troubleshooting that does not transfer from reading. You get it from volume and from writing down what the last one actually turned out to be.

---

## A success code that could not see the failure

**Signature.** Fourteen dictations. Thirteen log lines reporting the paste succeeded. Zero words delivered anywhere the user could see.

**Mechanism.** On macOS, clicking any window of an app activates that app. The stop button lived on the app's own floating indicator, so clicking it made the app frontmost, and the paste keystroke went into the app itself rather than the user's editor. The result code said `ok: true` because the keystroke genuinely was written and posted. The operating system gives no signal about where a synthetic keystroke lands. The success code was honest and it was measuring the wrong half of the operation.

**Rule.** An honest result code is not a delivery check. Any step whose success cannot be observed needs a separate question asked about it. The discard button, which pastes nothing, worked perfectly throughout.

---

## Green through two reviews, never executed

**Signature.** A path-resolution function shipped, was approved through two review passes, was touched by several later tasks, and threw on its first execution in a packaged bundle. Boot-blocking.

**Mechanism.** In development the native addons live at one path. In a packaged bundle they land at another. The function had exactly one code path, and that path was correct for the only environment anybody had ever run it in. There was no packaged branch missing, because there was no branching at all.

**Rule.** Green code and reviewed code are not the same claim as exercised code. A function with no environment-specific branch is not evidence it does not need one. It may only mean nobody has run it in the other environment yet.

---

## The obvious fix for a defect that was not there

**Signature.** The menu bar icon rendered blank on the first packaged launch. The project already knew a defect class that fits exactly: image assets inside the app archive failing to load, the same shape as a native addon problem it had already hit.

**Mechanism.** There wasn't one. The image loader returned a valid non-empty image from the real archive path, and a hardware re-check showed the icon rendering and animating correctly. The glyph was drawn at 68 of 256 alpha and I could not see it. A legibility problem wearing a loading problem's clothes.

**Rule.** An obvious fix for a plausible defect is still a guess until you check the mechanism. Had the fix landed it would have been a no-op sitting on top of an unrelated problem it could never have addressed, and it would have looked like it worked.

---

## Two clean populations, both worthless

**Signature.** A latency tuning session produced two full datasets, one at the default setting and one at the candidate setting. Both passed. Both were plausible. Neither errored or warned.

**Mechanism.** The function constructing the native engine passed no arguments, so the component's own constructor default silently decided the setting regardless of the config file. Both populations measured the same pinned default. The candidate value never reached the component at all.

**Rule.** The instrument needs the same scrutiny as the code under test. The fix added a readback line that logs what the component was actually configured with, because the failure was invisible by construction and would have stayed invisible.

---

## A test that proves the wiring, not the behaviour

**Signature.** The end-to-end suite clicks buttons by dispatching DOM events, and passes.

**Mechanism.** Not a defect. A scope boundary that is easy to misread. Dispatching a DOM click proves the event listener, the inter-process round trip, and the resulting state change. It says nothing about whether the operating system would deliver a real click to that pixel, or about focus stealing, or multi-display placement.

**Rule.** Write the boundary down where the test lives, not in your head. That suite's README now carries an explicit scope-honesty clause naming what it does not assert, because a green run reads as "the buttons work" to everyone who did not write it.

---

## A guard that named four of five spellings

**Signature.** A safety check covering a set of cases looked comprehensive and covered most of them.

**Mechanism.** It enumerated four variants of a value. There were five. Naming four of five reads as covering all five to every subsequent reader, including the person who wrote it.

**Rule.** An enumeration is a claim about completeness. Either pin it with a test that fails when the set changes, or do not enumerate. From the same project: every count written in prose has been wrong at some point, and every count pinned by a test is right.

---

## What these have in common

Five of the six are a passing signal that measured the wrong thing. None of them announced itself. In every case the data looked fine, and the only reason any of them surfaced is that somebody went and checked the mechanism instead of accepting a result that agreed with them.

That is the habit I would bring to a support engineering seat. The customer's stated cause and a green dashboard are the same kind of evidence, and both deserve the same question: what would I expect to see if this were true, and have I actually looked at it.
