---
inclusion: auto
name: tech
description: Hard technical rules (serverless, private buckets, no tracking) and the list of open decisions. Use for design, infrastructure, tasks or code work.
---

# Tech

## Hard rules

- No servers to patch. Managed and serverless services only.
- No public storage bucket. Buckets are private and read through the content delivery network with origin access control.
- Photo uploads go straight to storage with signed addresses. Files never pass through a function.
- Location data is removed from every published image. The original stays private.
- No tracking cookies. No third-party analytics. No third-party scripts on public pages.
- Uploads and removal requests are rate limited.
- Automated tests run on every change.
- No page is wider than the screen. A test measures every page at 390 px.
- On touch screens controls are at least 44 px high, fields use at least 16 px text, and nothing depends on hover.
- Keep every choice reversible. When a choice is made, the design says what it costs to change.

## High-level design

It lives in one place: [high-level-design.md](high-level-design.md). Do not repeat it here.

## Open decisions

Twenty points are open. They are listed in Open decisions in [requirements.md](requirements.md). Items 1 to 8 have no working answer: do not build on them, and keep related code behind a small interface. Items 9 to 20 carry a working assumption that the design and tasks use until the owner decides.

## Decisions (2026-11-06)

Full text in [requirements.md](requirements.md), section Decisions.

1. Sign-in: Cognito, behind a small interface.
2. Infrastructure tool: SAM. Stateful resources get fixed names and a retain policy so a later change of tool does not replace them.
3. Default photo licence: all rights reserved. A group may set another in its configuration.
4. Domain: the CDN address at first. Use relative addresses in pages.
5. Owner: a personal account at first.
6. Later modules: not in version 1.
