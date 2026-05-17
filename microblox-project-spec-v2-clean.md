# MicroBlox Project Specification for PubNub Volunteers and AI Coding Assistants

## Overview

MicroBlox is a lightweight, chat-native collaboration application for drafting, revising, reviewing, and approving IATI XML files before those files are published publicly as open aid data. The demonstration version should feel like a simple instant messaging app with a built-in structured editor, while the underlying system records XML-related collaboration as append-only ledger events that can be replayed into a current-state XML view.

The application is intended for humanitarian organization staff who need to collaborate privately on IATI activity data across users, teams, departments, organizations, and eventually partner organizations before consensus is reached to publish a final public XML file. PubNub is the real-time messaging backbone for the demo, providing channels, permissions, and presence features that support cross-device collaboration on phones and desktops.

This document is designed to be detailed enough that PubNub volunteers can use it directly and also detailed enough that they can pass it to AI coding assistants as project context, implementation guidance, and architecture intent.

## Project goal

The goal of the demo is to prove that a messaging application can support structured, permission-aware collaborative authoring of IATI XML in a way that is easier to understand and coordinate than passing full XML files around manually. The demo does not need to implement the entire IATI standard or production-grade blockchain infrastructure; it needs to demonstrate the workflow, interaction model, ledger logic, progressive visibility model, and export path clearly enough that the architecture is understandable and credible.

## Core concept

MicroBlox treats small units of XML collaboration as ledger-backed blocks or events that are exchanged through messaging. Each user maintains a local or user-scoped ledger populated only by blocks they authored themselves or blocks they received from others.

The user interface should not expose the ledger as the primary interaction model. Instead, it should present:

- a familiar chat experience,
- a current-state XML editor or viewer,
- a human-readable Markdown or readable projection of the same current state,
- and a list of XML files or activities the user is editing, reviewing, or following.

Behind the scenes, the current visible XML state should be calculated from the append-only stream of accepted or active events, not from in-place mutation of a single stored XML file.

## Why IATI matters

IATI is both an XML standard and an open data publishing framework used by organizations to report development and humanitarian aid activities in structured detail. An IATI activity file uses `iati-activities` as its root element and may contain one or more `iati-activity` records inside it.

In this project, public publication of an IATI file is the end-state of a private workflow. MicroBlox is intended to support the work that happens before publication: drafting, discussing, revising, reviewing, approving, and gradually expanding visibility until a file is ready to become public.

## Product vision

MicroBlox should be understood as more than a chat app with code pasted into messages. It is a structured collaboration environment for humanitarian reporting work that combines:

- messaging,
- structured XML editing,
- append-only provenance,
- progressive permissions,
- approval workflows,
- and eventual public XML publication.

The demonstration should focus on proving that this workflow is viable in a lightweight messaging environment. The broader long-term vision is that the same workflow could later support stronger search, corpus-scale ledger organization, and AI-assisted curation of IATI data.

## Users and roles

The demo should assume several user types, even if the implementation uses simplified mock roles:

- **Author**: creates a draft activity or adds new XML lines or fragments.
- **Collaborator**: receives edit requests and proposes revisions.
- **Reviewer**: checks content and may accept or reject proposed changes.
- **Approver**: has authority to move a draft to a wider visibility or published-ready state.
- **Follower**: can view selected files or fragments but may not edit them.
- **Future AI collaborator**: not required in the first demo, but the architecture should leave room for an AI agent to later behave like a proposing or reviewing participant under human oversight.

The UI should make it clear who can see, edit, review, or approve a draft item at a given time.

## Product experience

The product should be designed as a minimal and lightweight cross-device app with the following experience goals:

- Works well on desktop and mobile.
- Feels like a chat app first, not a heavy enterprise workflow tool.
- Lets users coordinate work through ordinary conversation and structured collaboration requests.
- Makes it easy to jump from a chat thread to the exact XML item under discussion.
- Shows the latest visible state of an XML draft in either raw XML mode or a human-readable mode.
- Makes it clear which changes are proposed, accepted, rejected, or pending review.

