# Knowledge Base

---
slug: knowledge-base
kb: default-kb
tier: 1
status: draft
updated_at: 2026-02-13T01:40:24.192Z
---

<!-- snippet:intro -->
# cta snippet

## Heading

<!-- snippet:usage -->
# usage snippet

# Production grade visual Skills Editor for internal use at Antares

## Executive summary

This report proposes an internal first, production grade “Skills Editor” that lets marketing and engineers compose Skill Trees from reusable Snippets, maintain draft, review, live environments, and publish Markdown exports into a Git repository with governance. Tier 2 changes ship via a direct commit, Tier 1 changes ship via a pull request, aligned with a human approval model (similar in spirit to human in the loop controls highlighted by entity["company","Vercel","deployment platform"] and modern agent tooling). citeturn7view0turn0search0turn0search4

Key recommendations:

- Use a GitHub App for repository write access, rather than personal tokens, because installation tokens are short lived, auditable, and designed for automation. citeturn1search3turn1search28turn1search11
- Create a canonical data model in Postgres, then “compile” immutable snapshots per publish, and export deterministic Markdown plus a tree manifest. Agents should point to snapshot IDs and commit SHAs for safe rollbacks. citeturn0search10turn9search0turn9search2turn9search3
- Treat imported knowledge as untrusted content and build layered defences, because prompt injection remains a significant risk whenever agents consume external or user supplied text. citeturn8view1turn8view0turn3search9
- Plan for connector scale by aligning with entity["company","Anthropic","ai company"]’s Model Context Protocol (MCP) approach, which targets the “N by M integrations” problem with a standard protocol. Use it later, not in V1. citeturn8view2turn0search2turn0search6

## Product scope and roadmap

**Core user story**
Marketing needs to edit reusable Snippets and assemble Skills visually without asking engineers. Engineers need governance, diffs, auditability, and deterministic exports into Git.

**MVP principles**
Internal first means single tenant, fast iteration, and strict focus on reliability, auditability, and shipping workflow over “connector completeness”.

### Prioritised MVP feature list

Must have, week 1 to 8:

- Skill Tree visual editor, hierarchical navigation, drag reorder, search.
- Snippet library, reusable Snippets referenced by multiple Skills, with “Used in” reverse index.
- Draft workspace for edits, plus Review and Live pointers (environments).
- Tiering model, Tier 2 direct publish, Tier 1 publish as PR.
- Diff viewer for Snippets and Skills, plus “impact list” showing which Skills change when a Snippet changes.
- Full audit trail, who changed what, when, and why, including immutable publish events.
- Export compiler creates Markdown files, stable paths, deterministic output.
- Git integration using GitHub App, single commit per publish, and PR creation for Tier 1.
- Basic collaboration, comments on Skills and Snippets, plus approval decision records.

Should have, month 3 to 6:

- Safer collaboration, per object edit locks, graceful conflict handling.
- Structured validation, linting, and schema enforcement at publish time.
- Import upgrades, URL fetch, file upload to storage, lightweight parsing.
- Notifications for review requests and PR events via webhooks.
- Read optimised “Live Snapshot API” for agents, cached, version pinned.

Could have, month 6 to 12:

- Connector framework via MCP servers, enabling “import from Atlassian, Linear, Google Docs” without bespoke integrations. citeturn8view2turn0search2
- Automatic suggestion, “AI assisted chunking” and “Skill draft generation” using AI SDK patterns. citeturn0search5turn7view0
- Multi team governance, fine grained policy rules, scheduled rollouts.
- Enterprise hardening, advanced retention, evidence export, compliance workflows.

**Why this sequencing**
The editor plus governed publish loop is your core wedge. Connectors can be layered later, and MCP provides a credible long term direction for scaling integrations. citeturn8view2turn0search2turn0search6

## Canonical data model and versioning design

This section specifies a Postgres first model that supports reuse, auditability, diffs, environments, and Git publish metadata.

### Entity model overview

- Snippet is the smallest reusable unit of truth.
- Skill is a node in a tree, composed from an ordered list of Snippet references.
- Version is an immutable snapshot of the entire knowledgebase as compiled at publish time.
- PublishEvent records an attempted publish, outcomes, commit SHAs, PR links, and approvals.
- ChangeLog stores granular edits for audit and diff context.
- Users and Roles implement RBAC, optionally backed by SAML SSO.

For database hosting, entity["company","Supabase","backend platform"] provides managed Postgres, Auth, Storage, and SAML support, and can enforce Row Level Security as defence in depth if you ever expose data directly to clients. citeturn12search3turn1search2turn3search2turn3search37

### Tables and fields

Below is a pragmatic schema. Types are Postgres oriented, UUID primary keys, timestamps in UTC.

**teams**

- id uuid pk
- name text
- slug text unique
- created_at timestamptz

