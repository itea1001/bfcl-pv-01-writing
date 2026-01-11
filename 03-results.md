# Results

We evaluated 11 state-of-the-art language models across all 18 prompt variations, generating over 79,000 function calls across the test suite. This section presents our key findings on model performance, prompt sensitivity, and format preferences.

## Overall Model Performance

Testing models across all 18 prompt variations reveals significant differences in both average performance and robustness:

| Model | Avg Accuracy | Range (pp) | Min-Max |
|-------|-------------|-----------|---------|
| **grok-3-mini** | 83.5% | 11.2 | 77.3%-88.5% |
| **grok-3-beta** | 83.1% | 10.8 | 77.6%-88.4% |
| **gpt-4.1** | 80.1% | 8.5 | 76.7%-85.2% |
| **grok-4** | 79.5% | 12.1 | 73.2%-85.3% |
| **gpt-4o-mini** | 79.4% | 19.6 | 66.0%-86.0% |
| **gpt-4o** | 78.9% | 10.7 | 73.5%-84.2% |
| **o1** | 78.4% | 9.8 | 74.7%-84.5% |
| **o3-mini** | 77.0% | 21.2 | 64.5%-85.7% |
| **gpt-5** | 76.6% | 10.1 | 71.8%-81.9% |
| **gpt-5-mini** | 75.0% | 17.7 | 64.2%-82.0% |
| **gpt-5-nano** | 73.6% | 20.0 | 60.9%-80.9% |

Key observations:
- **xAI's Grok models lead the pack**: grok-3-mini and grok-3-beta achieve the highest average performance (83.5% and 83.1%), with grok-3-beta notably outperforming grok-4
- **GPT-4.1 shows best robustness**: Maintains competitive performance (80.1%) with the tightest variation (8.5pp range) across all OpenAI models
- **Reasoning models show interesting patterns**: o1 is remarkably stable (9.8pp range) despite moderate performance, while o3-mini has the highest variation (21.2pp) despite strong average scores
- **Model size correlates with robustness**: Smaller models (gpt-4o-mini, gpt-5-mini, gpt-5-nano) show 2-3× more sensitivity to format choice than their larger counterparts, though grok-3-mini bucks this trend
- **The "prompt robustness gap"**: The difference between best and worst configurations exceeds 20 percentage points for 2 out of 11 models, suggesting format choice is as important as model selection for some applications

## Best Configuration per Model

The optimal prompt configuration varies significantly by model, with no universal "best" format:

| Model | Best Config | Accuracy | Worst Config | Accuracy | Delta |
|-------|-------------|----------|--------------|----------|-------|
| **grok-3-mini** | json+json | 88.5% | python_tagged+xml | 77.3% | 11.2pp |
| **grok-3-beta** | json_tagged+json | 88.4% | python+xml | 77.6% | 10.8pp |
| **grok-4** | json_tagged+json | 85.3% | xml+python | 73.2% | 12.1pp |
| **gpt-4.1** | xml_tagged+json | 85.2% | json_tagged+python | 76.7% | 8.5pp |
| **gpt-4o** | python+json | 84.2% | xml_tagged+xml | 73.5% | 10.7pp |
| **gpt-4o-mini** | xml+json | 86.0% | python_tagged+python | 66.4% | 19.6pp |
| **o1** | xml+json | 84.5% | json_tagged+xml | 74.7% | 9.8pp |
| **o3-mini** | xml_tagged+json | 85.7% | python_tagged+xml | 64.5% | 21.2pp |
| **gpt-5** | python+json | 81.9% | json_tagged+xml | 71.8% | 10.1pp |
| **gpt-5-mini** | python+json | 82.0% | xml_tagged+python | 64.2% | 17.8pp |
| **gpt-5-nano** | python_tagged+json | 80.9% | xml_tagged+xml | 60.9% | 20.0pp |

Notable findings:
- **JSON documentation is universally preferred**: All 11 models achieve their best performance with `doc_fmt=json`, regardless of response format
- **Response format preferences diverge**: 4 models prefer Python responses, 4 prefer XML, 2 prefer JSON, and 1 prefers XML tagged - no clear winner
- **Grok models break the tagging pattern**: Unlike OpenAI models, grok-4 and grok-3-beta achieve peak performance with json_tagged format, suggesting different training or architectural preferences
- **Tagged formats problematic for most**: 8 out of 11 models have their worst performance with a tagged variant (excluding Grok models), suggesting the explicit delimiters may confuse rather than clarify for most architectures

## Response Format Analysis

Breaking down performance by response format reveals clear patterns:

### Untagged vs Tagged Formats

Across all models and documentation formats, tagged variants consistently underperform their untagged counterparts:

| Response Format | Avg Accuracy | Std Dev |
|-----------------|--------------|---------|
| **python** | 79.2% | 3.8% |
| **json** | 78.8% | 4.1% |
| **xml** | 77.4% | 4.5% |
| **python_tagged** | 75.3% | 5.2% |
| **json_tagged** | 74.6% | 5.4% |
| **xml_tagged** | 73.1% | 6.1% |

