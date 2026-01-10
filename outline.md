# Blog Post Outline: Prompt Format Variations in Function Calling

## 1. Introduction and Motivation [DRAFTED - see 01-intro-motivation.md]
- Brief overview of function calling in LLMs
- Function calling is critical for LLM applications (tools, APIs, agents)
- The challenge: Different prompting formats exist (JSON, XML, Python) but little research on their impact
- Existing benchmarks focus on "what" models can do, not "how" format affects performance
- Real-world implications: choosing the wrong format could significantly impact application reliability
- Current practice is largely trial-and-error
- Our research question: How sensitive are LLMs to prompt format variations in function calling tasks?

## 2. Methodology [DRAFTED - see 02-methodology.md]

### 2.1 Prompt Variation Design
- **Response Format (res_fmt)**: JSON, XML, Python
  - With and without tags (e.g., `<function_call>`)
  - Controls how the model outputs function calls
- **Documentation Format (doc_fmt)**: JSON, XML, Python
  - Controls how function signatures are presented to the model in the system prompt
- Total: 18 combinations (3 res_fmt × 2 tagged/untagged × 3 doc_fmt)
- Examples of each format variation with side-by-side comparisons

### 2.2 Test Categories
- We evaluate on 8 core single-turn categories from the Berkeley Function-Calling Leaderboard (BFCL), chosen to cover the key complexity dimensions: simple, multiple, parallel, and parallel_multiple function calling, in both simulated and live API settings.
- Evaluation follows the original BFCL methodology

## 3. Key Findings

### 3.1 Overall Performance
- GPT-4.1: 80.13% (best, most robust)
- GPT-4o-mini: 79.37%
- GPT-4o: 78.91%

### 3.2 Prompt Sensitivity
- **GPT-4o-mini**: Most sensitive (19.6 percentage point range, 66-86%)
- **GPT-4o**: Moderate sensitivity (10.7 pp range)
- **GPT-4.1**: Most robust (8.5 pp range)
- Finding: Smaller models show 2-4× more variation than larger/newer models

### 3.3 Best Configurations per Model
- **GPT-4o-mini**: XML response + JSON docs (86.0%)
- **GPT-4o**: Python response + JSON docs (84.6%)
- **GPT-4.1**: XML_tagged response + JSON docs (85.5%)
- Key insight: No single "best" format across all models

### 3.4 Response Format Analysis
- **Tagged formats consistently underperform**: 74-77% vs 80-81% for untagged
- JSON documentation (doc_fmt=json) generally performs best across all models
- Response format preference varies by model architecture

### 3.5 Common Failure Patterns
- Type conversion errors (integers provided where floats expected)
- Missing optional parameters
- Boolean value errors (providing `false` when only `true` is valid)
- These errors vary significantly by prompt format

## 4. Discussion

### 4.1 Implications for Practitioners
- Format choice matters more than previously thought (up to 20pp difference)
- Smaller models require more careful format selection
- JSON documentation appears to be a safe default
- Avoid tagged response formats unless specifically required

### 4.2 Implications for Model Developers
- Prompt robustness should be a key evaluation metric
- Newer/larger models show better format invariance
- Training should emphasize format generalization

### 4.3 Limitations
- Focus on OpenAI models (GPT family)
- Single-turn scenarios only
- Limited to 8 core categories from BFCL

## 5. Future Work
- Extend to other model families (Anthropic, Google, open-source)
- Multi-turn scenarios
- Impact of prompt paraphrasing
- Interaction effects between format choices

## 6. Conclusion
- First systematic study of prompt format variations in function calling
- Significant performance differences (up to 20pp) based on format alone
- Format selection should be part of LLM application design process
- Open-source tools and data released for reproducibility

## Appendices
- Detailed results tables (all 18 variations × 3 models)
- Example prompts for each format
- Failure case analysis with examples
- Code repository and reproduction instructions

