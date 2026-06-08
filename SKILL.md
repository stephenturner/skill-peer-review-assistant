---
name: peer-review-assistant
description: >
  Peer review assistant powered by Consensus. Given a manuscript (PDF, DOCX, abstract, or pasted
  text), runs 4-6 targeted searches to verify background claims, identify missing citations, and
  assess whether the methods reflect current practice. Produces a structured review report as a
  Word document with sections for background accuracy, citation gaps, methods evaluation, and an
  overall recommendation.

  Trigger when the user asks: "help me review this paper", "write a peer review for this",
  "check if this paper is missing citations", "are the methods current?", "critique this
  manuscript", "find what this paper missed", or uploads a paper and asks for feedback or a
  formal review.

  Do NOT trigger for literature reviews of a topic the user is researching (use
  literature-review-helper), grant writing (use consensus-grant-finder), or research question
  validation (use research-question-validator).
---

# Peer Review Assistant

## Overview

This skill takes a manuscript and produces a structured peer review report grounded in what
Consensus finds. It is not a replacement for domain expertise — it is a force multiplier for
a reviewer who already has it. The searches surface what the authors may have missed: papers
they should have cited, methods that have been superseded, and claims that are inconsistent
with the broader literature.

Output: a Word document (.docx) the reviewer can edit, annotate, and submit or share.

---

## Data Integrity Principles

Everything in the review report must come from what the manuscript contains or what Consensus
returned in this session. Reviewers sign their names (or their confidentiality) to this work
— a fabricated missing citation or a hallucinated claim check can embarrass the reviewer and
mislead the authors.

**Only cite what Consensus returned.** Never include papers from training knowledge without
labeling them `[Not from Consensus — model knowledge]` and excluding them from all citation
counts. If a paper seems obviously relevant but Consensus didn't return it, note the omission
— don't manufacture the reference.

**Confirm before proceeding.** A search is not complete until the result is received and
inspected. Never mark a search done based on intent.

**Detect and report plan-tier caps.** After the first search, note how many results were
returned. If the response indicates a cap ("showing top 10 of 20 found"), log it and calibrate
confidence accordingly. A plan-tier ceiling can make a field look sparse — say so rather than
overstating the gap.

**Surface gaps, don't fill them.** If a search returns zero results, report it as a dimension
with incomplete coverage, not as confirmation that the authors are correct. Zero results on a
methods check may mean the terminology missed, not that the method is uncontested.

---

## Error Handling

1. On any search failure: wait 3 seconds, retry once.
2. Log every failure — which search, what error, whether the retry succeeded.
3. After 3 consecutive failures: stop and alert the user. Deliver whatever was collected before the failures and note which review dimensions are incomplete.
4. Never silently skip a failed search. Mark it in the report as `[Search failed — not included]`.

---

## Workflow

### Phase 1: Parse the Manuscript

Read the uploaded file or pasted text and extract:

- **Title and authors**
- **Research question or hypothesis**: what the paper claims to study or demonstrate
- **Key background claims**: 2-4 factual assertions in the introduction that the paper's
  framing depends on (e.g., "Current methods for X are limited by Y", "No study has examined Z")
- **Methods**: study design, datasets used, models or statistical approaches, evaluation
  metrics or endpoints
- **Main findings**: what the paper reports as its primary results
- **Conclusions**: what the authors claim their findings imply

If only an abstract is provided, extract what you can and note in the report that the review
is abstract-only and certain dimensions (especially methods) will have limited coverage.

Flag any claims in the abstract or introduction that are stated without a citation — these are
candidates for the claims-verification searches.

---

### Phase 2: Run 4-6 Consensus Searches Sequentially

Run all searches one at a time with at least a 1-second pause between calls. Confirm each
result before sending the next. Track: query sent, results received, results cited.

