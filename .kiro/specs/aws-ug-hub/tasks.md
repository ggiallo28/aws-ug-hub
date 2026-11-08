# Implementation Plan: aws-ug-hub

## Overview

This plan builds aws-ug-hub in small slices. Order: skeleton, configuration, one event page, first deploy, data and sign-in, photos, removal, the wall, organizer pages, phone experience, hardening and guide.

Requirements are in the [steering requirements](../../steering/requirements.md). The design is in the [high-level design](../../steering/high-level-design.md) and in [design.md](design.md). This file repeats neither.

How to read the IDs. The spec format uses a running number for each requirement. The draft ID is in brackets. Cite the bracketed ID in commits and reviews. R1 to R16 are Requirements 1 to 16. N1 to N7 are Requirements 17 to 23. R17 to R24 are Requirements 24 to 31. N8 is Requirement 32. A criterion such as R10.4 is written 10.4 (R10.4).

Working assumption, open decision 15 (the owner has not decided it): language, tools, CI and image library.

- Language and tools: TypeScript on Node for functions, the site build and tests. Vitest and fast-check for tests. Playwright for page tests. Cost to change: functions, build and tests are rewritten. Low now, high after task 7.
- CI service: GitHub Actions. Cost to change: one workflow file.
- Image library: sharp in a container image for the processing function. Cost to change: one module and the image build.

