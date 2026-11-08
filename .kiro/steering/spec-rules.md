---
inclusion: always
---

# Spec rules

These rules apply to every spec in this project, now and later. This file is always included. [structure.md](structure.md) says where files live. This file says how to write a spec.

## Single sources

Only this file is always included. The others load when needed. Before any spec work (requirements, design, tasks or code), load `requirements`, `high-level-design`, `product` and `tech`, and read the spec folder.

- Requirements: [requirements.md](requirements.md). The only place requirement text exists.
- High-level design: [high-level-design.md](high-level-design.md). The only place the overall design exists.
- Rules: [product.md](product.md), [tech.md](tech.md), [structure.md](structure.md).
- A spec never copies from these. It links to them and adds detail.

## New requirements

- A new feature that needs new requirements adds them to [requirements.md](requirements.md) first, in the same style, with the next free ID (R25 and N9 are next).
- Never renumber or reorder existing IDs. Specs, design and tasks cite them.
- Never rewrite approved requirements. If one is unclear or wrong, list it for the owner and wait.
- Changes to the requirements need the owner's approval before a spec builds on them.

## Spec folder

One folder per feature in `.kiro/specs/<feature-name>/` with `requirements.md`, `design.md` and `tasks.md`. Use kebab-case names.

### requirements.md in a spec

- Starts with an introduction that links to the steering requirements.
- Lists only the requirements the feature covers, by ID and title, with an area line.
- Each user story is one link to the requirement in the steering file.
- One acceptance criterion per steering criterion, same order and same count. Each is a reference that cites the criterion ID (R10.4, N4.1), keeps the same EARS keyword (WHEN, WHILE, IF/THEN, WHERE) and the same subject, and links to the steering requirement. No requirement text is copied.
- Headings need a running number for the format check. Keep the draft ID in the heading and cite that ID.

### design.md in a spec

- Starts with an overview that links to the high-level design.
- One short section per area, each with a link to the matching steering section and the criterion IDs it covers.
- Adds detail only. Never repeats a diagram, table or paragraph from the high-level design.
- If the feature changes the overall design, update [high-level-design.md](high-level-design.md) in the same change.
- Correctness properties cite the requirement IDs they check. They add no new requirements.
- States what each choice would cost to change. Keeps every choice reversible.

### tasks.md in a spec

- Small tasks in a sensible order. Smallest slice that works end to end first.
- Each task names the requirement or criterion IDs it satisfies and a way to check it is done: a test, a command, or a visible result.
- A task changes one thing. If it needs more than a day of work, split it.
- No task builds on an open decision that has no answer. Use a small interface and a note instead.
- Tests come with the task they test, not at the end.

## Order and review

1. Requirements first. Stop for review.
2. Design second. Stop for review.
3. Tasks third. Stop for review.
4. No application code until the tasks are approved.
5. After each document, list gaps, contradictions and ambiguities at the end of the reply. Do not fix them in approved documents.

## Open decisions

- The owner decides open decisions. Give two options and a recommendation with the reason and the cost to change. Do not decide them. Do not build on one without an answer.
- When one is answered, record the answer in [requirements.md](requirements.md) with the date, and update [high-level-design.md](high-level-design.md).

## Non-negotiable constraints

Every spec must respect these. If a spec would break one, stop and ask.

- Nothing about a specific group is in the code. It comes from the configuration file.
- No servers to patch. No public storage bucket.
- No tracking cookies. No third-party analytics. No face recognition.
- Each group has its own installation. No shared service.

## Writing

- English. Short sentences. Plain words. No marketing language.
- Link to files with markdown links. Do not only name them.
- Use "R10.4" style IDs, not "the removal requirement".

## The preview

- `docs/preview/` shows agreed behaviour. A behaviour the owner confirms there is written into [requirements.md](requirements.md) and the design before it becomes a task.
- When the preview and the requirements differ, the requirements win. List the difference as a gap. Do not change the requirements to match the preview without the owner.
- Tasks that build a page name the preview page they follow.
