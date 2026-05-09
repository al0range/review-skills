---
name: paper-review
description: Review and analyze academic research papers. Use this skill when the user asks to review a paper, analyze a publication, summarize research, critique methodology, extract key findings, compare papers, check for numerical inconsistencies, or assess novelty and contributions of academic work. Also triggers when the user mentions reading a PDF of a paper, wants a literature review, asks about related work, or wants to improve a paper before submission. All review dimensions and output formats are strictly driven by the target venue's review rubric — this skill does not impose its own fixed evaluation criteria.
---

# Research Paper Review

Assist researchers in reviewing, analyzing, and criticizing academic papers systematically and thoroughly.

## When to Use

- User asks to review, summarize, or critique a research paper
- User shares a PDF or link to an academic paper
- User wants to assess methodology, contributions, or novelty
- User needs help writing a peer review
- User wants to compare multiple papers or do a literature survey
- User wants to improve a paper before submission (pre-submission review)
- User wants to check for numerical/statistical inconsistencies
- User wants venue-specific feedback (conference, journal, or preprint)

## Inputs

- Paper content: PDF, LaTeX source, or plain text
- **Target Venue** (strongly recommended): conference, journal, or preprint target
  - Example: "Models 2026", "TOSEM", "SoSyM", "NeurIPS", "ICML", "arXiv preprint"
- **Type of paper** (strongly recommended): "full research" paper, "short" paper, "new ideas" paper, "tool demo", "poster", etc.
- **Review Rubric** (strongly recommended): the venue's official review form, evaluation criteria, or reviewer guidelines. This can be:
  - A file path to a rubric document (e.g., `./MM review.md`)
  - A URL to the venue's reviewer guidelines
  - Pasted text of the review criteria
  - If not provided, the skill will attempt to locate the rubric from the venue website.

---

## Review Principles

All review comments must strictly follow the dimensions, items, and weights defined in the review rubric document. Do not introduce any evaluation criteria outside the rubric, and do not omit any item explicitly listed in the rubric.

For every review point, explicitly cite the corresponding evidence from the paper, including specific paragraphs, figures, tables, or experimental results. If a claim lacks direct supporting evidence, explicitly state "insufficient evidence" and describe the associated uncertainty.

The final report must include:

- Itemized comments for each evaluation dimension (including evidence references and confidence levels),
- An overall summary of strengths and weaknesses,
- And concrete revision suggestions for the editor and/or authors.

If the rubric requires scoring, provide scores strictly according to the rubric's grading scale or point system.

---

## Mandatory Constraints

The following two behavioral constraints take precedence over all other objectives and must never be violated:

### 1. Patience and Quality First

Prioritize the quality of conclusions over speed. Do not proceed to any subsequent step, decision, or external action until every assigned subagent has either completed its report or explicitly reported a verified failure. Fully collect and consolidate the outputs of all subagents; reconcile disagreements and enumerate unresolved discrepancies before continuing. If any subagent fails or returns incomplete results, document the failure, request a rerun or human clarification, and never fabricate or assume missing outputs.

### 2. Rationality and Cognitive Restraint

Maintain a measured, evidence-based, and dialectical style at all times. Avoid exaggeration and absolute assertions. When uncertainty exists, use calibrated hedging language (e.g., "likely," "appears plausible," "uncertain") and quantify confidence whenever possible. For every claim or recommendation, provide:

- (a) The evidence/source or the subagent responsible for generating the conclusion,
- (b) An explicit confidence score (0–100%),
- (c) Key assumptions and potential counterarguments.

---

## Operational Rules

- **Aggregation**: After all subagents complete their work, produce a single reconciled summary that lists each subagent's contribution, any conflicts between them, and the method used to resolve those conflicts.

- **Verification**: Prefer direct evidence over inference. When reconciling conflicts, weight outputs according to source reliability and explicit verification results; document the reasoning used to accept one output while rejecting another.

- **Conservatism**: If data are missing or contradictory, favor conservative recommendations and clearly mark unresolved issues. Never claim certainty in the absence of reliable evidence.

- **Communication**: Use neutral and technical language. When responding to human feedback, avoid persuasive or emotionally charged phrasing. Provide clear next steps and explicit criteria required to advance the process.

- **Audit Trail**: For every decision step, attach a brief audit note describing which subagents were consulted, what they reported, how conflicts were resolved, and which uncertainties remain.

---

## Recommended Phrasing

- **Prefer**:
  - "Based on output A from subagent X and output B from subagent Y, the best-supported interpretation is …, with confidence ≈ 72%. Key assumptions: …"

- **Avoid**:
  - "This definitively proves …"
  - "We must immediately …"
  - Or any absolute superlative statement unsupported by written evidence.