There are twenty open decisions (see [Open decisions](../../steering/requirements.md#open-decisions)). A task marked "placeholder until open decision N" (items 1 to 8) uses a small interface and a placeholder value. It does not decide N. "Assumption" lines in tasks refer to items 9 to 20, which carry a working assumption until the owner decides. The Notes section lists them all.

No task here is application code yet. No code is written until the owner approves this plan.

## Tasks

- [ ] 1. Project skeleton and checks
  - [ ] 1.1 Create the repository layout and tooling
    - Create `src/` (config, site, api, functions, shared), `templates/`, `locales/`, `scripts/`, `tests/`, `infra/`.
    - Add `package.json` with exact versions, strict `tsconfig.json`, ESLint, Vitest and fast-check.
    - Add scripts `lint`, `test`, `build`.
    - Working assumption, open decision 15: tools as listed in the Overview.
    - Done when: `npm run lint && npm test` passes on one smoke test.
    - _Requirements: 21.1 (N5.1)_

  - [ ] 1.2 Add CI that runs lint and tests on every change
    - Working assumption, open decision 15: GitHub Actions.
    - Add `.github/workflows/ci.yml` running `npm ci`, `npm run lint`, `npm test` on push and on pull request.
    - Done when: the workflow runs green on a pull request.
    - _Requirements: 21.1 (N5.1)_

  - [ ] 1.3 Add repository checks for `.kiro/` and the MIT licence
    - Add `tests/repo.test.ts`. It asserts `.kiro/steering/`, `.kiro/specs/aws-ug-hub/` and the three spec files exist, and `LICENSE` is the MIT licence.
    - Done when: `npm test` runs the test and it passes. Deleting `LICENSE` makes it fail.
    - _Requirements: 21.2 (N5.2), 23.1 (N7.1)_

- [ ] 2. Configuration slice
  - [ ] 2.1 Build the configuration schema and loader
    - Read `config/group.yml` (YAML). Validate every key of `config/group.example.yml`: name, slug, Meetup address, contact address, languages, default language, accent colour, default licence, review days, retention days, Consent_Text per language.
    - Reject a Consent_Text missing for a configured language.
    - On error, print the file, the key path and what is wrong, and stop.
    - Done when: unit tests pass for the example file, for a missing key, for a bad colour and for a missing consent language. The message names the key path.
    - _Requirements: 1.1 (R1.1), 1.2 (R1.2), 1.3 (R1.3), 1.4 (R1.4), 1.5 (R1.5), 1.6 (R1.6), 1.7 (R1.7), 1.8 (R1.8), 14.1 (R14.1), 19.4 (N3.4)_

  - [ ] 2.2 Add the new optional keys with placeholder values (placeholder until open decisions 1, 4 and 8)
    - Add optional keys from [section 6](../../steering/high-level-design.md#6-how-the-configuration-file-drives-the-site): maximum upload size, maximum files per upload, rate limit values, base address, `site.talk_url`, first wall settings (cap per event, picks for each year position), manifest name, short name and colours.
    - Each key has a documented placeholder default in one file, `src/config/placeholders.ts`. A placeholder is not a decision.
    - Put the keys in the example file only as commented lines, and only after the owner approves the schema change.
    - When open decision 4, 8 or 1 is answered: change the placeholder values or the key shape in this file only.
    - Done when: tests show the example file still validates with no new key, and that each new key is read when present.
    - _Requirements: 6.3 (R6.3), 20.1 (N4.1), 20.2 (N4.2), 28.2 (R21.2), 28.3 (R21.3), 31.2 (R24.2)_

  - [ ] 2.3 Add a second sample configuration
    - Working assumption, open decision 18: this second sample configuration is the second-group test.
    - Add `tests/fixtures/group.second.yml` with an invented name, slug, Meetup address, two other languages, another colour and `cc-by-nc-4.0` as licence.
    - Add a helper that runs a test over both configurations.
    - Done when: the validator accepts both files in a test.
    - _Requirements: 1.9 (R1.9), 14.1 (R14.1)_

  - [ ] 2.4 Add the check command
    - Add `npm run check-config -- <file>`. Exit 0 when valid. Exit 1 with the message of task 2.1 when not.
    - Done when: a test runs the command on a valid file and on a broken file and checks the exit code and the message.
    - _Requirements: 1.1 (R1.1), 1.4 (R1.4), 2.2 (R2.2)_

  - [ ] 2.5 Add the no-group-name test
    - Read the name, slug and Meetup address of both configurations at test time. Scan `src/`, `templates/`, `locales/`, `scripts/` and `infra/` and fail if any appears.
    - Done when: the property test passes, and adding the example group name to a source file makes it fail.
    - **Property 4: No group name in the code**
    - **Validates: Requirements 1.9 (R1.9)**
    - _Requirements: 1.9 (R1.9)_

  - [ ] 2.6 Write the compiled `config.json`
    - Write `config.json` from an allow list of public keys. It carries nothing secret.
    - Done when: a snapshot test for both configurations passes, and a test fails if a key outside the allow list appears.
    - _Requirements: 1.1 (R1.1), 1.4 (R1.4), 1.5 (R1.5), 1.8 (R1.8), 14.1 (R14.1)_

- [ ] 3. One event page end to end
  - [ ] 3.1 Define the event record schema and an example event file
    - Fields: title, date and time, venue, Meetup address, speakers, description per language, attendance.
    - Validate an event JSON file with clear errors.
    - Working assumption, open decision 13: the import file is a JSON list of events.
    - Done when: unit tests pass for a valid event and for each missing field.
    - _Requirements: 3.1 (R3.1)_

  - [ ] 3.2 Add language strings and a build that fails on a missing string
    - Add `locales/<lang>.json` for the languages of both sample configurations. Add a loader.
    - Done when: a test removes one key and the build stops with an error that names the key and the language.
    - _Requirements: 14.1 (R14.1)_

  - [ ] 3.3 Add the accent colour contrast check
    - Compute the WCAG contrast ratio of the accent colour against its text and background colours. Stop the build below the AA ratio.
    - Done when: tests pass for the example colour and fail for a low-contrast colour with a clear message.
    - _Requirements: 15.1 (R15.1)_

  - [ ] 3.4 Build the page shell
    - Header with the group name, language links, and a footer with the photos and privacy link on every page. Use relative addresses. Follow `docs/preview/about.html`.
    - Working assumption, open decision 12: language links are plain links between the page sets.
    - Add a minimal privacy page now so the link works. Task 10.11 fills it.
    - Done when: a test builds the shell for both configurations and finds the footer link in every language.
    - _Requirements: 27.1 (R20.1), 14.1 (R14.1)_

  - [ ] 3.5 Build the event page template
    - Follow `docs/preview/evening.html` without photos. Show the fields of task 3.1 in each language.
    - Done when: a snapshot test for the example event shows title, date, venue, Meetup address, speakers, description and attendance.
    - _Requirements: 4.3 (R4.3), 3.1 (R3.1), 14.1 (R14.1)_

  - [ ] 3.6 Add the local site build command
    - Add `npm run build:site -- --config <file> --events <dir>`. It writes one set of pages per language.
    - Done when: a test builds the example event with both configurations and finds the pages in the output folder.
    - _Requirements: 1.9 (R1.9), 14.1 (R14.1), 4.3 (R4.3)_

  - [ ] 3.7 Build the events list with upcoming, past and the year filter
    - Follow `docs/preview/events.html`. Two lists. A year filter that needs no third-party script.
    - Done when: unit tests split upcoming and past events, and a DOM test shows the filter narrows the lists to the chosen year.
    - _Requirements: 4.1 (R4.1), 4.2 (R4.2)_

  - [ ] 3.8 Add bulk import of past events
    - Working assumption, open decision 13: only Organizers manage events; each bad item is reported with its position.
    - Add `npm run import-events -- <file.json>`. Validate each item with task 3.1. Report each bad item with its position and reason. Write the good ones.
    - Done when: a test with 2 good and 1 bad item imports 2 and reports 1.
    - _Requirements: 4.4 (R4.4), 3.1 (R3.1)_

  - [ ] 3.9 Build the event export page without photos
    - Working assumption, open decision 19: a standalone page at a stable address, with links to the public photo copies.
    - Follow `docs/preview/record.html`. A standalone page with date, venue, Meetup address and attendance. No external file is referenced.
    - Done when: a test finds the four fields and finds no external reference. Photos are added in task 7.17.
    - _Requirements: 5.1 (R5.1), 5.2 (R5.2)_

  - [ ] 3.10 Add the sideways overflow test for every page
    - Playwright opens every built page at 390 px and checks the page is not wider than the screen. A property test builds events with long titles and long words and measures them too.
    - New pages are found by listing the output folder. Add this step to CI.
    - Done when: `npm run test:overflow` passes, and a page with a 500 px wide element makes it fail.
    - **Property 7: No public page is wider than the screen**
    - **Validates: Requirements 29.9 (R22.9)**
    - _Requirements: 29.9 (R22.9)_

- [ ] 4. First deploy
  - [ ] 4.1 Write the SAM template with a private site bucket and the CDN
    - `infra/template.yaml` with the site bucket, CDN, origin access control and a bucket policy for the CDN only. Fixed resource names from the slug. Retain policy on stateful resources.
    - Done when: `sam validate --lint` passes. A template test asserts every bucket blocks public access and no bucket policy allows a public principal.
    - _Requirements: 17.1 (N1.1), 17.2 (N1.2), 17.3 (N1.3), 17.4 (N1.4)_

  - [ ] 4.2 Add the site upload and cache step
    - `scripts/publish-site` uploads the build output to the site bucket and clears the CDN paths.
    - Done when: a test with a fake S3 and CDN client checks the uploaded keys and the cleared paths.
    - _Requirements: 2.2 (R2.2), 17.3 (N1.3)_

  - [ ] 4.3 Add the install command
    - `npm run install-group` checks the configuration, runs `sam build` and `sam deploy` with a stack name from the slug, uploads the site and prints the address.
    - Done when: a test with a fake command runner checks the order, the credentials taken from the environment, and that an invalid configuration stops it before any deploy.
    - _Requirements: 2.1 (R2.1), 2.2 (R2.2)_

  - [ ] 4.4 Add the remove command
    - `npm run remove-group` asks whether to keep or delete the tables and buckets (default keep), runs `sam delete`, and prints what is left.
    - Done when: a test with fake prompts checks the default is keep and that `sam delete` is called.
    - _Requirements: 2.3 (R2.3)_

  - [ ] 4.5 Add the install timing script (placeholder: not yet measured)
    - Working assumption, open decision 16: the target is 30 minutes.
    - `scripts/time-install` runs the install and prints the elapsed time. Add `docs/install-timing.md` with the line "not yet measured".
    - Task 14.5 records the real run on an empty account.
    - Done when: a test checks the script prints an elapsed time with a fake runner.
    - _Requirements: 22.1 (N6.1)_

- [ ] 5. Checkpoint - first deploy
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 6. Data and sign-in
  - [ ] 6.1 Add the data table and the audit table to the template
    - On-demand billing, two indexes (by year, by status), TTL on rate counters, point-in-time recovery on the audit table, fixed names, retain policy. Follow [section 2](../../steering/high-level-design.md#2-data-model).
    - Add a shared policy that allows only `PutItem` on the audit table.
    - Done when: a template test finds both tables and finds no role with update or delete rights on the audit table.
    - _Requirements: 10.2 (R10.2), 17.1 (N1.1)_

  - [ ] 6.2 Write the append-only audit writer
    - `appendAudit` uses `PutItem` with a "not exists" condition. The module exports no update or delete.
    - Done when: the property test passes. It scans every source file and the template for any update, delete or batch write on the audit table, and checks a second write with the same key fails.
    - **Property 3: Audit table is append only**
    - **Validates: Requirements 10.2 (R10.2)**
    - _Requirements: 10.2 (R10.2)_

  - [ ] 6.3 Write the data access layer
    - Key builders and mappers for events, photos, wall settings, users and rate counters. No group key anywhere.
    - Done when: round-trip tests pass for each item type and the year index key is built from the start date.
    - _Requirements: 3.1 (R3.1), 4.1 (R4.1), 4.2 (R4.2)_

  - [ ] 6.4 Add the sign-in interface and the Cognito implementation (placeholder until open decision 6)
    - Define `AuthProvider` with `verifyToken` returning subject and email. Add the Cognito user pool to the template with fixed name and retain policy. Implement the interface for Cognito.
    - Whether sign-in uses a code or a link, and how sign out works, is not decided. Keep it out of the interface.
    - When open decision 6 is answered: change the Cognito settings and the sign-in page (task 6.8). The interface stays.
    - Done when: unit tests pass with a fake provider and a template test finds the user pool.
    - _Requirements: 17.1 (N1.1), 16.1 (R16.1)_

  - [ ] 6.5 Add roles and the role check
    - User items hold the role. `requireRole` runs on every write. Organizers add and remove Organizers and Contributors. Only Organizers publish and decide removal requests.
    - Working assumption, open decision 14: removing the last Organizer is refused.
    - Done when: a table test covers every role against every action and passes.
    - _Requirements: 16.1 (R16.1), 16.2 (R16.2), 16.3 (R16.3), 16.4 (R16.4)_

  - [ ] 6.6 Build the events API
    - Working assumption, open decision 13: only Organizers create, edit and delete events.
    - HTTP API in the template. Routes to create, edit, read and list events. Writes need the Organizer role. Attendance can be edited at any time.
    - Done when: handler tests show an Organizer creates and edits attendance, and a Contributor gets 403.
    - _Requirements: 3.1 (R3.1), 3.2 (R3.2), 16.3 (R16.3)_

  - [ ] 6.7 Create the first Organizer in the install command
    - The install command creates the first Organizer user from the contact address of the configuration.
    - Done when: a test with the fake provider finds one Organizer after install.
    - _Requirements: 2.1 (R2.1), 16.1 (R16.1)_

  - [ ] 6.8 Build the Organizer sign-in page and shell (placeholder until open decision 6)
    - Follow `docs/preview/admin-login.html` and `docs/preview/admin.css`. The page calls the `AuthProvider` only. It shows no behaviour that depends on a code or a link.
    - Done when: a DOM test shows the page works with the fake provider, and the overflow test (task 3.10) covers it.
    - _Requirements: 16.1 (R16.1), 29.9 (R22.9)_

  - [ ] 6.9 Build the publisher for events
    - Reuse the render code of tasks 3.5, 3.7 and 3.9. Read events from the table, write pages to the site bucket, clear the CDN paths. Run it after each event write.
    - Done when: a test with fake clients shows that saving an event writes the event page, the list and the export page and clears those paths.
    - _Requirements: 4.3 (R4.3), 5.1 (R5.1), 17.3 (N1.3)_

- [ ] 7. Photos
  - [ ] 7.1 Define the photo model and the status rules
    - Statuses: uploading, processing, ready, published, hidden_pending_removal, removed, failed. Only an Organizer publish action reaches published. Licence is copied from the default at upload. Photographer name is stored.
    - Done when: a property test applies random action sequences and finds that published is reached only through publish by an Organizer.
    - _Requirements: 8.1 (R8.1), 11.1 (R11.1), 11.3 (R11.3), 11.4 (R11.4)_

  - [ ] 7.2 Add photo storage, queue and CDN rules to the template
    - Private originals bucket. Photos bucket with `staging/` and `public/`. The CDN serves only `public/`. Queue with a dead letter queue on object created. Fixed names, retain policy.
    - Done when: a template test finds no CDN route to `staging/` or to the originals bucket, and all new buckets block public access.
    - _Requirements: 17.2 (N1.2), 17.3 (N1.3), 17.4 (N1.4), 7.5 (R7.5), 8.1 (R8.1)_

  - [ ] 7.3 Write the rate counter
    - Salted hash of the address, window, TTL, salt rotation. Used by uploads and removal requests.
    - Values come from the configuration (placeholder until open decision 4).
    - Done when: a property test shows the stored key never contains the raw address, and unit tests show the count and the expiry.
    - _Requirements: 20.1 (N4.1), 20.2 (N4.2), 19.1 (N3.1)_

  - [ ] 7.4 Build the signed upload address API (placeholder until open decision 4)
    - One address per file. Check role, type (JPEG, PNG, HEIC), size and rate counter before signing. Return the reason for each refused file. The address limits size and type. File bodies never reach a function.
    - Maximum size, files per upload and rate values come from the placeholders of task 2.2.
    - No licence acceptance step at upload (open decision 7 is not answered).
    - When open decision 4 is answered: change the values in `src/config/placeholders.ts` or the configuration. No code change.
    - Done when: handler tests cover each refusal reason, the rate limit, and a test checks the handler never receives a file body.
    - _Requirements: 6.1 (R6.1), 6.2 (R6.2), 6.3 (R6.3), 6.4 (R6.4), 6.5 (R6.5), 20.2 (N4.2)_

  - [ ] 7.5 Build the processing function for JPEG and PNG
    - Working assumption, open decision 9: the original keeps its location data and stays private. Open decision 15: sharp.
    - Container image. Read the original, refuse a file that is not an image, write the tile size (about 480 px) and the viewer size (about 1600 px) to `staging/` without location data. Set the status ready. The original is not changed.
    - Done when: tests with fixture images that carry GPS data find no location tags in either output, the sizes are right, and the original is unchanged.
    - _Requirements: 7.1 (R7.1), 7.2 (R7.2), 7.3 (R7.3), 7.5 (R7.5), 32.5 (N8.5)_

  - [ ] 7.6 Read the capture time
    - Read it from the file and keep it in the photo record. With none, use the upload time and set the estimate mark.
    - Done when: tests with and without a capture time in the fixtures pass.
    - _Requirements: 7.4 (R7.4), 26.1 (R19.1), 26.2 (R19.2)_

  - [ ] 7.7 Add HEIC support
    - Add HEIC decoding to the image. Test with real phone photos supplied by the owner, with the consent of whoever took them.
    - Done when: tests convert the HEIC fixtures, find no location tags, and keep the capture time.
    - _Requirements: 6.2 (R6.2), 7.1 (R7.1), 7.2 (R7.2), 7.3 (R7.3), 7.4 (R7.4)_

  - [ ] 7.8 Wire the queue and the failure path
    - Object created to queue to function. After the retries, the photo is set to failed and the message goes to the dead letter queue.
    - Done when: a handler test with a failing image sets failed after the retry limit, and a template test finds the dead letter queue.
    - _Requirements: 7.1 (R7.1), 7.2 (R7.2)_

  - [ ] 7.9 Build the publisher for photos
    - Publish copies the two files from `staging/` to `public/`, sets published, re-renders pages, clears CDN paths, writes an audit entry. Unpublish does the reverse.
    - Done when: the property test passes. For any set of photos in any status, the keys under `public/` are exactly those of published photos, and the CDN config serves nothing else.
    - **Property 1: Unpublished photos are not reachable**
    - **Validates: Requirements 8.1 (R8.1), 17.4 (N1.4)**
    - _Requirements: 8.1 (R8.1), 17.4 (N1.4)_

  - [ ] 7.10 Build the moderation API
    - Working assumption, open decision 20: Organizers preview unpublished photos from `staging/` through signed addresses.
    - Routes to list ready photos with signed preview addresses from `staging/`, set the alt text per language, change the licence, publish and hide. Organizer only.
    - Done when: handler tests cover each route, and a Contributor cannot publish.
    - _Requirements: 8.1 (R8.1), 11.4 (R11.4), 15.2 (R15.2), 16.3 (R16.3)_

  - [ ] 7.11 Add the cover photo rule
    - Working assumption, open decision 11: the Organizer sets the cover. Otherwise the first published photo. Otherwise a plain card.
    - Done when: unit tests cover the three cases.
    - _Requirements: 13.4 (R13.4)_

  - [ ] 7.12 Build the Organizer photos page
    - Follow `docs/preview/admin-photos.html`. Upload with a file picker, the list of photos with state, alt text, licence, photographer, publish and hide. The phone layout is task 12.8 and 12.9.
    - Done when: DOM tests with a fake API cover publish, hide and alt text, and the overflow test covers the page.
    - _Requirements: 6.1 (R6.1), 8.1 (R8.1), 11.1 (R11.1), 15.2 (R15.2)_

  - [ ] 7.13 Build the album page with the consent text
    - Follow `docs/preview/evening.html`. Show the Consent_Text in the page language, the photographer in small type, and the licence. Add the consent text to no other page and not as a banner.
    - Done when: tests find the consent text on the album page in each language and not on the events list or the home page.
    - _Requirements: 9.1 (R9.1), 9.2 (R9.2), 9.3 (R9.3), 11.2 (R11.2), 11.5 (R11.5), 13.1 (R13.1)_

  - [ ] 7.14 Build the enlarged view and the sharing preview
    - A viewer script with no third-party code. Keyboard: arrows, Escape, focus kept inside. The album page sets sharing preview tags from the cover photo.
    - Done when: DOM tests cover the keys and the focus, and a test finds the cover address in the preview tags.
    - _Requirements: 13.2 (R13.2), 13.3 (R13.3), 13.4 (R13.4)_

  - [ ] 7.15 Add playback in capture time order
    - Follow `docs/preview/evening.html`. Order by capture time, with a stable tie-break.
    - Done when: a unit test checks the order, and a DOM test checks the playback shows the photos in that order.
    - _Requirements: 13.5 (R13.5)_

  - [ ] 7.16 Add capture time editing
    - An Organizer changes the capture time of one photo, which removes the estimate mark. An Organizer moves all capture times of one event by a number of minutes.
    - Working assumption, not yet an open decision: the shift leaves estimate marks as they are.
    - Done when: API tests cover both actions and the re-render after each.
    - _Requirements: 26.3 (R19.3), 26.4 (R19.4)_

  - [ ] 7.17 Add published photos to the event export
    - Extend the export of task 3.9 with the published photos of the event.
    - Done when: a test finds the published photos in the export and no hidden or pending one.
    - _Requirements: 5.3 (R5.3)_

  - [ ] 7.18 Add the no-identification check
    - A test scans the source and the template for any call to a face recognition or tagging service. There is no tagging path.
    - Done when: the test passes, and adding such a call makes it fail. The opt-in of R12.3 is not built (open decisions 2 and 3).
    - _Requirements: 12.1 (R12.1), 12.2 (R12.2)_

- [ ] 8. Removal
  - [ ] 8.1 Build the public removal route with a rate limit (placeholder until open decision 4)
    - Working assumption, open decision 17: no contact is collected from the requester.
    - A public route to request removal of one photo. Rate limit with task 7.3. A cap of open requests per photo.
    - Rate values are placeholders.
    - Done when: handler tests show the limit refuses with 429 and the cap refuses another request on the same photo.
    - _Requirements: 10.1 (R10.1), 20.1 (N4.1)_

  - [ ] 8.2 Write the removal transaction
    - One transaction sets the photo to hidden_pending_removal (only from published), creates the request with its date, and writes the audit entry `removal.requested`.
    - Done when: a test shows all three happen or none do.
    - _Requirements: 10.1 (R10.1), 10.2 (R10.2)_

  - [ ] 8.3 Make the publisher hide a photo at once
    - On a removal request: delete the `public/` copies first, then re-render the album and event pages, then clear the CDN paths.
    - Add a hook for the wall file. Task 10.2 fills it.
    - Done when: the property test passes and a test checks the order: delete, then clear.
    - **Property 2: Pending-removal photos are not reachable**
    - **Validates: Requirements 10.1 (R10.1), 10.4 (R10.4)**
    - _Requirements: 10.1 (R10.1), 10.4 (R10.4), 10.6 (R10.6)_

  - [ ] 8.4 Add the Request removal button
    - A button on the album page and in the enlarged view. It calls the route and hides the photo on the page at once.
    - Done when: DOM tests cover the button and the page state after the call.
    - _Requirements: 10.1 (R10.1)_

  - [ ] 8.5 Notify the Organizers
    - An email topic in the template. The removal flow publishes a message with the photo and the request. The message holds no visitor address or contact.
    - Subscription by email needs a confirmation by each Organizer. Note it in the guide (task 14.3).
    - Done when: a handler test checks the message content, and a template test finds the topic.
    - _Requirements: 10.3 (R10.3)_

  - [ ] 8.6 Build the decision API (placeholder until open decision 5)
    - Organizer-only routes to approve or reject. Reject restores the photo to published with a reason in the audit entry (working assumption, open decision 5). Approve sets removed and deletes the derived copies.
    - What happens to the original is behind an `OriginalDisposition` interface. The default keeps the original. It does not delete.
    - When open decision 5 is answered: add the implementation of the interface. Nothing else changes.
    - Done when: handler tests cover both decisions, the audit entries and the role check.
    - _Requirements: 10.5 (R10.5), 16.4 (R16.4), 10.2 (R10.2)_

  - [ ] 8.7 Build the Organizer requests page
    - Follow `docs/preview/admin-requests.html`. List open requests with date and deadline. Approve and reject.
    - Done when: DOM tests with a fake API cover both decisions, and the overflow test covers the page.
    - _Requirements: 10.5 (R10.5), 16.4 (R16.4)_

  - [ ] 8.8 Add the review deadline reminder
    - A daily scheduler runs the sweeper. At `takedown_review_days`, it sends a reminder for each undecided request. The photo stays hidden.
    - Done when: a test with a fake clock finds one reminder per undecided request on the deadline day.
    - _Requirements: 10.5 (R10.5), 10.4 (R10.4)_

  - [ ] 8.9 Add the retention sweep (placeholder until open decision 5)
    - Find unpublished originals older than `retention_days`. The end action is behind a `RetentionAction` interface. The default only records the list in an audit entry and deletes nothing.
    - When open decision 5 is answered: add the implementation of the interface.
    - Done when: a fake clock test lists the right originals and writes one audit entry per run.
    - _Requirements: 19.3 (N3.3), 19.4 (N3.4)_

- [ ] 9. Checkpoint - photos and removal
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 10. The wall
  - [ ] 10.1 Write the wall file builder
    - A pure function from events, photos and settings to the entries of `wall.json`: photo id, event number, date, year, tile address, viewer address, photographer. Newest event first, capture time inside an event, cap per event.
    - Done when: the property test passes.
    - **Property 6: The wall file lists only published, picked photos within the cap**
    - **Validates: Requirements 24.3 (R17.3), 28.4 (R21.4)**
    - _Requirements: 24.2 (R17.2), 24.3 (R17.3), 32.1 (N8.1)_

  - [ ] 10.2 Make the publisher write the wall file
    - Rewrite `wall.json` on publish, pick, unpick, hide and removal. Send a cache of about one minute. Clear the cache on hide and removal. Fill the hook of task 8.3.
    - Done when: the property test passes. After any photo becomes hidden or removed, the file does not list it. A test checks the cache header.
    - **Property 5: Hidden or removed photos leave the wall**
    - **Validates: Requirements 10.6 (R10.6), 32.3 (N8.3)**
    - _Requirements: 32.1 (N8.1), 32.2 (N8.2), 32.3 (N8.3), 10.6 (R10.6)_

  - [ ] 10.3 Write the wall plan module (placeholder until open decision 1)
    - One module, `src/wall/plan.ts`, with a `planWall` function. It uses a number of picks for each year position, shows the rest of a year as one pile, and shows years after the last position as a pile only. It offers no number for a single calendar year.
    - Whether the wall uses numbers or percentage shares is not decided. Only this module knows.
    - When open decision 1 is answered: replace this module and the settings keys of task 2.2.
    - Done when: property tests show the tiles and the pile of a year hold exactly its picks.
    - _Requirements: 24.4 (R17.4), 24.5 (R17.5), 24.6 (R17.6)_

  - [ ] 10.4 Build the wall on the home page
    - Follow `docs/preview/home.html`. Draw tiles and piles from `wall.json`. Fixed tile size, the wall grows in length. A pile opens in place and folds back. An event with no pick has no tile.
    - Done when: DOM tests cover tiles, piles and open and fold, a test checks the page makes no request except `wall.json`, and the overflow test covers it.
    - _Requirements: 24.1 (R17.1), 24.7 (R17.7), 24.8 (R17.8), 24.9 (R17.9), 32.4 (N8.4)_

  - [ ] 10.5 Build the viewer overlay
    - Opens over the wall from a tile. Moves through all picks in wall order, across years and piles. Shows the photo, one button to the evening page, and the photographer in small type. No tags and no title line.
    - Done when: DOM tests cover opening, moving across a pile and the content of the overlay.
    - _Requirements: 25.1 (R18.1), 25.2 (R18.2), 25.3 (R18.3), 25.4 (R18.4)_

  - [ ] 10.6 Add the address marker to the viewer
    - Opening adds `#photo=<id>`. The back button closes. A loaded address with the marker opens on that photo.
    - Done when: DOM tests cover open, back and load with a marker.
    - _Requirements: 25.5 (R18.5), 25.6 (R18.6)_

  - [ ] 10.7 Add keyboard use and end stops to the viewer
    - Arrow keys and Escape. The previous control is disabled at the first pick. The next control is disabled at the last pick.
    - Done when: the property test passes.
    - **Property 8: Viewer controls stop at the ends**
    - **Validates: Requirements 25.7 (R18.7)**
    - _Requirements: 25.7 (R18.7), 25.8 (R18.8)_

  - [ ] 10.8 Add swipe to the viewer
    - On touch screens: swipe sideways to move, swipe down to close.
    - Done when: DOM tests with touch events cover both swipes.
    - _Requirements: 25.9 (R18.9)_

  - [ ] 10.9 Build the pick API and the wall settings (placeholder until open decision 1)
    - Organizer routes to pick and unpick, and to change the cap and the picks for each year position. Refuse a pick over the cap and say the cap. List events with published photos and no pick. Settings live in the `SETTINGS` item and start from the configuration.
    - Done when: handler tests cover pick, unpick, the refusal message and the list of events with no pick.
    - _Requirements: 28.1 (R21.1), 28.2 (R21.2), 28.3 (R21.3), 28.4 (R21.4), 28.5 (R21.5)_

  - [ ] 10.10 Build the Organizer wall page
    - Follow `docs/preview/admin-wall.html`. Pick and unpick, set the cap and the numbers, list events with no pick, preview the wall before leaving.
    - Done when: DOM tests with a fake API cover each action, and the overflow test covers the page.
    - _Requirements: 28.1 (R21.1), 28.2 (R21.2), 28.3 (R21.3), 28.5 (R21.5), 28.6 (R21.6)_

  - [ ] 10.11 Fill the photos and privacy page
    - Replace the minimal page of task 3.4. Show the Consent_Text, how to ask for a removal, and the retention period from the configuration.
    - The tagging text of R20.3 is not built. It depends on open decisions 2 and 3.
    - Done when: tests find the three parts in each language, and the page is linked from the footer of every page including the home page.
    - _Requirements: 27.1 (R20.1), 27.2 (R20.2), 19.3 (N3.3)_

- [ ] 11. Organizer area pages
  - [ ] 11.1 Build the evenings page
    - Follow `docs/preview/admin-evenings.html`. Create and edit events and attendance. The speakers list of the preview is not built because no requirement names it. Speakers stay a field of the event.
    - Done when: DOM tests with a fake API cover create and attendance edit, and the overflow test covers the page.
    - _Requirements: 3.1 (R3.1), 3.2 (R3.2), 16.3 (R16.3)_

  - [ ] 11.2 Build the team page
    - Follow `docs/preview/admin-team.html`. Add and remove Organizers and Contributors.
    - Done when: DOM tests cover add and remove for both roles, and the overflow test covers the page.
    - _Requirements: 16.1 (R16.1), 16.2 (R16.2)_

  - [ ] 11.3 Build the dashboard
    - Follow `docs/preview/admin-dashboard.html`. Show only what a requirement needs: the counts of photos and requests that wait.
    - Done when: a test with fake data finds both counts.
    - _Requirements: 30.2 (R23.2)_

- [ ] 12. Phone experience
  - [ ] 12.1 Add the breakpoints and the public tab bar (placeholder until open decision 8)
    - Keep the widths 640 px and 760 px as two constants in one file. Add the Tab_Bar fixed at the bottom and a slim top bar that scrolls away.
    - When open decision 8 is answered: change the constants.
    - Done when: Playwright tests at 390 px find the bar fixed at the bottom and the top bar scrolling away.
    - _Requirements: 29.1 (R22.1), 29.2 (R22.2)_

  - [ ] 12.2 Make the wall edge to edge
    - Three tiles per row at 640 px or less.
    - Done when: a Playwright test at 390 px counts three tiles per row and no side gap.
    - _Requirements: 29.3 (R22.3)_

  - [ ] 12.3 Add the year bar
    - One row that scrolls sideways, All fixed at the left, sticky at the top. While All is chosen, mark the year in view. Mark a chosen year in a different way.
    - Done when: Playwright tests cover the sticky position, the fixed All and the two marks.
    - _Requirements: 29.4 (R22.4), 29.5 (R22.5)_

  - [ ] 12.4 Add the back-to-top button and the Home tab scroll (placeholder until open decision 8)
    - A round button above the tab bar. It shows when scrolling up after going far down. It hides when scrolling down, after a pause, or near the top. The pause is one constant, about 3.5 s in the preview.
    - Tapping Home on the home page scrolls to the top.
    - Done when: DOM tests with a fake clock cover show and hide, and the Home tap.
    - _Requirements: 29.6 (R22.6), 29.7 (R22.7)_

  - [ ] 12.5 Add the compact footer, touch sizes and safe areas
    - Compact footer with the photos and privacy link and the statement that the project is not affiliated with Amazon Web Services. Controls at least 44 px high, fields at least 16 px, no action that needs hover, controls outside the safe areas.
    - Done when: a Playwright test with touch emulation measures the controls and fields on every public page, and a test finds the safe-area rules.
    - _Requirements: 29.8 (R22.8), 29.10 (R22.10), 29.11 (R22.11)_

  - [ ] 12.6 Keep the wall when only the height changes
    - Done when: a Playwright test resizes only the height and a mutation observer finds no tile recreated.
    - _Requirements: 29.12 (R22.12)_

  - [ ] 12.7 Add the Organizer tab bar and the More sheet
    - At 760 px or less: tabs Home, Photos, Wall, Requests, More. Counts on Photos and Requests. The More sheet lists Evenings, Team, Settings, View the site and Sign out. It closes on tap outside, swipe down, Escape or a second tap on More.
    - The Settings and Speakers entries of the preview have no requirement. List them as gaps and leave the entries out until the owner decides.
    - Done when: Playwright tests cover the tabs, the counts and the four ways to close.
    - _Requirements: 30.1 (R23.1), 30.2 (R23.2), 30.3 (R23.3), 30.4 (R23.4)_

  - [ ] 12.8 Turn tables into cards and show photo cards
    - At 760 px or less: each value shows its column name, main actions are full width. Photos show one per card with picture, state, actions and fields.
    - Done when: Playwright tests at 390 px cover the tables of requests, evenings and team, and the photo cards.
    - _Requirements: 30.5 (R23.5), 30.6 (R23.6)_

  - [ ] 12.9 Add Choose photos and Take a photo
    - On touch screens: two buttons instead of a drop area. The same type and size rules and the same reasons for any refused file when chosen, taken or dropped.
    - Done when: DOM tests cover the three ways and the refusal messages.
    - _Requirements: 30.7 (R23.7), 30.8 (R23.8)_

  - [ ] 12.10 Generate the web app manifest and icons
    - At build time, from the configuration: name, short name, colours, icons. The start address is the home page, opened without browser bars.
    - Done when: tests for both configurations find the values in the manifest, `display` set to standalone, and the icon files.
    - _Requirements: 31.1 (R24.1), 31.2 (R24.2), 31.3 (R24.3)_

- [ ] 13. Checkpoint - the full site
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 14. Hardening and guide
  - [ ] 14.1 Write the cost model and the budget alarm
    - A script computes the monthly cost for 30 events a year from stated usage and AWS prices. Add a budget alarm to the template.
    - The owner confirms the figure with real billing data. That step is not a coding task.
    - Done when: unit tests cover the model, and a template test finds the alarm.
    - _Requirements: 18.1 (N2.1), 18.2 (N2.2)_

  - [ ] 14.2 Add automated accessibility checks and the manual checklist
    - Run an accessibility rules engine on every built page, in both configurations. Add `docs/accessibility-checklist.md`.
    - Automated checks find part of the WCAG 2.2 AA problems. Full conformance needs manual testing with assistive technology and expert review.
    - Done when: the checks pass in CI, and the checklist file exists.
    - _Requirements: 15.1 (R15.1), 15.2 (R15.2)_

  - [ ] 14.3 Write the guide
    - Working assumption, open decision 16: the guide targets 30 minutes.
    - `docs/guide.md`: install, first event, roles, photos, removal, retention, the monthly cost, the email subscription confirmation, a recommendation of at least two Organizers, and the limits of caches.
    - The wording about licences that cannot be recalled is a placeholder until open decision 7.
    - Done when: the guide has the sections, and a link check test passes.
    - _Requirements: 22.1 (N6.1), 22.2 (N6.2), 19.3 (N3.3), 18.2 (N2.2)_

  - [ ] 14.4 Test that the guide covers every configuration key
    - Done when: a test fails if a schema key is not named in the guide.
    - _Requirements: 22.1 (N6.1), 1.1 (R1.1)_

  - [ ] 14.5 Add the timed run record
    - Add `docs/timed-run.md` as a template for a run by someone new on an empty account: install time and first event time against 30 minutes each. Update `docs/install-timing.md`.
    - The run itself is done by a person. It is not a coding task.
    - Done when: the file has the fields and the install timing note no longer says "not yet measured" only after a real run is recorded.
    - _Requirements: 22.1 (N6.1), 22.2 (N6.2)_

  - [ ] 14.6 Add the release check
    - `npm run release-check` builds both configurations, then runs the tests, the overflow test and the accessibility checks.
    - Done when: the command exits 0 on a clean tree and 1 when one step fails.
    - _Requirements: 21.1 (N5.1), 1.9 (R1.9), 29.9 (R22.9)_

- [ ] 15. Final checkpoint - all tests green
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` would be optional. None are. The owner asked for tests with the task they test, so every test is a core part of its task.
- Group numbers differ from the owner's list because checkpoints are numbered tasks. Mapping: owner group 1 to 4 are tasks 1 to 4, checkpoint 5, group 5 is task 6, group 6 is task 7, group 7 is task 8, checkpoint 9, group 8 is task 10, group 10 is task 11, group 9 is task 12, checkpoint 13, group 11 is task 14, group 12 is task 15. Organizer pages come before the phone work because the phone work needs them.
- Placeholders, by open decision:
  - Decision 1 (wall numbers or shares): 2.2, 10.3, 10.9, 10.10.
  - Decision 2 (tag yourself): R20.3 is not built; 10.11.
  - Decision 3 (face recognition and tagging opt-in): R12.3 is not built; 7.18.
  - Decision 4 (limits): 2.2, 7.3, 7.4, 8.1.
  - Decision 5 (after a removal decision, end of retention): 8.6, 8.9.
  - Decision 6 (sign-in details): 6.4, 6.8.
  - Decision 7 (licence at upload): no acceptance step in 7.4; wording in 14.3.
  - Decision 8 (small values): 2.2 (talk address), 12.1, 12.4.
- Working assumptions, by open decision (items 9 to 20):
  - Decision 9 (location data in the original): 7.5.
  - Decision 10 (signed download of originals for Organizers): no task builds it yet.
  - Decision 11 (cover photo rule): 7.11.
  - Decision 12 (language switch by plain links): 3.4.
  - Decision 13 (who manages events, the import file): 3.1, 3.8, 6.6.
  - Decision 14 (the last Organizer cannot be removed): 6.5.
  - Decision 15 (tools, CI and cost wording): 1.1, 1.2, 7.5.
  - Decision 16 (install target of 30 minutes): 4.5, 14.3.
  - Decision 17 (no contact for removal requests): 8.1.
  - Decision 18 (second sample configuration): 2.3.
  - Decision 19 (export is a standalone page with links to public copies): 3.9.
  - Decision 20 (staging preview through signed addresses): 7.10.
  - Not yet an open decision: the capture time shift leaves estimate marks as they are (7.16).
- Tasks that need a person or an AWS account outside the code: the real install timing (4.5, 14.5), the cost check with billing data (14.1), HEIC photos with consent (7.7), manual accessibility testing (14.2), tests on at least one iOS and one Android phone (risk in section 9, not a task).
- Each task changes one thing and should take about a day or less. Tasks 7.5 to 7.7 and 7.9 are the largest. Split them if they run over.
- Property tests are in the task that builds the thing they check: 1 in 7.9, 2 in 8.3, 3 in 6.2, 4 in 2.5, 5 in 10.2, 6 in 10.1, 7 in 3.10, 8 in 10.7.
- When the owner answers an open decision, remove its label from the tasks and move the answer to Decisions in the steering requirements.

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1.1"] },
    { "id": 1, "tasks": ["1.2", "1.3"] },
    { "id": 2, "tasks": ["2.1"] },
    { "id": 3, "tasks": ["2.2", "2.3"] },
    { "id": 4, "tasks": ["2.4", "2.6"] },
    { "id": 5, "tasks": ["2.5"] },
    { "id": 6, "tasks": ["3.1", "3.2", "3.3"] },
    { "id": 7, "tasks": ["3.4", "3.8"] },
    { "id": 8, "tasks": ["3.5"] },
    { "id": 9, "tasks": ["3.6"] },
    { "id": 10, "tasks": ["3.7"] },
    { "id": 11, "tasks": ["3.9"] },
    { "id": 12, "tasks": ["3.10"] },
    { "id": 13, "tasks": ["4.1"] },
    { "id": 14, "tasks": ["4.2", "4.5"] },
    { "id": 15, "tasks": ["4.3"] },
    { "id": 16, "tasks": ["4.4"] },
    { "id": 17, "tasks": ["6.1"] },
    { "id": 18, "tasks": ["6.2", "6.3"] },
    { "id": 19, "tasks": ["6.4"] },
    { "id": 20, "tasks": ["6.5"] },
    { "id": 21, "tasks": ["6.6"] },
    { "id": 22, "tasks": ["6.7", "6.8"] },
    { "id": 23, "tasks": ["6.9"] },
    { "id": 24, "tasks": ["7.1", "7.3"] },
    { "id": 25, "tasks": ["7.2"] },
    { "id": 26, "tasks": ["7.4"] },
    { "id": 27, "tasks": ["7.5"] },
    { "id": 28, "tasks": ["7.6"] },
    { "id": 29, "tasks": ["7.7"] },
    { "id": 30, "tasks": ["7.8"] },
    { "id": 31, "tasks": ["7.9"] },
    { "id": 32, "tasks": ["7.10"] },
    { "id": 33, "tasks": ["7.11", "7.13"] },
    { "id": 34, "tasks": ["7.12", "7.14"] },
    { "id": 35, "tasks": ["7.15", "7.16"] },
    { "id": 36, "tasks": ["7.17", "7.18"] },
    { "id": 37, "tasks": ["8.1"] },
    { "id": 38, "tasks": ["8.2"] },
    { "id": 39, "tasks": ["8.3"] },
    { "id": 40, "tasks": ["8.4", "8.5"] },
    { "id": 41, "tasks": ["8.6"] },
    { "id": 42, "tasks": ["8.7", "8.8"] },
    { "id": 43, "tasks": ["8.9"] },
    { "id": 44, "tasks": ["10.1"] },
    { "id": 45, "tasks": ["10.2", "10.3"] },
    { "id": 46, "tasks": ["10.4", "10.9"] },
    { "id": 47, "tasks": ["10.5", "10.11"] },
    { "id": 48, "tasks": ["10.6", "10.10"] },
    { "id": 49, "tasks": ["10.7"] },
    { "id": 50, "tasks": ["10.8"] },
    { "id": 51, "tasks": ["11.1", "11.2"] },
    { "id": 52, "tasks": ["11.3"] },
    { "id": 53, "tasks": ["12.1", "12.10"] },
    { "id": 54, "tasks": ["12.2"] },
    { "id": 55, "tasks": ["12.3"] },
    { "id": 56, "tasks": ["12.4"] },
    { "id": 57, "tasks": ["12.5"] },
    { "id": 58, "tasks": ["12.6"] },
    { "id": 59, "tasks": ["12.7"] },
    { "id": 60, "tasks": ["12.8"] },
    { "id": 61, "tasks": ["12.9"] },
    { "id": 62, "tasks": ["14.1", "14.2"] },
    { "id": 63, "tasks": ["14.3"] },
    { "id": 64, "tasks": ["14.4"] },
    { "id": 65, "tasks": ["14.5", "14.6"] }
  ]
}
```
