# MicroBlox Project Specification for PubNub Volunteers

## Overview

MicroBlox is a lightweight, chat-native collaboration application for drafting, revising, reviewing, and approving IATI XML files before those files are published publicly as open aid data.[cite:66][web:32][web:136] The demonstration version should feel like a simple instant messaging app with a built-in structured editor, while the underlying system records XML-related collaboration as append-only ledger events that can be replayed into a current-state XML view.[cite:64][web:124][web:208]

The application is intended for humanitarian organization staff who need to collaborate privately on IATI activity data across users, teams, departments, organizations, and eventually partner organizations before consensus is reached to publish a final public XML file.[cite:66][web:76] PubNub is the real-time messaging backbone for the demo, providing channels, permissions, and presence features that support cross-device collaboration on phones and desktops.[web:57][web:76][web:203]

## Project goal

The goal of the demo is to prove that a messaging application can support structured, permission-aware collaborative authoring of IATI XML in a way that is easier to understand and coordinate than passing full XML files around manually.[cite:64][cite:66] The demo does not need to implement the entire IATI standard or production-grade blockchain infrastructure; it needs to demonstrate the workflow, interaction model, ledger logic, and progressive visibility model clearly enough that the architecture is understandable and credible.[web:28][web:124][cite:196]

## Core concept

MicroBlox treats small units of XML collaboration as ledger-backed blocks or events that are exchanged through messaging.[cite:64] Each user maintains a local or user-scoped ledger populated only by blocks they authored themselves or blocks they received from others.[cite:64]

The user interface should not expose the ledger as the primary interaction model. Instead, it should present:

- a familiar chat experience,
- a current-state XML editor or viewer,
- a human-readable Markdown or readable projection of the same current state,
- and a list of XML files or activities the user is editing, reviewing, or following.[cite:64][cite:66][cite:201]

Behind the scenes, the current visible XML state should be calculated from the append-only stream of accepted or active events, not from in-place mutation of a single stored XML file.[web:124][web:205][web:208]

## Why IATI matters

IATI is both an XML standard and an open data publishing framework used by organizations to report development and humanitarian aid activities in structured detail.[web:32][web:28] An IATI activity file is an XML file containing one or more `iati-activity` records and must be well-formed XML.[web:30][web:207]

In this project, public publication of an IATI file is the end-state of a private workflow.[cite:66] MicroBlox is intended to support the work that happens before publication: drafting, discussing, revising, reviewing, approving, and gradually expanding visibility until a file is ready to become public.[cite:66][web:136]

## Users and roles

The demo should assume several user types, even if the implementation uses simplified mock roles:[cite:66]

- **Author**: creates a draft activity or adds new XML lines or fragments.
- **Collaborator**: receives edit requests and proposes revisions.
- **Reviewer**: checks content and may accept or reject proposed changes.
- **Approver**: has authority to move a draft to a wider visibility or published-ready state.
- **Follower**: can view selected files or fragments but may not edit them.

The UI should make it clear who can see, edit, review, or approve a draft item at a given time.[web:76][web:77]

## Product experience

The product should be designed as a minimal and lightweight cross-device app with the following experience goals:

- Works well on desktop and mobile.[web:117][web:57]
- Feels like a chat app first, not a heavy enterprise workflow tool.[web:117]
- Lets users coordinate work through ordinary conversation and structured collaboration requests.[web:118]
- Makes it easy to jump from a chat thread to the exact XML item under discussion.[web:118]
- Shows the latest visible state of an XML draft in either raw XML mode or a human-readable mode.[cite:201][web:124]

## Main workflow

A typical workflow in the demo should be:

1. A user opens or creates an IATI draft activity.[cite:66]
2. The user sends ordinary chat messages to coordinate work with another user or group.[web:57][web:117]
3. The user creates a new XML line or fragment, or flags an existing one for review, and sends it as a structured collaboration event through the messaging layer.[cite:64][web:118]
4. Another user receives the message, opens the related item in the editor, and views it in XML or readable mode.[cite:201][web:118]
5. That user proposes a revision by sending a new block or event that supersedes the earlier one for the same logical line or fragment.[cite:64][web:124]
6. Another collaborator explicitly accepts or rejects the revision; receipt alone does not count as acceptance.[web:186][cite:64]
7. Once the required approval rule is satisfied, the revised line or fragment becomes active in the current-state projection.[web:124][web:208]
8. Visibility may then be expanded to a wider audience through permissions changes.[cite:66][web:76]
9. Once enough consensus is reached, the approved current state can be exported as an IATI XML draft ready for publication.[web:136][web:30]

## Product principles

The implementation should follow these principles:

