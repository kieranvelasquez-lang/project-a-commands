---
description: Monthly European funding round retro — source, cross-reference Affinity, output email
allowed-tools: Read, Write, WebFetch, WebSearch, Bash(open:*), mcp__claude_ai_Affinity__search_companies, mcp__claude_ai_Affinity__semantic_search, mcp__claude_ai_Affinity__get_company_list_entries, mcp__claude_ai_Affinity__get_single_list_entry
---

# Deal Flow Retro Newsletter

Monthly digest of European Pre-Seed and Seed funding rounds — cross-referenced against Affinity — formatted as a ready-to-send email to the investment team and all partners, organized by investment thesis.

---

## Step 1 — Confirm date range

Based on today's date, suggest the most recently completed calendar month (e.g. if today is 1 Jun 2026, suggest "1 May – 31 May 2026").

Ask the user:

> "What month should this retro cover? Suggested: **[suggested range]**. Confirm or provide a different range."

Wait for confirmation before proceeding.

---

## Step 2 — Crunchbase CSV import

Ask the user to export data from Crunchbase Pro:

> "Please export this period's European funding rounds from Crunchbase Pro:
>
> 1. Go to **Crunchbase Pro → Search → Funding Rounds**
> 2. Set filters:
>    - **Headquarters Location:** Austria, Belgium, Bulgaria, Croatia, Cyprus, Czech Republic, Denmark, Estonia, Finland, France, Germany, Greece, Hungary, Ireland, Italy, Latvia, Lithuania, Luxembourg, Malta, Netherlands, Poland, Portugal, Romania, Slovakia, Slovenia, Spain, Sweden, United Kingdom, Switzerland, Norway
>    - **Announced Date:** [Start Date] to [End Date]
>    - **Funding Type:** Pre-Seed, Seed
> 3. Export to CSV
> 4. Paste the file path here (e.g. ~/Downloads/crunchbase-export.csv)"

Read the CSV using the Read tool. Map columns flexibly — Crunchbase column names vary but typically include:

| Crunchbase Column | Internal Field |
|---|---|
| Organization Name | Company |
| Organization Name URL | Website |
| Description / Short Description | Description |
| Funding Round Type | Stage |
| Money Raised | Amount |
| Lead Investors | Lead Investors |
| Headquarters Location | Country |
| Announced Date | Date |

If a column is missing or named differently, infer from context. Strip city names from Country (keep country only).

**Name/domain mismatch check:** After parsing, flag any row where the company name and the Organization Website domain look inconsistent (e.g. company name is "CodeWords" but website is `agemo.ai`). Print the flagged rows and ask:
> "The following companies have a potential name/website mismatch — please confirm the correct website URL before we continue:
> - [Company Name]: CSV website is [url] — is this correct, or should I use a different URL?
>
> Type the correct URL or 'ok' to keep as-is."

The website URL embedded in the HTML and used for Affinity domain searches must always come from the CSV (or a correction you provide here). Never substitute a different URL silently.

---

## Step 3 — Supplementary sourcing (EU-Startups)

WebFetch the EU-Startups weekly funding round-up articles covering the same period.

1. Fetch `https://www.eu-startups.com/category/funding/` to find the relevant weekly articles for the date range
2. Fetch each relevant article and parse: company name, country, amount, stage, investors, brief description
3. Compare against the Crunchbase data by company name (case-insensitive, strip legal suffixes like "GmbH", "Ltd", "SAS")
4. Add any companies NOT already present as supplementary entries

If the user also provides a Dealroom CSV path, read it and merge using the same deduplication logic. Dealroom data takes precedence over Crunchbase on conflicting fields.

**Stage filter:** After merging all sources, keep only: Pre-Seed, Seed. Remove everything else: Series A, Series A+, Series B+, Undisclosed, Fund closes, Growth Equity. Do not include a "Source" column in any output.

**Country filter:** After applying the stage filter, remove any company whose headquarters is **not** in the following list: Austria, Belgium, Bulgaria, Croatia, Cyprus, Czech Republic, Denmark, Estonia, Finland, France, Germany, Greece, Hungary, Ireland, Italy, Latvia, Lithuania, Luxembourg, Malta, Netherlands, Poland, Portugal, Romania, Slovakia, Slovenia, Spain, Sweden, United Kingdom, Switzerland, Norway. This explicitly excludes Turkey and any other non-listed country.

---

## Step 4 — Preview structured table + curation

Print the full filtered list as-is from the CSV/sources — **do not enrich yet**. Show whatever data is already present; missing fields display as "—".

