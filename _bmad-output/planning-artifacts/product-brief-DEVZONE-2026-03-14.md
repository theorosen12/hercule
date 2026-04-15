---
stepsCompleted: [1, 2, 3, 4, 5, 6]
status: complete
inputDocuments:
  - '_bmad-output/project-context.md'
date: 2026-03-14
author: Matthieu
---

# Product Brief: DEVZONE

<!-- Content will be appended sequentially through collaborative workflow steps -->

## Executive Summary

Nara is a collaborative memory-sharing mobile application that enables friends, families, and couples to collectively build, preserve, and relive their shared experiences. Unlike scattered photo dumps across messaging apps or solo travel journals like Polarsteps, Nara provides a structured storytelling framework where groups co-create lasting memories organized into meaningful moments — each with its own context, media, and narrative.

The app addresses a universal frustration: years of shared experiences fragmented across camera rolls, chat threads, and social platforms that were never designed for long-term preservation or collaborative storytelling. Nara fills this gap by offering a dedicated space where memories are built together, structured to tell a story, and preserved to be rediscovered for years to come.

---

## Core Vision

### Problem Statement

People share countless meaningful experiences with friends and family, yet have no dedicated tool to collectively capture, structure, and preserve those memories. Photos and videos end up scattered across individual devices, buried in messaging threads, or posted on social platforms designed for ephemeral consumption — not lasting preservation.

### Problem Impact

Over time, shared memories degrade and fragment. The photos from a group trip three years ago are split across five phones, partially shared in a WhatsApp thread nobody scrolls back to, and stripped of the context that made them meaningful. The story behind the memories — who was there, what happened, the sequence of moments — is lost entirely. There is currently no solution that addresses long-term preservation, collaborative contribution, and structured storytelling together.

### Why Existing Solutions Fall Short

- **Messaging apps (WhatsApp, Messenger):** Media gets buried in conversation history with no structure or organization. Impossible to find or relive memories months later.
- **Cloud photo albums (Google Photos, iCloud):** Storage-focused, no storytelling structure, limited collaborative features, and impersonal.
- **Social media (Instagram, TikTok):** Designed for public performance and ephemeral content, not intimate group memory preservation.
- **Travel journaling (Polarsteps):** Strong storytelling structure but limited to solo travel experiences — not collaborative and not applicable to everyday shared moments.

None of these tools solve the complete problem: enabling a group of people to co-create a structured, lasting memory together.

### Proposed Solution

Nara provides a collaborative memory-building platform structured around two key concepts:
- **Nara (Memory):** A shared experience defined by a title, date range, location, and a group of members who lived it together.
- **Moment:** A specific point within a memory, with its own title, date, location, and collection of photos and videos with descriptions.

This hierarchy gives every shared experience a narrative arc — not just a folder of media, but a story told through its moments. Every member can contribute their own media and perspectives, making the memory richer and more complete than any single person's camera roll.

### Key Differentiators

1. **Collaborative by design:** Every member of a memory contributes media and context, building a collective story rather than a single person's curation.
2. **Structured storytelling:** The Nara > Moments hierarchy creates a natural narrative arc that transforms raw media into meaningful, contextual stories.
3. **Built for lasting preservation:** Long-term memory preservation is the core promise, not a side effect of cloud storage.
4. **Not limited to travel:** Any shared experience — birthdays, family gatherings, festivals, weekends with friends — deserves to be remembered.
5. **Viral by nature:** Adding someone to a Nara introduces them to the app. They create their own Naras with other groups, spreading adoption organically beyond the first circle.

---

## Target Users

### Primary Users

Nara targets anyone who shares meaningful experiences with others and wants to preserve them — friends groups, families, and couples across all age ranges.

There are no rigid user roles. Any user can be a creator (initiating a Nara and inviting members) or a contributor (adding moments, photos, and videos). All members of a Nara have equal participation rights.

#### Persona 1: Léa, 27 — The Initiator

Léa just came back from a weekend trip with her closest friends. She knows everyone took great photos and videos, but they're scattered across 6 different phones. She opens Nara, creates a memory for the trip, invites the group, and starts adding her best moments. Within hours, her friends contribute theirs. The complete story of the weekend takes shape — richer than any single person's camera roll.

- **Motivation:** Preserve the full story of shared experiences, not just her own perspective
- **Behavior:** Curates intentionally — selects the most meaningful media rather than dumping everything
- **Current frustration:** Photos shared in group chats get buried and lost. There's no place to build a lasting, structured memory together

#### Persona 2: Karim, 34 — The Discovered User

Karim receives a notification: his sister added him to a Nara about their parents' anniversary dinner. He downloads the app, joins the memory, and sees photos and moments already organized with context. He adds his own video of the toast. For the first time, the family has a single, beautiful record of the evening.

