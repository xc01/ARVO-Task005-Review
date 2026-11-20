# PatchAudit

**PatchAudit: Uncovering Faulty Security Patches and Lingering Zero-Days at Scale**

| | |
|---|---|
| Original artifact | <https://anonymous.4open.science/r/patchaduit-4B70/> |
| Imported from | the publications page |
| Tool | `pubs2github` |


---

## Contents

The artifact contains 19 file(s) including Python, Config files, Data files, and Documentation.

```
├── badpatch-bench
│   ├── badpatch_bench_baseline_performance.csv
│   ├── badpatch_bench_final.csv
│   ├── construct_benchmark.py
│   └── finetuning.csv
├── measurement study
│   ├── all_negative.json
│   └── all_positive.json
├── phase1_code
│   ├── phase1.zip
│   └── readme.md
├── phase1_finetuning_dataset
│   ├── finetuning.py
│   └── finetuning_dataset_train.jsonl
├── phase2_code
│   ├── phase2_step1.zip
│   └── phase2_step2.zip
├── phase2_finetuning_dataset
│   ├── finetuning_dataset.jsonl
│   └── phase2_finetune.ipynb
├── zero-day bad patch discovery
│   ├── poc.zip
│   ├── README_zeroday_discovery.md
│   └── zero_day_discover.zip
├── .gitignore
└── README.md
```

---

## Original `README.md` (from the upstream artifact)

# PatchAudit Artifact

This is the artifact package for PatchAudit. This artifact includes the code for Phase 1 and Phase 2 and Phase 3, model weights and fine-tuning datasets and fine-tuning scripts for the Phase 1 model and Phase 2 model, and includes the evaluation results, large-scale study's result labels, and zero-day discovery implementation code.

---

## What is Included in This Package

```
patchaudit/
├── badpatch-bench/
│   ├── badpatch_bench_final.csv               # Benchmark with ground truth labels
│   ├── badpatch_bench_baseline_performance.csv # Performance metrics on benchmark
│   └── finetuning.csv                         # Fine-tuning dataset statistics
│   └── construct_benchmark.py — PatchAudit benchmark construction pipeline.
│
├── phase1_code/
│   ├── phase1.zip                             # Phase 1 implementation (also include the code for phase 3 verification)
│   └── readme.md                              # Phase 1 documentation
│
├── phase1_finetuning_dataset/
│   ├── finetuning_dataset_train.jsonl         # Phase 1 training data
│   └── finetuning.py                          # Training data generation script
│
├── phase2_code/
│   ├── phase2_step1.zip                       # Phase 2 step 1: data extraction & analysis
│   └── phase2_step2.zip                       # Phase 2 step 2: verification & classification (also include the code for phase 3 verification)
│
├── phase2_finetuning_dataset/
│   ├── finetuning_dataset.jsonl               # Phase 2 training data
│   └── phase2_finetune.ipynb                  # Phase 2 fine-tuning notebook
│
├── measurement study/
│   ├── all_positive.json                      # Confirmed bad patches (617 entries)
│   └── all_negative.json                      # Non-bad-patch samples (5919 entries)
│
├── zero-day bad patch discovery/
│   ├── README_zeroday_discovery.md            # Zero-day discovery documentation
│   └── zero_day_discover.zip                  # Zero-day discovery implementation
│   └── poc.zip                                # Zero-day vulnerability poc
│
└── readme.md                                   # Setup and usage instructions
```

---

## How to Download and Set Up

### 1. Install Dependencies

```bash
# Python dependencies
pip install langchain langchain-openai langgraph anthropic vllm transformers torch
pip install semgrep

# CodeQL (for static analysis)
# Download from: https://github.com/github/codeql-cli-binaries/releases
# Extract and add to PATH

# Node.js MCP servers (for GitHub and web scraping)
npm install -g @modelcontextprotocol/server-github firecrawl-mcp
```

### 2. Set Environment Variables

```bash
# Required API keys
export GITHUB_PERSONAL_ACCESS_TOKEN="your_github_token"
export FIRECRAWL_API_KEY="your_firecrawl_key"
export OPENAI_API_KEY="your_openai_key"


# Repository paths (adjust to your setup)
export BASE_REPOS_PATH="/path/to/your/repos"
export SARIF_DIR="/path/to/codeql/results"
```

---

## Configurations

### Phase 1 Configuration

