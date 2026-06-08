# Peer Review Assistant

A skill that helps you write a stronger peer review, faster. Upload a manuscript (or paste
an abstract), and it runs targeted Consensus searches to check whether the background claims
hold up, identify papers the authors should have cited, and flag whether the methods reflect
current practice.

The output is a structured review report as a Word document (.docx) — ready to edit, annotate,
and submit.

This is not a replacement for domain expertise. It's what a generous colleague would do if
they had time to cross-check the literature for you before you sat down to write.

## Installation

**Option 1: Download ZIP**

Click the green **Code** button at the top of this repo, then **Download ZIP**. Extract the ZIP and add the folder to your Claude skills directory.

**Option 2: Releases**

Go to the [Releases](https://github.com/stephenturner/skill-peer-review-assistant/releases) page and download the latest `.skill` file. Add it to your Claude skills in [customize/skills](https://claude.ai/customize/skills) on the web, or double-click it if you have Claude Desktop installed.

**Option 3: Build it yourself**

Build a `.skill` file from the source code, then add it to your Claude skills as described above.

```sh
git clone https://github.com/stephenturner/skill-peer-review-assistant.git
cd skill-peer-review-assistant
zip -r peer-review-assistant.skill SKILL.md references/
```

## Usage

Upload or paste the manuscript, then ask for a review, or directly call the skill with `/peer-review-assistant`. 
The skill reads the paper, runs the searches, and delivers the report.

**Trigger phrases:**

- "Help me review this manuscript"
- "Write a peer review for this paper"
- "Check if this paper is missing key citations"
- "Are the methods in this paper current?"
- "Critique this draft before I submit my review"
- "Find what this paper missed"

---

## Example queries

**1. AI model for clinical prediction**
> "I'm reviewing this paper on a transformer model for sepsis onset prediction from EHR data.
> Help me check whether the background claims are accurate, whether they've missed any recent
> benchmarks, and whether their evaluation metrics are the current standard."

*What the skill checks:* Whether the paper's framing of sepsis prediction accuracy gaps is
supported by recent literature, which high-impact EHR/sepsis ML papers are absent from the
reference list, and whether AUROC-only evaluation has been superseded by more informative
metrics in recent work.

---

**2. Microbiome-immunotherapy paper**
> "Can you help me review this manuscript? It's about gut microbiome composition predicting
> checkpoint inhibitor response in NSCLC. I want to know if their intro claims about the
> microbiome-immune axis are accurate and whether they've missed any landmark papers."

*What the skill checks:* Whether the mechanistic claims about microbiome-immune crosstalk
are well-supported, how the study compares to similar work in colorectal cancer and melanoma
that should contextualize the NSCLC findings, and whether 16S rRNA sequencing remains the
standard or whether metagenomic approaches have become expected.

---

**3. Protein language model paper**
> "I'm reviewing a paper applying ESM-2 embeddings to antimicrobial resistance phenotype
> prediction. Help me write the review — I want to check the methods, find missing citations,
> and see if the claim that 'no prior work has applied protein LLMs to AMR prediction' holds up."

*What the skill checks:* Whether that novelty claim is accurate (or whether the authors
missed adjacent work), whether ESM-2 is the current best choice or whether newer models
make the comparison incomplete, and which AMR prediction benchmarks the paper should be
evaluated against.

---

**4. Gene editing off-target effects**
> "Review this CRISPR base editing paper for me. Focus on whether the off-target detection
> methods are current — I have a feeling they're using an older approach that's been
> superseded."

*What the skill checks:* Whether the off-target detection method has been benchmarked against
more recent alternatives, which high-citation papers on base editing off-targets are absent
from the reference list, and whether the conclusions about specificity are consistent with
what comparable studies have found.

---

**5. Computational drug synergy prediction**
> "I need to review a paper on graph neural networks for predicting synergistic drug
> combinations. It's targeted at a bioinformatics journal. Help me check if it engages
> properly with both the GNN literature and the pharmacology literature — these papers
> often miss one side."

*What the skill checks:* Whether the paper cites the relevant GNN benchmarks from the
computational side, whether it engages with the pharmacological definitions of synergy
that a bioinformatics audience would expect, and whether the evaluation datasets are
standard in the field.

---

## Output

The review report (.docx) contains:

- **Summary**: what the paper does and overall impression
- **Background and claims**: whether the key intro assertions hold up against the literature
- **Missing citations**: papers found in Consensus that are relevant but absent from the reference list
- **Methods assessment**: whether the approach is current practice or has been superseded
- **Major concerns**: numbered, specific, with suggested remedies
- **Minor concerns**: numbered, presentation-level issues
- **Recommendation**: Accept / Minor revision / Major revision / Reject
- **Audit log**: every search query, result count, and plan-tier cap noted

