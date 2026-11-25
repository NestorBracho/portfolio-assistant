# 4. Answer in the language of the question

- **Status:** Proposed
- **Date:** 2026-11-25

## Context

nestorbracho.com has every page in English and Spanish, at its own address (`/en/…`, `/es/…`). The assistant has to
pick a language for each answer, and the brief leaves open whether that is the language of the question or the
language of the page ([section 7](../brief.md#7-decisiones-abiertas-resolver-antes-de-la-fase-1)). Languages other
than Spanish and English are out of scope.

The two can disagree. A recruiter may read the English page and ask in Spanish, or the other way round, and a
visitor who writes in one language expects to be answered in it.

## Decision

The assistant answers in the language of the question.

- The widget sends the language of the page with every message (`en` or `es`).
- The system prompt tells the model to answer in the language of the visitor's latest message. When that is
  unclear (a greeting, a single word, a name) or is neither Spanish nor English, it answers in the page's language.
- Fixed texts outside the model, like the off-topic answer or the message shown at the spending cap
  ([ADR 2](0002-a-monthly-spending-cap.md)), exist in both languages and follow the page's language.

## Consequences

- Visitors are answered in the language they chose, whatever page they are on.
- Detecting the language is left to the model, with no extra library or call. It can get mixed messages wrong, so
  the evaluations (phase 3) include questions in both languages, and some in a language the page isn't in.
- Much of the knowledge base is written in Spanish, so English answers are often a translation of what was
  retrieved. The citations still point to the original sections.
- The brand's spelling rules hold in both languages: "Nestor", without an accent.
