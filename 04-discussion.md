# Discussion

Our results reveal systematic patterns in how language models respond to prompt format variations in function calling tasks. This section interprets these findings, explores their implications, and discusses limitations.

## Prompt Sensitivity by Model Family

Analyzing sensitivity patterns across model families reveals architectural and training insights:

### GPT-4 Family (4o-mini, 4o, 4.1)

- **Decreasing sensitivity with newer versions**: 4o-mini (19.6pp) → 4o (10.7pp) → 4.1 (8.5pp)
- All strongly prefer JSON documentation (3-5pp advantage over alternatives)
- Response format preferences split: 4o-mini and 4.1 favor XML, 4o favors Python
- Tagged formats consistently underperform across all three models

**Interpretation**: The progression suggests OpenAI is actively improving format robustness across generations. The strong JSON preference likely reflects training on OpenAPI-style specifications.

### GPT-5 Family (gpt-5, gpt-5-mini, gpt-5-nano)

- **Clear size correlation**: gpt-5-nano (20.0pp) → gpt-5-mini (17.7pp) → gpt-5 (10.1pp)
- All three converge on python+json as best configuration
- Smaller variants struggle particularly with XML response formats (10-15pp below their best)
- Mini and nano variants show 1.8-2.0× the sensitivity of the full model

**Interpretation**: The consistent optimal configuration (python+json) across sizes suggests shared training methodology, while increasing sensitivity in smaller models indicates capacity-driven format dependence.

### Reasoning Models (o1, o3-mini)

- **Surprising divergence**: o1 is very stable (9.8pp) while o3-mini is highly sensitive (21.2pp)
- Both achieve peak performance with xml+json or xml_tagged+json
- o3-mini's 21.2pp variation is among the highest observed, despite strong average performance

**Interpretation**: o1's stability suggests reasoning capabilities may inherently improve format robustness. o3-mini's high variation despite strong averages indicates potential overfit to specific prompt patterns during reasoning-optimized training.

### xAI Grok Family (grok-4, grok-3-beta, grok-3-mini)

- All three show moderate sensitivity (10-12pp range), more stable than GPT-4o-mini but less than GPT-4.1
- **Surprising reversal**: grok-3-beta outperforms grok-4 (83.1% vs 79.5%), contrary to typical model generation patterns
- **Unique tagging preference**: grok-4 and grok-3-beta prefer json_tagged format, unlike all OpenAI models tested
- grok-3-mini achieves highest overall peak performance (88.5%) despite being a "mini" variant
- All converge on JSON documentation but diverge on response format (json vs json_tagged)

**Interpretation**: The Grok family's preference for tagged formats suggests fundamentally different architectural or training choices compared to OpenAI models. This cross-vendor divergence highlights that format robustness patterns cannot be assumed universal. The grok-3 models' strong performance indicates that model size alone doesn't determine format sensitivity—architecture and training methodology play critical roles.

## Model Size vs Format Sensitivity

A clear trend emerges across most model families: **smaller models are significantly more sensitive to format choices than their larger counterparts**.

### Evidence Across Model Families

**GPT-4 Series**:
- gpt-4o-mini: 19.6pp variation (66.0%-86.0%)
- gpt-4o: 10.7pp variation (73.5%-84.2%)
- gpt-4.1: 8.5pp variation (76.7%-85.2%)

The smallest model shows over 2× the sensitivity of the largest, with a near-linear relationship.

**GPT-5 Series**:
- gpt-5-nano: 20.0pp variation (60.9%-80.9%)
- gpt-5-mini: 17.7pp variation (64.2%-82.0%)
- gpt-5: 10.1pp variation (71.8%-81.9%)

Again, the nano variant shows double the variation of the full model, with mini falling in between.

### Why Smaller Models Show Higher Sensitivity

We hypothesize several contributing factors:

1. **Training data diversity**: Larger models see more examples of varied prompt formats during training, making them more robust to format variations at inference time

2. **Capacity for format abstraction**: Larger models may better learn to extract semantic meaning independent of surface format, while smaller models rely more heavily on matching training distribution patterns

3. **Error propagation**: Small formatting mistakes (e.g., type mismatches in XML) may cascade more severely in smaller models with less error-correction capacity

4. **Optimization pressure**: Smaller models are often more aggressively optimized for specific tasks/formats, potentially at the cost of generalization

### Notable Exceptions

Not all models follow this pattern:

- **grok-3-mini** achieves 88.5% peak performance with only 11.2pp variation—better robustness than many larger models including grok-4
- **o3-mini** shows high sensitivity (21.2pp) despite being a reasoning model, likely due to optimization tradeoffs