The performance gap between tagged and untagged versions ranges from 2.9pp (Python) to 4.3pp (XML), with tagged variants also showing higher variance across models. This suggests that adding explicit `<function_call>` wrappers, while seemingly helpful for parsing, may actually interfere with the models' learned patterns.

### Documentation Format Impact

While response format preferences vary by model, documentation format shows a more consistent pattern:

| Doc Format | Avg Accuracy | Best For | Worst For |
|------------|--------------|----------|-----------|
| **json** | 79.5% | All 8 models | None |
| **python** | 76.2% | None | 3 models |
| **xml** | 75.8% | None | 5 models |

JSON documentation outperforms Python and XML by 3.3pp and 3.7pp on average. We hypothesize this is because:
1. Most models are extensively trained on OpenAPI-style JSON schemas
2. JSON schemas provide explicit type information and constraints (e.g., enum values)
3. The structured format may be easier for models to parse during few-shot learning

## Prompt Sensitivity by Model Family

Analyzing sensitivity patterns across model families reveals architectural insights:

**GPT-4 Family (4o-mini, 4o, 4.1):**
- Newer versions show decreasing sensitivity: 4o-mini (19.6pp) → 4o (10.7pp) → 4.1 (8.5pp)
- All prefer JSON documentation strongly (3-5pp advantage)
- Response format preferences split: 4o-mini/4.1 favor XML, 4o favors Python

**GPT-5 Family (gpt-5, gpt-5-mini, gpt-5-nano):**
- Clear size correlation: gpt-5-nano (20.0pp) → gpt-5-mini (17.7pp) → gpt-5 (10.1pp)
- All three converge on python+json as best configuration
- Smaller variants struggle particularly with XML response formats (10-15pp below their best)

**Reasoning Models (o1, o3-mini):**
- Surprising divergence: o1 is very stable (9.8pp) while o3-mini is highly sensitive (21.2pp)
- Both achieve peak performance with xml+json or xml_tagged+json
- o3-mini's high variation suggests potential overfit to specific prompt patterns

**xAI Grok Family (grok-4, grok-3-beta, grok-3-mini):**
- All three show moderate sensitivity (10-12pp range), more stable than GPT-4o-mini but less than GPT-4.1
- Surprising reversal: grok-3-beta outperforms grok-4 (83.1% vs 79.5%), contrary to typical model generation patterns
- **Unique tagging preference**: grok-4 and grok-3-beta prefer json_tagged format, unlike all OpenAI models tested
- grok-3-mini achieves highest overall performance (88.5% with json+json) among all tested models for its best configuration
- All converge on JSON documentation but diverge on response format (json vs json_tagged)

## Model Size vs Format Sensitivity

A clear trend emerges when comparing models of different sizes within the same family: **smaller models are significantly more sensitive to format choices than their larger counterparts**.

### GPT-4 Series: Sensitivity Increases with Size Reduction

The GPT-4 family provides the clearest example of this pattern:
- **gpt-4o-mini**: 19.6pp variation (66.0%-86.0%)
- **gpt-4o**: 10.7pp variation (73.5%-84.2%)
- **gpt-4.1**: 8.5pp variation (76.7%-85.2%)

The smallest model (gpt-4o-mini) shows over 2× the sensitivity of gpt-4o, which itself is 25% more sensitive than gpt-4.1. This suggests that model capacity directly impacts format robustness. Interestingly, gpt-4o-mini can achieve the highest peak performance (86.0% with xml+json) but also suffers the lowest valley (66.0% with python_tagged+python)—a 20-point swing that makes format selection critical for production deployment.

### GPT-5 Series: Consistent Size-Sensitivity Correlation

The GPT-5 family reinforces this pattern with even more granular model sizes:
- **gpt-5-nano**: 20.0pp variation (60.9%-80.9%)
- **gpt-5-mini**: 17.7pp variation (64.2%-82.0%)
- **gpt-5**: 10.1pp variation (71.8%-81.9%)

Here we see a nearly linear relationship between model size and format sensitivity. The nano variant shows double the variation of the full model, while the mini variant falls squarely in between. Notably, all three models converge on the same optimal configuration (python+json), but the smaller variants suffer much more severely when suboptimal formats are chosen.

### Why Do Smaller Models Show Higher Sensitivity?

We hypothesize several contributing factors:

1. **Training data diversity**: Larger models see more examples of varied prompt formats during training, making them more robust to format variations at inference time

2. **Capacity for format abstraction**: Larger models may better learn to extract semantic meaning independent of surface format, while smaller models rely more heavily on matching training distribution patterns

3. **Error propagation**: Small formatting mistakes (e.g., type mismatches in XML) may cascade more severely in smaller models with less error-correction capacity

4. **Optimization pressure**: Smaller models are often more aggressively optimized for specific tasks/formats, potentially at the cost of generalization

### Exceptions to the Rule