Edit `phase1_code/phase1/simply_main.py`:

```python
# Output directory
results_dir = "/path/to/phase1/results"

# Repository base path
repos_base_path = "/path/to/repos"

# Input files
cve_pairs_json = "simple_cve_pairs.json"
cve_description_json = "cve_description.json"

# vLLM server
vllm_base_url = "http://localhost:8000/v1"
vllm_model = "Qwen/Qwen3-14B"

# Features
enable_thinking = True       # Enable Chain-of-Thought reasoning
enable_verification = True   # Enable multi-verdict verification
```

Edit `phase1_code/phase1/simple_verification_orchestrator.py`:

```python
# Static analyzer paths
BASE_REPOS_PATH = '/path/to/repos'
SARIF_DIR = "/path/to/codeql/results"
```

### Phase 2 Configuration

Edit `phase2_code/phase2/main.py`:

```python
# Processing range (for distributed processing)
START_INDEX = 0
END_INDEX = 100

# Output directory
results_dir = "/path/to/phase2/results"

# Input file
commits_json = "commits.json"
```

Edit `phase2_code/phase2/verification_orchestrator.py`:

```python
# Static analyzer paths
BASE_REPOS_PATH = '/path/to/repos'
SARIF_DIR = "/path/to/codeql/results"
```

---

## Inputs and Outputs

### Input

**CVE Commit Pairs** (`simple_cve_pairs.json` or `commits.json`):
```json
[
  {
    "cve_id": "CVE-2023-40581",
    "base_commit": "de015e930747165dbb8fcd360f8775fd973b7d6e",
    "base_commit_parent": "61bdf15fc740c66fadb1a9e8e1846de57232a0de",
    "later_commit": "ff07792676f404ffff6ee61b5638c9dc1a33a37a",
    "later_commit_parent": "216f6a3cb5780362d787a08a7c0c5c00e86304ba",
    "author": "yt-dlp",
    "repo": "yt-dlp"
  }
]
```

**CVE Descriptions** (`cve_description.json`):
```json
{
  "CVE-2023-40581": {
    "description": "Remote code execution via --exec flag on Windows...",
  }
}
```

**Repository Structure**:
- Repositories must be cloned locally at `{BASE_REPOS_PATH}/{author}/{repo}`
- Example: `/data/repos/yt-dlp/yt-dlp`

### Output

#### Decision
**Class label**: benign, incomplete, incorrect

#### Explanation
For bad patches (incomplete or incorrect), the output includes:

1. **Classification**:
   - **Type**: COMPLETING (patch was incomplete) or CORRECTING (patch had errors)
   - **Issue**: What security vulnerability exists in the base commit (C1)
   - **Cause**: Why the base commit had this issue

2. **Detailed Analysis**:
   - What the base commit (C1) failed to address or incorrectly implemented
   - How the later commit (C2) fixes or completes C1
   - Attack vectors that remained exploitable after C1
   - Root cause of the incomplete or incorrect patch (e.g., incomplete attack surface analysis, insufficient testing, coding errors)

---

## How to Run the Tool

### Phase 1: Lightweight Analysis

Phase 1 performs fast commit message-based analysis with early filtering.

#### Step 1: Start vLLM Server

```bash
# Terminal 1: Using base model
python -m vllm.entrypoints.openai.api_server \
    --model Qwen/Qwen3-14B \
    --port 8000 \
    --trust-remote-code
```

#### Step 2: Run Phase 1 Analysis

```bash
# Terminal 2
cd phase1_code/phase1
python simply_main.py
```

**Expected processing time**: ~1-2 minutes per CVE pair (with early filtering for ~80% of unrelated pairs)

### Phase 2: Deep Code Analysis

Phase 2 performs comprehensive code-level analysis with function extraction and interprocedural slicing.

#### Step 1: Start vLLM Server

```bash
# Terminal 1: Using base model
python -m vllm.entrypoints.openai.api_server \
    --model Qwen/Qwen3-14B \
    --port 8000 \
    --trust-remote-code
```

#### Step 2: Run Phase 2 Analysis

```bash
# Terminal 2
cd phase2_code/phase2
python main.py
```

**Expected processing time**: ~2-5 minutes per CVE pair

---

## How to Interpret the Outputs

### Phase 1 Output Files

#### 1. Filtered Result (Early Termination)
**Filename**: `{CVE_ID}_{base_hash}_{later_hash}_filtered.json`

