---
title: "A Practical Look at Generating Structured Outputs with LLMs"
date: 2025-03-08
author: peng
categories: [Intelligence & ML]
tags: [AI, LLM]
math: true
---

Over the past weeks, I’ve tested various methods to get LLMs to produce consistent, well-structured responses—an essential step when integrating these models into programmatic solutions, where free-form text can break automated workflows and data pipelines. Below are two approaches that helped me solve these problems: prompt engineering and Pydantic schema validation.

## Structured Data for Reliability
Converting raw LLM text into predictable formats like JSON streamlines integration with APIs, databases, and UIs. Without structure, even extracting a single field can become error-prone; for instance, `{"date": "2024-10-17"}` is much easier to process than “Next Thursday.”

---

## Method 1: Prompt Engineering (The “Ask Nicely” Approach)
This approach relies on explicit instructions in the prompt to guide the model’s output format.


```python
response = client.chat.completions.create(
    model="gpt-4",
    messages=[
        {"role": "user", "content": "Extract keywords from: 'Quantum computing is revolutionizing cryptography.' Return as JSON: {keywords: []}"}
    ]
)

print(response.choices[0].message.content)
# Might look like:
# {"keywords": ["Quantum computing", "cryptography"]}
```

### Pros
- **Universal Compatibility**: Works with most models, including older ones.  
- **Flexibility**: No dependency on specific libraries or SDKs.

### Cons
- **Fragility**: Even with precise prompts, the model might omit fields, add typos, or return invalid JSON.  
- **No Type Safety**: Manual validation is required, increasing code complexity.

### When to Use
- Prototyping or one-off tasks.  
- Working with endpoints/models that don’t support JSON schema enforcement.

---

## Method 2: Pydantic Schema Validation (The “Strict Enforcer” Approach)
Using OpenAI’s SDK with **Pydantic**, you can define strict schemas and parse responses directly into Python objects.

```python
from pydantic import BaseModel

class ProductReview(BaseModel):
    product: str
    sentiment: str  # "positive", "neutral", "negative"
    features: list[str]

response = client.chat.completions.parse(
    model="gpt-4o",
    messages=[
        {"role": "user", "content": "Analyze this review: 'The camera is amazing, but battery life disappoints.'"}
    ],
    response_format=ProductReview
)

parsed_data = response.choices[0].message.parsed
print(parsed_data)
# ProductReview(product='Camera', sentiment='positive', features=['camera', 'battery life'])
```

### Pros
- **Type Safety**: Invalid outputs raise validation errors immediately.  
- **Integration**: Directly maps to Python objects, reducing boilerplate code.  
- **Refusal Handling**: Detects safety-based refusals programmatically.

### Cons
- **Model Limitations**: Requires newer models that fully support `response_format`.  
- **Schema Rigidity**: May struggle with ambiguous inputs (e.g., extracting data from messy text).

### When to Use
- Production systems needing reliability.  
- Complex or nested data structures (e.g., multi-field documents).

---

## Pydantic in Practice: Lessons Learned

**What Went Well**  
- **Validation as a Guardrail**  
  In one text-classification workflow, the LLM sometimes returned incomplete fields for category tags. By defining a `CategoryItem(BaseModel)` with the required keys, Pydantic immediately caught missing tags and enforced type correctness (e.g., `tags: list[str]`). This quick feedback loop saved a lot of time troubleshooting downstream logic.

- **IDE Support**  
  Pydantic models provided clear hints and autocomplete in the IDE. Whenever a new field was added or renamed, my code editor flagged mismatches, reducing errors before runtime.

**Common Pitfalls**  
- **Schema Design Too Strict**  
  If you define a field as `int` but the LLM returns `"12 pages"` instead of `12`, you’ll get a validation error. One solution is to allow a `str` type and parse it manually. Another is to instruct the model more explicitly to return only numeric data—though that sometimes still fails if the user prompt is ambiguous.

- **Model/Endpoint Constraints**  
  Some endpoints on AWS or Azure don’t fully support strict JSON schema modes. Pydantic is best paired with newer OpenAI models that accept a `response_format`. If you must use an endpoint without this feature, be prepared to do manual validation or allow a more lenient schema.

---

## Practical Tips

1. **Start Simple**  
   Prompt-based JSON is faster to prototype. Transition to strict schemas later for reliability.

2. **Handle Edge Cases**  
   Sometimes the model returns incomplete or invalid data:
   ```python
   try:
       item = CategoryItem.parse_raw(response.content)
   except ValidationError as e:
       logger.warning("Validation error: %s", e)
       # Decide whether to retry or skip this item
   ```
   - **Why?** Even with a schema, a refusal or partial response can break validation. Catch these errors early and decide whether to log, skip, or retry.

3. **Combine Approaches**  
   You can still use a prompt that asks for JSON, plus enforce a schema:


```python
messages = [
    {"role": "system", "content": "Provide a JSON response with 'title' and 'tags'."},
    {"role": "user", "content": "Extract metadata from the paragraph: '...'"}
]

result = client.chat.completions.parse(
    model="your-model",
    messages=messages,
    response_format=MetaDataSchema
)
```
   - **Why?** Explicitly telling the model to produce JSON reduces errors, and the schema ensures type safety if the model strays.

4. **Use `anyOf` for Flexibility**  
   Sometimes your data can take multiple forms. A simple example:
   ```json
   {
     "anyOf": [
       { "$ref": "#/$defs/AddressFormatA" },
       { "$ref": "#/$defs/AddressFormatB" }
     ]
   }
   ```
   - **Why?** If the LLM might return different valid structures, `anyOf` helps accommodate variations without failing validation every time.

---

## Final Thoughts
Structured outputs are a major advantage in LLM workflows, but not a universal fix. Through trial and error, I’ve found:

- **Prompt-only** methods are quick to set up but risk invalid responses.  
- **Pydantic-backed schemas** provide stricter validation and easier debugging.  
- **Validation** should happen early—never fully trust raw model output in production.

Balancing simplicity with strong formatting constraints ensures your LLM-based system stays robust and dependable.
