---
inclusion: auto
name: product
description: Product purpose and rules that never change for aws-ug-hub. Use when writing or reviewing specs, designs, tasks or any feature work.
---

# Product

aws-ug-hub is a serverless site for AWS User Groups. Each group installs its own copy on its own AWS account. Version 1 covers events, photos and an exportable event record.

Independent community project. Not affiliated with, endorsed by or sponsored by Amazon Web Services.

## Rules that never change

- Nothing about a specific group is written in the code. Everything specific to a group comes from the configuration file.
- Each group has its own installation and its own data. There is no shared service and no groups table.
- Photos are hidden until an organizer publishes them.
- Consent and removal are part of the design, not an add-on.
- No face recognition. No tagging of people unless a group turns it on explicitly.
- No tracking cookies. No third-party analytics.
- On a phone the site works like an app: tab bar at the bottom, a wall edge to edge, big touch targets, no sideways scroll (R22 to R24).
- "Tag yourself" is an open decision. It is not built until the owner decides.
- Text is plain, short and in English for docs. Site text is in the languages set in the configuration.
- Do not build on an open decision until the owner has answered it (see Open decisions in the requirements).

## Requirements

The full approved requirements, R1 to R24 and N1 to N8, live in `.kiro/steering/requirements.md`. That file is the single source. Change them there only, with the owner's approval. Specs refer to them by ID and do not repeat them.
