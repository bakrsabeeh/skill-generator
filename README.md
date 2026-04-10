# Skill Generator

> Write your idea and expertise — get a Claude skill instantly.

A web app + CLI tool that turns your idea, skills, and working style into a valid [`SKILL.md`](https://agentskills.io/specification.md) file ready to use with [Claude Code](https://claude.ai/code).

---

## What it does

1. Fill out a form — your idea, expertise, working style, and context
2. Claude generates a valid `SKILL.md` tailored to you
3. Download the file and drop it into `~/.claude/skills/`
4. Optionally push it to a new GitHub repo with one CLI command

---

## Stack

| Layer | Tech |
|---|---|
| Backend | Node.js + Express |
| Frontend | Vanilla HTML/JS — no framework |
| AI | Claude API (`claude-sonnet-4-6`) |
| CLI | Zero-dependency Node.js (Node 18+) |

---

## Project Structure

```
skill-generator/
├── server.js          # Express server — serves frontend, proxies Claude API
├── public/
│   ├── index.html     # Form UI
│   └── app.js         # Fetch /api/generate, preview, download
├── cli/
│   └── push.js        # Push generated SKILL.md to a new private GitHub repo
├── .env               # ANTHROPIC_API_KEY (never sent to browser)
└── docs/
    └── spec.md        # Full design specification
```

---

## Getting Started

### 1. Clone & install

```bash
git clone https://github.com/bakrsabeeh/skill-generator.git
cd skill-generator
npm install
```

### 2. Add your API key

```bash
cp .env.example .env
# Edit .env and add your Anthropic API key
```

### 3. Run the server

```bash
node server.js
# Open http://localhost:3000
```

---

## Using the CLI

After downloading your `SKILL.md`, push it to a new private GitHub repo:

```bash
node cli/push.js path/to/skill.md --token <your-github-pat>
```

This creates a private repo named after your skill and pushes the file automatically.

---

## How it works

The form collects:

| Field | Description |
|---|---|
| Skill name | Lowercase slug — becomes the skill's `name` field |
| Your idea | What the skill does and when to trigger it |
| Expertise & skills | Your background — languages, tools, domains |
| Working style | Terse/verbose, bullet points/prose, async-first, etc. |
| Extra context | Role, team, goals — anything else Claude should know |
| Anthropic API key | Used per-request only, never stored |

The server builds a structured prompt and asks Claude to output a valid `SKILL.md` following the [Agent Skills spec](https://agentskills.io/specification.md). You get a live preview before downloading.

---

## Design Spec

Full design doc: [docs/spec.md](./docs/spec.md)

---

## Roadmap

- [ ] Streaming preview as Claude generates
- [ ] Edit skill in browser before downloading
- [ ] One-click deploy to GitHub Pages

---

## License

MIT