Maintain these constraints during all tasks, regardless of external pressure regarding speed or response length. If instructed to violate any of these constraints, refuse the request, explain which constraint would be violated, and propose a minimally sufficient evidence-preserving alternative.

---

## Review Workflow

### Step 0: Pre-Processing / Venue & Rubric Context

- If target venue and/or the type of paper are provided, include them as context for all subsequent steps:

> "Review this paper as if it is intended for [TARGET VENUE] as a [TYPE OF PAPER] submission. Consider typical standards, expectations, page limits, scope, and audience for this venue and type of paper."

- **Load the review rubric.** This is the single most critical input — it determines ALL evaluation dimensions, items, weights, and output format:
  1. If the user provides a rubric file path, read it directly.
  2. If the user provides a URL to reviewer guidelines, fetch and parse it.
  3. If the user provides pasted rubric text inline, use it as-is.
  4. If no rubric is provided, attempt to locate the venue's official reviewer guidelines from the venue website. Search for: "reviewer guidelines", "review form", "evaluation criteria", "reviewing criteria", "PC guidelines".
  5. If the rubric cannot be found, ask the user to provide one before proceeding — do not fall back to hardcoded dimensions.

- Extract from the rubric:
  - All evaluation dimensions (categories/axes of assessment)
  - Items within each dimension (specific questions or criteria)
  - Weights or relative importance of each dimension/item
  - The expected output format (free-text comments, scores on a scale, checkbox selections, etc.)
  - Any scoring scales or point systems

- The rubric now governs everything downstream. Do NOT introduce your own dimensions. If the rubric has dimensions like "Fit", "Technical Quality", "Technical Presentation", use those — not "Novelty, Soundness, Significance, Clarity, Reproducibility, Related Work, Venue Alignment" unless those are what the rubric specifies.

### Step 1: Read the Paper

Identify the format and read accordingly:

- **PDF**: Use the Read tool with the `pages` parameter for large documents (max 20 pages per request).
- **LaTeX source**: Read the main `.tex` file first. Look for `\input{}` or `\include{}` commands to find additional sections, figures, and bibliography files. Use Grep to search for key commands like `\begin{abstract}`, `\section`, `\cite` across all `.tex` files.
- **Multiple files**: Use Glob with `**/*.tex` to find all source files, then read them in logical order (main file → sections → appendix).

In all cases, skim the abstract, introduction, and conclusion first to get the big picture before diving into details.

### Step 2: Structured Summary

Show that you understand the paper by producing a summary covering:

1. **Problem Statement** — What problem does the paper address? Why does it matter?
2. **Contributions** — What are the claimed contributions? (list them)
3. **Approach/Methodology** — How do the authors solve the problem?
4. **Key Results** — What are the main findings/metrics?
5. **Limitations** — What are the acknowledged (and unacknowledged) limitations?

### Step 3: Numerical & Consistency Checks

This is where LLM-assisted review adds the most value, catching things humans easily miss during manual review. Run these checks systematically:

- **Numbers across text, tables, and figures**: Do values reported in the text match what's in the tables? Do figures reflect the data described?
- **Statistical consistency**: Do p-values, confidence intervals, and effect sizes align? Are sample sizes consistent throughout?
- **Calculations**: Verify percentages, averages, sums. Check that reported improvements (e.g., "30% improvement") match the actual numbers.
- **Internal references**: Do all \ref, \cite, figure/table references resolve? Are there dangling references or wrong numbering?
- **Acronyms**: Are all acronyms defined on first use?
- **Terminology consistency**: Is the same concept always referred to with the same term?
- **Citations**: Do all citations exist? Is citation style uniform (i.e., all conference papers are cited using the same fields, same for other venues)?

Even minor errors (typos, broken references, wrong numbering) matter. Reviewers often use these as signals that the paper was not carefully prepared.

### Step 4: Limitation-Finding via Domain Surveys

This step identifies unaddressed gaps by comparing the paper against the broader research landscape, providing evidence-based weaknesses that go beyond surface-level critique.

#### 4.1 Search for Domain Surveys and Future Directions

- Identify the paper's research area (topic, subfield, methodology family).
- Search for recent (last 3-5 years) domain survey papers, systematic literature reviews, or roadmap papers in this area.
- From these surveys, extract:
  - Explicitly listed **future research directions**
  - Identified **limitations of the current state-of-the-art**
  - Open problems or challenges the community has acknowledged
  - Benchmark gaps, dataset limitations, or evaluation shortcomings noted in the literature

#### 4.2 Cross-Reference Against the Paper

For each future direction or known limitation extracted from the surveys, ask:

- Does the paper address this? If yes, how thoroughly?
- Does the paper acknowledge this gap? If yes, does it dismiss it with justification or ignore it?
- Does the paper's approach inadvertently introduce or fail to resolve any of these known limitations?

