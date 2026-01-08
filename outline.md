# Blog Post Outline: Prompt Format Variations in Function Calling

## 1. Introduction
- Brief overview of function calling in LLMs
- The challenge: Different prompting formats exist (JSON, XML, Python) but little research on their impact
- Our research question: How sensitive are LLMs to prompt format variations in function calling tasks?

## 2. Motivation
- Function calling is critical for LLM applications (tools, APIs, agents)
- Existing benchmarks focus on "what" models can do, not "how" format affects performance
- Real-world implications: choosing the wrong format could significantly impact application reliability
- Current practice is largely trial-and-error

## 3. Methodology

### 3.1 Prompt Variations Tested
- **Response Format (res_fmt)**: JSON, XML, Python
  - With and without tags (e.g., `<function_call>`)
- **Documentation Format (doc_fmt)**: JSON, XML, Python
  - How function signatures are presented to the model
- Total: 18 combinations (3 res_fmt × 2 tagged/untagged × 3 doc_fmt)

### 3.2 Models Evaluated
- GPT-4o-mini-2024-07-18
- GPT-4o
- GPT-4.1
- [Additional reasoning models: o1, o3-mini, gpt-4.5-preview - in progress]

### 3.3 Test Categories
- 8 core categories from BFCL benchmark:
  - Non-live: simple, multiple, parallel, parallel_multiple
  - Live: live_simple, live_multiple, live_parallel, live_parallel_multiple
- Focus on single-turn function calling scenarios

### 3.4 Evaluation Pipeline
- Extended Berkeley Function-Calling Leaderboard (BFCL)
- Implemented XML parser with type attributes for accurate evaluation
- Added "Prompt Variation Accuracy" metric (average across 8 categories)

## 4. Key Findings

### 4.1 Overall Performance
- GPT-4.1: 80.13% (best, most robust)
- GPT-4o-mini: 79.37%
- GPT-4o: 78.91%

### 4.2 Prompt Sensitivity
- **GPT-4o-mini**: Most sensitive (19.6 percentage point range, 66-86%)
- **GPT-4o**: Moderate sensitivity (10.7 pp range)
- **GPT-4.1**: Most robust (8.5 pp range)
- Finding: Smaller models show 2-4× more variation than larger/newer models

### 4.3 Best Configurations per Model
- **GPT-4o-mini**: XML response + JSON docs (86.0%)
- **GPT-4o**: Python response + JSON docs (84.6%)
- **GPT-4.1**: XML_tagged response + JSON docs (85.5%)
- Key insight: No single "best" format across all models

### 4.4 Response Format Analysis
- **Tagged formats consistently underperform**: 74-77% vs 80-81% for untagged
- JSON documentation (doc_fmt=json) generally performs best across all models
- Response format preference varies by model architecture

### 4.5 Common Failure Patterns
- Type conversion errors (integers provided where floats expected)
- Missing optional parameters
- Boolean value errors (providing `false` when only `true` is valid)
- These errors vary significantly by prompt format

## 5. Discussion

### 5.1 Implications for Practitioners
- Format choice matters more than previously thought (up to 20pp difference)
- Smaller models require more careful format selection
- JSON documentation appears to be a safe default
- Avoid tagged response formats unless specifically required

### 5.2 Implications for Model Developers
- Prompt robustness should be a key evaluation metric
- Newer/larger models show better format invariance
- Training should emphasize format generalization

### 5.3 Limitations
- Focus on OpenAI models (GPT family)
- Single-turn scenarios only
- Limited to 8 core categories from BFCL

## 6. Future Work
- Extend to other model families (Anthropic, Google, open-source)
- Multi-turn scenarios
- Impact of prompt paraphrasing
- Interaction effects between format choices

## 7. Conclusion
- First systematic study of prompt format variations in function calling
- Significant performance differences (up to 20pp) based on format alone
- Format selection should be part of LLM application design process
- Open-source tools and data released for reproducibility

## Appendices
- Detailed results tables (all 18 variations × 3 models)
- Example prompts for each format
- Failure case analysis with examples
- Code repository and reproduction instructions

