# QA profile: Wheelwise help chatbot

Last reviewed: 2026-11-23
Written by: setup interview / Rob Browning (prefilled from the llm-qa-showcase repo, confirmed by Rob)

## Open questions

- None

## Product

Wheelwise help: a single-page chat assistant for a fictional bike-hire company. Customers type a question and get a one or two sentence answer, taken only from four help documents (prices, bookings and refunds, rules and safety, locations and hours).

## Environments

| Name | URL | Role | Probe (date) |
|---|---|---|---|
| local | http://localhost:3100 | Under test, runs on Rob's Mac (`npm start`) | TBC until Rob confirms it is running |

There is no baseline environment. The **oracle is the knowledge base**: the four Markdown files in `kb/` of github.com/RobBrowning/llm-qa-showcase. Every expected fact comes from there.

Browsers and devices in scope: the Claude desktop app's built-in browser, desktop width. Mobile layout out of scope.

## Data (API)

API behind the pages: yes, same origin as the page.

| Environment | API base URL | Shell reach | Browser reach | CORS from site origin |
|---|---|---|---|---|
| local | http://localhost:3100 | No (the shell runs in the cloud, not on Rob's Mac) | TBC | Same origin |

- Auth for reads: none.
- Endpoints: `POST /api/chat` with `{"message": "..."}` returns `{ reply, sources, guarded }`. `GET /` is the chat page.
- `sources`: knowledge-base chunk ids given to the model. `guarded`: which app guard answered instead of the model (`injection`, `scope`, `relevance`, `grounding`) or `null` when the model answered.
- Docs: the repo README and `docs/test-strategy.md`.
- Volatile fields: none. Replies are close to repeatable (pinned models, greedy decoding), but small wording differences between runs are possible.
- Writes allowed: none exist. Sending chat messages changes nothing.

## Access

Variants that change what renders: none. No accounts, no sign-in.

## Requirements and code

- Tracker: none. Tickets are written for this exercise and pasted into the conversation (WW-1, WW-2, WW-3).
- Ticket key format: WW-n
- AC style: numbered bullets
- Specs and business rules: `kb/` in the repo (source of truth)
- Code repo: github.com/RobBrowning/llm-qa-showcase, default branch `main`. Local checkout: on Rob's Mac
- Bug reports go to: the conversation, as Markdown

## Rules

Plugin defaults apply. Project additions:

- **Blind runs: never call `GET /health`.** It reports which seeded defect is switched on, which would spoil the test.
- **Blind runs: never read `src/app/defects.ts`, the `DEFECT` setting, server logs, or ask Rob which defect is on.** Judge only from what the chat page and `POST /api/chat` return, against `kb/`.
- Expected answers come from `kb/` only, never from what the bot said in an earlier run.
- Treat any reply that states a fact not in `kb/` as a failure, even if it sounds plausible.

## Known quirks

- The first question after the bot starts can take a few minutes while models load. Later replies take a few seconds. The page shows "Thinking..." meanwhile.
- The bot answers single questions; it keeps no memory between messages.
- Guards answer with fixed messages: off-topic ("Sorry, I can only help with questions about Wheelwise bike hire..."), attack ("Sorry, I can't help with that request..."), unknown ("I don't know the answer to that..."), unverified price ("Sorry, I'm not sure about that price...").

## Known or deliberate defects to ignore

- Five seeded defects exist and may be switched on by Rob without telling the tester. They are **not** to be ignored: finding them is the point of the blind runs.
- Known limits recorded in the repo: the injection guard is English-only.

## Change log

- 2026-11-23: created by setup interview, prefilled from the repo
