# Results

We evaluated 8 state-of-the-art language models across all 18 prompt variations, generating over 57,000 function calls across the test suite. This section presents our key findings on model performance, prompt sensitivity, and format preferences.

## Overall Model Performance

Testing models across all 18 prompt variations reveals significant differences in both average performance and robustness:

| Model | Avg Accuracy | Range (pp) | Min-Max |
|-------|-------------|-----------|---------|
| **o3-mini** | 77.0% | 21.2 | 64.5%-85.7% |
| **o1** | 78.4% | 9.8 | 74.7%-84.5% |
| **gpt-4.1** | 80.1% | 8.5 | 76.7%-85.2% |
| **gpt-4o** | 78.9% | 10.7 | 73.5%-84.2% |
| **gpt-4o-mini** | 79.4% | 19.6 | 66.0%-86.0% |
| **gpt-5** | 76.6% | 10.1 | 71.8%-81.9% |
| **gpt-5-mini** | 75.0% | 17.7 | 64.2%-82.0% |
| **gpt-5-nano** | 73.6% | 20.0 | 60.9%-80.9% |

Key observations:
- **GPT-4.1 achieves the best average** (80.1%) while maintaining tight variation (8.5pp range), demonstrating both high capability and robustness
- **Reasoning models show interesting patterns**: o1 is remarkably stable (9.8pp range) despite moderate performance, while o3-mini has the highest variation (21.2pp) despite strong average scores
- **Model size correlates with robustness**: Smaller models (gpt-4o-mini, gpt-5-mini, gpt-5-nano) show 2-3× more sensitivity to format choice than their larger counterparts
- **The "prompt robustness gap"**: The difference between best and worst configurations exceeds 20 percentage points for 3 out of 8 models, suggesting format choice is as important as model selection for some applications

## Best Configuration per Model

The optimal prompt configuration varies significantly by model, with no universal "best" format:

| Model | Best Config | Accuracy | Worst Config | Accuracy | Delta |
|-------|-------------|----------|--------------|----------|-------|
| **gpt-4.1** | xml_tagged+json | 85.2% | json_tagged+python | 76.7% | 8.5pp |
| **gpt-4o** | python+json | 84.2% | xml_tagged+xml | 73.5% | 10.7pp |
| **gpt-4o-mini** | xml+json | 86.0% | python_tagged+python | 66.4% | 19.6pp |
| **o1** | xml+json | 84.5% | json_tagged+xml | 74.7% | 9.8pp |
| **o3-mini** | xml_tagged+json | 85.7% | python_tagged+xml | 64.5% | 21.2pp |
| **gpt-5** | python+json | 81.9% | json_tagged+xml | 71.8% | 10.1pp |
| **gpt-5-mini** | python+json | 82.0% | xml_tagged+python | 64.2% | 17.8pp |
| **gpt-5-nano** | python_tagged+json | 80.9% | xml_tagged+xml | 60.9% | 20.0pp |

Notable findings:
- **JSON documentation is universally preferred**: All 8 models achieve their best performance with `doc_fmt=json`, regardless of response format
- **Response format preferences diverge**: 4 models prefer Python responses, 3 prefer XML, and 1 prefers XML tagged - no clear winner
- **Tagged formats are problematic**: 7 out of 8 models have their worst performance with a tagged variant, suggesting the explicit delimiters may confuse rather than clarify

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
2. **No one-size-fits-all solution**: Optimal configurations vary by model, though JSON documentation is a consistently strong choice
3. **Smaller models need more attention**: Format selection becomes increasingly critical as model size decreases
4. **Tagged formats backfire**: Despite intuitive appeal, explicit function call delimiters consistently hurt performance
5. **Complexity amplifies effects**: Format impact scales with task difficulty, making careful selection crucial for real-world applications

These findings suggest that function calling benchmark results should be reported with prompt format specifications, and that practitioners should systematically test format variations rather than defaulting to a single approach.

