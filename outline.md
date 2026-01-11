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

## 3. Key Findings [DRAFTED - see 03-results.md]

### 3.1 Overall Performance
- 11 models tested (OpenAI GPT-4/5 series, reasoning models o1/o3-mini, xAI Grok series)
- Grok models lead: grok-3-mini (83.5%), grok-3-beta (83.1%), grok-4 (79.5%)
- OpenAI: GPT-4.1 (80.1%), GPT-4o-mini (79.4%), GPT-4o (78.9%)
- Reasoning: o1 (78.4%), o3-mini (77.0%)

### 3.2 Prompt Sensitivity
- **Most sensitive**: o3-mini (21.2pp), gpt-4o-mini (19.6pp), gpt-5-nano (20.0pp)
- **Most robust**: GPT-4.1 (8.5pp), o1 (9.8pp), gpt-5 (10.1pp)
- **Grok models**: Moderate sensitivity (10-12pp range)
- Finding: Generally smaller models more sensitive, but grok-3-mini bucks trend

### 3.3 Best Configurations per Model  
- **All models prefer JSON documentation** (doc_fmt=json)
- Response format highly variable: Python (4 models), XML (4 models), JSON (2 models), XML_tagged (1 model)
- **Key finding**: Grok models prefer json_tagged format (unlike all OpenAI models)
- grok-3-mini achieves highest single configuration: 88.5% with json+json

### 3.4 Response Format Analysis
- **Tagged format effects are model-specific**: Hurt OpenAI models (74-77% vs 80-81%) but help Grok models
- JSON documentation universally best: 3-4pp advantage over Python/XML
- XML struggles with type inference (31.4% type mismatch errors)
- Python has enum value issues (38.4% of errors)

### 3.5 Model Size vs Format Sensitivity
- Clear trend: smaller models more sensitive to format choice
- GPT-4 series: gpt-4o-mini (19.6pp) vs gpt-4o (10.7pp) vs gpt-4.1 (8.5pp)
- GPT-5 series: nano (20pp) → mini (17.7pp) → full (10.1pp)
- Exceptions: grok-3-mini bucks trend with strong performance despite size

### 3.6 Common Failure Modes
- Type coercion failures (34%): XML parses arrays as strings
- Optional parameter omission (28%): Format affects default handling
- Tagging-induced confusion (22%): Tags cause premature closes or incomplete outputs
- Concrete examples showing when one format works but variations fail

### 3.7 Category-Level Analysis
- Simple calls: Low format impact (5-8pp)
- Multiple parallel calls: High impact (12-22pp)
- Error pattern analysis by format

### 3.8 Statistical Significance
- All performance differences statistically significant
- Confirms format effects are not due to chance

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

