---
inclusion: auto
name: requirements
description: The full approved requirements R1 to R24 and N1 to N8, glossary, out of scope and decisions. Use when writing a spec, design or task, or when a requirement ID is cited.
---

# Requirements Document

## Introduction

aws-ug-hub is a serverless site for AWS User Groups. Each group installs its own copy on its own AWS account. Version 1 covers events, photos and an exportable event record. Status: draft for discussion with the other organizers.

IDs R1 to R16 (functional) and N1 to N7 (non-functional) come from the approved draft. R17 to R24 and N8 were added on 2026-11-07 from the owner's decisions on the wall and on the phone experience (see Decisions). Steering, design and tasks refer to them by these IDs. Do not renumber them. The spec format needs a running heading number, so each heading reads "Requirement 1: R1: ..." through "Requirement 32: N8: ...". The draft ID after the running number is the one to cite.

### Goals

- **A home for the group's story.** One place where a user group keeps its events, its photos and its history, and where anyone can see how the group has lived over the years.
- **Shared with the community.** Built in the open and documented so that any other user group can pick it up, try it, improve it and give its changes back.
- **Easy to make your own.** A group adapts the site by editing a configuration file, not the code, and is running in a short afternoon.
- **Care for the people in the photos.** Consent and removal are part of the design from the first day.
- **Records that can be read and checked.** Every event leaves a page that others can open and verify.
- **Looked after by several people.** Maintained by more than one person, with a clear way for newcomers to start.

### Users

- **Organizer:** creates events, uploads and publishes photos, decides on removal requests.
- **Contributor:** uploads photos for an event. Cannot publish.
- **Visitor:** reads events and published photos. Needs no account.

### Constraints to keep visible

- Nothing about a specific group is written in the code (R1).
- No servers to patch (N1).
- No public bucket (N1).
- No tracking cookies and no third-party analytics (N3).
- No face recognition (R12, out of scope).

## Glossary

- **System**: The aws-ug-hub site, including its public pages, organizer functions, storage and deployment tooling.
- **Group**: One AWS User Group that installs its own copy of the System.
- **Configuration_File**: The single file that holds the Group's settings. The example is `config/group.example.yml`.
- **Organizer**: A user who creates events, uploads and publishes photos, and decides on removal requests.
- **Contributor**: A user who uploads photos for an event and cannot publish them.
- **Visitor**: A person who reads events and published photos and needs no account.
- **Event**: A meeting of the Group with a record in the System.
- **Event_Record**: The stored data for one Event: title, date and time, venue, Meetup address, speakers, description and attendance.
- **Event_Export**: A standalone page for one Event, suitable as proof of the Group's activity.
- **Photo**: An image uploaded to an Event.
- **Original**: The file as uploaded. It stays private.
- **Thumbnail**: The small version of a Photo created by the System. It is the tile size, about 480 px on the longest side, used on the Wall and in lists.
- **Web_Size**: The version of a Photo shown on public pages and in the Wall_Viewer. It is the viewer size, about 1600 px on the longest side.
- **Published_Photo**: A Photo that an Organizer has published and that is not hidden by a removal request.
- **Album**: The set of Published_Photos of one Event, shown as a gallery.
- **Removal_Request**: A request from a Visitor, made with the Request removal button, to remove one Photo.
- **Consent_Text**: The notice about photo publication and removal, set per language in the Configuration_File.
- **Licence**: The licence applied to a Photo. The Configuration_File holds the default.
- **Meetup_Address**: The web address of the Group's Meetup page or of an Event on Meetup.
- **CDN**: The content delivery network that serves the System's stored content.
- **Install_Command**: The documented command that deploys the System to an AWS account.
- **Remove_Command**: The documented command that removes the System from an AWS account.
- **Guide**: The documentation for installing and running the System.
- **WCAG_2_2_AA**: Web Content Accessibility Guidelines 2.2, level AA.
- **Wall**: The grid of picked Photos on the home page, newest Event first.
- **Pick**: A Published_Photo that an Organizer chose for the Wall.
- **Pile**: One tile that folds the rest of the Picks of a year. Opening it shows them in place.
- **Year_Position**: The place of a year counted back from the latest year: latest, previous, and so on. It is not a calendar year.
- **Wall_Viewer**: The overlay that opens over the Wall to browse the Picks.
- **Capture_Time**: The time a Photo was taken, read from the file. When the file has none, the upload time, marked as an estimate.
- **Tab_Bar**: The bar of main sections fixed at the bottom of the screen on a phone.

