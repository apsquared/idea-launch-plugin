---
name: setting-up-an-idea-test
description: Set up an Idea Launch test well — write a brief specific enough to target against, generate ad creative, and check every gate before the user spends an Ad Run. Use when the user wants to validate a product idea, test whether something is worth building, get ads running for a product they already shipped, or asks what is blocking their Idea Launch test from starting.
license: MIT
metadata:
  author: idea-launch
  version: "1.1"
---

# Setting up an Idea Launch test

Your job is to get the test to a state worth paying for, then hand it back. The
quality of the brief determines the targeting and the ad copy, so a vague brief
wastes a real Ad Run.

## You cannot launch, and should not imply otherwise

Starting or boosting an Ad Run spends one of the user's Ad Runs and is confirmed
by them in the web app. There is no tool for it. Finish with
`idealaunch_prepare_ad_run` and give the user the `launchUrl` (the idea's page
in the Idea Launch app, where they review the ad and start the test). Never say
you have started a test, and never promise results by a date. If the user asks
you to launch, say plainly that you cannot and hand over the link.

## Start from what already exists

Call `idealaunch_list_ideas` before creating anything; the user may already have
the idea. `idealaunch_get_idea` returns a readiness checklist for free (no AI
turn, no credit), and `idealaunch_get_account` says how many Ad Runs are
available. Accounts are capped at 50 ideas, so do not create throwaway ones.

## Which of the two paths applies

Ask if it is not obvious:

- **Validating an idea** — nothing shipped yet, or a new angle on something that
  is. Traffic goes to a generated sign-up page, and signups are the signal. Set
  `conversionMode: "waitlist"` with `idealaunch_update_idea_brief`; it starts
  empty, and an empty mode fails the destination gate.
- **Ads for something already live** — pass `websiteUrl` on
  `idealaunch_create_idea`, then set `conversionMode: "external_url"` and
  `externalDestinationUrl` with `idealaunch_update_idea_brief`. `websiteUrl`
  alone records the site; it does not route traffic there. Traffic then goes to
  their own site, and signups only count if their signup form reports back to
  Idea Launch — the app shows them how. Tell the user that, because zero
  signups from an unconnected form means "not measured", not "no demand".

The mechanics are identical. Only the destination and what the readout
emphasises differ.

## Write a brief worth targeting against

`idealaunch_draft_brief` returns suggestions and saves nothing. **Show them to
the user and get corrections before saving with `idealaunch_update_idea_brief`.**
The draft is a starting point; the user knows their customer and you do not.

Push for specificity on the four required fields:

| Field | Too vague | Specific enough |
|---|---|---|
| `targetCustomer` | "Founders" | "Solo technical founders who have shipped a product but have no distribution" |
| `whyTheydCare` | "It saves time" | "They lose a day a week to manual scheduling and have already tried three tools" |
| `currentAlternative` | "Nothing" | "A shared spreadsheet plus WhatsApp reminders" |
| `primaryMarket` | — | One of `US`, `CA`, `GB`, `AU`, `GL` |

"Nothing" is almost never the real alternative. If the user says it, ask what
the customer does today instead — that answer is the actual competitor and it
sharpens the ad.

## Then the creative

`idealaunch_generate_ad_assets` writes the tagline, ad copy, and an image prompt.
It defaults to not saving; review with the user, then call again with
`save: true`. It does **not** write the sign-up page headline, description, or
CTA — set `signupPageHeadline`, `signupPageDescription` and
`signupPageCtaLabel` with `idealaunch_update_idea_brief`, since they are what the
conversion-destination gate checks on the sign-up page path.
`idealaunch_preview_landing_page` shows where ad traffic will land and the copy
on that page; use it before handing over.

Every test needs a hero image. Where the server offers
`idealaunch_generate_hero_image` (Codex, ChatGPT and most other clients), it runs
in the background: poll `idealaunch_get_image_job` until it succeeds, show the
variants, then apply the user's pick with `idealaunch_apply_hero_image`. The
Claude connector has no image tools, so in Claude tell the user to add the hero
image in the Idea Launch web app, where they can upload their own or generate
one. Don't claim to have added an image you could not add.

Each generation step consumes one of the idea's AI turns, and the budget is per
idea, not per session. Do not regenerate repeatedly hoping for better output —
edit the wording directly instead.

## Check before they spend

`idealaunch_prepare_ad_run` creates nothing and charges nothing. It returns every
gate with a specific fix, and — when the user has an Ad Run credit available —
the platform that would be chosen, why, and the inferred audience. With no
credit it still returns the readiness checks; the preview is what is withheld.

Report the blockers as a short list of what is left, then the launch link.

## When to say it is not ready

Say so plainly rather than proceeding:

- The brief is still generic. Targeting built on "founders who want to save
  time" will underperform and the user will blame the product.
- The user wants to test five variations at once. One clean read first; the
  readout will recommend the next variation.
- The idea is far outside SaaS and app products, which is what the platform is
  tuned for. Say it may not be well served rather than taking the money.
- The user expects the test to prove the business works. Reset that: a small
  test measures how one audience responds to one message and offer. It can be
  inconclusive, and even a strong result is evidence about that positioning,
  not proof the business will succeed.

## Language

An Ad Run produces evidence about a positioning, not proof about a business.
Prefer: early signal, market read, worth another test. Avoid: validated, proven,
guaranteed.

Never discuss what a run costs to operate, ad spend, cost per click, or cost
per signup, and do not estimate them from other numbers. The Ad Run price
itself is public and fine to mention.

`idealaunch_list_signups` withholds email addresses unless `includeEmails` is
set. Only set it when the user asks for the addresses.