```
[N] companies after filtering — [Start Date] – [End Date]

# | Company            | Country   | Stage   | Amount | Lead Investors        | Description
--|--------------------|-----------|---------:|--------|----------------------|------------------------------
1 | Acme AI            | Germany   | Seed     | €12M   | Sequoia, Index       | AI-powered logistics platform
2 | Beta Labs          | France    | Pre-Seed | —      | —                    | —
...
```

Then ask:

> "Above is the full filtered list of [N] companies. Please select up to 50 to include in the retro:
> - Type **'all'** to use everything (only if ≤50)
> - List row numbers to **include**: e.g. `1,3,5-12,20`
> - List row numbers to **remove**: e.g. `remove 4,7,15`"

Apply the selection. Confirm final count before proceeding.

---

## Step 5 — Enrich missing fields (selected 50 only)

Now enrich only the selected companies. For any row missing a website URL or a description:
1. WebSearch `[Company Name] startup [Country]` to identify the likely official website
2. **Verify the URL before using it**: WebFetch the homepage to confirm it resolves and the content matches the company
   - If the fetch succeeds and content matches → use the URL and extract a 1-sentence description
   - If the fetch returns an error (403, 404, timeout, domain-for-sale page, or unrelated content) → do **not** embed the URL; mark the website field as `[needs review]`
3. Never embed a URL that has not been successfully verified via WebFetch

If a website or description cannot be found or verified after searching, flag that specific row with `[needs review]` in the relevant field.

---

## Step 6 — Thesis routing

After curation, assign each company to one of the six thesis areas using its description, sector, and country.

**Routing rules:**
- **Physical World Intelligence** — space (civilian/non-defense), ocean, land, subsurface, agriculture, infrastructure, construction, energy (hardware and software for the physical world), new materials, robotics infrastructure (software/AI-first: foundation models, physical intelligence, robot OS; hardware/physical world: space, ocean, agriculture, construction), AI inference infrastructure generally (inference serving, inference optimization/hardware-adjacent infra — not just robotics-specific)
- **Industrial Autonomy** — manufacturing, factory automation, factory software, supply chain, logistics
- **Regulated Industries** — healthcare tech, fintech, legal tech, gov tech, insurance tech, compliance
- **European Resilience** — defense, military, military space, dual-use hardware, cybersecurity infrastructure (not consumer security)
- **Frontier Tech** — semiconductors, quantum computing, frontier biotech, breakthrough energy hardware (e.g. fusion), novel AI architectures, fundamental CS algorithm research, competitive-programming/algorithmic-research background (route here on any plausible fit, even from a terse note)
- **Miscellaneous** — AI agents, LLM platforms, dev tools, enterprise SaaS (applied/practitioner work with no research or competitive-programming signal), gaming, consumer, edtech (no team member assigned — visibility only); excludes AI inference infra (routes to Physical World Intelligence)

When a company spans two areas, pick the primary one based on what they're **selling**, not what they use internally (e.g. an AI company selling into manufacturing → Industrial Autonomy if the product is hardware/process automation; → Miscellaneous if the product is software/AI).

Print the routing table:

```
# | Company        | Assigned Thesis         | Reason
--|----------------|-------------------------|--------------------------------------------
1 | Acme AI        | Physical World Intelligence | Software platform for warehouse automation
2 | Beta Labs      | European Resilience     | Drone swarm systems for defense applications
...
```

Ask:
> "Any routing corrections? Type 'ok' to proceed or list corrections (e.g. `3: Industrial Autonomy`, `7: Frontier Tech`)."

Apply corrections, then proceed.

---

## Step 7 — Affinity MCP contact check

Run an automated check against the Affinity Master Deals List (list ID 99030). **First run on the initial 5 companies as a test batch** — print results and wait for confirmation before running the rest.

### Per-company process

**Step A — Find company in Affinity:**
1. Call `search_companies(term=[Company Name])` → get `company_id`
   - If no match, try `semantic_search(query=[Company Name] [Country])`
   - If still no match → not in Affinity at all → `affinity_link = null`, `seen = false`, `in_contact_12mo = false`

**Step B — Check Master Deals List (list 99030):**
2. Call `get_company_list_entries(company_id=[company_id])` → look for an entry where `listId == 99030`
   - If none found → not in Master Deals List → `affinity_link = https://projecta.affinity.co/companies/[company_id]` (company exists in Affinity but not in pipeline), `seen = false`, `in_contact_12mo = false`
   - If found → record `list_entry_id`; set `affinity_link = https://projecta.affinity.co/companies/[company_id]`

