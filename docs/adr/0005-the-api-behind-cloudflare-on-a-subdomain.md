# 5. The API behind Cloudflare, on a subdomain

- **Status:** Accepted
- **Date:** 2026-11-25

## Context

The API runs on a single EC2 instance ([ADR 3](0003-ec2-with-docker-compose.md)) and is called by the chat widget
on the pages of nestorbracho.com. It is open to anyone, and every answer costs money ([ADR 2](0002-a-monthly-spending-cap.md)),
so it is an obvious target for scripts and floods. It needs HTTPS, and a way to rate-limit visitors by their real
IP address.

The main site already sits behind Cloudflare's free plan, in front of an origin that only accepts Cloudflare's
traffic. That setup has worked well, and the domain is already managed there.

## Decision

Serve the API at **`api.nestorbracho.com`**, as a proxied record in the same Cloudflare zone as the main site, with
the same setup:

- SSL/TLS set to **Full (strict)**. Visitors connect to Cloudflare, and Cloudflare reaches the instance over HTTPS
  with a Cloudflare origin certificate, issued from a CSR generated on the instance so its private key never
  leaves it. A reverse proxy in the Compose file (nginx, as on the main site) terminates that connection and
  passes requests to the API.
- The instance's security group only accepts HTTPS from Cloudflare's published IP ranges, so the API can't be
  reached directly even if its address leaks. How the deploy reaches the instance is decided with the deploy
  workflow in phase 1.
- The rate limits count requests by the `CF-Connecting-IP` header, which Cloudflare sets on every request and
  visitors can't forge past it, the same way the main site does. `X-Forwarded-For` is never trusted.
- API responses are never cached by Cloudflare: they are personal answers, sent with `Cache-Control: no-store`.
- The pages call the API from another origin, so the API sends CORS headers that only allow the site's own origins
  (`CORS_ALLOWED_ORIGINS`).

## Consequences

- TLS, basic DDoS protection and a hidden origin come at no cost, and work the same way as on the main site, so
  there is one way of doing things to maintain.
- The API has an address of its own, separate from the site's pages, which keeps the two deployments independent
  ([ADR 1](0001-a-separate-public-repository.md)).
- Cloudflare's free plan adds more protection if it is ever needed, such as rate-limiting rules at the edge, before
  requests reach the instance and cost anything.
- Cloudflare drops a request when the origin takes about 100 seconds to start answering. Answers must start well
  before that, which streaming them does anyway.
- Every request crosses another hop, which adds a little latency. It is measured with the rest in phase 1.
- Only Cloudflare trusts the origin certificate, and the security group has to follow Cloudflare's IP ranges, like
  the main site's firewall.
