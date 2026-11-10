---
name: reading-a-market-read
description: Interpret an Idea Launch market read correctly — the validation score, band, confidence, and recommended next move — and report it without overstating what a single ad test proves. Use when the user asks how their Idea Launch test went, what a validation score or band means, whether a result is good, whether to build or keep testing, or why a run is not live yet.
license: MIT
metadata:
  author: idea-launch
  version: "1.1"
---

# Reading an Idea Launch market read

An Ad Run measures whether strangers respond to **a specific positioning** — one
audience, one message, one offer, and the landing experience together. It does
not measure whether the business will work, and a small test can be
inconclusive. Every reading below inherits that limit.

Find the run with `idealaunch_list_ideas` and `idealaunch_list_ad_runs`, then
fetch the readout with `idealaunch_get_market_read`. Pass
`response_format: "json"` when you need the structured fields below: the
default markdown shows the score and the written `recommendation`, but not
`recommendedEvidenceAction` or the lineage readout. Do not compute your own
score from the metrics; the score is benchmarked against other runs and you
cannot reproduce that locally.

## Read the score and the confidence together

The score alone is not the finding. A score with low confidence is a shrug, not
a verdict.

`validationScore` and `validationBand`:

| Score | Band | What it supports |
|---|---|---|
| 80+ | `strong` | Real pull. The strongest case for building. |
| 60–79 | `promising` | Worth more evidence. Boost or run a variation. |
| 40–59 | `mixed` | The angle is doing something, but not clearly. Change one variable. |
| 20–39 | `weak` | Little pull for this positioning. Rethink the angle, not the product. |
| < 20 | `little_evidence` | Not enough happened to conclude anything. |

Confidence gates all of it. The field is `confidenceScore`, a number; the
labels below are how to describe it (the app uses the same cut-offs):

| `confidenceScore` | Say | Meaning |
|---|---|---|
| 70+ | high | The band can carry a decision |
| 40–69 | medium | Directional; prefer more evidence before committing |
| < 40 | low | Report the band as a hint, not a result |

`isProvisional` is true whenever `confidenceScore` is below 70. **When it is true, say
so in your first sentence.** A provisional 72 is not "promising, go build" — it
is "leaning positive, not settled".

## Check what the read is based on

`readoutBasis` changes what you may claim:

- **`conversion_backed`** — signups were tracked. You can talk about whether
  people wanted the thing.
- **`traffic_backed`** — no conversion tracking, so the read rests on clicks and
  landing-page traffic. You may talk about whether the *message* earned
  attention. You may **not** conclude anything about demand for the product.
  If the user wants a demand read, the fix is to set up signup tracking, not to
  run the test again.

A test that sends traffic to the user's own site is read as
`conversion_backed` unless they told Idea Launch their form will not report
signups. If their form was never connected, zero signups means "not measured".
Check before reporting it as no demand, and point them to the signup tracking
setup in the app.

## Lead with the move, not the numbers

`recommendedEvidenceAction` is the product's own recommendation. Lead with it.
A single run (`get_market_read`) uses `boost_run`, `start_angle_variation`,
`wait_for_more_data` or `none`. The idea-level and lineage readouts
(`list_ideas`, `lineage.overallReadout`) use `boost_best_run`,
`start_angle_variation`, `wait_for_active_runs` or `none`, plus a
`recommendedRunId`. `start_market_variation` exists in the schema but is not
currently produced.

| Action | Say |
|---|---|
| `boost_run` / `boost_best_run` | Put more behind the run that is already working. |
| `start_angle_variation` | Same audience, different message. The offer may be fine and the framing wrong. |
| `start_market_variation` | Same message, different market. |
| `wait_for_more_data` / `wait_for_active_runs` | Do not act yet. Say plainly that acting now would be reading noise. |
| `none` | No clear next step. Explain what is missing rather than inventing one. |

`signalDrivers` explains *why* the score landed where it did. Quote the drivers
that moved it rather than listing all of them.

## A run that is not live yet

If a run has no readout, check `idealaunch_get_run_events` before speculating.
A run with status `pending_review` is waiting on an Idea Launch content
review — nothing has been charged and nothing has reached the ad platform. (A
separate `providerReviewState` of `pending_review` means the ad platform's own
review, after launch.) Say that
plainly; it is not an error and not something the user needs to fix.

## Language

Use: early signal, market read, evidence of interest, worth another test, a
stronger case for building.

Avoid: validated, proven, confirmed, guaranteed, this will work, the idea is
good. One test on one positioning cannot support any of those.

Never report or estimate what the run cost, a cost per click, or a cost per
signup. Those numbers are deliberately absent from the readout, and inferring
them from anything else defeats the point.

## Common misreads

- **Treating a low-confidence score as a decision.** Check `isProvisional`
  first, every time.
- **Concluding demand from a `traffic_backed` read.** Clicks are attention, not
  intent.
- **Calling a weak band a dead product.** It is evidence about one positioning.
  The usual next step is a different angle, not abandoning the idea.
- **Averaging runs by hand.** Use `lineage.overallReadout` from
  `idealaunch_get_ad_run` (with `response_format: "json"`); runs are weighted
  by confidence, not averaged.
- **Recommending a launch yourself.** You cannot start a run. Report readiness
  and hand over the link.