**Step C — Check contact history:**
3. `seen = true` (already established by being in list 99030 — Step B above)
4. Call `get_single_list_entry(list_id=99030, list_entry_id=[list_entry_id], field_types=['relationship-intelligence'])`
   - Look for both `last-contact` and `last-interaction` fields in the response
   - If **both are null** → never formally contacted → `in_contact_12mo = false`
   - If **either has a date within the last 12 months** → `in_contact_12mo = true`
   - If **either has a date older than 12 months** → `in_contact_12mo = false`

### Test batch (first 5 companies)

Print raw results for the first 5:

```
Test batch — 5 companies

# | Company        | In Affinity? | In List 99030? | Last Contact / Interaction | Seen? | In Contact 12mo?
--|----------------|--------------|----------------|---------------------------|-------|------------------
1 | Acme AI        | Yes          | Yes            | 2025-09-03 (last-contact) | Yes   | Yes
2 | Beta Labs      | Yes          | Yes            | 2023-02-16 (last-contact) | Yes   | No (old)
3 | Gamma Systems  | Yes          | Yes            | null / null               | Yes   | No
4 | Delta Corp     | Yes          | No             | —                         | No    | No
5 | Echo Ventures  | No           | —              | —                         | No    | No
```

Ask:
> "Does this test batch look correct? If the logic looks right, type 'continue' to run the remaining [N] companies. Otherwise describe any issues."

Wait for confirmation, then run the full batch. After completing all companies, print the full summary table and ask:
> "Full Affinity check complete. Does this look correct? Type 'ok' to proceed."

---

## Step 8 — Pre-HTML missing info check

Before generating HTML, scan all 50 companies for missing data. If any row still has `[needs review]` in website, lead investors, or description, pause and output:

> "The following companies are missing information — please provide corrections before I generate the HTML:
> - [Company]: missing [website / lead investors / description]
> - ...
>
> Paste corrections in the format: `[Company Name]: website=[url], investors=[names], description=[text]`
> Type 'skip' to generate with blanks (rendered as '—')."

Wait for the user's response and apply any corrections before proceeding.

---

## Step 9 — Generate HTML email file

Write the file to `/Users/kvelasquez/Desktop/dealflow-retro-newsletter.html` using the Write tool.

Calculate:
- **CW number**: ISO week number of the **end date** of the period
- **Summary line**: "[N] rounds tracked across 6 thesis areas. [X] seen, [Y] in contact within 12 months."