- **Chat-first**: conversation is a first-class part of the workflow.[web:117]
- **Append-only**: XML collaboration state is stored as events or blocks, not overwritten in place.[web:124][web:205]
- **Projected current state**: the visible XML is a read model built from events.[web:124][web:208]
- **Explicit approval**: receiving a revision is not the same as accepting it.[web:186]
- **Selective visibility**: users only see blocks they authored or received, plus any shared metadata allowed by the design.[cite:64][web:151]
- **Progressive disclosure**: a draft may move from individual visibility to wider team, department, organization, and cross-organization visibility over time.[cite:66][web:76]
- **Public IATI output**: the live editing environment is private; publication is the eventual end-state.[web:136][cite:66]

## Functional requirements

The demo should include the following capabilities:

### Messaging

- One-to-one chat.[web:76]
- Small group chat or activity-specific channels.[web:109]
- Timestamped message threads.
- Optional message status or presence indicators if easy to implement.[web:203][web:107]
- Ability to flag a message as related to a specific XML item using message actions or similar metadata.[web:118]

### Files and activities

- A list of files or activities the user is working on or following.[cite:66]
- A visible status for each file, such as draft, in review, approved, or ready for publication.[web:124][cite:66]
- The ability to open a file and see its current visible state.

### Editor

- XML mode showing the current-state XML view.
- Readable mode or Markdown projection that presents the same content in a more human-friendly format.[cite:201]
- Ability to click from chat into the relevant line or fragment in the editor.[web:118]
- Ability to propose a new line or revision.
- Ability to accept, reject, or supersede a proposal.[web:186][web:124]

### Ledger behavior

- All XML collaboration actions should generate append-only events or blocks.[web:124]
- Prior events remain in history even when superseded.[web:124][web:205]
- Current state should reflect the latest accepted or active version for each addressed line or fragment.[cite:64][web:208]

### Export

- Ability to export the current approved draft to XML, even if using a simplified IATI subset.[web:30][web:136]

## Non-functional requirements

The demo should also satisfy these qualities:

- Lightweight and responsive on desktop and mobile.[web:117]
- Minimal enough for a volunteer-built demo, but polished enough to communicate the concept clearly.[cite:196]
- Easy for non-engineers to understand visually.
- Real-time feel for both chat and editing workflows.[web:57]
- Clear permission boundaries, even if identity and auth are simplified.[web:76][web:204]

## UI and screens

The demo should at minimum include the following views.

### 1. Chat / conversation list

This screen should show:

- direct chats,
- group chats,
- activity channels,
- unread indicators,
- and optionally who is online.[web:203][web:107]

### 2. Conversation view

This should be the main working surface for collaboration. It should show:

- ordinary chat messages,
- flagged messages requesting help with XML,
- structured XML-related messages or chips,
- quick actions to open the editor on the referenced item.[web:118]

### 3. File or activity list

This should show XML drafts the user is following, editing, or reviewing, including a simple workflow status.[cite:66]

### 4. Editor view

This should allow the user to switch between:

- raw XML view,
- readable/Markdown view.[cite:201]

The editor should focus on the currently selected line or fragment and allow proposing changes.

### 5. Review and approval state

The UI should expose whether a change is:

- proposed,
- pending review,
- accepted,
- rejected,
- superseded,
- or active.[web:124][web:186]

### 6. Visibility scope indicator

The app should display who currently has access to the draft or fragment, such as:

- private,
- team,
- department,
- organization,
- partner organization.[cite:66][web:76]

## Information architecture

A practical desktop layout could be:

- left sidebar: chats and activities,
- center pane: conversation thread,
- right pane: editor or review pane.

A practical mobile layout could use bottom navigation tabs such as:

- Chats
- Files
- Editor
- Review

The interface should stay intentionally simple. The goal is to demonstrate the workflow, not to reproduce the complexity of enterprise knowledge management software.

## Data and event model

The demo should use a unified message envelope and a small set of event types.

### Unified message envelope

All messages sent through PubNub should have a common structure such as:

```json
{
  "message_id": "uuid",
  "channel_id": "activity:GB-CHC-202918-JORA82",
  "message_class": "chat_message | xml_fragment_event",
  "sender_id": "user:alice",
  "recipient_scope": ["user:bob", "team:reporting"],
  "timestamp": "2026-05-16T21:58:00Z",
  "payload": {}
}
```

This allows normal chat and structured collaboration events to coexist in one messaging system.[web:57][web:76]

### Recommended XML collaboration event types

The demo should support a small event vocabulary, for example:

- `chat_message`
- `edit_request`
- `fragment_proposal`
- `fragment_acceptance`
- `fragment_rejection`
- `fragment_supersession`
- `visibility_change`
- `publish_event`.[cite:64][web:118][web:124]

### Minimum event fields

A structured XML event should include fields such as:

