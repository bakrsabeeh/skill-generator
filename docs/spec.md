# Skill Generator App — Design Spec

## Overview

A web app + CLI tool that takes a user's idea, expertise, working style, and context, then uses the Claude API to generate a valid `SKILL.md` file they can download and use with Claude Code.

---

## Target Users

Anyone who wants a personal Claude skill — from non-technical users to developers. No prior knowledge of the Agent Skills spec required.

---

## Architecture

```
skill-generator/
├── server.js          # Express backend — serves frontend + proxies Claude API
├── public/
│   ├── index.html     # Form UI
│   └── app.js         # Calls /api/generate, shows preview, triggers download
├── cli/
│   └── push.js        # Zero-dep Node.js CLI — pushes SKILL.md to a new private GitHub repo
└── .env               # ANTHROPIC_API_KEY (never sent to browser)
```

### Request flow

1. User fills out the form and submits
2. `app.js` POSTs form data to `/api/generate`
3. `server.js` builds a prompt and calls the Claude API (`claude-sonnet-4-6`)
4. Claude returns raw `SKILL.md` content
5. Frontend renders a preview + Download button
6. Optionally: user runs `node cli/push.js skill.md --token ghp_xxx` to push to a new private GitHub repo

---

## Form Fields

| Field | Type | Required | Notes |
|---|---|---|---|
| `Skill name` | Text | Yes | Validated: lowercase, hyphens only, 1-64 chars |
| `Your idea` | Textarea | Yes | What the skill does and when to trigger it |
| `Expertise & skills` | Textarea | Yes | Languages, domains, tools, experience level |
| `Working style` | Textarea | Yes | Terse/verbose, bullet points/prose, async-first, etc. |
| `Extra context` | Textarea | No | Role, company, team, goals, anything else relevant |
| `Anthropic API key` | Password | Yes | Sent to server per-request, never stored |

---

## Backend — `server.js`

- **Framework:** Express (minimal dependencies)
- **Routes:**
  - `GET /` — serves `public/index.html`
  - `POST /api/generate` — accepts JSON body, calls Claude API, returns `SKILL.md` text
- **API key handling:** Read from request body, used for that request only. Never logged, never stored.
- **Error responses:** JSON `{ error: "message" }` with appropriate HTTP status codes

### Claude prompt template

```
You are an expert at writing Agent Skills following the Agent Skills specification.

Generate a valid SKILL.md file for the following person:

Skill name: {name}
Idea: {idea}
Expertise & skills: {expertise}
Working style: {working_style}
Extra context: {context}

Rules:
- Frontmatter must have `name` (matching the slug provided) and `description` (1-1024 chars, include trigger phrases)
- Body under 500 lines
- Use H2 for sections, bullet points over prose
- Direct, instructional tone — second person
- No emojis, no filler

Output ONLY the raw SKILL.md content. No explanation, no markdown fences.
```

**Model:** `claude-sonnet-4-6`

---

## Frontend — `public/`

### `index.html`

- Single page, no framework
- HTML5 `required` on all required fields
- Skill name field: live regex validation (`/^[a-z0-9][a-z0-9-]*[a-z0-9]$/`) with inline error
- Submit button disabled while request is in-flight
- Preview area renders the raw `SKILL.md` in a `<pre>` block
- Download button triggers `SKILL.md` file download via Blob URL

### `app.js`

- Handles form submit: validates, POSTs to `/api/generate`, renders result
- Inline error display for API failures
- Warns user (but does not block) if generated skill exceeds 500 lines

---

## CLI — `cli/push.js`

Zero-dependency Node.js script (Node 18+). Creates a new private GitHub repo and pushes the `SKILL.md` file.

**Usage:**
```bash
node cli/push.js <path-to-skill.md> --token <github-pat>
```

**Steps:**
1. Read `SKILL.md`, extract `name` from frontmatter
2. `POST /user/repos` to GitHub API — creates private repo named after the skill
3. Init a local git repo in a temp dir, add the file, commit, push

**Error handling:**
- Missing file → clear message + exit 1
- Missing token → clear message + exit 1
- Repo already exists → clear message + exit 1
- GitHub API errors → print response message + exit 1

---

## Error Handling Summary

| Scenario | Handled by | Response |
|---|---|---|
| Invalid skill name | Client-side JS | Inline field error, blocks submit |
| Empty required fields | HTML5 + JS | Blocks submit |
| Claude API error | Server → client | Inline error message in UI |
| Generated skill > 500 lines | Server → client | Warning shown, download still available |
| Missing CLI args | CLI | Usage message + exit 1 |
| GitHub repo already exists | CLI | Clear error + exit 1 |

---

## Out of Scope

- User accounts / login
- Saving generated skills server-side
- Editing the skill in the browser after generation
- OAuth GitHub flow (user provides PAT to CLI directly)
- Streaming Claude response (full response returned at once)