## Requirements

### Requirement 1: R1: Group identity from a file

**User Story:** As an Organizer, I want the group identity to come from one configuration file, so that my group can use the System without code changes.

#### Acceptance Criteria

1. THE System SHALL read the group name from the Configuration_File.
2. THE System SHALL read the group slug from the Configuration_File.
3. THE System SHALL read the Meetup group address from the Configuration_File.
4. THE System SHALL read the languages from the Configuration_File.
5. THE System SHALL read the accent colour from the Configuration_File.
6. THE System SHALL read the contact address from the Configuration_File.
7. THE System SHALL read the photo licence default from the Configuration_File.
8. THE System SHALL read the Consent_Text from the Configuration_File.
9. THE System SHALL contain no name of any specific Group in its code.

### Requirement 2: R2: One-command install

**User Story:** As an Organizer, I want to install the System with one command, so that my group is running in a short afternoon.

#### Acceptance Criteria

1. WHEN an Organizer runs the Install_Command with AWS credentials, THE System SHALL deploy to the AWS account of those credentials.
2. WHEN an Organizer runs the Install_Command, THE System SHALL deploy without code changes.
3. WHEN an Organizer runs the Remove_Command, THE System SHALL remove the deployed System from the AWS account.

### Requirement 3: R3: Event record

**User Story:** As an Organizer, I want to create and edit events, so that the group's history is complete.

#### Acceptance Criteria

1. WHEN an Organizer creates an Event, THE System SHALL store the title, date and time, venue, Meetup_Address, speakers, description and attendance in the Event_Record.
2. WHEN an Organizer edits the attendance of an Event after the Event, THE System SHALL store the new attendance.

### Requirement 4: R4: History

**User Story:** As a Visitor, I want to browse upcoming and past events, so that I can see how the group has lived over the years.

#### Acceptance Criteria

1. THE System SHALL show upcoming Events and past Events in separate lists.
2. WHEN a Visitor selects a year, THE System SHALL filter the lists by that year.
3. THE System SHALL provide a page for each Event.
4. WHEN an Organizer provides a JSON file of past Events, THE System SHALL import those Events in bulk.

### Requirement 5: R5: Event record export

**User Story:** As an Organizer, I want to export an event as a standalone page, so that I can show it as proof of the group's activity.

#### Acceptance Criteria

1. WHEN an Organizer exports an Event, THE System SHALL produce an Event_Export as a standalone page.
2. THE Event_Export SHALL contain the date, venue, Meetup_Address and attendance of the Event.
3. THE Event_Export SHALL contain the Published_Photos of the Event.

### Requirement 6: R6: Upload

**User Story:** As a Contributor, I want to upload several photos to an event, so that the album can be built quickly.

#### Acceptance Criteria

1. WHEN an Organizer or Contributor uploads Photos to an Event, THE System SHALL accept several files in one upload.
2. THE System SHALL accept JPEG, PNG and HEIC files.
3. THE System SHALL accept files up to the maximum size set in the Configuration_File.
4. WHEN an Organizer or Contributor uploads Photos, THE System SHALL receive the files without sending them through a function.
5. IF a file is not JPEG, PNG or HEIC, or exceeds the maximum size, THEN THE System SHALL reject that file and report the reason.

### Requirement 7: R7: Processing

**User Story:** As an Organizer, I want uploaded photos to be processed safely, so that location data is not exposed and originals stay private.

#### Acceptance Criteria

1. WHEN a Photo is uploaded, THE System SHALL create a Thumbnail.
2. WHEN a Photo is uploaded, THE System SHALL create a Web_Size version.
3. WHEN a Photo is uploaded, THE System SHALL remove location data from the Thumbnail and the Web_Size version.
4. WHEN a Photo is uploaded, THE System SHALL keep the capture date.
5. WHEN a Photo is uploaded, THE System SHALL keep the Original private.

