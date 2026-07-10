# Project A — Claude Commands

Custom Claude Code slash commands for the Project A investment team.

---

## Commands

| Command | What it does |
|---|---|
| `/morning-recap` | Pulls new #deal-flow Slack messages since the last recap, routes by deep dive, and posts the Morning Recap to #automation-tests |
| `/deal-flow-review` | Pulls deal flow for a specific team member and date range, sends a formatted DM to their Slack |
| `/investment-team-dealflow-meetings` | Formats a Granola meeting transcript into ready-to-send Gmail email summaries (V1 internal, V2 compliance) |
| `/evertrace-signals` | Reads an Evertrace CSV export, routes companies by deep dive, and posts a signal digest to #automation-tests |
| `/dealflow-retro-newsletter` | Monthly European Pre-Seed and Seed funding round retro — Crunchbase CSV, manual top-50 curation, Affinity MCP check, HTML email output |
| `/dealflow-screener` | Paste any ad-hoc deal flow list — Claude enriches, routes companies and profiles by deep dive, posts to #automation-tests |
| `/call-prep` | Give Claude a company name, website, and/or pitch deck — get a concise VC investment brief that opens as an HTML page in your browser |

---

## Setup guide

Follow these steps in order. The whole process takes about 30 minutes.

> **Note:** Where these files are hosted may change after the IT infrastructure review. Check with your manager before starting if you are unsure of the correct clone source.

---

### Step 1 — Install Claude Code

