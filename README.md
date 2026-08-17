# Implementing Prompt Chaining with Bedrock Agent Flows and API

Amazon Bedrock Flows is the AWS service for building LLM applications from connected nodes. A flow starts at a flow input, passes data through prompt nodes (and optionally condition, agent, code, or Lambda nodes), and ends at one or more flow output nodes. The visual editor and the JSON definition are equivalent representations of the same flow.

`Linear chaining` splits a complex task into a sequence of narrower prompts. Each step has a single responsibility, takes the previous step's output as a template variable, and emits its own output for the next step. Splitting a task this way makes each step independently testable and improves consistency over a single long prompt.

[Linear prompt chaining example](linear-prompt-chaining.md)

`Conditional routing` introduces branching. A classifier prompt produces a single value, a condition node compares that value to predefined options, and execution flows down the matching branch. Two patterns matter for reliability:

- The classifier must produce a value the condition node can match exactly. "Output exactly one word" in the prompt, or structured output, prevents punctuation and explanations from breaking the comparison.

- Specialist prompts downstream of the condition node need their data wired directly from the flow input, not from the classifier. The classifier chooses the path; it does not pass the user's content forward.

The flow trace records the input and output of every node that ran, which is the primary debugging tool when a flow returns an unexpected result.

[Conditional prompt chaining example](conditional-prompt-chaining.md)

# Text Helper Flow

A productivity startup at a fictional content-marketing company wants a single text helper that detects what a user wants to do with a chunk of text — summarize it, rewrite it for clarity, or something else — and returns the appropriate response. Every request goes through the same entry point, but the work that follows depends on the user's intent.

Build a Bedrock Flow that classifies the intent of an input message and routes it to the appropriate specialist prompt.

[Exercise: Text helper](text-helper.md)
