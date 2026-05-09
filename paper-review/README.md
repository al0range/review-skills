# paper-review

**Rubric-driven academic paper review skill** — a fork and significant rework of [BESSER-PEARL/agent-skills research-paper-review](https://github.com/BESSER-PEARL/agent-skills).

## What's Different from Upstream

This fork transforms the review workflow from a fixed-dimension framework into a **rubric-driven** one:

| Aspect | Upstream (research-paper-review) | This Fork (paper-review) |
|--------|----------------------------------|---------------------------|
| **Evaluation dimensions** | Hardcoded 7 dimensions (Novelty, Soundness, Significance, Clarity, Reproducibility, Related Work, Venue Alignment) | Strictly driven by the target venue's review rubric — no fixed dimensions |
| **Output format** | Fixed structure (Strengths, Weaknesses, Questions, Minor Issues, Venue-Specific) | Driven by the rubric's output specification (scoring, comments, checkboxes as required) |
| **Limitation finding** | Generic critical analysis | **New Step 4**: domain survey-based gap identification — searches for "future research directions" in survey papers, cross-references against the paper, and formulates evidence-based weaknesses |
| **Evidence quality** | Implicit | Explicit confidence scores (0–100%), mandatory evidence citations per claim, "insufficient evidence" flagging |
| **Multi-agent workflow** | Single-agent | Multi-subagent aggregation with conflict reconciliation and audit trails |
| **Quality control** | Not specified | **Mandatory Constraints**: Patience/Quality First + Rationality/Cognitive Restraint, guarding against overconfident and unsupported claims |

## Key Features

- **Rubric-driven**: All review dimensions, items, weights, and scoring come from the venue's official review form. The skill does not impose its own evaluation criteria.
- **Limitation-finding via domain surveys**: Searches recent survey papers for acknowledged gaps and future directions, then compares the paper under review against those gaps.
- **Evidence-first**: Every claim requires a paper citation (section, figure, table), a confidence score, and explicit assumptions.
- **Multi-agent aggregation**: Designed for parallel subagent execution with structured conflict resolution and audit trails.
- **Conservatism guardrails**: Cannot label papers as "simple combination"/"building blocks" without meeting 3 concrete evidence conditions.

## What It Does

When you share a research paper (PDF, LaTeX, or plain text) and a review rubric, this skill:

1. **Loads the rubric** — parses dimensions, items, weights, scoring scales
2. **Reads the paper** — handles PDF, LaTeX source, and plain text
3. **Summarizes** — problem, contributions, methodology, results, limitations
4. **Checks consistency** — numerical, statistical, citations, acronyms, references
5. **Finds gaps via surveys** — searches domain surveys for "future work" directions, cross-references against the paper
6. **Evaluates per rubric** — scores/assesses every rubric item with evidence and confidence
7. **Produces rubric-formatted feedback** — itemized comments, overall assessment, revision suggestions, scorer's matrix if required
8. **Prioritizes actions** — top 10 highest-impact-to-effort fixes

## Installation

```bash
npx skills add al0range/review-skills@paper-review
```

## Usage

Share a paper, a rubric, and optionally the venue:

```
Review this paper for ICWE 2026 as a full research paper submission.
Here is the review rubric: [paste or file path]
```

```
Use the NeurIPS 2025 reviewer guidelines to review this PDF.
```

## License

MIT — same as upstream. Copyright (c) BESSER-PEARL contributors.

## Credits

- Original skill: [BESSER-PEARL/agent-skills](https://github.com/BESSER-PEARL/agent-skills) by [Armen Sulejmani](https://github.com/armensulejmani), [Ivan David Alfonso](https://github.com/ivan-alfonso), [Jordi Cabot](https://github.com/jcabot)
- Fork and rework: [al0range/review-skills](https://github.com/al0range/review-skills)
