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

## 3. Results [DRAFTED - see 03-results.md]

### 3.1 Overall Model Performance
- Performance tables for 11 models across 18 prompt variations
- Grok models lead: grok-3-mini (83.5%), grok-3-beta (83.1%)
- GPT-4.1 most robust (8.5pp variation range)
- Smaller models generally show 2-3× more sensitivity

### 3.2 Best Configuration per Model
- Tables showing optimal and worst configurations
- All 11 models achieve best performance with JSON documentation
- Response format preferences diverge across models and vendors
- Grok models prefer tagged formats (unlike OpenAI models)

### 3.3 Response Format Analysis
- Tagged vs untagged performance tables
- Documentation format impact: JSON outperforms Python/XML by 3.3-3.7pp

### 3.4 Category-Level Analysis
- Simple calls: Low format impact (5-8pp)
- Multiple parallel calls: High impact (12-22pp)

### 3.5 Error Pattern Analysis
- Distribution tables by error type and format
- XML: 31.4% type mismatch errors
- Python: 38.4% invalid enum errors

### 3.6 Statistical Significance
- All differences statistically significant (p < 0.05)
- Confirms format effects are genuine

## 4. Discussion [DRAFTED - see 04-discussion.md]

### 4.1 Prompt Sensitivity by Model Family
- **GPT-4 Family**: Decreasing sensitivity across generations (19.6pp → 10.7pp → 8.5pp)
- **GPT-5 Family**: Clear size correlation, all prefer python+json
- **Reasoning Models**: o1 stable (9.8pp), o3-mini highly sensitive (21.2pp)
- **xAI Grok Family**: Moderate sensitivity, unique tagging preference

### 4.2 Model Size vs Format Sensitivity
- Smaller models show 2-3× more variation than larger counterparts
- Hypotheses: training diversity, capacity for abstraction, error propagation
- Exceptions: grok-3-mini achieves strong robustness despite "mini" size

### 4.3 Common Failure Modes
- Type coercion failures (34%): XML parsing issues with arrays
- Optional parameter omission (28%): Format affects default handling
- Tagging-induced confusion (22%): Tags cause incomplete outputs
- Detailed examples showing when formats fail

### 4.4 Cross-Vendor Differences
- OpenAI models: 7 out of 8 perform worst with tagged variants
- xAI Grok models: 2 out of 3 prefer tagged formats
- Format patterns not universal across vendors

### 4.5 Implications for Practitioners
- Format choice impacts performance up to 20pp
- JSON documentation as safe default
- Response format requires model-specific testing
- Smaller models need more careful format selection

### 4.6 Implications for Model Developers
- Prompt robustness as evaluation metric (variation range)
- Training for format generalization is feasible
- Cross-vendor divergence suggests tractable design choices

### 4.7 Limitations
- Focus on OpenAI and xAI models
- Single-turn scenarios only
- Limited to 8 core BFCL categories
- Exact match evaluation methodology

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

