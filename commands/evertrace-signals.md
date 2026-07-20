---
allowed-tools: Read, mcp__claude_ai_Slack__slack_send_message
---

# Evertrace Signals

You are processing a CSV export from Evertrace and posting a formatted signal digest to Slack.

---

## Step 1 — Ask three questions upfront. Wait for a single combined response before proceeding.

```
Three quick questions:

1. CSV file path? (drag the file into the terminal or paste the path)
2. Week label? (auto-detected as "w/c [MONDAY_OF_CURRENT_ISO_WEEK]" — confirm or correct)
3. Theme mapping? Paste profile names grouped by theme — e.g. "physical: Cord, Robin Bilgil / industrial: Josep Nolla / regulated: Jakub / resilience: SomeName". Valid keys: physical, industrial, regulated, resilience, frontier, misc. Use company name or founder name (partial is fine). Leave blank to use auto-detection as fallback.
```

To compute the auto-detected week label: find today's date, determine which day of the week it is (Monday=1 … Sunday=7 in ISO week), subtract (dayOfWeek - 1) days to get Monday, format as "w/c D Mon YYYY" (e.g. "w/c 23 Mar 2026"). Show this in the question so Kieran can confirm or override.

---

## Step 2 — Read corrections memory

Read the corrections file at: `~/.claude/projects/-Users-kvelasquez-Projects/memory/evertrace-signals-corrections.md`

If the file does not exist, continue without corrections. Store any theme routing overrides, name corrections, and company name corrections in memory for application at Step 4.

---

## Step 3 — Read and parse the CSV

Use the Read tool on the CSV path provided. Row 1 = headers. Parse remaining rows correctly handling RFC 4180 quoted fields (a field wrapped in `"..."` may contain commas — do not split on them).

Expected columns (exact names): `Company Name`, `Company Website URL`, `Company Description`, `Signal Type`, `First Name`, `Last Name`, `Linkedin URL`, `Linkedin Handle`, `Past Companies`, `City`, `Country`, `Industries`

Build a list of row objects keyed by column name. Keep `Signal Type` for each row — it will be "New Company" or "Stealth Position" (or similar).

---

## Step 4 — Deduplicate

Key each row by `Linkedin Handle` (if present and non-empty), otherwise by `FirstName_LastName` (concatenate with underscore, lowercase).

If the same key appears more than once:
- If entries include both "New Company" and "Stealth Position" signal types → keep "New Company", discard the stealth entry
- Otherwise keep the first occurrence, discard duplicates

---

## Step 5 — Detect theme per row

Apply theme assignments in this priority order:

1. **Manual mapping from Question 4** — if Kieran provided a theme mapping, match each profile by fuzzy-matching company name or founder first/last name (case-insensitive, partial match is fine). These take highest priority.
2. **Corrections memory overrides** — exact company name match, case-insensitive.
3. **Auto-detection** — for rows with no manual or corrections assignment, search `Industries` + `Company Description` + `Past Companies` + `Company Name` (all concatenated, case-insensitive) against these patterns in order:

| Theme key | Short label | Keywords (case-insensitive regex) |
|---|---|---|
| `resilience` | European Resilience | `defence`, `defense`, `military`, `weapon`, `aerospace` |
| `frontier` | Frontier Tech | `semiconductor`, `chip`, `quantum`, `deep.?tech`, `frontier.?bio`, `synthetic.?bio`, `genomics`, `drug.?discovery`, `biotech`, `fusion`, `energy.?storage`, `battery.?tech`, `novel.?architect`, `algorithm.?research`, `competitive.?programming`, `\bicpc\b`, `codeforces`, `research.?scientist`, `\bphd\b`, `cybersecurity`, `cyber.?security`, `infosec`, `pentesting`, `security.?tooling` |
| `physical` | Physical World Intelligence | `ocean`, `maritime`, `subsea`, `subsurface`, `offshore`, `satellite`, `\bspace\b`, `agriculture`, `agri`, `farming`, `infrastructure`, `construction`, `energy`, `cleantech`, `climate`, `material`, `geospatial`, `robotics`, `robot`, `inference` |
| `industrial` | Industrial Autonomy | `logistics`, `freight`, `warehousing`, `warehouse`, `procurement`, `shipping`, `supply.?chain`, `manufacturing`, `factory`, `industrial` |
| `regulated` | Regulated Industries | `payment`, `banking`, `neobank`, `crypto`, `blockchain`, `insurance`, `insurtech`, `regtech`, `accounting`, `legaltech`, `compliance`, `fintech`, `wealth`, `lending`, `invoice`, `healthcare`, `medtech`, `healthtech`, `health`, `clinical`, `real.?estate`, `proptech` |
| `misc` | Miscellaneous | (catch-all — `\bai\b`, `artificial intelligence`, `machine learning`, `saas`, `automation`, `developer tools`, `software`, `data analytics`, `no.?code`, `low.?code`, `api`, `devops`, `cloud`, `gaming`, `consumer`, `edtech`, `creator`, `fitness`, and everything that matches none of the above) |

Match in table order (first match wins — resilience and frontier checked first for specificity; physical before industrial to capture agriculture/energy correctly; misc is the final catch-all). Apply name corrections from corrections memory to `First Name` and `Last Name` fields. Apply company name corrections to `Company Name` field.

---

## Step 6 — Build the formatted Slack message

### Theme → team member tags

