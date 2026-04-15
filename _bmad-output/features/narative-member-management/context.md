# Linear Context — Narative Member Management

## Source
- Issue: NAR-461 — Narative Member Management — Add Members & Validate Join Requests
- Status: Backlog | Priority: Medium
- Assignee(s): Matthieu Gouverith
- Labels: Feature

## Epic / Project
- Epic: N/A
- Project: N/A

## Feature Description

From the NarativePage, members can navigate to a dedicated members page that currently displays a static list of members. This feature enhances that page with two new capabilities:

1. **Add Members** — Allow members to invite/add new members directly from the members page.
2. **Validate Join Requests** — Allow members to review and approve/reject pending membership requests for the narative.

### Goals

- Replace the static member list with a fully interactive member management experience.
- Empower existing members (not just admins) to manage membership.
- Surface pending join requests so they can be acted upon without leaving the app.

### Acceptance Criteria

- [ ] From the NarativePage, members can navigate to the Members page.
- [ ] The Members page displays the current list of members (as today).
- [ ] A member can add a new person to the narative from the Members page.
- [ ] A member can see the list of pending join requests.
- [ ] A member can approve or reject a join request.
- [ ] A member can remove an existing member from the narative.
- [ ] The feature is accessible to all members of the narative (not restricted to admins).

### Out of Scope

- Admin-only roles or permission tiers (all members have equal access to this feature).

## Key Decisions & Constraints (from comments)

- Feature is accessible to **all narative members** — no admin restriction.
- Entry point is **NarativePage → Members page** (existing navigation).
- Three distinct capabilities: adding new members, validating pending join requests, and removing existing members.

## Open Questions (from comments)

- How does "adding a member" work mechanically? (search by username/email? invite link?)
- Are join requests already tracked in the backend, or does that data model need to be created?
- What happens when a request is rejected — is the requester notified?
- Is there a limit to how many pending requests can exist?

## Linked Resources

None attached at issue creation.

## Sibling Issues (same epic)

N/A — no parent epic.

## Raw Notes

- User description (original, in French): "Depuis la page narativepage, on peut acceder à une autre page ou on trouve la liste des membres, statique. Je voudrais pouvoir ajouter des membres depuis cette page, ainsi que valider les demandes d'ajout à ce narative. Cette feature doit etre accesibles aux membres."
- Git branch: `matthieugouverith/nar-461-narative-member-management-add-members-validate-join`
- Linear URL: https://linear.app/naraa/issue/NAR-461/narative-member-management-add-members-and-validate-join-requests
- Created: 2026-04-12
