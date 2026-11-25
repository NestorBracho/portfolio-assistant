# 3. A single EC2 instance with Docker Compose

- **Status:** Accepted
- **Date:** 2026-11-25

## Context

The brief puts the assistant on AWS with Docker and leaves the service open: ECS, Elastic Beanstalk, or EC2 with
Docker Compose ([section 7](../brief.md#7-decisiones-abiertas-resolver-antes-de-la-fase-1)).

What has to run: the Django API, PostgreSQL with pgvector and Redis from phase 1 or 2, and, from phase 7, a
self-hosted Langfuse. Traffic is that of a personal site, a few conversations a day. The project is paid out of
pocket, and every phase has to end deployed ("in production from phase 1"), so the hosting must be quick to set up
and cheap to keep running.

## Decision

Run everything on a single EC2 instance, as services of one Docker Compose file: the API, PostgreSQL with pgvector
and Redis. GitHub Actions deploys by pulling the new images and restarting the services, after the evaluations pass.

The alternatives were set aside for now:

- **ECS (Fargate)** fits several containers well, but each service is billed on its own, and a managed database
  (RDS) and cache (ElastiCache) on top would cost several times what the assistant needs at this traffic.
- **Elastic Beanstalk** is what I have used most, but it suits a single application better than a set of services
  that includes a database and a cache.

## Consequences

- One machine, one bill, and a setup that runs the same locally: `docker compose up` brings up the whole stack.
- There is no high availability. If the instance stops, the assistant stops, and the widget must fail quietly and
  offer the contact form instead.
- Backups are my job, not a managed service's: a nightly `pg_dump` to S3, which also has to be tested by restoring
  it.
- The images, not the instance, are the deployable unit, so moving to ECS later changes the hosting, not the code.
- Self-hosted Langfuse needs ClickHouse, a cache and blob storage of its own, which is heavy for a small instance.
  Phase 7 decides between a larger instance, a second one, or Langfuse Cloud's free tier, and records it in an ADR
  of its own.
- To be confirmed before phase 1: whether the API sits behind Cloudflare, like the main site, on a subdomain of
  nestorbracho.com.