### Requirement 8: R8: Unpublished by default

**User Story:** As an Organizer, I want new photos hidden until I publish them, so that nothing appears before I have checked it.

#### Acceptance Criteria

1. THE System SHALL keep every new Photo hidden from Visitors until an Organizer publishes it.

### Requirement 9: R9: Consent notice

**User Story:** As a Visitor, I want to see the consent notice on every album, so that I know how photos are published and removed.

#### Acceptance Criteria

1. THE System SHALL show the Consent_Text on every Album page.
2. THE System SHALL show the Consent_Text in the language of the page.
3. THE System SHALL show the Consent_Text on the Album page only. THE System SHALL NOT show it on the Wall and SHALL NOT show it as a banner.

### Requirement 10: R10: Removal request

**User Story:** As a Visitor, I want to ask for a photo to be removed, so that I can protect my privacy.

#### Acceptance Criteria

1. WHEN a Visitor presses Request removal on a Photo, THE System SHALL hide that Photo from Visitors at once.
2. WHEN a Visitor presses Request removal on a Photo, THE System SHALL record the Removal_Request with its date.
3. WHEN a Visitor presses Request removal on a Photo, THE System SHALL notify the Organizers.
4. WHILE a Removal_Request is undecided, THE System SHALL keep the Photo hidden from Visitors.
5. THE System SHALL let an Organizer decide on a Removal_Request within the number of days set in the Configuration_File.
6. WHEN a Photo is hidden or removed, THE System SHALL stop serving it to Visitors within seconds and SHALL clear it from the cache. The effect is fast, not instant.

### Requirement 11: R11: Credit and licence

**User Story:** As a Contributor, I want my name shown with my photos and a clear licence applied, so that my work is credited.

#### Acceptance Criteria

1. THE System SHALL store the photographer's name for each Photo.
2. THE System SHALL show the photographer's name with each Published_Photo.
3. THE System SHALL apply the default Licence from the Configuration_File to each Photo.
4. WHERE an Organizer sets another Licence for a Photo, THE System SHALL apply that Licence to that Photo.
5. THE System SHALL show the photographer's name in small type on the Album page and in the Wall_Viewer, and SHALL NOT show it on Wall tiles.

### Requirement 12: R12: No identification of people

**User Story:** As a Visitor, I want the System not to identify people in photos, so that my privacy is protected.

#### Acceptance Criteria

1. THE System SHALL run no face recognition by default.
2. THE System SHALL tag no people by default.
3. WHERE a Group turns on face recognition or tagging explicitly, THE System SHALL run face recognition or tagging as configured.

### Requirement 13: R13: Gallery

**User Story:** As a Visitor, I want to view an album as a gallery, so that I can look at the photos of an event.

#### Acceptance Criteria

1. THE System SHALL show an Album as a gallery.
2. THE System SHALL provide an enlarged view of a Photo.
3. THE System SHALL let a Visitor operate the enlarged view with the keyboard.
4. THE System SHALL set the sharing preview of an Album page to the cover photo of the Album.
5. THE System SHALL let a Visitor play the Photos of an Album in the order of their Capture_Time.

### Requirement 14: R14: Languages

**User Story:** As a Visitor, I want every page in my language, so that I can read the site.

#### Acceptance Criteria

1. THE System SHALL provide every page in each language set in the Configuration_File.

### Requirement 15: R15: Accessibility

**User Story:** As a Visitor, I want the site to be accessible, so that I can use it with assistive technology.

#### Acceptance Criteria

1. THE System SHALL meet WCAG_2_2_AA.
2. THE System SHALL let an Organizer set a text description for each Photo.

### Requirement 16: R16: Roles

**User Story:** As an Organizer, I want to manage who can do what, so that the site is looked after by several people.

#### Acceptance Criteria

1. THE System SHALL let an Organizer add and remove other Organizers.
2. THE System SHALL let an Organizer add and remove Contributors.
3. THE System SHALL let only Organizers publish Photos.
4. THE System SHALL let only Organizers decide on a Removal_Request.

