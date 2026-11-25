# 1. A separate, public repository

- **Status:** Accepted
- **Date:** 2026-11-25

## Context

The assistant answers on the home page of nestorbracho.com, whose code lives in the `blogweb` repository. The brief
([section 7](../brief.md#7-decisiones-abiertas-resolver-antes-de-la-fase-1)) left open whether its backend should go
in that repository or in one of its own, and whether that repository should be public.

The two have little in common beyond the page the widget sits on. `blogweb` is a Django site on SQLite, deployed to
a single free-tier VM. The assistant needs PostgreSQL with pgvector, Redis, Docker, a CI that runs paid evaluations
on every pull request, and AWS. Its releases are also gated by different checks: a regression in the evaluations
should block an assistant deploy, never a blog post.

The second goal of the project is to be evidence of AI engineering for my CV, and evidence that can't be read
proves little.

## Decision

The assistant's backend gets a repository of its own, separate from `blogweb`, and the repository is public.

Being public sets rules for everything in it:

- No secrets. Settings come from the environment ([`.env.example`](../../.env.example)), the real `.env` is ignored,
  and gitleaks checks every commit before it is made.
- No client data. The knowledge base only holds what is already public on my CV and my site, anonymous clients stay
  anonymous in every file, and the content rules of the brief ([section 3](../brief.md#3-restricciones-de-contenido-obligatorias))
  apply to the repository as much as to the assistant's answers.

## Consequences

- Each repository keeps its own stack, CI and deployment, and neither can break the other's.
- Anyone can read the code, the evaluations and these records, which is the point.
- The chat widget still belongs in `blogweb`, so phase 1 changes both repositories, and the API between them (the
  chat endpoint, CORS, rate limits) has to be kept compatible on both sides.
- Some of the knowledge base starts as copies of documents in `blogweb` (`knowledge/cv-content.md`,
  `knowledge/home-cases.md`). They can drift apart, and they already have: this copy anonymizes a client that the
  one in `blogweb` names. Whatever the assistant reads comes from this repository.
- Anything committed by mistake is public for good, even if a later commit deletes it. The pre-commit hooks are the
  safeguard, so they must not be skipped.