## Main workflow

A typical workflow in the demo should be:

1. A user opens or creates an IATI draft activity.
2. The user may upload a whole XML file, upload a fragment, or start from scratch.
3. If XML is uploaded, the system parses and canonicalizes it before creating blocks or events from it.
4. The user sends ordinary chat messages to coordinate work with another user or group.
5. The user creates a new XML line or fragment, or flags an existing one for review, and sends it as a structured collaboration event through the messaging layer.
6. Another user receives the message, opens the related item in the editor, and views it in XML or readable mode.
7. That user proposes a revision by sending a new block or event that supersedes the earlier one for the same logical line or fragment.
8. Another collaborator explicitly accepts or rejects the revision; receipt alone does not count as acceptance.
9. Once the required approval rule is satisfied, the revised line or fragment becomes active in the current-state projection.
10. Visibility may then be expanded to a wider audience through permissions changes.
11. Once enough consensus is reached, the approved current state can be exported as an IATI XML draft ready for publication.

## Product principles

The implementation should follow these principles:

- **Chat-first**: conversation is a first-class part of the workflow.
- **Append-only**: XML collaboration state is stored as events or blocks, not overwritten in place.
- **Projected current state**: the visible XML is a read model built from events.
- **Explicit approval**: receiving a revision is not the same as accepting it.
- **Selective visibility**: users only see blocks they authored or received, plus any shared metadata allowed by the design.
- **Progressive disclosure**: a draft may move from individual visibility to wider team, department, organization, and cross-organization visibility over time.
- **Canonicalized ingest**: uploaded XML should be normalized before fragmentation into blocks.
- **Public IATI output**: the live editing environment is private; publication is the eventual end-state.
- **Human-in-the-loop governance**: future AI assistance must remain subject to human approval and audit.

## Ledger hierarchy model

MicroBlox should organize ledger data in a hierarchy that resembles how IATI activity data is commonly structured and distributed. This hierarchy should be logical, not necessarily literal filesystem storage, but it should shape addressing, permissions, UI navigation, and export behavior.

A recommended hierarchy is:

- **Organization**
- **Activity file**
- **Activity**
- **Fragment or line block**.

### Organization level

This corresponds to the reporting or publishing organization, such as a publisher identified by `reporting-org@ref` or another publisher identifier.

### Activity file level

This corresponds to one logical `iati-activities` XML file, which may contain one or more `iati-activity` records.

### Activity level

This corresponds to one `iati-activity` record, ideally keyed by `iati-identifier`.

### Fragment or line level

This corresponds to one editable unit such as `title`, `description`, `participating-org`, `recipient-country`, or a smaller line-oriented block when needed.

This model is preferred over a flat block list because it maps more naturally to IATI reality and supports file-level, activity-level, and fragment-level permissions and projections.

## XML upload and canonicalization workflow

If a user uploads a whole XML file or a fragment, the application should canonicalize the XML before breaking it into blocks or fragment events. This matters because semantically identical XML can differ in whitespace, formatting, namespace placement, and attribute ordering, which would otherwise create unnecessary divergence in hashes and block generation.

A recommended upload flow is:

1. Accept uploaded XML file or fragment.
2. Validate that the payload is well-formed XML.
3. Canonicalize the XML using a defined C14N method, preferably Canonical XML 1.1 for compatibility with XML signature practices.
4. Compute a canonical hash.
5. Derive internal fragment objects or line blocks from the canonicalized XML tree.
6. Store or retain the original uploaded payload for audit if useful.
7. Use the canonicalized representation as the basis for later block exchange, hashing, signing, and projection.

Canonicalization is for internal normalization and integrity, not for human-readable display. Readable XML and Markdown views should be generated separately from the structured current-state model.

## Functional requirements