## Non-functional Requirements

### Requirement 17: N1: Serverless

**User Story:** As an Organizer, I want no servers to patch, so that the System is cheap to look after.

#### Acceptance Criteria

1. THE System SHALL run with no servers for the Group to patch.
2. THE System SHALL keep stored content in private buckets.
3. THE System SHALL serve stored content through the CDN.
4. THE System SHALL expose no public bucket.

### Requirement 18: N2: Cost

**User Story:** As an Organizer, I want low running cost, so that a small group can afford the System.

#### Acceptance Criteria

1. WHERE possible within the System's design, THE System SHALL stay inside the AWS free tier.
2. THE Guide SHALL state the expected monthly cost for a Group with 30 Events a year.

### Requirement 19: N3: Privacy

**User Story:** As a Visitor, I want no tracking, so that my visit stays private.

#### Acceptance Criteria

1. THE System SHALL set no tracking cookies.
2. THE System SHALL use no third-party analytics.
3. THE Guide SHALL state the retention period for unpublished Originals.
4. THE Configuration_File SHALL hold the retention period for unpublished Originals.

### Requirement 20: N4: Abuse

**User Story:** As an Organizer, I want limits on public actions, so that the System resists abuse.

#### Acceptance Criteria

1. THE System SHALL rate limit Removal_Requests.
2. THE System SHALL rate limit uploads.

### Requirement 21: N5: Quality

**User Story:** As an Organizer, I want automated checks and kept specifications, so that several people can change the System safely.

#### Acceptance Criteria

1. WHEN a change is made to the System, THE System SHALL run its automated tests.
2. THE System SHALL keep the specification and steering files in `.kiro/`.

### Requirement 22: N6: Documentation

**User Story:** As an Organizer of a new Group, I want a clear guide, so that I can install the System and publish my first event.

#### Acceptance Criteria

1. THE Guide SHALL let a new Group install the System in 30 minutes.
2. THE Guide SHALL let a new Group publish its first Event in 30 minutes.

### Requirement 23: N7: Licence

**User Story:** As a member of another Group, I want an open licence on the code, so that I can use and change it.

#### Acceptance Criteria

1. THE System SHALL be released under the MIT licence.

## Added requirements

These were added on 2026-11-07 from the owner's decisions. See Decisions.

### Requirement 24: R17: The wall

**User Story:** As a Visitor, I want to see the life of the group at a glance on the home page, so that I get a first impression without reading every evening.

#### Acceptance Criteria

1. THE System SHALL show the Wall on the home page as a continuous grid of Picks.
2. THE System SHALL order the Wall by Event, newest first, and inside an Event by Capture_Time.
3. WHEN an Organizer picks a Published_Photo, THE System SHALL include it in the Wall, up to the cap of Picks per Event set in the settings.
4. THE System SHALL show for each Year_Position at most the number of Picks set in the settings, and SHALL fold the rest of the Picks of that year into one Pile.
5. WHERE a year is after the last Year_Position set in the settings, THE System SHALL show that year as a Pile only.
6. THE System SHALL take the number of Picks for a year only from its Year_Position, and SHALL offer no number for a single calendar year.
7. WHEN a Visitor opens a Pile, THE System SHALL show the rest of the Picks of that year in place and SHALL offer a control that folds them back.
8. THE System SHALL keep the tiles at their size and let the Wall grow in length as far as needed.
9. WHERE an Event has no Pick, THE System SHALL show no tile for it on the Wall and SHALL keep the Event on the Events page with all its Published_Photos.

### Requirement 25: R18: Wall viewer

**User Story:** As a Visitor, I want to browse the picked photos large without leaving the home page, so that I can look at them in sequence.

#### Acceptance Criteria