**Search 1 — Field consensus on the central question** (always run)
What does the broader literature say about the paper's central research question? This
establishes whether the paper's framing and conclusions align with or contradict the field.
```
query: "<core research question or topic as a focused academic query>"
```
Pay attention to: direction of findings (do most papers agree?), presence of systematic
reviews or meta-analyses (which would be the authoritative reference the authors should cite),
and any high-citation papers that are absent from the manuscript's reference list.

**Search 2 — Background claim verification** (always run)
Take the most important unsupported or load-bearing claim from the introduction and check it.
```
query: "<the specific claim as a searchable statement>"
year_min: <current year - 5>
```
Compare what Consensus returns against what the paper asserts. If the evidence is mixed or
points the other way, that is a major concern to flag. If the claim is well-supported, note
that the authors' framing is accurate — positive findings belong in the review too.

**Search 3 — Citation completeness** (always run)
Search for recent, high-impact papers in this space that the manuscript may have missed.
```
query: "<core topic and method or domain>"
year_min: <current year - 3>
```
Compare author names and titles in the results against the manuscript's reference list. Any
paper with high citation count or direct relevance that is absent is a candidate for the
"missing citations" section of the report. Do not flag a paper as missing unless it is
genuinely relevant — avoid the reviewer habit of demanding citations to their own work.

**Search 4 — Methods currency** (always run)
Are the methods the paper uses the current standard, or have they been superseded by better
approaches that the authors don't acknowledge?
```
query: "<method name or approach> <domain or application>"
year_min: <current year - 3>
```
Look for: newer methods that substantially outperform what the paper uses (without being
acknowledged), benchmarking papers that would contextualize the paper's results, or papers
that identified known limitations of the method the authors are applying.

**Search 5 — Second background claim or specific controversy** (run if the manuscript makes
a second major unsupported claim, or if Search 1 surfaces a controversy the authors didn't
acknowledge)
```
query: "<second claim or contested finding>"
```

**Search 6 — Adjacent literature** (run if the manuscript operates at the intersection of two
fields, e.g., AI applied to biology, and the authors appear to have cited only one side)
```
query: "<adjacent field concept> <paper's domain>"
```
This is especially common in AIxBio papers where authors from a computational background
may have missed key biology literature, or vice versa.

After all searches complete, record the final tally: searches sent, results received per
search, any failures or retries.

---

### Phase 3: Synthesize the Review

Before writing, make two structural assessments:

**Tone calibration**: Is this a manuscript with a strong core idea that needs refinement
(lean constructive), a paper with a fundamental flaw that undermines the conclusions (lean
direct), or a paper that is clearly out of scope for the target journal? Calibrate the
tone of the report accordingly — peer review is most useful when it is honest but specific.

**Severity triage**: Separate concerns into:
- **Major concerns**: issues that affect the validity of the conclusions (missing key control,
  wrong statistical test, central claim contradicted by the literature, missing seminal citation
  that would change interpretation)
- **Minor concerns**: presentation issues, missing non-critical citations, terminology, scope
  of discussion claims

A good review has 1-4 major concerns and 2-6 minor ones. More than that usually signals
the reviewer is padding. Fewer major concerns than the evidence warrants is a disservice
to the field.

---

### Phase 4: Generate the .docx Report

Read the DOCX skill at `/mnt/skills/public/docx/SKILL.md` before generating. Write the
document as a Node.js script using the `docx` library and execute via bash_tool.

Install if needed:
```bash
npm install -g docx
```

Save to `/mnt/user-data/outputs/peer-review-[short-title].docx`.

#### Report structure

**Header block** (not a numbered section)
- Manuscript title
- Date of review
- Note: "This review was prepared with the assistance of Consensus (consensus.app), which
  was used to search peer-reviewed literature for claim verification, citation completeness,
  and methods assessment. All search results are documented in the Audit Log."

**Section 1 — Summary**
2-3 sentences: what the paper does, what it claims to show, and one sentence on the overall
impression. This is what the editor reads first — make it accurate and specific.

