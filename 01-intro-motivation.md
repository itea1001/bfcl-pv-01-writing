# Introduction and Motivation

Function calling has become a cornerstone capability for modern large language models (LLMs), enabling them to interact with external tools, APIs, and databases. From personal assistants scheduling meetings to autonomous agents managing complex workflows, the ability to reliably invoke functions with correct parameters is critical for real-world LLM applications.

Despite its importance, there's a surprising gap in our understanding: **How sensitive are LLMs to the format in which we present function calling instructions?** Current practice shows wide variation in prompting approaches. Some systems use JSON-formatted schemas, others prefer Python-style function signatures, and still others adopt XML-based formats. Yet most practitioners choose formats based on intuition or anecdotal evidence rather than systematic evaluation.

This matters more than you might think. Existing function calling benchmarks like the Berkeley Function-Calling Leaderboard (BFCL) focus on *what* models can do—measuring their ability to handle different complexity levels and scenarios. But they don't tell us *how* format choices affect performance. If a model achieves 85% accuracy with one format and 70% with another, that's the difference between a production-ready system and one that frustrates users.

The real-world stakes are high. Application developers often discover format sensitivity only after deployment, when subtle prompt changes lead to unexpected failures. Consider a customer service bot that works perfectly in testing but degrades in production because the function documentation format doesn't match what the model was tuned on. Or an API integration that requires expensive model calls because the chosen format needs multiple retry attempts.

Current development workflows treat format selection as an afterthought, often copying examples from documentation without understanding the tradeoffs. This trial-and-error approach wastes development time and can miss significantly better configurations.

In this work, we conduct the first systematic study of prompt format variations in function calling. We evaluate how different ways of presenting both function documentation and expected response formats affect model performance across the BFCL benchmark. Our goal is simple: give practitioners and researchers the data they need to make informed decisions about format selection, and highlight prompt robustness as a key dimension for evaluating and improving LLMs.

**Our key findings preview:**
- Format choice can swing accuracy by up to 20 percentage points on the same tasks
- Smaller models show 2-4× more sensitivity to format variations than larger ones
- No single format wins across all models—optimal choice depends on model architecture
- Tagged response formats consistently underperform their untagged counterparts
- JSON documentation appears to be the most reliable default choice

The rest of this post details our methodology, presents comprehensive results across multiple models and format combinations, and discusses implications for both practitioners building applications and researchers developing the next generation of function-calling LLMs.

