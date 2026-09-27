---
description: "Write tool and function descriptions that models use correctly: naming, parameter docs, when-to-use guidance, error messages, and testing tool selection."
---

# 06 · Prompting for Tool & Function Calling

Many models can call **tools** (also called functions): you describe available
operations — search, look up an order, run a calculation — and the model responds with a
structured request to call one, which your code executes and returns. How to wire this up
in each provider's API is covered in the
[LLM Dev Mastery Path](https://sigilipelli.github.io/llm-dev-mastery-path/). This lesson
is about the part that's pure prompt engineering: **tool descriptions are prompts**, and
their quality decides whether the model picks the right tool with the right arguments.

## A tool definition is documentation for a reader with no context

Most APIs accept tool definitions as a name, a description, and a JSON Schema for
parameters. A weak definition:

```json
{
  "name": "search",
  "description": "Searches.",
  "parameters": {"type": "object", "properties": {"q": {"type": "string"}}}
}
```

A strong one:

```json
{
  "name": "search_orders",
  "description": "Find a customer's orders in the order database. Use this when the user asks about the status, contents, or delivery of a specific order. Do not use it for product questions (use search_catalog). Returns up to 10 orders, newest first.",
  "parameters": {
    "type": "object",
    "properties": {
      "customer_email": {"type": "string", "description": "Email address on the account, exactly as the user provided it."},
      "order_id": {"type": "string", "description": "Optional. Order ID in the form ORD-123456. Omit if the user didn't give one; don't guess."},
      "since_date": {"type": "string", "description": "Optional. ISO date YYYY-MM-DD; only orders on or after this date."}
    },
    "required": ["customer_email"]
  }
}
```

What changed: a specific name; what the tool does and **when to use it (and when not
to)**; what it returns; parameter descriptions with formats; and explicit "don't guess"
guidance for optional fields.

## Guidance in the system prompt

Tool descriptions say what each tool does; the system prompt says how to use them
together:

```text
Tool use:
- If you need order information, call search_orders before answering; never state an
  order status you haven't retrieved.
- If a tool returns an error, explain the problem to the user in plain words; don't
  retry more than once with the same arguments.
- Ask the user for missing required information (such as their email) instead of
  inventing it.
- Actions that change data (cancel_order) require the user's explicit confirmation in
  their latest message.
```

## Design tools for the model

- **Few, well-separated tools** beat many overlapping ones. If two tools could plausibly
  handle a request, the model will sometimes pick the wrong one.
- **Clear return values:** return concise, structured results with field names that
  explain themselves. Don't return 5,000 lines of raw data when 10 fields will do.
- **Helpful error messages:** `"order_id must look like ORD-123456; got '12345'"` lets the
  model fix its call. `"400 Bad Request"` doesn't.
- **Safe defaults:** make destructive operations separate tools that require
  confirmation, and enforce permissions in code — not just in the description.
- **Treat tool results as untrusted content** when they contain third-party text
  (lesson 04).

## Testing tool selection

Tool use is testable like classification: for each test message, what tool (if any)
should be called, with what arguments?

```python
tests = [
    {"msg": "Where's my order ORD-482913? email ana@example.com",
     "tool": "search_orders", "args": {"customer_email": "ana@example.com", "order_id": "ORD-482913"}},
    {"msg": "Do you sell waterproof hiking boots?", "tool": "search_catalog", "args": {}},
    {"msg": "Thanks, that's all!", "tool": None, "args": {}},
]

def fake_model_tool_choice(msg: str) -> tuple[str | None, dict]:
    """Stand-in for the model's tool call decision."""
    if "ORD-" in msg:
        return "search_orders", {"customer_email": "ana@example.com", "order_id": "ORD-482913"}
    if "sell" in msg:
        return "search_orders", {"customer_email": ""}   # wrong tool on purpose
    return None, {}

for t in tests:
    tool, args = fake_model_tool_choice(t["msg"])
    tool_ok = tool == t["tool"]
    args_ok = all(args.get(k) == v for k, v in t["args"].items())
    print(f"{'PASS' if tool_ok and args_ok else 'FAIL'}  expected={t['tool']} got={tool}")
```

Output:

```text
PASS  expected=search_orders got=search_orders
FAIL  expected=search_catalog got=search_orders
PASS  expected=None got=None
```

With a real model, run each test several times; tool selection near boundaries (product
vs order question) is where instability shows up.

## Worked example: fixing an over-eager tool

Symptom: the assistant calls `search_orders` for "What's your returns policy?", passing an
empty email. Fixes, one at a time:

1. Tool description: add "Do not use for general policy questions."
2. Parameter description: "customer_email is required; if unknown, ask the user."
3. Add a `get_policy` tool (or policy text in the system prompt) so there's a correct
   option to choose.
4. Add three general-policy questions to the test set.

Often step 3 matters most: models call the wrong tool when no right tool exists.

## How It Actually Works

When you pass tool definitions, the provider renders them into the model's context
(typically in the system area, in a format the model was trained on). The model then
generates either normal text or a structured tool-call block, which the API parses and
returns to you. Choosing a tool is a prediction like any other: the model compares the
user's request with tool names and descriptions and continues with the most likely
action. So everything you know about clear instructions, definitions, and examples
applies to tool descriptions.

Argument values are generated text too — they can be hallucinated, mis-formatted, or
copied from untrusted content. That's why arguments must be validated in code, and why
"don't guess; ask the user" guidance measurably matters.

## Common mistakes

- **One-line descriptions** with no when-to-use guidance.
- **Overlapping tools** with vague boundaries.
- **Opaque error messages** the model can't act on.
- **Trusting argument values** without validation.
- **Permission rules only in prose.**

## Exercise

1. Design 3 tools for an assistant you'd find useful (e.g. calendar lookup, note search,
   weather). Write full definitions with when/when-not guidance.
2. Write 12 test messages: 3 per tool, and 3 needing no tool. Include two ambiguous ones.
3. Run them against a real model with tool support (or reason through them with the
   mock pattern above) and record tool and argument accuracy.
4. Fix the weakest description and re-run.