### HTML structure

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<style>
  body { font-family: Arial, sans-serif; font-size: 13px; line-height: 1.5; max-width: 1100px; margin: 40px auto; padding: 0 20px; color: #000; }
  .section-label { color: #888; font-size: 11px; margin-bottom: 4px; text-transform: uppercase; letter-spacing: 0.05em; }
  .field-box { background: #f5f5f5; padding: 10px 14px; margin-bottom: 24px; border-radius: 4px; font-family: monospace; font-size: 12px; white-space: pre-wrap; word-break: break-all; }
  .email-body { padding: 10px 0; }
  .email-body p { margin: 0 0 16px 0; }
  h2 { font-size: 15px; border-bottom: 1px solid #ddd; padding-bottom: 6px; margin-bottom: 20px; }
</style>
</head>
<body>

<h2>Deal Flow Retro Newsletter — CW [X] | [Start Date] – [End Date]</h2>

<div class="section-label">RECIPIENTS — paste into To: field</div>
<div class="field-box">Investment Team &lt;investmentteam@project-a.vc&gt;, Anton Waitz &lt;anton.waitz@project-a.vc&gt;, Uwe Horstmann &lt;uwe.horstmann@project-a.vc&gt;, Florian Heinemann &lt;florian.heinemann@project-a.vc&gt;, Thies Sander &lt;thies.sander@project-a.vc&gt;, Philipp Werner &lt;philipp.werner@project-a.vc&gt;, Malin Posern &lt;malin.posern@project-a.vc&gt;, Jack Wang &lt;jack.wang@project-a.vc&gt;</div>

<div class="section-label">SUBJECT</div>
<div class="field-box">Deal Flow Retro Newsletter — CW [X] | [DD Mon] – [DD Mon YYYY]</div>

<div class="section-label">BODY — select all text below and copy into Gmail</div>
<div class="email-body">

<p>Hi everyone,</p>

<p>Please find below this month's European funding round retro newsletter covering <strong>[Start Date] – [End Date]</strong>. [Summary line]</p>

<!-- ONE SECTION PER THESIS — repeat this block 5 times -->
<h3 style="font-family:Arial,sans-serif;font-size:14px;margin:28px 0 10px 0;border-left:3px solid #444;padding-left:10px;">[Thesis Name]</h3>
<table style="border-collapse:collapse;width:100%;font-family:Arial,sans-serif;font-size:12px;margin-bottom:32px;">
  <tr>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">#</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Company</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Country</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Stage</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Amount</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Lead Investors</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Description</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Seen?</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">In Contact (12mo)?</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Affinity</th>
  </tr>
  <!-- INSERT ROWS HERE -->
</table>

<p>Best,<br>Kieran</p>

</div>
</body>
</html>
```

### Row template (all styles inline — Gmail strips style blocks)

```html
<tr style="background:[#fff or #fafafa alternating];">
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;">[#]</td>
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;"><a href="[website]" style="color:#000;text-decoration:underline;">[Company Name]</a></td>
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;">[Country]</td>
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;">[Pre-Seed or Seed]</td>
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;">[€12M or —]</td>
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;">[Lead Investors or —]</td>
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;color:#555;">[Description or —]</td>
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;font-weight:bold;color:[#1a7a1a if Yes, #c0392b if No];">[Yes or No]</td>
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;font-weight:bold;color:[#1a7a1a if Yes, #c0392b if No];">[Yes or No]</td>
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;">[<a href="[affinity link]" style="color:#1a5fa8;">View</a> or —]</td>
</tr>
```

### Sort order within each thesis table

1. Pre-Seed rows first, then Seed rows
2. Within each stage group: Seen=No rows first (alphabetical), then Seen=Yes rows (alphabetical)

### Thesis table order in the email

1. Physical World Intelligence
2. Industrial Autonomy
3. Regulated Industries
4. European Resilience
5. Frontier Tech
6. Miscellaneous

### Field rules

- **Company**: linked inline to verified website; plain text if no website available
- **Amount**: format as "€12M" / "$5M" / "£8M"; render "—" if unknown — never write "Unknown"
- **Lead Investors**: up to 3 names; append `+ N more` if there are more; "—" if unknown
- **Description**: 3–5 word one-liner (e.g. "AI-powered logistics platform"); "—" if unavailable
- **Seen?**: bold green `#1a7a1a` if Yes; bold red `#c0392b` if No
- **In Contact (12mo)?**: bold green `#1a7a1a` if Yes; bold red `#c0392b` if No
- **Affinity**: `<a href="..." style="color:#1a5fa8;">View</a>` for any company found in Affinity (regardless of seen/contact status); "—" only if company has no Affinity presence at all

---

## Step 10 — Open in browser

```bash
open /Users/kvelasquez/Desktop/dealflow-retro-newsletter.html
```

---

## Step 11 — Done message

Output:

```
Done. [N] companies in this retro — [X] seen, [Y] in contact within 12 months.

To copy the email body into Gmail:
1. Select all body text in the browser (from "Hi everyone" to "Kieran")
2. Cmd+C to copy
3. Click into Gmail compose body → Cmd+V (NOT Cmd+Shift+V — preserves table formatting)

Recipients and subject are shown above the body — copy those separately.
```

---

## Key rules

- Never write "Unknown" — use "—"
- All table styles must be inline (Gmail strips `<style>` block rules on copy-paste)
- Subject format: `Deal Flow Retro Newsletter — CW [X] | [DD Mon] – [DD Mon YYYY]`
- CW number = ISO week number of the period's **end date**
- Country = country only, never city (strip city names from Crunchbase location strings)
- Stage = Pre-Seed or Seed only — no Series A in any output
- Website URLs must be verified via WebFetch before embedding — never use an unverified URL
- Country whitelist: EU 27 + UK + Switzerland + Norway only — all others excluded
- Affinity test gate: always run the first 5 companies as a test batch and wait for confirmation before processing the rest
- **Seen?** = company is in Master Deals List (list 99030) — this alone is sufficient; no contact check required
- **In Contact (12mo)?** = `last-contact` or `last-interaction` date exists in Affinity (email, meeting, or formal interaction synced) and is within 12 months of today
- **Affinity column**: link for any company in Affinity; "—" only if not found in Affinity at all
- Affinity MCP call sequence: `search_companies` → `get_company_list_entries` (filter `listId==99030`) → `get_single_list_entry(field_types=['relationship-intelligence'])` → check both `last-contact` and `last-interaction`
- Affinity Master Deals List: https://projecta.affinity.co/lists/99030