1. WHEN a Visitor selects a tile of the Wall, THE System SHALL open the Wall_Viewer over the Wall without leaving the page.
2. THE System SHALL let the Visitor move through every Pick in Wall order, across years, including the Picks folded in Piles.
3. THE System SHALL offer in the Wall_Viewer a button that opens the evening page of the Photo, and SHALL offer no other action there.
4. THE System SHALL show in the Wall_Viewer the Photo, that button and the photographer's name in small type, and SHALL show no tags and no title line.
5. WHILE the Wall_Viewer is open, THE System SHALL add a marker to the address so that the back button of the browser closes it.
6. WHEN a Visitor loads an address that carries that marker, THE System SHALL open the Wall_Viewer on that Photo.
7. WHEN the Wall_Viewer is at the newest or the oldest Pick, THE System SHALL disable the control that has no Photo to go to.
8. THE System SHALL let a Visitor operate the Wall_Viewer with the keyboard.
9. WHERE the screen is a touch screen, THE System SHALL let a Visitor swipe sideways to move between Picks and swipe down to close the Wall_Viewer.

### Requirement 26: R19: Capture time

**User Story:** As an Organizer, I want photos in the right order even when a file has no time, so that the wall and the evening playback make sense.

#### Acceptance Criteria

1. THE System SHALL use the Capture_Time read from the file of a Photo.
2. IF a Photo has no Capture_Time, THEN THE System SHALL use the upload time and SHALL mark it as an estimate.
3. THE System SHALL let an Organizer change the Capture_Time of one Photo, which removes the estimate mark.
4. THE System SHALL let an Organizer move all the Capture_Times of one Event by a number of minutes.

### Requirement 27: R20: Photos and privacy page

**User Story:** As a Visitor, I want one short page that explains how photos are used, so that I know my rights and how to act.

#### Acceptance Criteria

1. THE System SHALL show a link to the photos and privacy page in the footer of every public page, including the home page.
2. THE System SHALL show on that page the Consent_Text, how to ask for a Photo to be removed, and the retention period for unpublished Originals.
3. WHERE a Group has turned on tagging, THE System SHALL show on that page how a person can tag themselves.

### Requirement 28: R21: Organizer wall page

**User Story:** As an Organizer, I want to choose what appears on the wall, so that the home page shows the best of the group.

#### Acceptance Criteria

1. THE System SHALL let an Organizer pick and unpick Published_Photos for the Wall.
2. THE System SHALL let an Organizer set the cap of Picks per Event.
3. THE System SHALL let an Organizer set the number of Picks for each Year_Position, and add or remove positions.
4. IF an Organizer tries to pick more Photos of one Event than the cap, THEN THE System SHALL refuse and SHALL say what the cap is.
5. THE System SHALL list the Events that have Published_Photos and no Pick.
6. THE System SHALL let an Organizer preview the Wall before leaving the page.

### Requirement 29: R22: Public site on a phone

**User Story:** As a Visitor on a phone, I want the site to work like an app, so that I can use it with one thumb.

#### Acceptance Criteria

1. WHEN the screen is 640 px wide or less, THE System SHALL show the main sections in a Tab_Bar fixed at the bottom of the screen.
2. WHEN the screen is 640 px wide or less, THE System SHALL show a slim top bar with the name of the Group that scrolls away with the page.
3. WHEN the screen is 640 px wide or less, THE System SHALL show the Wall with three tiles per row, edge to edge.
4. WHEN the screen is 640 px wide or less, THE System SHALL show the year bar as one row that scrolls sideways, with All fixed at its left, and SHALL keep the year bar at the top of the screen while the Wall scrolls.
5. WHILE All is chosen, THE System SHALL mark in the year bar the year the Visitor is looking at, and SHALL mark a chosen year filter in a different way.
6. WHEN a Visitor scrolls back up after going far down, THE System SHALL show a round back-to-top button above the Tab_Bar, and SHALL hide it when the Visitor scrolls down, stops, or is near the top.
7. WHEN a Visitor taps the Home tab while on the home page, THE System SHALL scroll to the top.
8. WHEN the screen is 640 px wide or less, THE System SHALL show a compact footer that keeps the photos and privacy link and the statement that the project is not affiliated with Amazon Web Services.
9. THE System SHALL NOT make any page wider than the screen.
10. WHERE the screen is a touch screen, THE System SHALL make controls at least 44 px high, SHALL use text of at least 16 px in fields, and SHALL NOT depend on hover for any action.
11. THE System SHALL keep controls out of the safe areas of the screen, such as the notch and the home bar.
12. WHEN only the height of the screen changes, THE System SHALL NOT rebuild the Wall.