**users**

- id uuid pk
- email text unique
- display_name text
- team_id uuid fk teams
- status enum(active, disabled)
- created_at timestamptz
- last_login_at timestamptz

**roles**

- id uuid pk
- key enum(admin, owner, reviewer, editor, viewer)
- description text

**user_roles**

- user_id uuid fk users
- role_id uuid fk roles
- scope enum(global, team)
- scope_team_id uuid nullable
- created_at timestamptz
- pk(user_id, role_id, scope, scope_team_id)

**snippets**

- id uuid pk
- slug text unique, stable for file paths
- title text
- body_markdown text
- tier smallint check in (1,2,3)
- status enum(draft, review, live, archived)
- owner_team_id uuid fk teams
- created_by uuid fk users
- updated_by uuid fk users
- created_at timestamptz
- updated_at timestamptz
- revision int default 1
- archived_at timestamptz nullable

**skills**

- id uuid pk
- slug text unique
- title text
- description text nullable
- tier smallint check in (1,2,3)
- status enum(draft, review, live, archived)
- owner_team_id uuid fk teams
- created_by uuid fk users
- updated_by uuid fk users
- created_at timestamptz
- updated_at timestamptz
- revision int default 1
- archived_at timestamptz nullable

**skill_tree_edges**

This encodes the tree and ordering.

- parent_skill_id uuid fk skills
- child_skill_id uuid fk skills
- sort_order int
- created_at timestamptz
- created_by uuid
- pk(parent_skill_id, child_skill_id)

**skill_snippet_refs**

This encodes composition and ordering.

- skill_id uuid fk skills
- snippet_id uuid fk snippets
- sort_order int
- include_mode enum(full, excerpt)
- excerpt_start int nullable
- excerpt_end int nullable
- created_at timestamptz
- created_by uuid
- pk(skill_id, snippet_id, sort_order)

**environments**

- id uuid pk
- key enum(draft, review, live)
- description text

**environment_pointers**

This says which Version is currently active in an environment.

- environment_id uuid fk environments
- current_version_id uuid fk versions nullable
- updated_at timestamptz
- updated_by uuid
- pk(environment_id)

**versions**
Immutable, compiled snapshots.

- id uuid pk
- version_number bigserial unique
- created_at timestamptz
- created_by uuid
- source_environment enum(draft, review)
- promoted_to_live_at timestamptz nullable
- snapshot_json jsonb, full compiled tree, resolved Snippet bodies
- manifest_json jsonb, list of exported file paths, checksums
- git_repo text
- git_base_branch text
- git_commit_sha text nullable
- git_tag text nullable
- build_status enum(queued, building, succeeded, failed)
- build_error text nullable

**publish_events**

- id uuid pk
- requested_by uuid fk users
- requested_at timestamptz
- source_version_id uuid fk versions
- policy_result enum(tier2_direct, tier1_pr_required, blocked)
- reason text nullable
- git_branch text
- git_commit_sha text nullable
- pr_number int nullable
- pr_url text nullable
- status enum(queued, running, success, failed, awaiting_review, merged)
- completed_at timestamptz nullable

**approvals**

- id uuid pk
- publish_event_id uuid fk publish_events
- reviewer_user_id uuid fk users
- decision enum(approved, rejected)
- comment text nullable
- decided_at timestamptz

**change_log**

Append only event log for audit and diffs.

- id uuid pk
- actor_user_id uuid fk users
- acted_at timestamptz
- object_type enum(snippet, skill, ref, edge, env_pointer)
- object_id uuid
- action enum(create, update, delete, reorder, publish_request, publish_complete, approve, reject)
- before_json jsonb nullable
- after_json jsonb nullable
- client_info jsonb nullable, user agent, ip, session id
- correlation_id uuid nullable, groups multiple changes in one UI session

### JSON examples

Snippet JSON example:

```json
{
  "id": "9f0c6e10-1c2d-4d9a-8b74-6f3a4d0a5c6a",
  "slug": "attribution-definition",
  "title": "Attribution definition",
  "tier": 2,
  "status": "draft",
  "owner_team": "marketing",
  "body_markdown": "## Attribution\nAttribution is the rule-set used to assign credit...",
  "revision": 7,
  "updated_by": "user_123",
  "updated_at": "2026-02-12T10:15:00Z"
}
```

Skill snapshot fragment example:

```json
{
  "skill": {
    "slug": "campaign-analysis",
    "title": "Campaign analysis",
    "tier": 1,
    "children": ["channel-analysis", "creative-analysis"],
    "snippets": [
      {"slug": "attribution-definition"},
      {"slug": "brand-voice-guidelines"},
      {"slug": "reporting-template-weekly"}
    ]
  }
}
```

<!-- snippet:cta -->
# cta snippet