```json
{
  "cve_id": "CVE-2023-40581",
  "base_commit": "de015e9",
  "later_commit": "ff07792",
  "filtered": true,
  "filter_reason": "Later commit does not mention CVE ID or base commit hash"
}
```

**Interpretation**: This pair was filtered out early because the later commit does not reference the CVE or base commit. No further analysis needed.

#### 2. Analysis Result
**Filename**: `{CVE_ID}_{base_hash}_{later_hash}_complete.json`

```json
{
  "cve_id": "CVE-2023-40581",
  "base_commit": "de015e9",
  "later_commit": "ff07792",
  "plan": {
    "steps": [
      {
        "tool": "get_issue",
        "params": {"owner": "yt-dlp", "repo": "yt-dlp", "issue_number": 7805}
      }
    ]
  },
  "gathered_info": {
    "github_issues": [...],
    "urls": [...]
  },
  "qwen_answer": "YES",
  "qwen_confidence": "HIGH",
  "final_answer": "YES"
}
```

**Interpretation**:
- `qwen_answer`: Initial LLM decision (YES = later commit fixes base commit, NO = unrelated)
- `qwen_confidence`: HIGH (requires verification) or LOW (skips verification)
- `gathered_info`: Context gathered from GitHub issues and URLs
- `final_answer`: Decision after gathering context

#### 3. Verification Result
**Filename**: `{CVE_ID}_{base_hash}_{later_hash}_verification.json`

```json
{
  "metadata": {
    "cve_id": "CVE-2023-40581",
    "base_commit": "de015e9",
    "later_commit": "ff07792"
  },
  "qwen_answer": "YES",
  "qwen_confidence": "HIGH",
  "gpt_answer": "YES",
  "match": true,
  "static_analysis": {
    "consensus": true,
    "path": "positive",
    "verdict_2": true,
    "verdict_3": false,
    "final_decision": true
  },
  "bad_patch_classification": {
    "type": "COMPLETING",
    "issue": "C1 escaped quotes but failed to escape %, \\, and newlines",
    "cause": "Incomplete analysis of Windows shell metacharacters"
  },
  "final_status": "confirmed"
}
```

**Interpretation**:

**Verdict 1 (LLM Consensus)**:
- `qwen_answer` and `gpt_answer`: Both LLMs must agree
- `match: true`: LLMs agree, proceed to static analysis
- `match: false`: LLMs disagree, requires human review

**Verdicts 2 & 3 (Static Analysis)**:
- For "YES" answers:
  - Verdict 2: C1 and C2_parent share same vulnerability (both vulnerable)
  - Verdict 3: C1_parent and C2_parent share same vulnerability
  - Pass if: Verdict 2 OR Verdict 3 = true
  
- For "NO" answers:
  - Verdict 2: C2_parent is safe (no vulnerabilities)
  - Verdict 3: C1 and C2_parent have different vulnerabilities
  - Pass if: Verdict 2 OR Verdict 3 = true

- `consensus: true`: Both Semgrep and CodeQL agree
- `final_decision: true`: All verdicts pass

**Bad Patch Classification** (only for confirmed bad patches):
- `type`: COMPLETING (incomplete patch) or CORRECTING (incorrect patch)
- `issue`: Specific vulnerability in base commit
- `cause`: Root cause of the issue

**Final Status**:
- `confirmed`: All verdicts pass
- `verdicts_failed`: Static analysis failed
- `static_disagreement`: Semgrep and CodeQL disagree
- `llm_disagreement`: Qwen and GPT disagree

### Phase 2 Output Files

#### 1. Analysis Result
**Filename**: `{CVE_ID}_{c1hash}_{c2hash}.json`

```json
{
  "cve_id": "CVE-2023-40581",
  "c1_hash": "de015e9307",
  "c2_hash": "ff07792676",
  "plan": [
    {
      "step_number": 1,
      "action": "get_issue",
      "parameters": {"owner": "yt-dlp", "repo": "yt-dlp", "issue_number": "7805"},
      "reasoning": "Retrieve security advisory to understand vulnerability"
    },
    {
      "step_number": 2,
      "action": "extract_function_with_info",
      "parameters": {
        "function_name": "shell_quote",
        "commit_hash": "de015e9307",
        "filepath": "yt_dlp/utils.py"
      },
      "reasoning": "Extract function to analyze escaping logic"
    }
  ],
  "execution_results": [
    {
      "step": 1,
      "action": "get_issue",
      "status": "success",
      "output": "Security Advisory GHSA-...: RCE via --exec flag..."
    },
    {
      "step": 2,
      "action": "extract_function_with_info",
      "status": "success",
      "output": "def shell_quote(s):\n    return s.replace('\"', '\\\\\"')..."
    }
  ],
  "analysis": {
    "reasoning": "Base commit added escaping for quotes but missed other metacharacters...",
    "decision": "YES"
  }
}
```