- `block_id`
- `activity_id`
- `fragment_id` or `line_id`
- `event_type`
- `sender_id`
- `recipient_scope`
- `sequence`
- `xpath`
- `content`
- `canonical_xml`
- `json_repr`
- `supersedes_block_id`
- `proposal_id`
- `status`
- `timestamp`
- `signature` or placeholder signature field.[web:91][web:102][cite:64]

### Important rule

A new revision should not erase the old one. The new event should supersede the earlier event for the same logical line or fragment, while the earlier event remains in history.[web:124][web:205]

## Current-state projection model

The visible XML file should be built as a projection or read model from the append-only event stream.[web:124][web:208] The projection logic should:

1. group events by addressed line or fragment,
2. order them by sequence or trusted timestamp,
3. determine which event is currently active,
4. ignore superseded or rejected events in the rendered current-state view,
5. render the latest accepted state in XML order.[web:124][web:205][web:208]

This current-state view can be cached for performance, but conceptually it is always derived from the ledger rather than being the canonical source of truth.[web:205][web:208]

## Consensus and approval model

MicroBlox should use workflow-level consensus, not infrastructure-level distributed consensus.[web:186][web:187] In practice, this means:

- one user proposes a revision,
- another user explicitly accepts or rejects it,
- acceptance is a distinct event and should not be inferred from message receipt alone.[web:186]

For the demo, a line or fragment can become active when one explicit acceptance is recorded, or when a configurable threshold is met if the team wants to demonstrate multi-party review.[web:183][web:185]

A clear acceptance pattern would be:

- `fragment_proposal`: here is a revised line or fragment,
- `fragment_acceptance`: I approve proposal X,
- `fragment_rejection`: I reject proposal X,
- `fragment_supersession`: proposal Y replaces proposal X.[web:185][web:186]

## Permissions and visibility model

Permissions are central to the concept. The application should support the idea that a draft becomes visible in stages over time.[cite:66][web:76]

A suggested visibility ladder is:

- individual,
- team,
- department,
- organization,
- partner organization,
- public publication outside the drafting environment.[cite:66][web:76]

The demo should make clear that:

- users only see fragments they are permitted to receive,[cite:64][web:151]
- the same draft may have different current visible projections for different users,[web:151][web:153]
- visibility changes are themselves meaningful events in the workflow.[cite:66][web:124]

## PubNub-specific implementation guidance

The volunteers should map the architecture onto PubNub as follows:

- Use PubNub channels for direct chats, group chats, and activity-specific collaboration streams.[web:109][web:76]
- Use Access Manager tokens and permissions to restrict which users can read or write which channels.[web:76][web:204]
- Use presence to indicate online state and channel participation if helpful.[web:203][web:107]
- Use message actions or metadata to flag chat items that reference XML fragments needing collaboration.[web:118]

A practical channel strategy might include:

- direct channels for one-to-one conversation,
- group channels for teams,
- activity channels for file-specific collaboration,
- optional restricted channels for sensitive fragments.[web:76][web:109]

## XML and readable-mode rendering

The canonical model should be the structured event stream plus the current-state projection.[web:124][cite:201] Markdown or readable mode should be treated as a usability layer, not as the source of truth.[cite:201]

This means:

- XML view shows the rendered current state,
- readable/Markdown view shows a human-friendly projection of the same state,
- edits from readable mode must still be turned back into structured events before they affect the ledger.[cite:201][web:124]

The team should avoid designing a system where free-form Markdown is the primary editable source and XML is reconstructed from it without a structured model.[cite:201]

## Recommended IATI subset for the demo

To keep the prototype manageable, the demo should support only a narrow subset of IATI activity content.[web:25][web:28] A good subset would be:

- `iati-identifier`
- `reporting-org`
- `title`
- `description`
- `participating-org`
- `activity-status`
- `activity-date`
- `recipient-country`.[web:25][web:30]

This is enough to demonstrate real structured XML collaboration without attempting the full standard.[web:28]

## Example IATI snippet for the demo

A useful baseline snippet is:

```xml
<iati-activity>
  <iati-identifier>GB-CHC-202918-JORA82</iati-identifier>
  <reporting-org ref="GB-CHC-202918" type="21">
    <narrative xml:lang="EN">Oxfam GB</narrative>
  </reporting-org>
  <title>
    <narrative xml:lang="EN">Solid Waste Management for Za'atari Camp and surrounding host communities</narrative>
  </title>
  <participating-org role="1" type="21">
    <narrative xml:lang="EN">Oxfam GB</narrative>
  </participating-org>
</iati-activity>
```

This gives the team something concrete to model in the demo.[web:25]

## Example readable projection

A readable projection of the same content might look like:

```md
# Activity

- Identifier: GB-CHC-202918-JORA82
- Reporting organization: Oxfam GB
- Title: Solid Waste Management for Za'atari Camp and surrounding host communities

## Participating organisations

- Role 1, Type 21: Oxfam GB
```

