---
name: grill-me
description: Interview the user relentlessly about a plan or design until reaching shared understanding, resolving each branch of the decision tree. Use when user wants to stress-test a plan, get grilled on their design, or mentions "grill me".
---

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time.

If a question can be answered by exploring the codebase, explore the codebase instead.

## How to ask

- **Show progress on every question.** Number each one against the total, e.g. "Question 3 of ~10". Plan the full list up front so the total is real. If the total changes because an answer opens or closes a branch, say so and give the new total. I always want to know how much questioning is left.
- **Keep each question under 300 words**, in plain terminology. Define a specialised term in one sentence the first time it appears, especially for infrastructure and cloud topics (Azure, Redis, networking). If I do not understand something, I will ask you to elaborate; do not pre-empt that with long explanations.
- **Be honest and constructive.** I am a front-end developer; many plans touch areas outside my expertise. If an idea is a different kind of thing than I think it is, say so plainly. Name the trade-offs, recommend one option, and explain why. Do not rubber-stamp.
- **Offer options as a short labelled list** (A, B, C) with my recommended option stated explicitly, then ask for agreement.
- **Track two running lists** and hand both over at the end: the decisions we reached, and the questions that must go to someone else (a stakeholder, an architect) because neither of us can answer them.

## Before the first question

Research first. Explore the codebase, related branches, wiki pages and any links I provide, and check the current state of any framework or vendor feature involved. Open with a short summary of what that research settles, so questions are only about what is genuinely open.