- **Motivation:** Didn't know he needed this until he experienced it — now he creates Naras for his own friend group
- **Behavior:** Starts as a contributor, becomes an initiator as he sees the value
- **Current frustration:** Family photos end up in a WhatsApp group that nobody scrolls back through

### User Journey

1. **Discovery:** A friend or family member invites the user to join a Nara → they download the app to participate
2. **Onboarding:** The user joins the memory, sees contributions from others, and adds their own meaningful photos and videos
3. **First Aha Moment:** Seeing everyone's contributions come together into a complete, structured story — richer than any individual's perspective
4. **Core Usage:** Creating Naras for new shared experiences (trips, gatherings, everyday moments) and contributing to others' Naras — both during the experience and after
5. **Deep Aha Moment:** Months later, rediscovering an old Nara through the feed — reliving a memory with full context, media, and the story as it was lived. A refreshing, emotional experience
6. **Long-term Engagement:** The feed surfaces old memories to relive. New experiences trigger new Naras. The app becomes the place where all meaningful shared moments live — growing more valuable over time

---

## Success Metrics

### User Success Metrics

- **Invite-to-user conversion:** Near 100% of people invited to a Nara download the app and join — the shared memory itself is the most compelling onboarding experience
- **Contribution frequency:** Users actively add moments, photos, and videos to Naras they're part of — both during experiences and after
- **Rediscovery engagement:** Users return to browse and relive old memories through the feed, demonstrating long-term emotional value
- **Content quality:** Successful Naras have multiple moments with rich descriptions and media from multiple contributors — not just photo dumps

### Business Objectives

- **Growth through virality:** Primary acquisition is organic — users discover Nara by being invited into a memory, then create their own Naras with other groups, expanding the network
- **Retention cadence:** Users return at least every 3 months, aligned with the natural rhythm of group events (trips, gatherings, celebrations). Usage is event-driven and bursty, not daily — this is expected and healthy
- **No monetization at this stage:** Focus is on building a growing, engaged user base. Monetization strategy to be defined once product-market fit is established

### Key Performance Indicators

| KPI | Target | Measurement |
|-----|--------|-------------|
| Invite-to-join rate | ~100% | % of invited users who download and join their first Nara |
| Creator conversion | High | % of users who go from contributor to creating their own Nara |
| Contribution rate | Majority | % of Nara members who actively add media or moments |
| Nara richness | High quality | Average number of moments, descriptions length, and unique contributors per Nara |
| Retention (3-month) | Strong | % of users who return to the app within a 3-month window |
| Feed engagement | Regular | Frequency of users browsing old memories via the feed |

---

## MVP Scope

### Core Features

The following features are already built and functional:

- **Authentication:** Google Sign-In, Apple Sign-In
- **Nara (Memory) management:** Create, edit Naras with title, date range, location, and members
- **Moments:** Create moments within Naras with title, date, location, and media (photos/videos with descriptions)
- **User profiles:** Profile pages with contact discovery and friend suggestions
- **Media viewer:** Instagram story-style format to browse all media within a moment
- **Feed:** Story-style format surfacing moments from the user's Naras
- **Map view:** Visualize all Naras on a map
- **Notifications:** Push notifications for Nara activity
- **Nara invitations:** Invite members to a Nara (functional)

### MVP Gaps to Close

These improvements are required before scaling the user base:

1. **Invitation flow redesign:** Improve the UX for inviting new people to a Nara — make it seamless to bring in users who aren't on the app yet
2. **New user onboarding:** Users who join via an invitation should land directly into the Nara they were invited to, not an empty app
3. **Feed upgrade:** Replace the current random moment display with a meaningful feed combining "On This Day" memory resurfacing and recent activity from the user's Naras

### Out of Scope for MVP

The following features are explicitly deferred to post-MVP:

- Comments or reactions on moments/media
- AI features (auto-summaries, auto-tagging, memory highlights)
- Public or semi-public sharing outside the app
- Search across Naras and moments
- Chat/messaging within a Nara
- Web version (mobile-only for now)
- Monetization features
- Analytics dashboard beyond Pendo

### MVP Success Criteria

- Invitation flow converts near 100% of invited users into active participants
- New users who join via invite immediately see and contribute to shared memories
- The feed drives regular return visits through "On This Day" resurfacing and recent activity
- Users organically create new Naras with other groups after their first experience

### Future Vision

**2-3 Year Horizon:**

- **Massive memory platform:** Scale from intimate friend groups to a widely-adopted platform for preserving shared experiences at any scale
- **AI-powered features:** Intelligent summaries, auto-tagging, memory highlights, and smart curation to enhance storytelling with minimal effort
- **Beyond personal use:** Expand to event organizers, company events, schools, sports teams — any organization that wants to collaboratively capture and preserve shared experiences
- **Platform expansion:** Web version, advanced search, public/semi-public sharing options, and monetization strategy built on a proven, engaged user base