**Section 2 — Background and Claims**
Assessment of the key claims in the introduction. For each claim checked:
- State the claim as the authors wrote it
- What Consensus found: supporting evidence, contradictory evidence, or inconclusive results
- Verdict: Accurate and well-supported / Partially supported — important nuance missing /
  Contradicted by recent literature / Unverifiable with available searches

If a claim is accurate, say so. Reviewers who only flag problems give authors an incomplete picture.

**Section 3 — Missing Citations**
Papers found in Consensus searches that are absent from the manuscript's reference list and
are directly relevant to the paper's claims, methods, or conclusions. For each:
- Full citation with hyperlink to Consensus URL
- One sentence on why this paper is relevant and what the authors should engage with it on

Only flag papers that are genuinely relevant. Do not pad this section. If no important papers
were missed, say so — that is useful information for the editor.

**Section 4 — Methods Assessment**
For each method or approach the paper uses:
- Is this the current standard, or have better approaches been published?
- Are there known limitations of this method that the authors should acknowledge?
- Do the evaluation metrics or endpoints match what is standard in this field?

Cite specific papers from Search 4 when flagging methods concerns.

**Section 5 — Major Concerns**
Numbered list. Each concern:
- A clear statement of the problem
- Why it matters for the paper's conclusions
- What the authors would need to do to address it (specific, not vague)

**Section 6 — Minor Concerns**
Numbered list, same format but for issues that don't threaten the conclusions.

**Section 7 — Recommendation**
One of: Accept / Minor revision / Major revision / Reject with invitation to resubmit /
Reject. One sentence justifying the recommendation based on the major concerns listed above.
Do not hedge with long qualifications — a clear recommendation is more useful than a
diplomatic non-answer.

**Section 8 — Audit Log**
Full transparency on the search process. Includes:

| # | Query | Filters | Results Received | Purpose |
|---|-------|---------|-----------------|---------|
| 1 | [query] | [year_min etc.] | [N] | Field consensus |
| 2 | [query] | | [N] | Claim verification |
| ... | | | | |

- Detected Consensus plan tier and per-query cap
- Any searches that failed and were not recovered
- Note: "Papers cited in this review were drawn only from Consensus results returned in
  this session. Papers not returned by Consensus are not cited regardless of relevance
  from prior knowledge."

#### Styling

- Arial 12pt body, navy (#1a3a5c) headings
- Major concerns in bold; minor concerns in regular weight
- Table header rows: light blue (#e8f0f8) fill
- Keep the report to 3-5 pages excluding the audit log — a review longer than that
  typically contains padding

---

### Phase 5: Deliver

Save the .docx and present with `present_files`.

Brief in-chat summary:

> "Peer review report saved as [filename]. Key findings:
> - **Overall**: [one sentence recommendation]
> - **[N] major concerns** — [the most important one in a clause]
> - **[N] missing citations** identified — [most notable one]
> - **Methods**: [one sentence on currency of methods]
>
> The Audit Log in the document shows every search query and result count. All citations
> link to Consensus."

---

## Notes

- **Rate limit**: 1 Consensus query per second. Always sequential, never parallel.
- **Abstract-only reviews are possible but limited**: note in the report which sections have
  reduced coverage and why. Methods assessment in particular requires the full methods section.
- **Do not invent concerns**: if the searches don't surface a problem on a given dimension,
  say the dimension looks solid rather than generating generic criticism.
- **Positive findings matter**: note when background claims are accurate, when the methods
  are current, and when the citation list appears complete. An honest review acknowledges
  strengths — it's more useful to authors and signals a credible reviewer to the editor.
- **Conflict of interest flag**: if any author name in the manuscript matches a frequent
  author in the Consensus results (a sign the reviewer may know the authors' prior work),
  note this in the chat summary so the user can assess whether to disclose.
- **Terminology sensitivity**: AI/biology intersection papers often use field-specific jargon.
  If Search 1 returns sparse results, try a terminology variant before concluding the field
  is thin. Note any terminology adjustments in the audit log.
