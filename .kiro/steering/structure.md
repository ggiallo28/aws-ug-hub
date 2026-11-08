---
inclusion: auto
name: structure
description: Where files live in this repo and how steering and specs split the work. Use when creating or moving files or specs.
---

# Structure

## Where things live

- `.kiro/steering/`: rules (`product.md`, `tech.md`, `structure.md`), the high-level design (`high-level-design.md`, one place only) and all requirements (`requirements.md`). Included when needed, except `spec-rules.md`, which is always included.
- `.kiro/specs/aws-ug-hub/`: `requirements.md` (a reference list by ID, no repeated text), `design.md` (detail only, refers to `high-level-design.md`) and `tasks.md`. These add detail. They do not repeat the steering.
- `.kiro/steering/requirements.md`: the full approved requirements R1 to R24 and N1 to N8. Single source. Do not rewrite it without the owner. If a gap or contradiction shows up, list it for the owner.
- `config/group.example.yml`: example configuration. The real file per group is not committed to the shared code.
- `docs/`: guides and the draft issue list.
- `docs/wall-decisions.md`: the owner's decisions D1 to D21 on the wall, already turned into requirements R17 to R21 and N8.
- `docs/preview/`: a static prototype of the site (public pages and organizer area) with sample content, built by `docs/preview/build.py`. It is the design reference for behaviour and looks. It is not production code. Change `build.py` and the CSS and scripts, not the generated HTML. Run `python3 build.py` after a change.

## Rule: steering holds the whole, specs hold the detail

- Steering holds the high-level design and all requirements. Load them for any spec work.
- Specs go deeper on one area. Each spec starts by naming the requirements it covers, by number (R1 to R24, N1 to N8).
- Do not copy requirement text into a spec. Refer to it by ID and point to `.kiro/steering/requirements.md`.
- Each task names the requirements it satisfies and how to check it is done.
- The high-level design is only in `.kiro/steering/high-level-design.md`. The spec `design.md` links to it and does not repeat it.
- If a spec changes the overall design, update `high-level-design.md` in the same change.

## Working rules

- Small tasks, smallest end-to-end slice first: configuration, one event page, then photos.
- Do not write application code before the design and tasks are approved.
- No group name, colour, language, URL or text in code. Read it from the configuration.