The demo should include the following capabilities:

### Messaging

- One-to-one chat.
- Small group chat or activity-specific channels.
- Timestamped message threads.
- Optional message status or presence indicators if easy to implement.
- Ability to flag a message as related to a specific XML item using message actions or similar metadata.

### Files and activities

- A list of organizations, files, or activities the user is working on or following.
- A visible status for each file or activity, such as draft, in review, approved, or ready for publication.
- The ability to open a file and see its current visible state.
- The ability to navigate to an activity within a multi-activity file.

### Editor

- XML mode showing the current-state XML view.
- Readable mode or Markdown projection that presents the same content in a more human-friendly format.
- Ability to click from chat into the relevant line or fragment in the editor.
- Ability to propose a new line or revision.
- Ability to accept, reject, or supersede a proposal.

### Ledger behavior

- All XML collaboration actions should generate append-only events or blocks.
- Prior events remain in history even when superseded.
- Current state should reflect the latest accepted or active version for each addressed line or fragment.
- The current XML view should be rebuilt from ledger events, not edited in place as a canonical source.

### Export

- Ability to export the current approved draft to XML, even if using a simplified IATI subset.

## Non-functional requirements

The demo should also satisfy these qualities:

- Lightweight and responsive on desktop and mobile.
- Minimal enough for a volunteer-built demo, but polished enough to communicate the concept clearly.
- Easy for non-engineers to understand visually.
- Real-time feel for both chat and editing workflows.
- Clear permission boundaries, even if identity and auth are simplified.
- Clean separation of transport, event logic, projection logic, and UI rendering, so AI coding assistants can reason about the architecture modularly.

## UI and screens

The demo should at minimum include the following views.

### 1. Chat / conversation list

This screen should show:

- direct chats,
- group chats,
- activity channels,
- unread indicators,
- and optionally who is online.

### 2. Conversation view

This should be the main working surface for collaboration. It should show:

- ordinary chat messages,
- flagged messages requesting help with XML,
- structured XML-related messages or chips,
- quick actions to open the editor on the referenced item.

### 3. Organization / file / activity navigation

This should show the logical hierarchy of:

- organizations,
- files inside an organization,
- activities inside a file,
- and status of those items.

### 4. Editor view

This should allow the user to switch between:

- raw XML view,
- readable/Markdown view.

The editor should focus on the currently selected line or fragment and allow proposing changes.

### 5. Review and approval state

The UI should expose whether a change is:

- proposed,
- pending review,
- accepted,
- rejected,
- superseded,
- or active.

### 6. Visibility scope indicator

The app should display who currently has access to the draft or fragment, such as:

- private,
- team,
- department,
- organization,
- partner organization.

## Information architecture

A practical desktop layout could be:

- left sidebar: chats and organizations/files/activities,
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

This allows normal chat and structured collaboration events to coexist in one messaging system.

### Recommended XML collaboration event types

The demo should support a small event vocabulary, for example:

- `chat_message`
- `edit_request`
- `fragment_proposal`
- `fragment_acceptance`
- `fragment_rejection`
- `fragment_supersession`
- `visibility_change`
- `publish_event`.

### Minimum event fields

A structured XML event should include fields such as:

- `block_id`
- `activity_id`
- `file_id`
- `organization_id`
- `fragment_id` or `line_id`
- `event_type`
- `sender_id`
- `recipient_scope`
- `sequence`
- `xpath`
- `content`
- `canonical_xml`
- `canonical_hash`
- `json_repr`
- `supersedes_block_id`
- `proposal_id`
- `status`
- `timestamp`
- `signature` or placeholder signature field.

### Important rule

A new revision should not erase the old one. The new event should supersede the earlier event for the same logical line or fragment, while the earlier event remains in history.

## Current-state projection model

The visible XML file should be built as a projection or read model from the append-only event stream. The projection logic should:

1. group events by addressed line or fragment,
2. order them by sequence or trusted timestamp,
3. determine which event is currently active,
4. ignore superseded or rejected events in the rendered current-state view,
5. render the latest accepted state in XML order.

This current-state view can be cached for performance, but conceptually it is always derived from the ledger rather than being the canonical source of truth.

## Consensus and approval model

MicroBlox should use workflow-level consensus, not infrastructure-level distributed consensus. In practice, this means:

- one user proposes a revision,
- another user explicitly accepts or rejects it,
- acceptance is a distinct event and should not be inferred from message receipt alone.

For the demo, a line or fragment can become active when one explicit acceptance is recorded, or when a configurable threshold is met if the team wants to demonstrate multi-party review.

A clear acceptance pattern would be:

- `fragment_proposal`: here is a revised line or fragment,
- `fragment_acceptance`: I approve proposal X,
- `fragment_rejection`: I reject proposal X,
- `fragment_supersession`: proposal Y replaces proposal X.

## Permissions and visibility model

Permissions are central to the concept. The application should support the idea that a draft becomes visible in stages over time.

A suggested visibility ladder is:

- individual,
- team,
- department,
- organization,
- partner organization,
- public publication outside the drafting environment.

The demo should make clear that:

- users only see fragments they are permitted to receive,
- the same draft may have different current visible projections for different users,
- visibility changes are themselves meaningful events in the workflow.

## PubNub-specific implementation guidance

The volunteers should map the architecture onto PubNub as follows:

- Use PubNub channels for direct chats, group chats, and activity-specific collaboration streams.
- Use Access Manager tokens and permissions to restrict which users can read or write which channels.
- Use presence to indicate online state and channel participation if helpful.
- Use message actions or metadata to flag chat items that reference XML fragments needing collaboration.

A practical channel strategy might include:

- direct channels for one-to-one conversation,
- group channels for teams,
- activity channels for file-specific collaboration,
- optional restricted channels for sensitive fragments.

## XML and readable-mode rendering

The canonical model should be the structured event stream plus the current-state projection. Markdown or readable mode should be treated as a usability layer, not as the source of truth.

This means:

- XML view shows the rendered current state,
- readable/Markdown view shows a human-friendly projection of the same state,
- edits from readable mode must still be turned back into structured events before they affect the ledger.

The team should avoid designing a system where free-form Markdown is the primary editable source and XML is reconstructed from it without a structured model.

## Recommended IATI subset for the demo

To keep the prototype manageable, the demo should support only a narrow subset of IATI activity content. A good subset would be:

- `iati-identifier`
- `reporting-org`
- `title`
- `description`
- `participating-org`
- `activity-status`
- `activity-date`
- `recipient-country`.

This is enough to demonstrate real structured XML collaboration without attempting the full standard.

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

This gives the team something concrete to model in the demo.

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

This should be generated from the same structured current state, not maintained separately.

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
    "organization_id": "GB-CHC-202918",
    "file_id": "activity-file-001",
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
    "organization_id": "GB-CHC-202918",
    "file_id": "activity-file-001",
    "activity_id": "GB-CHC-202918-JORA82",
    "fragment_id": "participating-org-01",
    "supersedes_block_id": "b1",
    "canonical_xml": "<participating-org role=\"3\" type=\"21\"><narrative xml:lang=\"EN\">Oxfam GB</narrative></participating-org>",
    "canonical_hash": "sha256:example",
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

## AI-agent positioning and future role

MicroBlox should be positioned as a human-centered system that can later support AI agents as assisting collaborators rather than autonomous publishers. This is a good strategic framing because the MicroBlox architecture already includes structured events, approval steps, permissions, and auditability, all of which are useful foundations for human-in-the-loop AI workflows.

In future versions, AI agents could:

- suggest likely missing IATI fields,
- recommend edits to narratives or metadata,
- detect possible duplicates,
- answer questions about previously reported activities,
- summarize draft activities for reviewers,
- and help humans navigate large volumes of IATI content.