| Theme key | Members | Slack pings |
|---|---|---|
| `physical` | Daria Gneusheva | `<@U0AA0044W1K>` |
| `industrial` | Oskar Lingk | `<@U0AA1BDG7D4>` |
| `regulated` | Marjorie Lengereau | `<@U0AAGADJCQ1>` |
| `resilience` | Jack Wang, Miha Pavlovic | `<@U0A9X1FNV19> <@U0A9X1DAVTM>` |
| `frontier` | Jack Wang, Omar Hedeya | `<@U0A9X1FNV19> <@U0A9MUM30AK>` |
| `misc` | (no team member) | (no ping) |

### Theme order (always): physical → industrial → regulated → resilience → frontier → misc

### Unified list — all entries grouped by theme

Header:
```
**Evertrace Signals — w/c [DATE]**
```

For each theme that has ≥1 entry (New Company or Stealth):
```
**[Short label]**  [tags]
• [bullet]
• [bullet]
```

Within each theme block: New Company entries first, then Stealth entries. No separator between them.

**New Company bullet format:**
```
• <https://WEBSITE|CompanyName> — Description. _(ex-Co1, Co2)_ | _Founder: <https://LINKEDIN_URL|First Last> | City, Country_
```

Rules:
- **Company link**: if `Company Website URL` is non-empty, wrap company name: `<https://url|CompanyName>`. If URL is empty, use plain `CompanyName`
- **Description**: use `Company Description` if it is ≤120 chars and ends with `.`, `!`, or `?`. Otherwise omit it.
- **ex- field**: take `Past Companies`, split by common delimiters (`,` `/` `|`), strip whitespace, exclude any token that looks like a university (contains: `university`, `université`, `universität`, `college`, `school`, `institute`, `academy`), take first 2 remaining tokens. Format: `_(ex-Co1, Co2)_`. If no valid tokens remain, omit the whole ex- field.
- **Founder link**: if `Linkedin URL` is non-empty use `<https://url|First Last>`. If empty but `Linkedin Handle` is non-empty, construct URL as `https://www.linkedin.com/in/[handle]/` and use that. If both empty, use plain `First Last`.
- **Location**: `City, Country`. If City is empty, use `Country` only. If Country is also empty, omit location entirely.
- Assemble as: `• [company] — [description] [ex-field] | _Founder: [founder] | [location]_`
- If description is omitted, write: `• [company] [ex-field] | _Founder: [founder] | [location]_`
- If ex-field is omitted, skip that part.

**Stealth bullet format:**
```
• <https://www.linkedin.com/in/[handle]/|Full Name>, ex-Co1, Co2 (City, Country)
```

Rules:
- Use LinkedIn URL if available, else construct from handle, else plain name
- ex- companies: same filtering rules (max 2, no universities)
- Location in parentheses at end
- Omit location if neither city nor country

### Footer (always last, one blank line before it):

```
_Source: Evertrace · Screened by Kieran · [DATE]_
```

DATE in footer = same week label date string (e.g. "23 Mar 2026").

### Formatting rules

- Use `**double asterisk**` for bold (not `__`)
- One blank line between theme sections
- No subtitle line (do NOT include `_High-signal screen: VC-backed & serial founders, Score 10/10, EU-based_` or anything like it)
- If full message exceeds ~4000 characters: split into two messages. Second message header: `**Evertrace Signals — w/c [DATE] *(continued)***`
- Do NOT include the `allowed-tools` block or any meta-commentary in the posted messages
- Dividers: do NOT use `---` in Slack messages — causes an `invalid_blocks` error
- LinkedIn URLs: always use the full `https://www.linkedin.com/in/slug/` format — never output just `/in/slug`
- Link URLs: never use percent-encoded characters (e.g. `%C3%AB`) in the URL portion of `<url|text>` — use simplified ASCII slugs instead
- Inline team tags: when a team member should be tagged for a specific founder entry (e.g. nationality-based), append their `<@USERID>` immediately after the founder's linked name on that bullet — NOT in the section header. Example: `• <https://www.linkedin.com/in/slug/|Coralie Chaufour> <@U0AAGADJCQ1>, ex-Entrepreneur First (Paris, France)`

---

## Step 7 — Show preview

Print the full formatted message(s) to the terminal. Then ask:

```
Does this look right? Type 'post' to send to #automation-tests, or paste a corrected version.
```

If Kieran pastes a corrected version, use that as the message content going forward. If they type 'post', proceed with what was previewed.

---

## Step 8 — Post to #automation-tests

Use `mcp__claude_ai_Slack__slack_send_message` with channel `C0AKKPK3J1K`.

If there are two parts, post them sequentially (Part 1 first, then Part 2).

Confirm with the Slack message permalink URL(s).

---

## Step 9 — Corrections

Ask:
```
Any corrections needed? Paste them (e.g. "CompanyX should be Fintech", "Name was spelled wrong → correct spelling") and I'll learn. Type 'no' to finish.
```

If corrections are given, append them to `~/.claude/projects/-Users-kvelasquez-Projects/memory/evertrace-signals-corrections.md` under the appropriate section:
- Theme routing overrides: `CompanyName → theme_key | added YYYY-MM-DD` (valid keys: physical, industrial, regulated, resilience, frontier, misc)
- Name corrections: `WRONG → CORRECT | added YYYY-MM-DD`
- Company name corrections: `WRONG → CORRECT | added YYYY-MM-DD`

Confirm what was saved.