1. Go to [claude.ai/download](https://claude.ai/download) and download the Mac desktop app
2. Sign in with your Anthropic account — create one at [claude.ai](https://claude.ai) if needed. You need a paid Claude Pro or Team plan.
3. Open a terminal window inside Claude Code, or use your system Terminal

---

### Step 2 — Clone the repo and install the commands

Run these commands in Terminal one at a time:

```bash
git clone https://github.com/kieranvelasquez-lang/project-a-commands.git ~/Projects/project-a-commands
```

```bash
mkdir -p ~/.claude/commands
```

```bash
cp ~/Projects/project-a-commands/commands/*.md ~/.claude/commands/
```

```bash
mkdir -p ~/.claude/project-a/memory
```

Open a new Claude Code session. You should see all seven commands in the slash command menu.

---

### Step 3 — Connect Claude to Slack (Slack MCP)

Several commands read from and post to the Project A Slack workspace. Each intern connects their own Slack account — Claude will post and read messages as you.

1. Open Claude Code → **Settings → MCP Servers**
2. Click **Add MCP Server** → select **Slack** (ask your manager for the server config if it does not appear)
3. Follow the OAuth flow using your Project A Slack account (`yourname@project-a.vc`)
4. Approve all permissions — Claude needs read and write access to channels and DMs

**Verify:** Open a new Claude Code session and type `/morning-recap`. If Slack is connected, Claude will start pulling from #deal-flow.

---

### Step 4 — Set your #automation-tests channel ID

All automation outputs post to a private staging channel before going live. You need to create your own — the previous intern's channel no longer exists.

1. In Slack, create a new **private** channel named `#automation-tests`
2. Right-click the channel → **View channel details** → copy the Channel ID at the bottom (starts with `C0...`)
3. Run this in Terminal, replacing `PASTE_YOUR_ID_HERE` with your actual ID:

```bash
sed -i '' 's/YOUR_AUTOMATION_TESTS_CHANNEL_ID/PASTE_YOUR_ID_HERE/g' ~/.claude/commands/*.md
```

This updates all skill files at once. You only need to do this once.

---

### Step 5 — Connect Affinity MCP

Required for `/dealflow-retro-newsletter`, which checks companies against the Master Deals List.

1. Open Claude Code → **Settings → MCP Servers**
2. Add the Affinity MCP server (ask your manager for the server config URL)
3. Authorise with your Project A Affinity credentials

---

### Step 6 — Install Claude in Chrome

The dealflow meeting workflow requires the Claude in Chrome browser extension.

1. Install the Claude extension from the Chrome Web Store — search "Claude for Chrome" or ask your manager for the direct link
2. Sign in with your Anthropic account
3. Confirm the Claude icon appears in your browser toolbar

---

### Step 7 — Set up Claude's Board in Affinity and configure the Chrome shortcut

Claude's Board is a saved Affinity view that the Chrome extension scrapes before each dealflow meeting. You must create your own — the previous intern's board URL will not work for you.

**Create your board:**

1. Go to the Master Deals List in Affinity: `https://projecta.affinity.co/lists/99030`
2. Click **Views** → duplicate the existing "Claude's Board" view
3. Name your copy `Claude's Board`
4. Copy the URL of your new view — it will look like `https://projecta.affinity.co/lists/99030/views/XXXXXXX-claudes-board`

**Create the Chrome shortcut:**

1. Open the Claude in Chrome extension → **Shortcuts**
2. Create a new shortcut named `Investment-team-dealflow-meeting`
3. Paste the prompt below, replacing `YOUR_CLAUDES_BOARD_URL` with your actual Affinity view URL:

```
Navigate to YOUR_CLAUDES_BOARD_URL
Wait for the page to fully load. You will see a flat list/table of companies,
each with a Name, Website, Owners, and Stage column.
Your task is to extract all companies and group them by their Stage value.
You only care about three stages: "Partner Call", "Due Diligence",
and "Legal/Financial DD". Ignore any rows with other stage values.
For each row, extract THREE pieces of information:
1. Company name — the bold text in the Name column
2. Company website — the smaller grey URL text sitting directly below the
   company name in the same Name column (e.g. "gondorindustries.com").
   This is NOT a separate column — it appears as a subtitle under the name.
   Do not skip this or leave it blank.
3. Owners — all names listed in the Owners column
Scroll down until you have seen every row in the table before outputting anything.
Then output the results in exactly this format, with each company on its own line:
---
Dealflow
Legal & Financial DD
[Company Name] - [website] - @[First name], @[First name]...
Due Diligence
[Company Name] - [website] - @[First name], @[First name]...
Partner Calls
[Company Name] - [website] - @[First name], @[First name]...
---
Rules:
- Each company MUST be on its own new line
- Every entry MUST include the website — if you cannot find it, write "no website"
  rather than skipping it
- Use first names only for owners (e.g. "Miha Pavlovič" → "Miha")
- Prefix every owner name with @ (e.g. "@Miha, @Jack")
- List sections in this order: Legal & Financial DD first, then Due Diligence,
  then Partner Calls
- If an owner column shows "+N more", click into that row to reveal all hidden
  owner names before extracting
- Output only the formatted list with no extra commentary
```

---

### Step 8 — Set up Granola

The dealflow meeting workflow uses Granola for AI-generated meeting transcripts. You need to configure a custom recording format.

1. Download and install Granola from [granola.ai](https://granola.ai)
2. Sign in with your work account
3. Go to **Settings → Note Formats** (ask the outgoing intern or watch the setup video [link TBD])
4. Create a new format called `Investment Team Dealflow Meeting` and paste in this context exactly:

```
This is a biweekly internal investment team dealflow meeting at Project A, a
Berlin-based VC firm. The agenda is structured across three categories: Legal &
Financial DD, Due Diligence, and Partner Calls. Before the meeting, a deal list
will be pasted into the notes containing every company being discussed, their
website, and assigned team members. This list must be fully preserved in the
output — do not drop any company, even if nothing was said about them during
the meeting.

Section: Deal Updates
Output Format:
List every company from the pre-meeting notes under its original category heading.
Use exactly three section headings in this order: Legal & Financial DD, Due
Diligence, Partner Calls.
For each company, output exactly one entry in this format:
Company Name (website URL) (Country) - @Owner1, @Owner2 // Summary // Next steps

Rules:
- The company name is the human-readable name (e.g. "Gondor Industries"), not
  the domain or URL. The website URL follows in parentheses exactly as it appears
  in the pre-meeting notes. These are two separate fields — never substitute one
  for the other.
- All @names from the pre-meeting notes must appear after the dash. Do not drop,
  abbreviate, or reorder them.
- Do not move companies between sections. Each company stays under the category
  heading it was listed under in the pre-meeting notes.
- After the double slash, write the status update or key discussion points from
  the meeting in one concise block. Keep next steps and action items inline —
  do not create a separate action items section.
- If a company was listed in the pre-meeting notes but nothing was said about it
  during the meeting, write: Company Name (URL) (Country) - @Names // No update.
- Do not remove, merge, reorder, or add any companies beyond what appears in the
  pre-meeting notes.
- Do not create any sections beyond the three listed above.
```

> **Important:** The outgoing intern will demonstrate switching to this format on a walkthrough video [link TBD]. Watch this before your first dealflow meeting.

---

## Getting updates

When commands are updated, pull the latest and reinstall:

```bash
cd ~/Projects/project-a-commands && git pull
cp commands/*.md ~/.claude/commands/
```

After pulling, re-run the channel ID `sed` command if new Slack-posting skills were added.

---

## Deep Dives — Routing

All commands use the same routing table (updated July 2026):

| What they build | Deep Dive |
|---|---|
| Space (civilian/non-defense), ocean, land, subsurface, agriculture, infrastructure, construction, energy (hardware and software for the physical world), new materials | **Physical World Intelligence** — Daria |
| Manufacturing, factory automation, factory software, supply chain, logistics | **Industrial Autonomy** — Oskar |
| Fintech, payments, healthcare, real estate, insurance, compliance, legal, payroll, tax, blockchain, crypto, web3 | **Regulated Industries** — Marjorie |
| Defense tech, military, weapons, military space | **European Resilience** — Jack, Miha |
| Semiconductors, chips, quantum computing, novel computation; breakthrough energy hardware (novel generation/storage); frontier biotech (synthetic biology, genomics, drug discovery); novel AI architectures; fundamental CS algorithm research, competitive-programming/algorithmic-research background (e.g. "ex-competitive programmer", "AI research background", "PhD, novel model work") — route here on any plausible fit, even from a terse note; robotics infrastructure | **Frontier Tech** — Jack, Omar |
| AI agents, orchestration, LLM infra, dev tools, enterprise AI-native SaaS (applied/practitioner work with no research or competitive-programming signal — e.g. "AI engineer", "ML ops", generic "works in AI"), gaming, consumer, edtech, creator, fitness | **Miscellaneous** — (no team member, visibility only) |

**Hardcoded overrides:**
- Cybersecurity (all types) → Frontier Tech (Omar)
- AI sales tools (commissions, revenue ops) → Miscellaneous
- Blockchain / crypto / web3 → Regulated Industries
- Energy (software, cleantech, grid, infrastructure) → Physical World Intelligence (Daria); breakthrough energy hardware (e.g. fusion) → Frontier Tech
- Agriculture, construction, infrastructure → Physical World Intelligence (Daria)
- Space: civilian/commercial (satellites, launch, orbital infra) → Physical World Intelligence (Daria); military/defense space → European Resilience
- Robotics — all robotics infrastructure (software/AI-first: foundation models, physical intelligence, robot OS; hardware/physical world: space, ocean, agriculture, construction) → Frontier Tech (Omar); factory/industrial → Industrial Autonomy; defense robotics → European Resilience
- Biotech frontier (synthetic biology, genomics, drug discovery) → Frontier Tech (Omar); commercial healthtech/medtech → Regulated Industries
- Semiconductors, chips, quantum → Frontier Tech
- AI/CS research or competitive-programming signal (novel model/algorithm work, CS research roles, PhDs on new methods, competitive-programming background) → Frontier Tech, even on a terse note; applied AI product/practitioner roles with no research signal → Miscellaneous

---

## Memory files

Each skill learns from your corrections over time. Files are stored at `~/.claude/project-a/memory/` on your machine — not in the repo. They are personal to you and do not carry over to the next intern.

| File | Used by |
|---|---|
| `morning-recap-corrections.md` | `/morning-recap`, `/dealflow-screener` |
| `investment-team-dealflow-meetings-corrections.md` | `/investment-team-dealflow-meetings` |
| `evertrace-signals-corrections.md` | `/evertrace-signals` |

---

## Commands in detail

### `/morning-recap`
Pulls every new message from #deal-flow since the last Morning Recap (auto-anchored to the most recent recap post), routes all deals to the correct deep dive, captures Slack-explicit action items and thread commentary, enriches entries missing a description, and posts to #automation-tests.

**Flow:** Auto-anchor → Pull Slack → Parse + route → Capture action items + commentary → Enrich → Post to #automation-tests

**Requires:** Slack MCP

---

### `/deal-flow-review`
Given a team member's name and a date range, pulls all #deal-flow messages where they were tagged (action items) or where entries match their deep dive (thesis matches). Formats a clean summary and sends it as a Slack DM to that person.

**Usage:** `/deal-flow-review` → enter name → select date range (this week / last 2 weeks / custom)

**Requires:** Slack MCP

---

### `/investment-team-dealflow-meetings`
Paste a Granola meeting transcript and get two ready-to-send email summaries — Version 1 (internal, full detail) and Version 2 (clean, compliance-facing). HTML files open directly in your browser for copy-paste into Gmail.

**V1 recipients:** Investment Team, Anton Waitz, Uwe Horstmann, Florian Heinemann, Thies Sander, Philipp Werner, Malin Posern, Jack Wang

**V2 recipients:** Investment Team, Anton Waitz, Vincent Synde, Miriam Ayasse, Martin Laudien, Christian Kurz, Andreas Kühnke, Anton Grabovski, Elias Wahl, Christoph Heiland, Frieder Zinkel

**Usage:** `/investment-team-dealflow-meetings` → paste transcript when prompted

**Requires:** Nothing beyond Claude Code

---

### `/evertrace-signals`
Export a CSV from Evertrace, run this command, answer three questions (CSV path, week label, optional theme mapping), and get a clean signal digest posted to #automation-tests.

**Usage:** `/evertrace-signals` → follow prompts (drag CSV into terminal for the file path)

**Requires:** Slack MCP

---

### `/dealflow-retro-newsletter`
Monthly digest of European Pre-Seed and Seed funding rounds across six thesis tables. Import a Crunchbase Pro CSV, Claude merges EU-Startups data, you manually curate to top 50, Claude routes by thesis and runs an Affinity MCP check, then generates a formatted HTML email.

**Flow:** Confirm date range → Export Crunchbase CSV → EU-Startups merge → Curate top 50 → Enrich → Route → Affinity check → HTML email

**Affinity check:** Seen in Master Deals List? + In contact in last 12 months?

**Recipients:** Investment Team, Anton Waitz, Uwe Horstmann, Florian Heinemann, Thies Sander, Philipp Werner, Malin Posern, Jack Wang

**Usage:** `/dealflow-retro-newsletter` → follow prompts

**Requires:** Crunchbase Pro (CSV export). Affinity MCP connected.

---

### `/dealflow-screener`
Paste any ad-hoc deal flow list (from an email, WhatsApp, DM, etc.) and get a unified post to #automation-tests — companies and profiles merged by deep dive. Enriches companies, flags ambiguous entries, pauses for context on unroutable profiles.

**Usage:** `/dealflow-screener` → paste your list

**Requires:** Slack MCP

---

### `/call-prep`
Give Claude a company name plus any available materials (website URL, pitch deck PDF path, DOCX memo) and get a concise VC investment brief that opens as an HTML page in your browser.

**Covers:** Snapshot · Problem & Solution · Product & Technology · Traction & Metrics · Team · Market & Timing · Investment Framing · Key Questions (5 max)

**Usage:** `/call-prep` → provide company name and materials

**Requires:** Nothing beyond Claude Code

---

## Repo structure

```
project-a-commands/
├── README.md
└── commands/
    ├── morning-recap.md
    ├── deal-flow-review.md
    ├── evertrace-signals.md
    ├── investment-team-dealflow-meetings.md
    ├── dealflow-retro-newsletter.md
    ├── dealflow-screener.md
    └── call-prep.md
```

Memory files are personal to each user and live at `~/.claude/project-a/memory/` — not in the repo.

---

## Questions / issues

Speak to your manager or raise an issue in this repo.
