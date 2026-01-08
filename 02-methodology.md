# Methodology

Our experimental design focuses on systematically varying two key aspects of function calling prompts: how we present function documentation to the model, and what format we ask the model to use in its responses.

## Prompt Variation Design

We vary prompts along two orthogonal dimensions:

**1. Response Format (res_fmt)**: How the model should structure its function call outputs
- **JSON**: Structured key-value format
- **XML**: Markup-based format with opening/closing tags  
- **Python**: Function call syntax as it would appear in code

For each base format, we also test a **tagged variant** that wraps the entire response in explicit delimiters (e.g., `<function_call>...</function_call>` for XML). This gives us 6 response format options total (3 base × 2 tag variants).

**2. Documentation Format (doc_fmt)**: How function signatures and parameters are presented in the system prompt
- **JSON**: OpenAPI-style schemas with type annotations
- **XML**: XML Schema-inspired documentation
- **Python**: Python docstring format with type hints

Combining these gives us **18 unique prompt configurations** (6 response formats × 3 documentation formats).

### Example Format Variations

Here's how the same function appears across documentation formats:

**JSON Documentation:**
```json
{
  "name": "calculate_area",
  "description": "Calculate the area of a rectangle",
  "parameters": {
    "type": "object",
    "properties": {
      "width": {"type": "number", "description": "Width in meters"},
      "height": {"type": "number", "description": "Height in meters"}
    },
    "required": ["width", "height"]
  }
}
```

**Python Documentation:**
```python
def calculate_area(width: float, height: float) -> float:
    """
    Calculate the area of a rectangle
    
    Args:
        width: Width in meters
        height: Height in meters
    """
```

**XML Documentation:**
```xml
<function name="calculate_area">
  <description>Calculate the area of a rectangle</description>
  <parameters>
    <param name="width" type="number" required="true">
      Width in meters
    </param>
    <param name="height" type="number" required="true">
      Height in meters
    </param>
  </parameters>
</function>
```

And the corresponding response format variations would look like:

**JSON Response:** `{"function": "calculate_area", "parameters": {"width": 5.0, "height": 3.0}}`

**Python Response:** `calculate_area(width=5.0, height=3.0)`

**XML Response:** 
```xml
<function_call>
  <name>calculate_area</name>
  <arg name="width">5.0</arg>
  <arg name="height">3.0</arg>
</function_call>
```

## Evaluation Setup

We evaluate on 8 single-turn categories from the Berkeley Function-Calling Leaderboard: simple, multiple, parallel, and parallel_multiple function calling, each in both simulated and live API settings. These categories cover the key complexity dimensions in function calling while keeping the evaluation tractable.

All evaluations follow the original BFCL methodology, using the same test cases and scoring criteria. For each prompt variation, we measure accuracy as the percentage of test cases where the model produces a correctly formatted function call with valid parameter values.

We test each configuration across multiple state-of-the-art models to understand how format sensitivity varies with model size and architecture.

