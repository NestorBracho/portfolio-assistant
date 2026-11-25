# 2. A monthly spending cap of 10 EUR

- **Status:** Accepted
- **Date:** 2026-11-25

## Context

The assistant is open to anyone who visits the home page, and every answer costs money in Claude API calls. A burst
of traffic, a bot or a loop in the code could run up a bill that a personal project shouldn't have. The brief makes
controlled cost one of its principles: there is a monthly cap, and the assistant degrades gracefully when it reaches
it.

## Decision

Spending on the Claude API is capped at **10 EUR a month**, set in `ANTHROPIC_MONTHLY_BUDGET_EUR`.

- The assistant measures its own spending. From phase 1 every message stores its tokens and an estimated cost, so
  the month's total is a query, not a guess.
- When the month's spending reaches the cap, the assistant stops calling the stronger model and answers with the
  cheaper one. If spending keeps growing, it stops calling the API and answers with a static message that points to
  the contact form. The exact thresholds are part of phase 6, with the router.
- The Anthropic Console gets a spending limit of its own, slightly above the cap, as a safety net in case the
  assistant's own count is wrong.

## Consequences

- The worst case is known in advance: a bad month costs about 10 EUR.
- Near the end of a busy month, visitors may get shorter answers or only the contact form. That is the intended
  trade-off: a lead can still reach me even when the assistant can't answer.
- The API bills in US dollars and the cap is in euros, so the estimate converts with a fixed rate. It should err on
  the high side.
- The estimate depends on the price of each model, which is configuration, not code. It has to be updated when
  prices or models change, or the cap will be checked against the wrong numbers.
- Evaluation runs in CI also call the API. Their cost is measured separately (phase 3), so that running the tests
  doesn't use up the visitors' budget.