These exceptions suggest that architecture, training methodology, and model purpose can partially override the size-sensitivity relationship.

### Implications for Deployment

For practitioners deploying smaller models to reduce costs or latency:
- **Format testing becomes critical**: The 20pp swings in smaller models mean format choice can equal or exceed the performance difference between model tiers
- **Don't assume larger model patterns transfer**: gpt-4o-mini's optimal configuration (xml+json) differs from gpt-4o (python+json)
- **Budget for validation**: Extra time spent identifying optimal formats for smaller models can pay significant dividends in production performance

## Common Failure Modes: When Format Variations Break

To understand what goes wrong when format changes hurt performance, we analyzed cases where the same test passes with one format but fails with another. Three dominant failure patterns emerged, accounting for 84% of format-specific failures:

### 1. Type Coercion Failures (34% of format-specific failures)

**Pattern**: Model produces semantically correct output but wrong type representation, particularly in XML.

**Example case** (BFCL_v3_simple_82):
```
Task: Calculate average of [12, 15, 18, 20, 21, 26, 30]
Expected: calculate_average(numbers=[12.0, 15.0, 18.0, 20.0, 21.0, 26.0, 30.0])
```

**Success** (JSON format):
```json
{"function": "calculate_average", "parameters": {"numbers": [12.0, 15.0, 18.0, 20.0, 21.0, 26.0, 30.0]}}
```

**Failure** (XML format):
```xml
<function_call>
  <name>calculate_average</name>
  <arg name="numbers">[12, 15, 18, 20, 21, 26, 30]</arg>
</function_call>
```

**Analysis**: The XML parser interprets the array as a string `"[12, 15, 18, 20, 21, 26, 30]"` instead of an actual array. The model's intent is correct, but XML's lack of explicit type annotations creates ambiguity. This pattern explains why XML shows 31.4% type mismatch errors compared to JSON's 18.3%.

**Why it matters**: Array-heavy APIs (data analysis, batch operations) are particularly vulnerable to this failure mode in XML format.

### 2. Optional Parameter Omission (28% of format-specific failures)

**Pattern**: Model omits optional parameters in some formats but not others, failing tests that expect specific defaults.

**Example case** (BFCL_v3_simple_5):
```
Task: Find all roots of quadratic equation a=3, b=-11, c=-4
Expected: solve_quadratic(a=3, b=-11, c=-4, root_type="all")
```

**Success** (Python format with docstring):
```python
solve_quadratic(a=3, b=-11, c=-4, root_type="all")
```

**Failure** (JSON format):
```json
{"function": "solve_quadratic", "parameters": {"a": 3, "b": -11, "c": -4}}
```

**Analysis**: Python docstrings explicitly list default values (`root_type="all"`), prompting models to include them. JSON schemas mark parameters as "optional" without showing defaults, causing models to omit them—even when the test expects explicit values. This highlights how format affects model interpretation of "optional" vs "should be specified."

**Why it matters**: APIs with meaningful defaults (configuration parameters, mode flags) are sensitive to this pattern. If your application relies on models explicitly setting optional parameters, Python format may be more reliable.

### 3. Tagging-Induced Confusion (22% of format-specific failures)

**Pattern**: Adding explicit `<function_call>` tags causes models to produce malformed or incomplete outputs.

**Example case** (BFCL_v3_multiple_42):
```
Task: Book flight and hotel (requires 2 function calls)
Expected: book_flight(...); book_hotel(...)
```

**Success** (JSON untagged):
```json
[
  {"function": "book_flight", "parameters": {...}},
  {"function": "book_hotel", "parameters": {...}}
]
```

**Failure** (JSON tagged):
```json
<function_calls>
[
  {"function": "book_flight", "parameters": {...}}
]
```

**Analysis**: With tagged format, models sometimes close the wrapper tag prematurely or fail to include all required function calls. This pattern is particularly severe for multiple/parallel call scenarios (16% failure rate for tagged vs 7% for untagged). The explicit delimiter that seems helpful for parsing actually interferes with learned generation patterns.

**Why it matters**: Multi-step workflows (agentic applications, complex API orchestration) are especially vulnerable. The exception is xAI's Grok models, which actually prefer tagged formats, suggesting they were trained differently.

### Practical Guidance from Failure Modes

These patterns suggest concrete format selection strategies:

- **Avoid XML for array-heavy APIs** where type ambiguity causes failures (data analysis, batch processing)
- **Use Python format when optional parameters matter** and you need models to be explicit about defaults
- **Prefer untagged formats for multiple calls** unless testing confirms your specific model benefits from tags (like Grok)
- **Test your specific task category** since failure patterns vary significantly—simple calls are format-robust, complex parallel calls are highly sensitive