### Requirement 30: R23: Organizer area on a phone

**User Story:** As an Organizer on a phone, I want the organizer area to work like an app, so that I can upload and approve photos at an event.

#### Acceptance Criteria

1. WHEN the screen is 760 px wide or less, THE System SHALL show the organizer sections Home, Photos, Wall, Requests and More in a Tab_Bar at the bottom of the screen.
2. THE System SHALL show on the Photos and Requests tabs the number of items that wait for an Organizer.
3. WHEN an Organizer opens More, THE System SHALL show a sheet above the Tab_Bar with Evenings, Speakers, Team, Settings, View the site and Sign out.
4. WHEN an Organizer taps outside the sheet, swipes it down, presses Escape or taps More again, THE System SHALL close the sheet.
5. WHEN the screen is 760 px wide or less, THE System SHALL show tables as cards, each value with the name of its column, and the main actions as full-width buttons.
6. WHEN the screen is 760 px wide or less, THE System SHALL show the Photos of an Event one per card, with the picture, the state, the actions and the fields.
7. WHERE the screen is a touch screen, THE System SHALL offer Choose photos and Take a photo instead of a drop area.
8. THE System SHALL accept the same file types and the same size limit when photos are chosen, taken or dropped, and SHALL report the reason for a refused file.

### Requirement 31: R24: Installable on a phone

**User Story:** As a Visitor or an Organizer, I want to add the site to my home screen, so that it opens like an app.

#### Acceptance Criteria

1. THE System SHALL provide a web app manifest and icons so that the site can be added to the home screen.
2. THE System SHALL take the name, the short name and the colours of the manifest from the Configuration_File.
3. WHEN the site is opened from the home screen, THE System SHALL open at the home page without browser bars.

### Requirement 32: N8: Wall delivery

**User Story:** As an Organizer, I want the wall to be fast and cheap to serve, so that a small group can afford it.

#### Acceptance Criteria

1. THE System SHALL read the Wall from one prebuilt file served by the CDN with a cache of about one minute.
2. WHEN a Photo is published, picked, unpicked, hidden or removed, THE System SHALL rewrite that file.
3. WHEN a Photo is hidden or removed, THE System SHALL clear the cache of that file.
4. THE System SHALL make no call to the API for each visit to the Wall and SHALL NOT rebuild the whole site for a change of the Wall.
5. THE System SHALL make the tile size and the viewer size once, at upload, and SHALL NOT resize images on the fly.

## Out of scope for version 1

- Face recognition.
- Comments.
- A shared service for several groups.
- A mobile app.

## Decisions

Decided by the owner on 2026-11-06 (items 1 to 7) and 2026-11-07 (item 8). The agent gave options and a recommendation. The owner chose. Changing one needs the owner's approval. The design says what each would cost to change.

1. **Sign-in for organizers:** Cognito.
2. **Infrastructure tool:** SAM (AWS Serverless Application Model). The owner chose it over CDK for a lower barrier for contributors.
3. **Default licence for photos:** all rights reserved by the photographer. `config/group.example.yml` keeps `all-rights-reserved`, so the most cautious choice applies to every new group. A group may set CC BY-NC 4.0 in its own configuration.
4. **Domain:** the CDN address at first. A group domain can be added later.
5. **Owner:** a personal account at first. The agent recommended an organization with the other organizers from the start. The owner chose otherwise. Moving later is a repository transfer.
6. **Later modules:** not in version 1.