This should be generated from the same structured current state, not maintained separately.[cite:201]

## Example event payloads

### Chat message

```json
{
  "message_id": "m1",
  "message_class": "chat_message",
  "sender_id": "user:a",
  "recipient_scope": ["user:b"],
  "timestamp": "2026-05-16T18:00:00Z",
  "payload": {
    "text": "Can you review the participating-org line?"
  }
}
```

### Edit request

```json
{
  "message_id": "m2",
  "message_class": "xml_fragment_event",
  "sender_id": "user:a",
  "recipient_scope": ["user:b"],
  "timestamp": "2026-05-16T18:01:00Z",
  "payload": {
    "event_type": "edit_request",
    "activity_id": "GB-CHC-202918-JORA82",
    "fragment_id": "participating-org-01",
    "xpath": "/iati-activities/iati-activity/participating-org[1]",
    "note": "Please revise this line"
  }
}
```

### Fragment proposal

```json
{
  "message_id": "m3",
  "message_class": "xml_fragment_event",
  "sender_id": "user:b",
  "recipient_scope": ["user:a"],
  "timestamp": "2026-05-16T18:05:00Z",
  "payload": {
    "event_type": "fragment_proposal",
    "proposal_id": "p2",
    "activity_id": "GB-CHC-202918-JORA82",
    "fragment_id": "participating-org-01",
    "supersedes_block_id": "b1",
    "canonical_xml": "<participating-org role=\"3\" type=\"21\"><narrative xml:lang=\"EN\">Oxfam GB</narrative></participating-org>",
    "json_repr": {
      "element": "participating-org",
      "attributes": {"role": "3", "type": "21"},
      "children": [
        {"element": "narrative", "attributes": {"xml:lang": "EN"}, "text": "Oxfam GB"}
      ]
    },
    "status": "proposed"
  }
}
```

### Fragment acceptance

```json
{
  "message_id": "m4",
  "message_class": "xml_fragment_event",
  "sender_id": "user:a",
  "recipient_scope": ["user:b"],
  "timestamp": "2026-05-16T18:07:00Z",
  "payload": {
    "event_type": "fragment_acceptance",
    "proposal_id": "p2",
    "fragment_id": "participating-org-01",
    "status": "accepted"
  }
}
```

## Demo scope

The demonstration should stay intentionally narrow. In scope:

- a small set of mock users,
- a simplified IATI activity subset,
- chat plus XML collaboration,
- XML and readable-mode projections,
- append-only event handling,
- explicit acceptance,
- staged visibility changes.[cite:196][cite:66]

Out of scope:

- full IATI implementation,
- production-grade cryptography,
- real blockchain consensus,
- enterprise deployment,
- comprehensive schema validation,
- robust offline sync across all edge cases.[web:124][cite:196]

## Success criteria

The demo should be considered successful if it can show the following end-to-end scenario:

- two or more users can chat in real time,[web:57]
- one user can flag or request collaboration on a specific XML item,[web:118]
- another user can open that item in the editor and propose a revision,
- the revision is recorded as a new event,
- another user explicitly accepts the revision,[web:186]
- the current-state XML view updates to show the accepted latest version,[web:124][web:208]
- the file's visibility can be expanded to a wider audience,[cite:66][web:76]
- the resulting state can be exported as XML.[web:30][web:136]

## Open implementation choices

The volunteers should have freedom to decide certain details as long as the core concept is preserved. Examples include:

- whether the ledger is stored entirely client-side, lightly server-assisted, or mocked,
- whether revisions operate at a strict line level or slightly larger fragment level,
- whether acceptance rules are one-approver or multi-approver,
- how much of signing and hashing is real versus demonstrative,
- how the readable view is styled and how permissions are visualized.

## Guidance for AI coding assistants

If this specification is provided to an AI coding assistant, the assistant should be told to optimize for:

- simplicity of the demo,
- clarity of the workflow,
- chat-first UX,
- event-driven architecture,
- append-only ledger behavior,
- client-friendly current-state projections,
- and a modular codebase that cleanly separates transport, event logic, projection logic, and UI rendering.[web:124][web:205]

The assistant should avoid designing the system as a generic text editor with a chat sidebar. The intended architecture is the reverse: a messaging application that uses structured events and projections to produce collaborative XML editing behavior.[cite:64][cite:66]

## Final note

The most important idea for the demo is that users should feel they are collaborating in a lightweight messaging app, while the system quietly maintains a structured audit trail of XML collaboration events that can be projected into the current visible state of an IATI draft.[cite:64][web:124] That combination of chat, permissions, event history, and structured export is the core of MicroBlox.[cite:66][web:136]
