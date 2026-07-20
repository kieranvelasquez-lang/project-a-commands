---
description: Screen an ad-hoc deal flow list — enriches and routes companies and profiles into one unified list, posting to #automation-tests
allowed-tools: Read, WebSearch, WebFetch, mcp__claude_ai_Slack__slack_send_message, mcp__claude_ai_Slack__slack_search_users
---

# Ad-hoc Deal Flow Screen

## Team member → thesis mapping

| Deep Dive | Member | User ID |
|---|---|---|
| Physical World Intelligence | Daria Gneusheva | `U0AA0044W1K` |
| Industrial Autonomy | Oskar Lingk | `U0AA1BDG7D4` |
| Regulated Industries | Marjorie Lengereau | `U0AAGADJCQ1` |
| European Resilience | Jack Wang | `U0A9X1FNV19` |
| European Resilience | Miha Pavlovic | `U0A9X1DAVTM` |
| Frontier Tech | Jack Wang | `U0A9X1FNV19` |
| Frontier Tech | Omar Hedeya | `U0A9MUM30AK` |

---

## Step 1 — Collect the list

If the user has not already pasted the list in their prompt, ask:

> "Paste the deal flow list — one entry per line. Include any URLs, LinkedIn links, or descriptions if you have them."

Accept any format. Do not require structure.

---

## Step 2 — Parse and separate entries

The list may contain section headers (e.g. "Pipeline", "Declined Q1", "Portfolio - Closed", "Market Radar", "Tracking"). **Ignore these labels entirely** — process every entry the same way regardless of which section it appears in. Never skip entries based on section labels.

Separate all entries into two buckets:

**Companies** — has a company website URL, or is clearly a company/product name (not a person's name).

**Profiles** — has a LinkedIn URL, or is clearly a person's name (with or without a title/background note).

For each entry in both buckets, capture:
- **Name**
- **URL** (company website or LinkedIn URL, if provided)
- **Description/context** — any inline text following the name or URL (keep verbatim)

---

## Step 3 — Enrich companies

For each company entry:

### URL was provided
- **Company URL**: WebFetch the homepage for a one-sentence description. If WebFetch fails, try WebSearch with company name + "startup".
- If the original list included an inline description, use that even if enrichment fails.

### No URL provided
- Run WebSearch for "[company name] startup". Only use a result if the company name is an **exact match** — no partial matches, no assumed synonyms.
  - Exact match found → WebFetch for a one-sentence description.
  - No exact match found → **flag the entry**.

### Flagged companies — pause before composing Part 1

After processing all companies, if any are flagged, stop and ask:

> "I couldn't find exact matches for the following companies. Provide a URL for each, or type 'skip' to leave them unlinked:
> - [Company 1]
> - [Company 2]"

If user provides URLs, re-run enrichment. If 'skip', include name only — no link, no description. Only proceed once all flags are resolved.

---

## Step 4 — Route companies

First, load staged corrections: read `~/.claude/projects/-Users-kvelasquez-Projects/memory/morning-recap-corrections.md`. Any company listed there overrides default routing.

**Default routing table:**

| What they build | Deep Dive |
|---|---|
| Space (civilian/non-defense), ocean, land, subsurface, agriculture, infrastructure, construction, energy (hardware and software for the physical world), new materials, robotics infrastructure (software/AI-first: foundation models, physical intelligence, robot OS; hardware/physical world: space, ocean, agriculture, construction), AI inference infrastructure generally | Physical World Intelligence |
| Manufacturing, factory automation, factory software, supply chain, logistics | Industrial Autonomy |
| Fintech, payments, healthcare, real estate, insurance, compliance, legal, payroll, tax, blockchain, crypto, web3 | Regulated Industries |
| Defense tech, military, weapons, military space | European Resilience |
| Semiconductors, chips, quantum computing, novel computation; breakthrough energy hardware (novel generation/storage methods); frontier biotech (synthetic biology, genomics, drug discovery); novel AI architectures/paradigms, fundamental CS algorithm research, competitive-programming/algorithmic-research background (e.g. "ex-competitive programmer", "AI research background", "PhD, novel model work") — route here on any plausible fit, even from a terse note | Frontier Tech |
| AI agents, orchestration, LLM infra, dev tools, enterprise AI-native SaaS, general AI tech stack (applied/practitioner work with no research or competitive-programming signal — e.g. "AI engineer", "ML ops", generic "works in AI"); gaming, consumer, edtech, creator, fitness — excludes AI inference infra (routes to Physical World Intelligence, see hardcoded rules) | Miscellaneous (no team member — visibility only) |

**Hardcoded routing rules:**
- Cybersecurity — all cybersecurity (commercial pentesting/infosec/SOC/security tooling and offensive/defense-grade) → Frontier Tech (Omar)
- AI sales tools (commissions, sales enablement, revenue ops) → Miscellaneous
- Blockchain / crypto / web3 → Regulated Industries
- Energy — energy software, cleantech, grid, energy infrastructure → Physical World Intelligence (Daria); breakthrough energy hardware (novel generation/storage, e.g. fusion) → Frontier Tech
- Agriculture, construction, infrastructure → Physical World Intelligence (Daria)
- Space — civilian/commercial space (satellites, launch, orbital infra) → Physical World Intelligence (Daria); military/defense space → European Resilience
- Robotics — all robotics infrastructure (software/AI-first: foundation models, physical intelligence, robot OS; hardware/physical world: space, ocean, agriculture, construction) → Physical World Intelligence (Daria); robotics (factory/industrial floor) → Industrial Autonomy; defense robotics → European Resilience
- AI inference — inference infrastructure/platforms/tooling generally, not just robotics-specific (e.g. inference serving, inference optimization/hardware-adjacent infra) → Physical World Intelligence (Daria)
- Biotech — frontier (synthetic biology, genomics, drug discovery) → Frontier Tech (Omar); commercial healthtech/medtech/clinical → Regulated Industries
- Semiconductors, chips, quantum computing → Frontier Tech
- AI/CS research or competitive-programming signal (novel model/algorithm work, CS research roles, PhDs on new methods, competitive-programming background) → Frontier Tech, even on a terse note; applied AI product/practitioner roles with no research signal → Miscellaneous

If a company still cannot be routed with confidence, place it in **⚠️ Flagged for Review**.

---

## Step 5 — Handle profiles

Sort all profile entries into two groups:

**Has context** — the entry includes an inline description, background note, or enough information to route confidently (e.g. "Lars Rehfeldt — Aegiron", "Andrei Ciobotar — background in AI/IoT"). Route these directly using the same thesis routing table. The inline text becomes the description shown in Part 2.

**No context** — a LinkedIn URL or bare name with no inline description. Cannot route without more information.

If any profiles have no context, pause and output to the terminal:

> "Here are [N] profiles I couldn't route — no context available. Click through each and reply with a brief note (background, what they're working on, which thesis you think):
> 1. [Name — linkedin.com/in/slug] (or just [Name] if no URL)
> 2. ...

Wait for the user's response. Accept notes in any format (e.g. "1. robotics, ex-Palantir", "3. crypto, skip"). Once received, route all profiles using the notes provided. If the user provides a thesis directly, use it. If they say 'skip' for an entry, omit it from Part 2 entirely.

If all profiles already have context, skip this pause and proceed directly.

**Profile routing** uses the same deep dive table as companies — route based on domain expertise and background:
- Space (civilian), ocean, subsurface, agriculture, construction, infrastructure, energy, new materials, robotics (all forms except factory/industrial and defense) → Physical World Intelligence
- Manufacturing, factory automation, supply chain, logistics → Industrial Autonomy
- Fintech, crypto, compliance, healthcare, real estate → Regulated Industries
- Defense, military, weapons, aerospace, military space → European Resilience
- Semiconductors, chips, quantum, frontier biotech, novel energy hardware (fusion/etc.), fundamental AI/CS research (including competitive-programming background, CS research roles, PhDs working on novel algorithms/model paradigms — route here on any plausible fit, even from a terse note); cybersecurity → Frontier Tech
- AI/ML practitioners and infra roles with no research or competitive-programming signal (e.g. "AI engineer", "ML infra", generic "works in AI"); consumer, gaming, edtech, fitness, creator → Miscellaneous

---

## Step 6 — Resolve @mention user IDs

All core team member IDs are hardcoded in the table above — use them directly. Only call `slack_search_users` for non-team members (e.g. action item owners outside the core team). Format as `<@USERID>`.

---

## Step 7 — Compose unified list

Companies and profiles go into one single message, merged by thesis. Within each thesis section, list companies first, then profiles — but both use the same bullet format.

```
**Ad-hoc Deal Flow Screen — [Month D, YYYY]**

**Physical World Intelligence** <@U0AA0044W1K>
- <https://company.com|CompanyName> — One-sentence description.
- <https://www.linkedin.com/in/slug/|Full Name> — background context note.

**Industrial Autonomy** <@U0AA1BDG7D4>
- <https://company.com|CompanyName> — One-sentence description.
- <https://www.linkedin.com/in/slug/|Full Name> — background context note.

**Regulated Industries** <@U0AAGADJCQ1>
- <https://company.com|CompanyName> — One-sentence description.
- <https://www.linkedin.com/in/slug/|Full Name> — background context note.

**European Resilience** <@U0A9X1FNV19> <@U0A9X1DAVTM>
- <https://company.com|CompanyName> — One-sentence description.
- <https://www.linkedin.com/in/slug/|Full Name> — background context note.

**Frontier Tech** <@U0A9X1FNV19> <@U0A9MUM30AK>
- <https://company.com|CompanyName> — One-sentence description.
- <https://www.linkedin.com/in/slug/|Full Name> — background context note.

**Miscellaneous**
- <https://company.com|CompanyName> — One-sentence description.
- <https://www.linkedin.com/in/slug/|Full Name> — background context note.

**⚠️ Flagged for Review**
- CompanyName — reason
```

**Section order (fixed — never deviate):**
1. Physical World Intelligence
2. Industrial Autonomy
3. Regulated Industries
4. European Resilience
5. Frontier Tech
6. Miscellaneous

**Formatting rules:**
- Bold: `**double asterisk**`
- Italic: `_underscore_`
- Links: `<https://url|display text>` — never `[text](url)`
- @mentions: `<@USERID>` — never plain `@name`
- Ampersands: raw `&` — never `&amp;`
- One blank line between thesis sections
- Do not include `| _Raised:_`
- Do not append `_Sent using Claude_`
- Company entries with no URL: plain name only, no link; use description if provided in the original list
- Profile entries with no LinkedIn URL: plain `Full Name` only
- Action item format: `| _Action: <@USERID>_` appended at end of line
- Only include sections that have entries; only include Flagged if there are flags
- If the message exceeds ~4000 characters, split into Part 1 / Part 2 with _(continued)_ on the second

---

## Step 8 — Post to #automation-tests

Post the unified message to channel ID `C0AKKPK3J1K` using `slack_send_message`.

Then output to terminal:
> "Screened [N] companies + [M] profiles → posted to #automation-tests."

---

## Step 9 — Ask for routing corrections

Ask:
> "Were any thesis routings wrong? If yes, tell me: 'CompanyName should be [Thesis]' and I'll learn it. Type 'no' to finish."

---

## Step 10 — Learn from corrections (only if user provides them)

For each routing correction:

1. Determine if **company-specific** or a **general rule**.
   - General rules → ask Kieran to confirm before editing the skill directly.
   - Company-specific → stage in corrections file.

2. Append to `~/.claude/projects/-Users-kvelasquez-Projects/memory/morning-recap-corrections.md`:
```
- CompanyName → Thesis Name | added YYYY-MM-DD
```

3. Tell the user:
> "Learned [N] new correction(s): [list]. Staged for future runs."
