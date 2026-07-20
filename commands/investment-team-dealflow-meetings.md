---
description: Format Granola investment team dealflow meeting transcript into ready-to-send email summaries
allowed-tools: Write, Read, Bash(open:*)
---

# Investment Team Dealflow Meeting Notes Formatter

The user has pasted a Granola meeting transcript. Format it into ready-to-copy email blocks for Gmail.

## Step 1: Ask one question

Ask the user exactly this, then wait for their full answer before proceeding:

"One quick question:
1. Is this a meeting requiring Version 2 as well? (yes/no)"

## Step 2: Extract from the transcript

First, read the corrections memory file at `~/.claude/project-a/memory/investment-team-dealflow-meetings-corrections.md` using the Read tool. If it exists, load all corrections into context:
- **Name corrections**: apply case-insensitively to all first names found
- **Company name corrections**: apply case-insensitively to all company names found
- **HQ country corrections**: when a company appears in the transcript, check if its HQ country matches a correction entry — if so, replace it with the correct country

Note which corrections were applied (you'll report them in Step 5).

Then extract:
- **Date**: Find the meeting date. Format as DD.MM.YYYY (e.g. 07.03.2026). No slashes, no day name.
- **Sender name**: Find the note-taker/intern's first name for the sign-off.
- **Companies**: Group all companies into sections by their Affinity deal stage:
  - Legal & Financial DD
  - Due Diligence
  - Partner Calls
  - Omit any section with no companies.

Also extract the URL map directly from the transcript (company name → URL). The Granola transcript includes website URLs inline with each company entry.

## Step 3: Company entry format (both versions)

[Company Name] ([HQ Country]) - @[First names] // [content] // [content]

- HQ is always a **country** (not a city)
- Names after @ are first names only, comma-separated
- Use // to separate content segments
- No bullet points within entries

When writing company `<p>` tags in the HTML (Step 4), if a company has a URL in the URL map, format it as:

```html
<a href="https://company.com" style="color: #000; text-decoration: underline;">Company Name</a> (Germany) - @Johann // ...
```

If no URL: plain text as before. Apply to both V1 and V2.

---

## Version 1 — Internal Summary

**Audience:** Investment team only (internal)
**Tone:** Concise, factual, neutral — aim for 1–2 lines per company

**Include the most important points only:**
- What the company does (one short phrase)
- Key figures if directly relevant (raise size, ARR, round stage)
- IC outcome if applicable
- One clear next step
- A notable team impression or open question only if it was a central point of discussion

**Do not:** pad entries with every detail mentioned. If a point wasn't significant, leave it out. Shorter is better.

---

## Version 2 — Clean Summary

**Audience:** Wider partnership group (compliance-facing)
**Tone:** Factual, concise, neutral

**Include only:**
- Company name and HQ country
- Deal lead + responsible partner (first names)
- Brief factual description of what the company does
- Current deal status

**Strip everything:** internal commentary, team impressions/opinions, open questions, specific figures unless they define the deal stage, any color commentary.

---

## Step 4: Write HTML files to Desktop and open them

After formatting the content, use the Write tool to create HTML files on the Desktop, then open them with Bash.

### Always create Version 1 HTML file:

Write the file to `~/Desktop/investment-team-dealflow-meetings-v1.html` with this structure:

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<style>
  body { font-family: Arial, sans-serif; font-size: 14px; line-height: 1.6; max-width: 800px; margin: 40px auto; padding: 0 20px; color: #000; }
  .section-label { color: #888; font-size: 12px; margin-bottom: 4px; }
  .field-box { background: #f5f5f5; padding: 10px 14px; margin-bottom: 20px; border-radius: 4px; font-family: monospace; font-size: 13px; white-space: pre-wrap; word-break: break-all; }
  .email-body { padding: 10px 0; }
  .email-body p { margin: 0 0 16px 0; }
  b { font-weight: bold; }
  h2 { font-size: 16px; border-bottom: 1px solid #ddd; padding-bottom: 6px; }
</style>
</head>
<body>

<h2>Version 1 — Internal Summary</h2>

<div class="section-label">RECIPIENTS — paste into To: field</div>
<div class="field-box">Investment Team &lt;investmentteam@project-a.vc&gt;, Anton Waitz &lt;anton.waitz@project-a.vc&gt;, Uwe Horstmann &lt;uwe.horstmann@project-a.vc&gt;, Florian Heinemann &lt;florian.heinemann@project-a.vc&gt;, Thies Sander &lt;thies.sander@project-a.vc&gt;, Philipp Werner &lt;philipp.werner@project-a.vc&gt;, Malin Posern &lt;malin.posern@project-a.vc&gt;, Jack Wang &lt;jack.wang@project-a.vc&gt;</div>

<div class="section-label">SUBJECT</div>
<div class="field-box">Internal Summary Dealflow [DATE]</div>

<div class="section-label">BODY — select all text below and copy into Gmail</div>
<div class="email-body" id="v1-body">
<p>Hi everyone,</p>
<p>Please find below the internal update on today's deal flow meeting:</p>
<p><b>Dealflow</b></p>
<p><b>Legal &amp; Financial DD</b></p>
<p>[company entries]</p>
<p><b>Due Diligence</b></p>
<p>[company entries]</p>
<p><b>Partner Calls</b></p>
<p>[company entries]</p>
<p>Best,<br>[Sender first name]</p>
</div>

</body>
</html>
```

Fill in:
- `[DATE]` with the formatted date
- `[Sender first name]` with the sender's name
- Each `[company entries]` block with the actual company entries for that section, each as its own `<p>` tag
- Remove any section (including its bold header) that has no companies

### If user answered "yes" to V2, also create Version 2 HTML file:

Write the file to `~/Desktop/investment-team-dealflow-meetings-v2.html` with the same structure but:
- Title: `Version 2 — Clean Summary`
- Recipients: `Investment Team &lt;investmentteam@project-a.vc&gt;, Anton Waitz &lt;anton.waitz@project-a.vc&gt;, Vincent Synde &lt;vincent.synde@project-a.vc&gt;, Miriam Ayasse &lt;miriam.ayasse@project-a.vc&gt;, Martin Laudien &lt;martin.laudien@project-a.vc&gt;, Christian Kurz &lt;christian.kurz@project-a.vc&gt;, Andreas Kühnke &lt;andreas.kuehnke@project-a.vc&gt;, Anton Grabovski &lt;anton.grabovski@project-a.vc&gt;, Elias Wahl &lt;elias.wahl@project-a.vc&gt;, Christoph Heiland &lt;christoph.heiland@project-a.vc&gt;, Sebastian Köppe &lt;sebastian.koeppe@project-a.vc&gt;`
- Subject: `Summary Dealflow [DATE]`
- Body: `Please find below the update on today's deal flow meeting:`
- Entries: clean/factual version only

### Then open the file(s):

Use Bash to run:
- `open ~/Desktop/investment-team-dealflow-meetings-v1.html`
- `open ~/Desktop/investment-team-dealflow-meetings-v2.html` (if V2 was created)

## Step 5: Tell the user

After opening the file(s), output this message (adjust based on corrections applied):

"Done. The file(s) have opened in your browser.

To copy the email body into Gmail:
1. Select all the body text in the browser (click into the body, Cmd+A won't work — manually select from 'Hi everyone' to your name)
2. Cmd+C to copy
3. Click into Gmail compose body → Cmd+V to paste (NOT Cmd+Shift+V)

Recipients and subject are shown above the body — copy those separately into the To: and Subject: fields."

If any corrections from the corrections memory were applied, append: "Applied [N] automatic correction(s) from memory: [list them]."

## Step 6: Ask for corrections feedback

After the Done message, ask exactly this and wait for their response:

"Did the output need any name or company corrections? If yes, paste your corrected version below and I'll learn from it. Type 'no' to finish."

## Step 7: Learn from corrections (only if user pastes corrected output)

If the user types 'no', stop here.

If the user pastes a corrected version:
- Compare it against the output you generated
- Identify changed first names, company names, and HQ countries
- Check if each correction already exists in the corrections file
- If the corrections file doesn't exist yet, create it at `~/.claude/project-a/memory/investment-team-dealflow-meetings-corrections.md` with this header:

```markdown
# Investment Team Dealflow Meetings — Corrections Memory

## Name Corrections
<!-- Format: WRONG → CORRECT | added YYYY-MM-DD -->

## Company Name Corrections
<!-- Format: WRONG → CORRECT | added YYYY-MM-DD -->

## HQ Country Corrections
<!-- Format: CompanyName: WRONG → CORRECT | added YYYY-MM-DD -->
```

- Append only new (not already present) corrections to the appropriate section:
  - Names: `- WRONG → CORRECT | added YYYY-MM-DD`
  - Company names: `- WRONG → CORRECT | added YYYY-MM-DD`
  - HQ countries: `- CompanyName: WRONG → CORRECT | added YYYY-MM-DD`
- Tell the user: "Learned [N] new correction(s): [list]. Applied automatically on future runs."

---

## Key rules

- Section order: Legal & Financial DD → Due Diligence → Partner Calls
- Omit any section (including its bold header) that has no companies
- Date format: DD.MM.YYYY — no slashes, no day name
- HQ is always country, never city — if HQ cannot be determined from the transcript or corrections memory, leave it blank (e.g. `Company Name () -`) for the user to fill in
- Each company entry is its own `<p>` tag in the HTML — no wrapping or indentation issues
- Company links use inline `style` attributes (not CSS classes) — Gmail strips `<style>` block rules on copy-paste; inline styles survive