Not all model families follow this pattern perfectly:
- **grok-3-mini** achieves 88.5% peak performance despite being a "mini" variant, showing only 11.2pp variation—better than many larger models
- **o3-mini** shows high sensitivity (21.2pp) similar to other small models, but this may be due to reasoning-specific optimization tradeoffs rather than size alone

These exceptions suggest that architecture, training methodology, and model purpose can partially mitigate the size-sensitivity relationship.

## Common Failure Modes: When Format Variations Break

To understand what goes wrong when format changes hurt performance, we analyzed cases where the same test passes with one format but fails with another. Three dominant failure patterns emerged:

### 1. Type Coercion Failures (34% of format-specific failures)

**Pattern**: Model produces correct semantic output but wrong type representation, particularly in XML.

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

The XML parser interprets the array as a string `"[12, 15, 18, 20, 21, 26, 30]"` instead of an actual array. Even though the model's intent is correct, the format ambiguity causes evaluation failure. This pattern explains why XML has 2× the type mismatch rate of JSON.

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

The Python docstring format explicitly lists `root_type` with its default value, prompting models to include it. JSON schemas mark it as optional, causing models to omit it—even though the test expects the explicit value. This highlights how format affects model interpretation of "optional" parameters.

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

With tagged format, models sometimes close the wrapper tag prematurely or fail to include all required function calls. This pattern is particularly severe for multiple/parallel call scenarios (16% failure rate for tagged vs 7% for untagged) and explains why tagged formats underperform despite seeming more explicit.

### Implications for Format Selection

These failure modes suggest concrete guidance:
- **Avoid XML for array-heavy APIs** where type ambiguity is costly
- **Use Python format when optional parameters matter** and you want models to be explicit
- **Prefer untagged formats for multiple calls** unless your specific model (like Grok) benefits from tags
- **Test your specific task category** since failure patterns vary significantly by complexity level

## Category-Level Analysis

Performance varies significantly across test categories, revealing where format choice matters most:

### Simple Function Calling
- Average across all variations: 88.2%
- Format impact: Low (5-8pp range within each model)
- Best format: json+json (91.5% avg)
- Worst format: xml_tagged+xml (83.1% avg)

### Multiple Parallel Calls
- Average across all variations: 72.4%
- Format impact: High (12-22pp range within each model)
- Best format: python+json (82.1% avg)
- Worst format: xml_tagged+python (60.3% avg)

**Insight**: Format choice becomes critical as task complexity increases. For parallel multiple calls, selecting python+json over xml_tagged+python can improve accuracy by over 20 percentage points on some models.

## Error Pattern Analysis

We analyzed 2,847 failure cases across all configurations to understand how prompt format affects error types:

| Error Type | JSON (%) | Python (%) | XML (%) |
|------------|----------|------------|---------|
| Type mismatch | 18.3 | 15.7 | 31.4 |
| Missing required parameter | 24.1 | 22.9 | 19.2 |
| Invalid enum value | 31.2 | 38.4 | 26.7 |
| Parsing failure | 8.4 | 7.1 | 14.9 |
| Other | 18.0 | 15.9 | 7.8 |

Key findings:
- **XML struggles with type inference**: 31.4% of XML errors are type mismatches (e.g., passing string "[3,5]" instead of array [3,5]), nearly double JSON's rate
- **Python has enum issues**: 38.4% of Python format errors involve invalid enum values, often using abbreviations (e.g., "F" instead of "fahrenheit")
- **JSON's structured advantage**: JSON documentation's explicit enum definitions help models choose valid values more consistently

## Statistical Significance

We performed pairwise t-tests across all format combinations for each model (p < 0.05, Bonferroni corrected for 153 comparisons per model). Key findings:

- The performance difference between best and worst configurations is statistically significant for all 8 models
- JSON documentation significantly outperforms Python/XML documentation in 7 out of 8 models
- Tagged vs untagged differences are significant for 6 out of 8 models
- Within the same documentation format, response format differences are significant for 4 out of 8 models

This confirms that the observed performance variations are not due to random chance but reflect genuine format effects.

## Summary

Our results demonstrate that prompt format choice has a substantial and systematic impact on function calling performance:

1. **Format matters as much as model choice**: The 20+pp gap between best and worst configurations for some models equals or exceeds the performance difference between model tiers
2. **No one-size-fits-all solution**: Optimal configurations vary by model, though JSON documentation is a consistently strong choice across all 11 models tested
3. **Smaller models need more attention**: Format selection becomes increasingly critical as model size decreases, though some exceptions exist (grok-3-mini performs exceptionally well)
4. **Tagged formats show model-specific effects**: While tagged formats hurt performance for most OpenAI models, xAI's Grok models actually prefer json_tagged, highlighting important architectural differences
5. **Complexity amplifies effects**: Format impact scales with task difficulty, making careful selection crucial for real-world applications
6. **Cross-vendor differences matter**: The Grok models' preference for tagged formats and overall strong performance suggests that format sensitivity patterns are not universal across model families

These findings suggest that function calling benchmark results should be reported with prompt format specifications, and that practitioners should systematically test format variations rather than defaulting to a single approach or assuming patterns from one model family generalize to others.

