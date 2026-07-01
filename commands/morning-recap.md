---
description: Pull yesterday's #deal-flow Slack messages, enrich and route them, then publish the Morning Recap to #automation-tests
allowed-tools: Read, Write, WebSearch, WebFetch, mcp__claude_ai_Slack__slack_read_channel, mcp__claude_ai_Slack__slack_read_thread, mcp__claude_ai_Slack__slack_send_message, mcp__claude_ai_Slack__slack_search_users
---

# Morning Recap

## Team member Slack display names
User IDs are hardcoded below — do not call `slack_search_users` for core team members.

| Thesis | Member | User ID |
|---|---|---|
| Physical World Intelligence | Daria Gneusheva | `U0AA0044W1K` |
| Industrial Autonomy | Oskar Lingk | `U0AA1BDG7D4` |
| Regulated Industries | Marjorie Lengereau | `U0AAGADJCQ1` |
| European Resilience | Jack Wang | `U0A9X1FNV19` |
| European Resilience | Miha Pavlovic | `U0A9X1DAVTM` |
| Frontier Tech | Jack Wang | `U0A9X1FNV19` |
| Frontier Tech | Omar Hedeya | `U0A9MUM30AK` |

---

## Step 1 — Check corrections delta

Read `~/.claude/project-a/memory/morning-recap-corrections.md` using the Read tool.

This file is a **staging area** for new corrections that haven't yet been baked into the skill. The default routing rules below already incorporate all previously confirmed corrections.

- If the file is empty or has no entries: proceed — no delta to apply.
- If the file has entries: load them into context as overrides. Note them briefly (e.g. "1 staged correction: CompanyX → Regulated Industries"). These take precedence over default routing for any matching company name.

Do not re-read or re-explain the full routing table — just flag what's new.

---

## Step 2 — Pull from Slack (auto-anchor)

Anchor automatically:

1. Call `slack_read_channel` on #deal-flow (channel ID: `C0AB6LUVCN4`) with `oldest` set to 48 hours ago (current Unix timestamp minus 172800).
2. Scan the returned messages for the most recent one that begins with `**Morning Recap —`. Use that message's `ts` value as the anchor.
3. **If no such message is found in the 48-hour window**, stop and ask:
   > "I couldn't find a Morning Recap to anchor from. Which message should I start at? Please give me the sender name, date, and time (e.g. 'Philipp Werner, April 2, 9:15 AM') and I'll pull that message plus everything after it."
   - Wait for the user's response before proceeding.
   - Once they provide a start message, use `slack_read_channel` with `oldest` set to just before that message's timestamp to retrieve it, then process **that message and all messages after it** (inclusive — the start message itself is not a recap boundary, it's the first entry to process).

Once the anchor `ts` is established from a found Morning Recap, call `slack_read_channel` again with `oldest` set to that `ts` to fetch only messages posted after the last Morning Recap.

**Exclusion rule (Morning Recap anchor only):** When anchoring from a found Morning Recap, that message is a boundary marker only — do not parse its text for company or LinkedIn entries. Do not read or process its thread replies. Only process messages with `ts` strictly greater than the anchor `ts`. This exclusion does NOT apply when using a user-specified start message.

Remember: Project A is Berlin — CET = UTC+1, CEST = UTC+2 from late March.

For each message retrieved:
- Capture timestamp, full content, sender display name, any names already assigned inline
- If a message has thread replies, use `slack_read_thread` to capture additional context (links, descriptions, investor info, assignee notes, action items)

---

## Step 3 — Parse each entry (no enrichment yet)

For every company or LinkedIn profile mentioned, do the following:

### 3a. Identify type
Is this a company deal or a LinkedIn profile link?

### 3b. Extract inline data only
Pull directly from Slack — no WebFetch or WebSearch yet:
- Company/person name
- URL from Slack message
- Inline description from Slack message (verbatim or lightly cleaned)
- Funding info only if explicitly mentioned in Slack
- Thesis routing — apply staged corrections first, then default routing table below

**First: apply staged corrections.** If the company name appears in `morning-recap-corrections.md`, use that thesis — do not apply default routing rules.

**Otherwise, use default routing:**

| What they build | Thesis | Team |
|---|---|---|
| Space (civilian/non-defense), ocean, land, subsurface, agriculture, infrastructure, construction, energy (hardware and software for the physical world), new materials, robotics infrastructure | Physical World Intelligence | Daria Gneusheva |
| Manufacturing, factory automation, factory software, supply chain, logistics | Industrial Autonomy | Oskar Lingk |
| Fintech, payments, healthcare, real estate, insurance, compliance, legal, payroll, tax, blockchain, crypto, web3 | Regulated Industries | Marjorie Lengereau |
| Defense tech, military, weapons, military space | European Resilience | Jack Wang, Miha Pavlovic |
| Semiconductors, chips, quantum computing, novel computation; breakthrough energy hardware (novel generation/storage methods); frontier biotech (synthetic biology, genomics, drug discovery); novel AI architectures/paradigms, fundamental CS algorithm research | Frontier Tech | Jack Wang, Omar Hedeya |
| AI agents, orchestration, LLM infra, dev tools, enterprise AI-native SaaS, general AI tech stack; gaming, consumer, edtech, creator, fitness | Miscellaneous | (no team member — visibility only) |

