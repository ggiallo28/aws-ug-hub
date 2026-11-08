# Design Document

## Overview

The design lives in [high-level-design.md](../../steering/high-level-design.md) #[[file:.kiro/steering/high-level-design.md]]. That file is the single place for it. This spec adds detail only. It repeats nothing from that file.

Requirements R1 to R24 and N1 to N8 are in the [steering requirements](../../steering/requirements.md). Criterion IDs such as R10.4 are the requirement ID plus the criterion number.

## Architecture

### Architecture on AWS

Steering: [section 1, Architecture on AWS](../../steering/high-level-design.md#1-architecture-on-aws).

Covers N1.1 to N1.4, N2.1, R6.4, R7.5.

Details to be added here when a task needs them.

### Data model

Steering: [section 2, Data model](../../steering/high-level-design.md#2-data-model).

Covers R3.1, R3.2, R4.1, R4.2, R8.1, R10.2, R11.1, R11.3, R11.4, R16.1, R16.2, R17.3, R19.2, R21.2.

Details to be added here when a task needs them.

### How each requirement is met

Steering: [section 3, How each requirement is met](../../steering/high-level-design.md#3-how-each-requirement-is-met).

Covers R1.1 to R24.3 and N1.1 to N8.5.

Details to be added here when a task needs them.

### The wall

Steering: [section 11, The wall and the phone experience](../../steering/high-level-design.md#11-the-wall-and-the-phone-experience) and [section 2, Data model](../../steering/high-level-design.md#2-data-model).

Covers R17.1 to R17.9, R18.1 to R18.9, R19.1 to R19.4, R20.1 to R20.3, R21.1 to R21.6, N8.1 to N8.5.

Details to be added here when a task needs them.

### The phone experience

Steering: [section 11, The wall and the phone experience](../../steering/high-level-design.md#11-the-wall-and-the-phone-experience).

Covers R22.1 to R22.12, R23.1 to R23.8, R24.1 to R24.3.

Details to be added here when a task needs them.

### Photo pipeline

Steering: [section 4, Photo pipeline](../../steering/high-level-design.md#4-photo-pipeline-upload-to-publication-to-removal).

Covers R6.1 to R6.5, R7.1 to R7.5, R8.1, R11.4, R15.2, R19.2, N3.3, N3.4, N4.2, N8.5.

Details to be added here when a task needs them.

### Removal request flow and audit log

Steering: [section 5, Removal request flow and audit log](../../steering/high-level-design.md#5-removal-request-flow-and-audit-log).

Covers R10.1 to R10.6, R16.4, N4.1, N8.3.

Details to be added here when a task needs them.

### Configuration

Steering: [section 6, Configuration](../../steering/high-level-design.md#6-how-the-configuration-file-drives-the-site).

Covers R1.1 to R1.9, R9.2, R14.1, R24.2, N3.4.

Details to be added here when a task needs them.

### Install and removal

Steering: [section 7, Install and removal](../../steering/high-level-design.md#7-install-and-removal-for-a-new-group).

Covers R2.1 to R2.3, N6.1, N6.2.

Details to be added here when a task needs them.

### Decisions

Steering: [section 8, Decisions and what they cost to change](../../steering/high-level-design.md#8-decisions-and-what-they-cost-to-change).

Decisions 1 to 9 are made. They are Cognito, SAM, all rights reserved as the default licence, the CDN address first, a personal account first, no later modules in version 1, the wall, the phone experience, and the preview as a design reference. They are recorded in the steering requirements and in section 8. This spec follows them and repeats no detail.

Twenty points stay open. They are listed in [Open decisions](../../steering/requirements.md#open-decisions) in the steering requirements. Items 1 to 8 have no working answer, and this spec does not rely on them. Items 9 to 20 carry a working assumption that this spec uses until the owner decides.

### Risks

Steering: [section 9, Risks](../../steering/high-level-design.md#9-risks).

Covers R10.1, R15.1, N2.1, N4.1.

Details to be added here when a task needs them.

### Gaps

Steering: [section 10, Gaps and ambiguities](../../steering/high-level-design.md#10-gaps-and-ambiguities-in-the-requirements).

Covers R10.5, R11.3, R9.1, R12.3, R17.4, R21.3, R22.6, R6.3, N2.1, N5.1.

Gaps 19 and 20 were added to the steering list. Gap 19 is contributor licence acceptance at upload (R11.3). Gap 20 is that the consent text should say a Creative Commons licence cannot be recalled (R9.1).

Gaps 21 to 28 were added after that. They are: wall numbers or shares (R17.4, R21.3), tag yourself (R12.3, open decision 2), sign-in link or code, the talk address from configuration, breakpoints, the upload limit placeholder (R6.3), the wall height, and the back-to-top timing (R22.6).

Every gap is now an open decision in the steering requirements.

Details to be added here when a task needs them. The requirements are not changed by this spec.

## Components and Interfaces

Components and their interfaces are listed in [section 1](../../steering/high-level-design.md#1-architecture-on-aws). Interface detail will be added here when a task needs it.

## Data Models

Data models are in [section 2](../../steering/high-level-design.md#2-data-model). Detail will be added here when a task needs it.

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system. Properties are the bridge between the requirements and automated checks.*

These follow directly from the high-level design. No new requirements are added.

### Property 1: Unpublished photos are not reachable

For any photo that is not published, no CDN path serves its thumbnail, web size or original.

**Validates: Requirements 8.1, 17.4** (R8.1, N1.4)

### Property 2: Pending-removal photos are not reachable

For any photo in the state hidden pending removal, no CDN path serves it and no public page lists it, until an Organizer decides.

**Validates: Requirements 10.1, 10.4** (R10.1, R10.4)

### Property 3: Audit table is append only

For any function in the System, there is no update or delete path to the audit table. See [section 5](../../steering/high-level-design.md#5-removal-request-flow-and-audit-log).

**Validates: Requirements 10.2** (R10.2)

### Property 4: No group name in the code

For any source file in the System, the file contains no name of a specific Group.

**Validates: Requirements 1.9** (R1.9)

### Property 5: Hidden or removed photos leave the wall

For any photo that becomes hidden or removed, the wall file does not list it, and no cached wall copy lists it once the cache time has passed.

**Validates: Requirements 10.6, 32.3** (R10.6, N8.3)

### Property 6: The wall file lists only published, picked photos within the cap

For any set of events and photos, the wall file lists only photos that are published and picked, and never more than the cap of picks per event.

**Validates: Requirements 24.3, 28.4** (R17.3, R21.4)

### Property 7: No public page is wider than the screen

For any public page, the page is not wider than 390 px, so there is no sideways scroll.

**Validates: Requirements 29.9** (R22.9)

### Property 8: Viewer controls stop at the ends

For any wall with at least one pick, the viewer's previous control is disabled at the first pick and its next control is disabled at the last pick.

**Validates: Requirements 25.7** (R18.7)

## Error Handling

Failure paths are described in [section 4](../../steering/high-level-design.md#4-photo-pipeline-upload-to-publication-to-removal) and [section 9](../../steering/high-level-design.md#9-risks). Detail will be added here when a task needs it.

The install flow uses SAM. It must be timed on an empty account against N6.1 and N6.2.

## Testing Strategy

Tests run on every change (N5.1). Each property above is a property-based test with at least 100 iterations, tagged as Feature: aws-ug-hub, Property N. Infrastructure and page rendering use example, snapshot and smoke tests instead. The sideways overflow test runs every page at 390 px. The design reference is `docs/preview`. Detail will be added here when a task needs it.
