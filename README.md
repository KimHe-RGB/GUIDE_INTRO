# GUIDE (AutoEvidence)

[![Python Version](https://img.shields.io/badge/python-3.9-blue.svg)](https://www.python.org/downloads/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](#contributing)

[English](README.md)

## What is GUIDE 🗺️

GUIDE (previous name: AutoEvidence) is an AI-powered, human-in-the-loop clinical guideline generation interface.

### GUIDE Invents AI Generation of Clinical Guideline ✨✨🤖🧠✨✨

GUIDE supports evidence-based medicine (EBM) workflow for clinical practice guideline generation and updating.

The system is designed to support:

- Clinical practice guideline development
- Guideline updating
- Systematic review workflows
- Evidence mapping
- Evidence-to-Recommendation drafting
- Expert consensus preparation

### GUIDE Advocates Human-in-the-loop 🧑‍⚕️🩺 🤝 🤖💻

GUIDE is designed as an interactive tool for augmenting human experts. High-volume and rule-based tasks are delegated to specialized AI agents, while medical experts retain oversight, adjudication authority, and final responsibility for judgment-intensive steps.

It is **NOT** intended to replace:

- Human clinical judgment
- Formal guideline panels
- Independent risk-of-bias assessment
- Regulatory review
- Patient-specific medical advice

## Quick Start 🚀

### Prerequisites ⚙️🔧

- The System is developed and tested on a MacBook Pro 2024 M3 chip, performance may be restricted on different machines.
- The following environment configuration is mandatory.

- MacOS or Linux
- Python 3.9+
- npm (For Frontend)
- Redis (For task queue and WebSocket scaling)
- conda (For development deployment)
- Docker & Docker Compose (For production deployment)
- API key for LLM Models
- **guide_web**
  - Make sure the frontend project is placed under the project root folder

### Start as Development

Create a conda environment first, this may take a long time when you first run:

```bash
conda env create -f environment.yml
```

After you have conda environment setup, use this one-line script to deploy:

```bash
bash ./start_development.sh
```

### Start as Production (Require Docker 🐳)

Use this one-line script to deploy:

```bash
bash ./start_docker.sh
```

### API Docs

APIs are prefixed with `/api`. Swagger/OpenAPI documentation is available at `/docs`

## LLM Model Configuration

Place model configuration JSON files under `config/llm_models/`. Example:

```json
{
  "api_key": "sk-xxx",
  "model": "gpt-4.1",
  "base_url": "https://api.openai.com/v1"
}
```

The Config loader scans `config/llm_models/*.json` at startup. Each file name becomes the model alias you can reference throughout the application.

## Why GUIDE? 🤔

Clinical practice guidelines must remain timely, trustworthy, and clinically accountable. However, conventional guideline development and updating workflows are often slowed by the rapid growth of biomedical literature, complex evidence appraisal, and the need for expert consensus.

GUIDE addresses these challenges through a structured human-AI collaboration model:

- AI agents accelerate evidence retrieval, screening, extraction, and preliminary synthesis.
- AI reviewer monitor uncertainty, disagreement, and inconsistent outputs.
- Experts intervene proactively or when triggered by predefined quality-control signals.
- A Delphi-style multi-agent panel supports transparent recommendation drafting and consensus formation.

## Workflow Overview

GUIDE implements a seven-steps clinical guideline workflow, using a multi-agent workflow system:

1. Clinical question decomposition
2. PICO construction and expert verification
3. Literature search strategy generation and retrieval
4. Title/abstract and full-text screening
5. Data extraction and body-of-evidence assembly
6. GRADE-based evidence quality assessment
7. Evidence-to-Recommendation synthesis and Delphi-style consensus

```text
Clinical Question
      ↓
Protocol Agent + Expert Verification
      ↓
Search Strategy Generation Agents 
      ↓
Title/Abstract Screening Agents
      ↓
Full-Text Screening Agents + Expert Adjudication
      ↓
Data Extraction Agents + Source Verification
      ↓
GRADE Quality Assessment Agents + Expert Audit
      ↓
Delphi-Style Multi-Agent Consensus
      ↓
Evidence-to-Recommendation Synthesis
```

## Introduction to Each Step

<!-- markdownlint-disable MD033 -->

### 1. Clinical Question to Protocol 🙋 ➡️ 🗺️

GUIDE starts from decomposing an arbitary clinical questions into **PICO elements**, and generates a structured **Review Protocol** table, which serves as the key controlling paradigm for subsequent steps.

The **Review Protocol** covers the following terms:

- Objectives
- Type of review
- Population
- Intervention
- Comparator
- Outcomes
- Study design
- Crossover studies
- Other exclusions
- Population stratification
- Search criteria

To enhance human-computer-interaction, all fields mentioned above are editable by human experts to control downstream evidence retrieval and screening, outcome extraction, etc.

<div class="figure-grid">
      <figure class="figure-card">
            <img src="figures/1-Clinical%20Question%20to%20Protocol.jpg" alt="Clinical question to protocol interface">
            <figcaption><strong>Protocol Generation</strong> A free-text clinical question is transformed into editable PICO fields and a structured review protocol.</figcaption>
      </figure>
</div>

### 2. Search Strategy Generation 🗺️ ➡️ 🔍

Generate database-specific search strategies from **Review Protocol**, including:

- Dataset-specific Boolean queries
- MeSH term expansion
- Synonym and concept expansion
- Study-design filters when needed

GUIDE supports dual-agent search strategy generation, where two LLM agents with different LLM configurations independently generate search strategies. AI reviewer compares retrieval results and flags potential unreliable searches, such as large discrepancies in retrieval volume or low overlap between search outputs.

<div class="figure-grid">
      <figure class="figure-card">
            <img src="figures/2-Search%20Strategy%20Generation.jpg" alt="Dual-agent search strategy generation interface">
            <figcaption><strong>Dual search strategy generation.</strong> Dual agents draft database-specific retrieval logic</figcaption>
      </figure>
</div>

### 3. Literature Retrieval 🔍 ➡️ 📚📚📚

Execute literature searches through biomedical databases and retrieve:

- Article titles
- Abstracts
- Metadata
- Author information
- Publication dates
- Journal information
- Full-text PDFs when available

Supported sources include:

- **PubMed** via NCBI Entrez / E-utilities
- **Scopus** via Elsevier API
- Optional scholarly enrichment through **OpenAlex**

<div class="figure-grid">
      <figure class="figure-card">
            <img src="figures/3-Literature%20Retrieval.jpg" alt="Literature retrieval and screening queue interface">
            <figcaption><strong>Retrieved Articles</strong> Search results are collected into a working set that supports bulk review, export, and staged for batch screening</figcaption>
      </figure>
</div>

### 4. Two-Stage Article Screening 📚📚📚 ➡️ ✨📚✨

GUIDE supports sequential evidence screening:

- **Title and abstract screening**
- **Full-text eligibility assessment**

Screening agents evaluate articles against the Protocol-derived inclusion and exclusion criteria and provide:

- Include / exclude decisions
- Rationale
- Confidence levels
- Disagreement flags (only in the second screening stage)

In dual-agent mode, two independent agents screen the same evidence pool. When disagreement occurs, the AI reviewer triggers expert adjudication.

<div class="figure-grid">
      <figure class="figure-card">
            <img src="figures/4-Title%20and%20abstract%20screening_1.jpg" alt="Title and abstract screening workspace">
            <figcaption><strong>Title-Abstract Screening</strong></figcaption>
      </figure>
      <figure class="figure-card">
            <img src="figures/4-Title%20and%20abstract%20screening_2.jpg" alt="Screening reason popover">
            <figcaption><strong>Agent Rationale Review</strong></figcaption>
      </figure>
      <figure class="figure-card">
            <img src="figures/4-Full-text%20eligibility%20assessment_1.jpg" alt="Full-text screening workspace">
            <figcaption><strong>Get PDF files forFull-text Screening</strong></figcaption>
      </figure>
      <figure class="figure-card">
            <img src="figures/4-Full-text%20eligibility%20assessment_3.jpg" alt="Full-text Extraction">
            <figcaption><strong>Batch Upload PDF and Extract Full-text with OCR</strong></figcaption>
      </figure>
      <figure class="figure-card">
            <img src="figures/4-Full-text%20eligibility%20assessment_2.jpg" alt="Batch PDF upload and article assignment modal">
            <figcaption><strong>Full-text Screening Results</strong></figcaption>
      </figure>

</div>

### 5. Full-Text Evidence Extraction 📚 ➡️ 📖

For included studies, GUIDE extracts structured evidence from full-text articles to support guideline evidence tables and downstream evidence synthesis.

The extracted information includes:

- Study identifier
- Study type and design
- Countries and study setting
- Study duration, recruitment period, intervention duration, and follow-up duration
- Inclusion criteria
- Exclusion criteria
- Recruitment or participant selection methods
- Participant characteristics, including age, gender/sex, and family origin or ethnicity where reported
- Population indirectness assessment
- Intervention and comparator details for each study arm
- Number of participants in each study arm
- Funding sources
- Conflict-of-interest information
- Protocol-defined outcomes reported in the study
- Actual outcome names used in the original study
- Outcome measurement tools or definitions
- Outcome time points
- Mapping between protocol outcomes and study-reported outcomes
- Protocol outcomes not reported by the study

GUIDE supports extraction of multiple outcome types, including:

- Dichotomous or binary outcomes
- Continuous outcomes
- Time-to-event outcomes
- Ordinal outcomes
- Count outcomes
- Rate outcomes
- Repeated-measure outcomes

GUIDE also extracts numerical results into structured tables according to outcome type, including:

- Group totals
- Event numbers
- Numerical values
- Time-to-event values
- Counts, rates, proportions, or percentages
- Diagnostic accuracy data
- Effect estimates
- Confidence intervals
- P values
- Other study-reported statistics

When information is missing, unclear, or not applicable, GUIDE marks it explicitly to support transparent expert review.

The system supports text and table extraction from PDFs and can provide source-position markers to help experts rapidly verify extracted evidence.

<div class="figure-grid">
      <figure class="figure-card">
            <img src="figures/5-Full-Text%20Evidence%20Extraction_1.jpg" alt="Data extraction tables">
            <figcaption><strong>Extracted Tables from each included articles</strong></figcaption>
      </figure>
      <figure class="figure-card">
            <img src="figures/5-Full-Text%20Evidence%20Extraction_2.jpg" alt="Clinical Evidence Summary Table">
            <figcaption><strong>Clinical Evidence Summary Table</strong></figcaption>
      </figure>
      <figure class="figure-card">
            <img src="figures/5-Full-Text%20Evidence%20Extraction_3.jpg" alt="Outcomes discrepency report">
            <figcaption><strong>Show dual-agent discrepency in extracted outcome fields for human edition</strong></figcaption>
      </figure>
</div>

### 6. GRADE-Based Effect Estimate and Quality Assessment 💯📈

GUIDE supports GRADE-ready evidence synthesis by selecting an appropriate effect-estimate approach for each outcome before certainty assessment.

For each outcome, GUIDE identifies:

- Outcome type
- Study design
- Available effect measure
- Follow-up time point
- Crude or adjusted estimate
- Comparability of outcome definitions
- Direction of benefit
- Whether meta-analysis is appropriate

Domain experts group study-level outcomes into comparison- and outcome-level evidence bodies for effect synthesis.

GUIDE supports effect estimates including:

- Risk ratio (RR)
- Odds ratio (OR)
- Hazard ratio (HR)
- Risk difference (RD)
- Mean difference (MD)
- Standardized mean difference (SMD)
- Hedges g
- Rate ratio or incidence rate ratio (IRR)
- Sensitivity and specificity
- Likelihood ratios
- Diagnostic odds ratio
- Single-arm proportion, mean, or rate

When appropriate, GUIDE calculates pooled effect estimates using random-effects meta-analysis via R script.

GUIDE then supports GRADE-oriented certainty assessment across key domains:

- Risk of bias
- Indirectness
- Imprecision
- Inconsistency
- Publication bias
- Overall certainty of evidence

AI agents can perform preliminary effect-estimate synthesis and grading, while expert review remains central for judgment-intensive domains and final certainty decisions.

<div class="figure-grid">
      <figure class="figure-card">
            <img src="figures/6-GRADE-Based%20Effect%20Estimate%20and%20Quality%20Assessment_1.jpg" alt="GRADE grouping and meta-analysis preparation view">
            <figcaption><strong>Automatic & Manual grouping for Meta analysis preparation</strong> </figcaption>
      </figure>
      <figure class="figure-card">
            <img src="figures/6-GRADE-Based%20Effect%20Estimate%20and%20Quality%20Assessment_2.jpg" alt="Meta-analysis result and certainty summary">
            <figcaption><strong>Pooled results and certainty summary.</strong> Meta-analysis outputs are presented with effect estimates, inconsistency, and a GRADE-ready narrative interpretation.</figcaption>
      </figure>
</div>

### 7. Body-of-Evidence Assembly

GUIDE organizes extracted study-level evidence into structured bodies of evidence for each clinical outcome.

This includes:

- Outcome-level evidence tables
- Study-level effect summaries
- Cross-study comparison
- Inconsistency analysis
- Certainty-of-evidence profiles
- Evidence summaries ready for recommendation drafting

### 8. Delphi-Style Multi-Agent Consensus

GUIDE includes a multi-agent virtual consensus module that simulates a guideline committee.

Supported roles may include:

- Clinical Practice Expert
- Research Methodology Expert
- Evidence Synthesis Expert
- Research Ethics and Quality Expert
- Biostatistician
- Health Economics Expert
- Patient and Public Representative
- Health Policy and Implementation Expert

Each AI panelist provides an independent role-specific assessment. A committee chair agent synthesizes agreement, disagreement, and unresolved issues. Multiple deliberation rounds can be conducted until consensus criteria are met or expert intervention is required.

<div class="figure-grid">
      <figure class="figure-card">
            <img src="figures/8-Delphi-Style%20Multi-Agent%20Consensus.jpg" alt="Delphi-style multi-agent consensus discussion">
            <figcaption><strong>Virtual committee deliberation.</strong> Role-specific panelists debate recommendation wording and certainty until the committee converges on a consensus.</figcaption>
      </figure>
</div>

### 9. Evidence-to-Recommendation Synthesis

GUIDE supports structured Evidence-to-Recommendation synthesis by integrating:

- Benefits and harms
- Certainty of evidence
- Outcome importance
- Clinical applicability
- Cost and resource considerations
- Patient values and preferences
- Implementation considerations

This module helps bridge the gap between evidence appraisal and clinically interpretable recommendations.

<div class="figure-grid">
      <figure class="figure-card">
            <img src="figures/9-Evidence-to-Recommendation%20Synthesis.jpg" alt="Evidence-to-recommendation synthesis report">
            <figcaption><strong>Recommendation report synthesis.</strong> The committee output is consolidated into a structured Evidence-to-Recommendation report with rationale, tradeoffs, and implementation framing.</figcaption>
      </figure>
</div>

### 10. Human-in-the-Loop Reviewer Mechanism

GUIDE is built around an AI reviewer triggered governance model.

The AI reviewer can flag:

- Search result discrepancies
- Low overlap between retrieval strategies
- Screening disagreements
- Inconsistent extracted data
- Divergent certainty ratings
- Potential methodological concerns
- Low-confidence decisions

This design allows experts to focus on high-value arbitration rather than continuously monitoring every automated step.

### 11. AI Assistant Interface

A LangChain-based conversational agent orchestrates the workflow through natural language.

The Assistant can:

- Call internal workflow tools
- Maintain context across sessions
- Stream tool execution events through WebSocket
- Delegate long-running tasks to specialized agents
- Translate expert instructions into executable operations
- Help revise search strategies, screening criteria, or evidence tables

<div class="figure-grid">
      <figure class="figure-card">
            <img src="figures/11-AI%20Assistant%20Interface.jpg" alt="AI assistant interface for workflow orchestration">
            <figcaption><strong>Conversational workflow control.</strong> The assistant translates natural-language instructions into tool-backed workflow actions such as PICO extraction and strategy refinement.</figcaption>
      </figure>
</div>

<!-- markdownlint-enable MD033 -->

## License 📄

MIT License

## Contributing

Pull requests are welcome. For substantive changes, open an issue first to discuss the proposed update and keep the README, figures, and deployment workflow aligned with the published site.