**Hardcoded routing rules (confirmed corrections, already baked in):**
- Cybersecurity — all cybersecurity (commercial pentesting/infosec/SOC/security tooling and offensive/defense-grade) → Frontier Tech (Omar)
- AI sales tools (commissions, sales enablement, revenue ops) → Miscellaneous
- Blockchain / crypto / web3 → Regulated Industries (even if building tooling or infra for blockchain networks)
- Energy — energy software, cleantech, grid, energy infrastructure → Physical World Intelligence (Daria); breakthrough energy hardware (novel generation/storage, e.g. fusion) → Frontier Tech
- Agriculture, construction, infrastructure → Physical World Intelligence (Daria)
- Space — civilian/commercial space (satellites, launch, orbital infra) → Physical World Intelligence (Daria); military/defense space → European Resilience
- Robotics (software/AI-first — foundation models, physical intelligence, robot OS) → Frontier Tech (Omar)
- Robotics (hardware/physical world — space, ocean, agriculture, construction) → Physical World Intelligence (Daria); robotics (factory/industrial floor) → Industrial Autonomy; defense robotics → European Resilience
- Biotech — frontier (synthetic biology, genomics, drug discovery) → Frontier Tech (Omar); commercial healthtech/medtech/clinical → Regulated Industries
- Semiconductors, chips, quantum computing → Frontier Tech
- Fundamental AI research (novel model architectures or paradigms, CS algorithm research) → Frontier Tech; applied AI products → Miscellaneous

### 3c. Flag unknowns
Flag entries with no URL and no name, but still capture them — never skip entries.

### 3d. Capture inline assignments, action items, and thread commentary

After routing, check for explicit Slack-level assignments that override default routing:

**In the original message:**
- Is a specific team member @mentioned alongside the deal? If yes, route to that person's thesis.

**In thread replies:**
- Look for language like "tagging this to X", "passing to X", "X will reach out", "X is following up", "pinging X on this".
- Track the full chain: if A assigns to B, and B reassigns to C, follow to C.

**Routing override rules:**
- If the final explicit assignee is on a different thesis than default routing → override the thesis to match the assignee.
- If tagger and thread responder are on the **same thesis team**, no routing change needed (e.g. Marjorie sends + tags Miha, Jack also responds → still European Resilience).
- If a cross-thesis reassignment exists (e.g. Miha tags Daria, Daria passes to Marjorie) → route to final assignee's thesis (Fintech), and flag the action item owner.

**Action item flag:**
Record who owns the action item (if anyone). This will be displayed inline in the Morning Recap as `| _Action: <@USERID>_`.

If no explicit action item exists, leave this field blank — do not invent one.

**Thread commentary to surface:**
For each thread reply that is from a human (non-bot) and contains substantive text beyond a pure routing phrase, capture the reply text and the sender's user ID as a comment to surface under the entry.

- A reply is "pure routing" only if its **entire body** is a routing/assignment phrase with no additional content (e.g. "Tagging to <@X>" alone). If the reply contains routing AND commentary, capture the full text.
- Ignore empty replies and bot/app messages.
- Store these as an ordered list of `{userId, text}` per entry — they will be rendered in Step 6.

### 3e. Capture inline notes from the main message body

For each company or LinkedIn profile link extracted from the main message, also capture any inline text the poster wrote around that link:

1. **Same-line text**: Take the text on the same line as the URL, stripping the URL itself. Common patterns: `"Met at NOAH: https://url"`, `"https://url — pass, no EU traction"`, `"https://url (great team)"`.
2. **Preceding-line fallback**: If the URL line has no text besides the URL itself, check the immediately preceding non-empty line. If it reads as a natural annotation (not itself a URL, not a section header, not a standalone company name), capture it as the inline note.
3. Do **not** capture lines that are clearly the company name or a category header.
4. Attribute to the original poster's user ID.
5. If no inline note is found, leave blank — do not invent one.

These notes are separate from the Slack description used for the entry's one-liner. Store as `{userId: originalPosterUserId, text: inlineNote}` per entry alongside thread comments.

---

## Step 4 — Enrich entries with no Slack description

For any entry where the Slack message included **no description**, enrich before composing the Morning Recap:

**Companies:**
- Use WebFetch on the company's actual website homepage to pull a one-sentence description.
- If WebFetch fails, try WebSearch (company name + "startup" or "what does it do").
- If both fail, flag the entry — do not invent a description.
- Do **not** research funding here.

**LinkedIn profiles:**
- LinkedIn blocks automated fetching. If no description was in Slack, omit the description in the Summary — do not attempt WebFetch.

---

## Step 5 — Resolve @mention user IDs

All core team member IDs are hardcoded in the table above — use them directly. Only call `slack_search_users` for non-team members who appear as action item owners (e.g. a founder, an angel, or a guest collaborator). Format all pings as `<@USERID>` in the post body.

---

## Step 6 — Compose Morning Recap

Match this format exactly:

```
**Morning Recap — [Month D, YYYY]**
_Review of yesterday's dealflow · [Month D, YYYY]_

**Physical World Intelligence** <@U0AA0044W1K>
- <https://company.com|CompanyName> — One-sentence description.
- <https://company.com|CompanyName2> — One-sentence description. | _Action: <@USERID>_
  › _<@U456>: "Met them at NOAH, really impressive team"_
  › _<@U789>: "I know the founder, happy to intro"_
- <https://linkedin.com/in/handle|Full Name> — One-sentence bio/context.
  › _<@U456>: "Strong background in robotics, worth a look"_

**Industrial Autonomy** <@U0AA1BDG7D4>
- <https://company.com|CompanyName> — One-sentence description.

**Regulated Industries** <@U0AAGADJCQ1>
- <https://company.com|CompanyName> — One-sentence description. | _Action: <@U0AAGADJCQ1>_

**European Resilience** <@U0A9X1FNV19> <@U0A9X1DAVTM>
- <https://company.com|CompanyName> — One-sentence description.

**Frontier Tech** <@U0A9X1FNV19> <@U0A9MUM30AK>
- <https://company.com|CompanyName> — One-sentence description.

**Miscellaneous**
- <https://company.com|CompanyName> — One-sentence description.

---

**⚠️ Flagged for Review**
- CompanyName — reason
```

Rules:
- Include **all** entries from the day — not just Net New. Companies already in Affinity belong here too.
- Only include thesis sections that have entries.
- Only include "Flagged for Review" if there are actual flags.
- One blank line between each thesis section.
- Do **not** include `| _Raised:_` in the Recap.
- Action item format: `| _Action: <@USERID>_` appended at end of line, only when a confirmed action item owner exists.
- LinkedIn profiles with no Slack description: show link and name only (`- <https://linkedin.com/in/handle|Full Name>`), no description fragment.
- Do NOT include `_Sent using Claude_` — the MCP appends it automatically.
- If post exceeds ~4000 characters, split into Part 1 / Part 2.

**Commentary sub-bullets (inline notes from main message and thread replies):**
- If an entry has any inline notes (from the main message body, Step 3e) or thread comments (from Step 3d), render them as indented sub-bullets immediately after the entry line.
- Main message note first (if any), then thread replies in chronological order.
- Format each: `  › _<@USERID>: "note text"_` — two spaces + `›` + space + italic attribution.
- Truncate individual notes at ~200 characters, appending `…` if cut.
- Entries with no commentary at all render exactly as before — no sub-bullets.
- Commentary sub-bullets count toward the ~4000-char split threshold.

### Formatting rules (CRITICAL — never deviate)

- Bold: `**double asterisk**`
- Italic: `_underscore_`
- Links: `<https://url|display text>` — never markdown `[text](url)` format
- @mentions: `<@USERID>` — never plain `@name`
- Ampersands: raw `&` — never `&amp;`
- Commentary sub-bullets: `  › _<@USERID>: "text"_` — two spaces + `›` glyph + italic (never markdown `>` blockquote)
- No "sourced by" attribution
- Do NOT append `_Sent using Claude_`
- Dividers: do NOT use `---` in Slack messages — causes an `invalid_blocks` error
- LinkedIn URLs: always use the full `https://www.linkedin.com/in/slug/` format — never output just `/in/slug`
- Link URLs: never use percent-encoded characters (e.g. `%C3%AB`) in the URL portion of `<url|text>` — use simplified ASCII slugs instead

---

## Step 7 — Post to #automation-tests

Post the Morning Recap to channel ID `YOUR_AUTOMATION_TESTS_CHANNEL_ID` using `slack_send_message`.

Then output to the terminal:
> "Morning Recap is live in #automation-tests. Copy to #deal-flow when ready."

---

## Step 8 — Ask for routing corrections

Ask:
> "Were any thesis routings wrong? If yes, tell me: 'CompanyName should be [Thesis]' and I'll learn it. Type 'no' to finish."

---

## Step 9 — Learn from corrections (only if user provides them)

For each routing correction provided:

1. Determine if this is **company-specific** (e.g. "AcmeCorp → Fintech") or **general** (e.g. "AI sales tools → Surf and Turf").
   - **General rules** should be baked directly into the hardcoded routing rules in Step 3b of this skill — ask Kieran to confirm before editing.
   - **Company-specific corrections** go into the staging file.

2. For company-specific corrections, check if the company already exists in `~/.claude/project-a/memory/morning-recap-corrections.md`. If the file doesn't exist, create it with this header:

```markdown
# Morning Recap — Corrections Memory

## Staged Corrections (not yet baked into skill)
<!-- Format: CompanyName → Thesis | added YYYY-MM-DD -->
<!-- Valid theses: Physical World Intelligence, Industrial Autonomy, Regulated Industries, European Resilience, Frontier Tech, Miscellaneous -->
```

3. Append only new entries (skip duplicates):
```
- CompanyName → Thesis Name | added YYYY-MM-DD
```

4. Tell the user:
> "Learned [N] new correction(s): [list]. Staged for future runs. Let me know if any should be baked into the skill as a permanent rule."
