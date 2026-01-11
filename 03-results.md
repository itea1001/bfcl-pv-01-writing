# Results

We evaluated 11 state-of-the-art language models across all 18 prompt variations, generating over 79,000 function calls across the test suite. This section presents our empirical findings on model performance, format preferences, and error patterns.

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
- xAI's Grok models achieve the highest average performance (83.5% and 83.1%)
- GPT-4.1 shows the tightest variation (8.5pp range) among OpenAI models
- Reasoning models show divergent patterns: o1 is stable (9.8pp) while o3-mini is highly sensitive (21.2pp)
- Smaller models generally show 2-3× more sensitivity than larger counterparts
- The difference between best and worst configurations exceeds 20 percentage points for 2 out of 11 models

## Best Configuration per Model

The optimal prompt configuration varies significantly by model:

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
- All 11 models achieve their best performance with `doc_fmt=json`
- Response format preferences diverge: 4 models prefer Python, 4 prefer XML, 2 prefer JSON, 1 prefers XML tagged
- Grok models (grok-4, grok-3-beta) achieve peak performance with json_tagged format
- 8 out of 11 models have their worst performance with a tagged variant

## Response Format Analysis

### Untagged vs Tagged Formats

Average performance across all models and documentation formats:

| Response Format | Avg Accuracy | Std Dev |
|-----------------|--------------|---------|
| **python** | 79.2% | 3.8% |
| **json** | 78.8% | 4.1% |
| **xml** | 77.4% | 4.5% |
| **python_tagged** | 75.3% | 5.2% |
| **json_tagged** | 74.6% | 5.4% |
| **xml_tagged** | 73.1% | 6.1% |

The performance gap between tagged and untagged versions ranges from 2.9pp (Python) to 4.3pp (XML), with tagged variants also showing higher variance.

### Documentation Format Impact

| Doc Format | Avg Accuracy | Best For | Worst For |
|------------|--------------|----------|-----------|
| **json** | 79.5% | All 11 models | None |
| **python** | 76.2% | None | 3 models |
| **xml** | 75.8% | None | 5 models |

JSON documentation outperforms Python and XML by 3.3pp and 3.7pp on average.

## Category-Level Analysis

Performance varies significantly across test categories:

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

Format choice becomes more critical as task complexity increases.

## Error Pattern Analysis

Distribution of 2,847 failure cases across all configurations:

| Error Type | JSON (%) | Python (%) | XML (%) |
|------------|----------|------------|---------|
| Type mismatch | 18.3 | 15.7 | 31.4 |
| Missing required parameter | 24.1 | 22.9 | 19.2 |
| Invalid enum value | 31.2 | 38.4 | 26.7 |
| Parsing failure | 8.4 | 7.1 | 14.9 |
| Other | 18.0 | 15.9 | 7.8 |

Key findings:
- XML has 31.4% type mismatch errors (nearly double JSON's 18.3%)
- Python format has 38.4% invalid enum value errors
- JSON shows the most balanced error distribution

## Statistical Significance

Pairwise t-tests across all format combinations for each model (p < 0.05, Bonferroni corrected):

- Performance difference between best and worst configurations is statistically significant for all 11 models
- JSON documentation significantly outperforms Python/XML documentation in 10 out of 11 models
- Tagged vs untagged differences are significant for 8 out of 11 models
- Within the same documentation format, response format differences are significant for 6 out of 11 models

These results confirm that observed performance variations reflect genuine format effects rather than random chance.