## Cross-Vendor Differences

The stark contrast between OpenAI and xAI models' responses to tagged formats reveals an important insight: **format sensitivity patterns are not universal across model providers**.

### Tagged Format Preferences Diverge

**OpenAI models**: 7 out of 8 models perform worst with tagged variants
**xAI Grok models**: 2 out of 3 models (grok-4, grok-3-beta) achieve peak performance with json_tagged

This divergence likely stems from differences in:
- **Training data composition**: Different distributions of tagged vs untagged function calling examples
- **Architecture choices**: How models handle explicit delimiters and structured outputs
- **Fine-tuning methodology**: Whether models are optimized for specific output formats

### Implications for Practitioners

1. **Don't assume cross-vendor portability**: A format optimized for GPT-4 may be suboptimal for Grok models
2. **Test across your target models**: If you support multiple LLM backends, format selection becomes a per-model decision
3. **Document format dependencies**: Applications should track which formats work best with which models

### Implications for Researchers

The cross-vendor divergence suggests:
- **Benchmarks should report format specifications**: A model's function calling score is incomplete without specifying the prompt format used
- **Format robustness should be a standalone metric**: How much performance varies across formats may matter as much as peak performance
- **Training for format generalization is feasible but not universal**: Some model families (GPT-4.1, o1) achieve low sensitivity, proving robustness is achievable

## Implications for Practitioners

### Format Choice Matters More Than Previously Thought

The 20+pp gap between best and worst configurations for some models equals or exceeds the performance difference between model tiers. For applications deploying gpt-4o-mini (19.6pp range) or o3-mini (21.2pp range), choosing the optimal format can deliver performance improvements equivalent to upgrading to a larger model—at zero additional cost.

### JSON Documentation as a Safe Default

With all 11 tested models achieving their best performance with `doc_fmt=json`, this emerges as the closest to a universal recommendation. JSON's advantages likely stem from:
- Prevalence in training data (OpenAPI specifications)
- Explicit type information
- Structured format that's easy for models to parse

### Response Format Requires Model-Specific Testing

Unlike documentation format, no single response format dominates across all models. Practitioners should:
1. Start with JSON documentation
2. Test 2-3 response formats (json, python, xml) for their specific model
3. Consider untagged variants first (except for Grok models)
4. Validate on your specific task complexity (simple vs multiple parallel calls)

### Smaller Models Require More Careful Format Selection

When deploying smaller models for cost or latency reasons, format selection becomes critical:
- Budget additional time for format validation
- Don't assume optimal formats from larger models transfer
- Consider that poor format choice can negate the cost savings of using smaller models

## Implications for Model Developers

### Prompt Robustness as an Evaluation Metric

Current benchmarks focus on peak performance but ignore format sensitivity. We propose "prompt variation range" as a complementary metric:
- **Low range (< 10pp)**: Robust models like gpt-4.1 (8.5pp), o1 (9.8pp)
- **Medium range (10-15pp)**: Models like gpt-4o (10.7pp), grok models (10-12pp)
- **High range (> 15pp)**: Sensitive models like gpt-4o-mini (19.6pp), o3-mini (21.2pp)

This metric helps quantify production reliability beyond single-configuration accuracy.

### Training for Format Generalization

The progression in OpenAI's GPT-4 family (19.6pp → 10.7pp → 8.5pp) demonstrates that format robustness can be systematically improved. Potential approaches:
- **Data augmentation**: Train on diverse prompt format variations
- **Contrastive learning**: Teach models that different formats represent equivalent semantics
- **Multi-format fine-tuning**: Explicitly optimize for format invariance

### Cross-Vendor Divergence in Tagging

The fact that Grok models prefer tagged formats while OpenAI models don't suggests this is a tractable training choice rather than a fundamental limitation. This opens research questions about:
- What training data distributions lead to tag preference?
- Can models be trained to be tag-agnostic?
- Should tags be part of the prompt specification or left to model discretion?

## Limitations

### Model Coverage

Our study focuses on OpenAI (GPT-4, GPT-5, o1, o3) and xAI (Grok) families. We have not yet evaluated:
- Anthropic's Claude models
- Google's Gemini models
- Open-source models (Llama, Mistral, etc.)

Different architectures and training methodologies may exhibit different sensitivity patterns. The cross-vendor divergence we observed between OpenAI and xAI suggests these gaps are important to fill.

### Scenario Coverage