7. **The wall (decided 2026-11-06).** Decisions D1 to D21 are recorded in `docs/wall-decisions.md`: a curated home page grid, picks with a cap per evening, shares by year position, piles, an overlay viewer with an address marker, a prebuilt wall file with a short cache, two image sizes made at upload, capture time with an editable estimate, a short photos and privacy page, three tiles per row on a phone. They are now requirements R17 to R21, N8 and the additions R9.3, R10.6, R11.5, R13.5. The owner accepted these risks: a photo can be seen on the home page with no removal action next to it (the path is viewer, evening page, removal request, with one footer link as signpost, not checked with a lawyer); the 150-a-year stress test makes a long wall; a removal reaches visitors in seconds, not instantly.
8. **The phone experience (decided 2026-11-07).** On a phone the public site and the organizer area work like an app: a Tab_Bar at the bottom, a slim top bar that scrolls away, a wall edge to edge, a year bar with All fixed, a back-to-top button that appears when scrolling up, a compact footer, a swipe viewer, cards instead of tables, choose-or-camera photo upload, no sideways scroll, 44 px touch targets, safe areas, and a manifest so the site can be added to the home screen. They are requirements R22 to R24.
9. **The preview is a design reference.** `docs/preview/` is a static prototype built by `docs/preview/build.py` with sample content. It shows the agreed behaviour. It is not production code and its content is not the source of the requirements. When the two differ, the requirements win and the difference is listed as a gap.

## Open decisions

These are not decided. The agent has not decided them. Each has a gap number in section 10 of [high-level-design.md](high-level-design.md). Options and a recommendation are given when the owner asks for them.

Items 1 to 8 have no working answer. Do not build on them: code waits behind an interface or a placeholder.

Items 9 to 20 carry a working assumption. The design and the tasks use it so work can go on. It is not a decision. The owner confirms it or changes it, and then it moves to Decisions.

1. **How the wall shares photos between years.** A number of picks for each year position (decisions D3 and D15), or a percentage share of a wall height chosen in screens (the preview). Gap 21 and 27.
2. **Tag yourself.** Whether it is version 2 or later, and what privacy review it needs. It is not built until decided. The preview shows it with a "version 2" mark. Gap 22.
3. **Face recognition and tagging opt-in.** R12.3 lets a group turn it on, and the out of scope list names face recognition. Gap 1.
4. **Limits.** Maximum file size, files per upload, and the rate limit values for uploads and removal requests. The preview uses 25 MB as a placeholder. Gaps 7 and 26.
5. **What happens to a photo after a removal decision.** Whether an approved request deletes the original at once, whether a rejected request restores the photo, what happens at the review deadline, and what happens at the end of retention. Gaps 2 to 5.
6. **Sign-in details with Cognito.** A code or a link by email, and how sign out works. Gap 23.
7. **Licence at upload.** Whether contributors accept the licence when they upload, and the wording about Creative Commons not being recalled. Gaps 19 and 20.
8. **Small values from the preview.** One width for the phone layout instead of 640 px and 760 px, the time after which the back-to-top button hides, and where the talk proposal address lives in the configuration. Gaps 24, 25 and 28.

9. **Location data in the original.** Gap 6. Working assumption: the original is kept as uploaded, with its location data, and stays private. Only the tile size and the viewer size lose it.
10. **How an Organizer gets an original.** Gap 8. Working assumption: a signed download for Organizers only.
11. **Cover photo.** Gap 9. Working assumption: an Organizer sets it; otherwise the first published photo; otherwise a plain card.
12. **How a Visitor switches language.** Gap 10. Working assumption: plain links between the page sets of each language.
13. **Who manages events.** Gap 11. Working assumption: only Organizers create, edit and delete events. The import file is a JSON list of events and each bad item is reported with its position.
14. **The last Organizer.** Gap 12. Working assumption: removing the last Organizer is refused.
15. **Tools, CI and cost wording.** Gap 13. Working assumption: TypeScript on Node, Vitest, fast-check, Playwright, GitHub Actions for CI, and the `sharp` library in a container image. N2.1 says "where possible", which cannot be tested; the guide states the measured cost instead.
16. **Install time target.** Gap 14. R2 says "a short afternoon" and N6 says 30 minutes. Working assumption: 30 minutes is the target.
17. **Contact of the person who asks for a removal.** Gap 15. Working assumption: none is collected, so the outcome is not sent to them.
18. **Test for a second group.** Gap 16. Working assumption: a second sample configuration runs through every build and test (R1.9), and that is the test.
19. **Format of the event export.** Gap 17. Working assumption: a standalone page at a stable address, with links to the public copies of the photos.
20. **How Organizers preview photos that are not published.** Gap 18. Working assumption: from the `staging/` prefix through signed addresses.
