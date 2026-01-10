# Blog Post Outline: Prompt Format Variations in Function Calling

## 1. Introduction and Motivation [DRAFTED - see 01-intro-motivation.md]
- Function calling as cornerstone capability for modern LLMs (tools, APIs, agents)
- The gap: Wide variation in prompting formats but little systematic research on impact
- Why it matters: Wrong format choice can cause 15+ percentage point accuracy drops
- Real-world stakes: Production failures, expensive retries, wasted development time
- Current practice: Trial-and-error approach without understanding tradeoffs
- Our contribution: First systematic study of format variations in function calling
- Key findings preview: Up to 20pp swings, smaller models 2-4× more sensitive, no universal best format

## 2. Methodology [DRAFTED - see 02-methodology.md]

### 2.1 Prompt Variation Design
- Two orthogonal dimensions varied systematically:
  - **Response Format (res_fmt)**: JSON, XML, Python (6 variants with/without tags)
  - **Documentation Format (doc_fmt)**: JSON, XML, Python (how functions presented in system prompt)
- Total: 18 unique configurations (6 response × 3 documentation formats)
- Concrete examples included for calculate_area function showing:
  - JSON docs: OpenAPI-style schemas
  - Python docs: Docstring format with type hints  
  - XML docs: XML Schema-inspired
  - Corresponding response formats for each

### 2.2 Evaluation Setup
- 8 single-turn BFCL categories covering key complexity dimensions (simple/multiple/parallel/parallel_multiple in simulated and live settings)
- Follow original BFCL methodology and scoring
- Accuracy measured as % correctly formatted calls with valid parameters
- Tested across multiple state-of-the-art models

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