However, in the intended design, AI actions should still be represented as structured proposals or recommendations subject to human review and approval before they become active or publishable.

## Future development roadmap

The demonstration should focus on the private collaboration workflow. The following ideas are future-facing and should be identified as later platform capabilities rather than first-demo requirements.

### Future capability 1: corpus-scale ledger snapshots

In principle, each user ledger could eventually incorporate a snapshot of published IATI corpus data organized by organizations, files, and activities. This would allow MicroBlox to function not only as a drafting system but also as a personal or organization-scoped IATI knowledge workspace.

### Future capability 2: duplicate checking and prior-report lookup

With broader corpus access, MicroBlox could help users ask whether a specific activity appears already reported before they create a new draft. That would reduce duplication and improve the quality of pre-publication review.

### Future capability 3: AI-assisted querying and curation

A later AI layer could sit on top of published and draft data to help users search, summarize, compare, and reason over IATI records, while keeping humans responsible for approvals and publication decisions.

These future capabilities should be described in the document so the long-term platform direction is visible, but they must remain out of scope for the volunteer demo build.

## Demo scope

The demonstration should stay intentionally narrow. In scope:

- a small set of mock users,
- a simplified IATI activity subset,
- chat plus XML collaboration,
- XML upload and canonicalization,
- organization/file/activity navigation,
- XML and readable-mode projections,
- append-only event handling,
- explicit acceptance,
- staged visibility changes.

Out of scope:

- full IATI implementation,
- production-grade cryptography,
- real blockchain consensus,
- enterprise deployment,
- comprehensive schema validation,
- robust offline sync across all edge cases,
- corpus-scale full local replication,
- AI-agent autonomy in publication decisions.

## Success criteria

The demo should be considered successful if it can show the following end-to-end scenario:

- two or more users can chat in real time,
- one user can upload or start an activity draft and have it normalized for collaboration,
- one user can flag or request collaboration on a specific XML item,
- another user can open that item in the editor and propose a revision,
- the revision is recorded as a new event,
- another user explicitly accepts the revision,
- the current-state XML view updates to show the accepted latest version,
- the file's visibility can be expanded to a wider audience,
- the resulting state can be exported as XML.

## Open implementation choices

The volunteers should have freedom to decide certain details as long as the core concept is preserved. Examples include:

- whether the ledger is stored entirely client-side, lightly server-assisted, or mocked,
- whether revisions operate at a strict line level or slightly larger fragment level,
- whether acceptance rules are one-approver or multi-approver,
- how much of signing and hashing is real versus demonstrative,
- how the readable view is styled,
- how permissions are visualized,
- how much of canonicalization and validation is surfaced to users versus hidden internally.

## Guidance for AI coding assistants

If this specification is provided to an AI coding assistant, the assistant should be told to optimize for:

- simplicity of the demo,
- clarity of the workflow,
- chat-first UX,
- event-driven architecture,
- append-only ledger behavior,
- client-friendly current-state projections,
- organization/file/activity hierarchy,
- XML canonicalization before fragmentation,
- and a modular codebase that cleanly separates transport, event logic, projection logic, permission handling, and UI rendering.

The assistant should avoid designing the system as a generic text editor with a chat sidebar. The intended architecture is the reverse: a messaging application that uses structured events and projections to produce collaborative XML editing behavior.

The assistant should also avoid treating Markdown as the canonical source of truth. Markdown or readable mode is only a projection over structured state.

The assistant should treat future AI-agent support as a design consideration for extensibility, not a required first-build feature.

## Final note

The most important idea for the demo is that users should feel they are collaborating in a lightweight messaging app, while the system quietly maintains a structured audit trail of XML collaboration events that can be projected into the current visible state of an IATI draft. That combination of chat, permissions, event history, canonicalized ingest, hierarchical organization, and structured export is the core of MicroBlox.