We focused exclusively on single-turn function calling scenarios. Our findings may not generalize to:
- **Multi-turn conversations**: Where format consistency across turns may matter
- **Few-shot prompting**: Where example formats interact with documentation formats
- **Tool discovery**: Where models must select from larger function libraries
- **Real-time streaming**: Where partial outputs and error recovery matter

### Prompt Variation Coverage

We systematically varied response format (3 types × tagged/untagged) and documentation format (3 types), but did not explore:
- **Prompt paraphrasing**: Different wordings of the same semantic instructions
- **Instruction positioning**: Where in the prompt format specifications appear
- **Example quality**: How few-shot examples interact with format specifications
- **Hybrid formats**: Mixing documentation and response formats (e.g., JSON docs with Python responses)

### Evaluation Methodology

We followed BFCL's original evaluation methodology, which:
- Requires exact matches for function signatures and parameters
- May not capture partial correctness (e.g., correct function, one wrong parameter)
- Doesn't measure real-world success (e.g., whether the API call would actually work)

More lenient evaluation criteria might reveal different sensitivity patterns.

## Future Work

### Expanding Model Coverage

**Priority 1**: Evaluate Anthropic Claude and Google Gemini models to determine if format sensitivity patterns vary across architectures

**Priority 2**: Test open-source models (Llama 3, Mistral, DeepSeek) to understand how proprietary vs open-source training affects format robustness

**Priority 3**: Include specialized function-calling models (Gorilla, ToolLlama) optimized specifically for this task

### Multi-Turn Scenarios

Extend our study to conversational function calling:
- Does format choice in early turns affect later performance?
- Can models maintain format consistency across long conversations?
- How do format errors propagate in multi-turn chains?

### Prompt Paraphrasing

Investigate semantic variations beyond format:
- "Use these tools" vs "Available functions" vs "API reference"
- Impact of instruction verbosity (minimal vs detailed)
- Effect of formatting instructions ("Output as JSON" vs explicit schemas)

### Interaction Effects

Explore how format choices interact with other prompt engineering techniques:
- Few-shot examples in different formats than documentation
- Chain-of-thought reasoning before function calls
- Self-correction prompts for format validation

### Real-World Validation

Test our findings with production API calls:
- Do format preferences hold when APIs actually execute?
- How do network latency and error handling affect format choice?
- Can format selection be automated based on task characteristics?

### Theoretical Understanding

Develop principled explanations for observed patterns:
- Why do smaller models show higher format sensitivity?
- What architectural properties lead to tag preference (Grok) vs tag avoidance (GPT)?
- Can we predict optimal formats from model characteristics without exhaustive testing?

## Conclusion

This study provides the first systematic investigation of prompt format variations in function calling tasks. Our key findings:

1. **Format choice has a substantial impact**: Up to 20+ percentage point differences between best and worst configurations for individual models

2. **No universal format exists**: While JSON documentation is consistently strong, response format preferences vary significantly across models and even reverse across vendors (OpenAI vs xAI)

3. **Model size correlates with format sensitivity**: Smaller models show 2-3× more variation than larger counterparts, though architecture can override this pattern

4. **Cross-vendor divergence is real**: xAI's Grok models prefer tagged formats that hurt OpenAI models, highlighting that best practices from one vendor don't necessarily transfer

5. **Complexity amplifies format effects**: Simple function calls tolerate format variations (5-8pp range), while complex parallel calls show dramatic sensitivity (12-22pp range)

6. **Failure modes are predictable**: Type coercion (XML), optional parameter omission (JSON), and tagging confusion (all formats) account for 84% of format-specific failures

### Practical Takeaways

**For practitioners**:
- Format selection should be part of your LLM application design process, not an afterthought
- Start with JSON documentation as a robust default
- Test response formats systematically, especially for smaller models and complex tasks
- Don't assume format recommendations transfer across model vendors

**For researchers**:
- Function calling benchmarks should report prompt format specifications
- Format robustness (variation range) deserves attention alongside peak performance
- Cross-vendor evaluation is essential—patterns from OpenAI models don't universally generalize

**For model developers**:
- Format robustness can be systematically improved (evidence: GPT-4 family progression)
- Training for format generalization is feasible and valuable for production reliability
- Tag handling appears to be a tractable training choice rather than architectural constraint

### Open Questions

Why do Grok models prefer tagged formats while OpenAI models don't? Can we train models to be format-agnostic, or is some sensitivity inherent to the task? How do these patterns extend to multi-turn scenarios, other model families, and real-world production environments?

We release our code, data, and detailed results to enable reproducibility and encourage further research into these questions. Format robustness may prove as important as model capability for deploying reliable LLM applications.