**Interpretation**:
- `plan`: Multi-step execution plan generated by LLM
- `execution_results`: Results from each tool call (GitHub issues, function extraction, code slicing)
- `analysis.decision`: Initial decision (YES/NO)
- `analysis.reasoning`: Detailed reasoning based on gathered evidence

#### 2. Verification Result
**Filename**: `{CVE_ID}_{c1hash}_{c2hash}_verification_complete.json`

```json
{
  "metadata": {
    "cve_id": "CVE-2023-40581",
    "c1_hash": "de015e9",
    "c2_hash": "ff07792",
    "final_status": "confirmed",
    "total_rounds_executed": 2
  },
  "rounds": [
    {
      "round_number": 1,
      "gpt_verdict": {
        "qwen_answer": "YES",
        "gpt_answer": "YES",
        "match_result": "MATCH"
      },
      "static_analysis": {
        "path": "positive",
        "verdict_2": true,
        "verdict_3": false,
        "final_decision": true
      }
    }
  ],
  "bad_patch_classification": {
    "type": "COMPLETING",
    "issue": "Base commit escaped quotes and ampersands but failed to escape percent signs, backslashes, and newlines in Windows shell commands",
    "cause": "Incomplete attack surface analysis - developer focused on common metacharacters but did not comprehensively test all Windows cmd.exe special characters"
  },
  "final_result": {
    "Qwen Answer": "YES",
    "GPT Answer": "YES",
    "Match Result": "MATCH"
  }
}
```

**Interpretation**:

**Multi-Round Verification**:
- `total_rounds_executed`: Number of verification rounds (max 3)
- Each round contains GPT verdict and static analysis results
- System may iterate if verdicts fail, providing feedback to LLM for re-analysis

**Bad Patch Classification** (more detailed than Phase 1):
- `type`: COMPLETING or CORRECTING
- `issue`: Comprehensive description of vulnerability in C1
- `cause`: Detailed root cause analysis including developer reasoning

**Final Status**:
- `confirmed`: All verdicts pass, bad patch confirmed
- `benign`: All verdicts pass for "NO" answer, commits are unrelated
- Other statuses indicate verification failures

---

## Benchmark and Evaluation

The `badpatch-bench/` directory contains:

- **construct_benchmark.py**: Construction pipeline for BadPatch-Bench, implementing the procedure of §4 (NVD collection, language filtering, post-`C1` commit retrieval, and the with-/without-explicit-reference split). Run `python construct_benchmark.py --help` for options and expected outputs.
- **badpatch_bench_final.csv**: Benchmark dataset with ground truth labels for evaluating PatchAudit's detection accuracy — 4,118 labeled commit pairs over 1,147 CVEs and 265 projects (363 positive / 3,755 negative), matching Table 1 of the paper
- **badpatch_bench_baseline_performance.csv**: Performance metrics showing precision, recall, and F1 scores across different model configurations
- **finetuning.csv**: Statistics on the fine-tuning datasets used for Phase 1 and Phase 2 models
---

## Large-Scale Study Results

The `measurement study/` directory contains results from the large-scale study of CVE patches:

- **all_positive.json**: 617 confirmed bad patches with detailed labels categorizing the root causes and security issues
- **all_negative.json**: 5919 non-bad-patch samples used as the negative set in the study

---

## Zero-Day Discovery

The `zero-day bad patch discovery/` directory contains:

- **README_zeroday_discovery.md**: Documentation on the methodology and findings from applying PatchAudit to discover previously unknown vulnerable patches
- **zero_day_discover.zip**: Implementation code and analysis results for zero-day vulnerability discovery

---

For phase-specific details, see:
- [Phase 1 README](phase1_code/readme.md)
- [Zero-Day Discovery README](zero-day bad patch discovery/README_zeroday_discovery.md)