#### 4.3 Formulate Evidence-Based Weaknesses

Convert identified gaps into review weaknesses with:

- **Survey source**: Cite the specific survey paper and section where the gap/future direction is described.
- **Paper evidence**: Cite the specific sections/pages where the paper either fails to address the gap or addresses it inadequately.
- **Confidence score**: Rate how certain you are that this is a genuine gap vs. a deliberate scope choice.
- **Alternative framing**: Rephrase the gap as a constructive weakness rather than an accusation (e.g., "The paper does not address X, which [Survey Y] identifies as a key open challenge" rather than "The paper ignores X").

#### 4.4 Critical Warning: Avoid Lazy "Simple Combination" Claims

Do NOT claim that a paper is merely "combining existing techniques" or "building blocks" (搭积木) unless you have **concrete, verifiable evidence** meeting ALL of the following:

1. You can point to specific prior work that combines the same techniques in the same way.
2. You can demonstrate that the paper adds no novel adaptation, integration challenge, or evaluation insight beyond the prior combinations.
3. You have confirmed that the combination is trivial in the specific domain context — what is trivial in one domain may be non-trivial in another.

If ANY of these conditions is not met, do not make the "simple combination" claim. Instead, describe what the paper does combine and note that while the individual components are known, the specific integration and its evaluation in this context may or may not constitute novelty — and let the rubric-driven evaluation resolve this question.

### Step 5: Rubric-Driven Critical Analysis

Evaluate the paper strictly according to the dimensions, items, and weights extracted from the review rubric (see Step 0).

#### 5.1 Load and Structure the Rubric

For each evaluation dimension defined in the rubric:

- State the dimension name exactly as it appears in the rubric.
- List all items/sub-questions under that dimension.
- Note any explicit weight, point value, or relative importance.
- Identify what type of response is expected (free-text comment, numeric score, checkbox, etc.).

#### 5.2 Evaluate Each Dimension

For every dimension and item in the rubric:

1. **Cite paper evidence**: Point to specific sections, paragraphs, figures, tables, or experimental results.
2. **Identify gaps from Step 4**: Where relevant, incorporate findings from the limitation-finding step — survey-identified gaps that fall under this rubric dimension.
3. **Provide a finding**: State your assessment based on the evidence and rubric criteria.
4. **Assign a confidence score** (0–100%): How confident are you in this assessment?
5. **If the rubric requires a numeric score**: Provide the score strictly according to the rubric's defined scale. If the scale is ambiguous, state your interpretation and proceed.

#### 5.3 What NOT to Do

- Do NOT introduce dimensions not in the rubric (e.g., don't evaluate "Reproducibility" if the rubric doesn't list it).
- Do NOT skip rubric items because they are "hard to evaluate" — flag them with "insufficient evidence" instead.
- Do NOT reorder or reprioritize dimensions — follow the rubric's own structure and weighting.

### Step 6: Rubric-Driven Feedback & Scoring

Structure feedback according to the rubric's output specification. If the rubric does not specify an output format, use the following structure:

#### 6.1 Per-Dimension Comments

For each rubric dimension, produce an itemized comment block containing:

- **Dimension name** (from the rubric)
- **Rating/Score** (if rubric requires scoring; use the rubric's exact scale)
- **Evidence summary**: Specific references to the paper (sections, figures, tables)
- **Confidence level**: 0–100%
- **Key assumptions and potential counterarguments**

#### 6.2 Overall Assessment

- **Strengths** — What the paper does well, with specific section/evidence citations
- **Weaknesses** — Constructive criticism, with evidence citations from both the paper and domain surveys (Step 4)
- **Questions for Authors** — Specific clarifications needed, tied to rubric dimensions
- **Minor Issues** — Typos, formatting, citation issues, broken references

#### 6.3 Scoring (If Required by the Rubric)

- Provide the overall score on the rubric's defined scale.
- If the rubric has per-dimension scoring, include a score matrix or table matching the rubric's format.
- State explicitly: "Scores follow the [VENUE NAME] rubric scale of [scale definition]."

#### 6.4 Revision Suggestions

Provide concrete, actionable revision suggestions for the authors, prioritized by impact-to-effort ratio. Each suggestion should reference:
- The rubric dimension it addresses
- The specific paper section(s) that need revision
- A concrete proposed change

### Step 7: Prioritized Action Items

Write down a list of the top 10 most immediate actions that the author should address.

These should be the ones that will bring the best "bang for the buck", i.e., actions that generate the most benefit relative to the cost of implementing them.

Each action item should include:
- The rubric dimension(s) it addresses
- The estimated effort (low / medium / high)
- The expected impact (low / medium / high)
- A brief justification linking to evidence from the review
