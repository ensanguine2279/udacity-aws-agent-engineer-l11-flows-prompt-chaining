# Prompt Chaining with Bedrock Flows

## Bedrock Flows fundamentals

A flow in Amazon Bedrock is an LLM application built from connected nodes. The flow visual editor exposes the most common node types — prompt, condition, agent, code, knowledge base, S3, iterator, collector, do-while loop, and Lambda — and a chat panel for testing the flow as it is built. The same flow can be exported as JSON for version control and code review.

The walkthrough builds a two-step blog post generator: a first prompt produces a structured outline from the user's topic, and a second prompt expands that outline into a full draft.

## Flow: `blog-post-chain`

```
Flow Input (idea_brief)  →  OutlineGenerator  →  DraftWriter  →  Flow Output
```

The blog post flow has three logical pieces:

| Node                            | Purpose                                     | Receives                                  | Builds                   |
| ------------------------------- | ------------------------------------------- | ----------------------------------------- | ------------------------ |
| Outline generator (prompt node) | Produces a structured outline from a topic. | The user's topic from the flow input.     | A multi-section outline. |
| Draft writer (prompt node)      | Expands the outline into a full blog post.  | The outline emitted by the previous node. | A complete draft.        |
| Flow output                     | Returns the draft to the caller.            | The draft writer's output.                | —                        |

The outline generator and the draft writer use different roles, different output formats, and different prompts. Each node's prompt has a single template variable — `{{topic}}` for the outline, `{{outline}}` for the draft — and each variable is bound to an upstream node's output through a connection in the editor.

## Why split the task

A single prompt might not work for complex tasks. Splitting the task into multiple steps gives each step a narrower responsibility and produces more consistent output. A multi-step flow also makes both steps observable: the trace records the input and output of every node, so we can diagnose where a problem creeps in.

## Reading the trace

After running a flow, the `Show trace` view lists every node that executed. For each node, the trace shows the input value the node received and the output it produced. When a flow returns an unexpected result, the trace usually pinpoints the failing node.

Traces for both test inputs can be found in `flow-trace-input-x.json`

---

## Node 1: OutlineGenerator

**System prompt:**

```
You are a content strategist for a B2B software blog. When given a brief idea, produce a structured blog post outline.

Your output must follow this exact format:
- Headline options: three alternative titles
- Target audience: one sentence
- Sections: a numbered list of section names with a single-sentence description of each
- Key points: three to five bullet points that must appear somewhere in the post

Here is the idea brief:

<brief>
{{idea_brief}}
</brief>

Generate the blog post outline now.
```

**Input variable:** `idea_brief` (String)

**Expected output:** A structured outline with headline options, target audience, numbered sections, and key bullet points — no prose.

---

## Node 2: DraftWriter

**System prompt:**

```
You are a professional copywriter for a B2B software blog. When given a structured outline, write a complete, publish-ready blog post.

Rules:
- Follow the section structure from the outline exactly
- Use the key points from the outline — do not drop or add any
- Write in a clear, professional tone aimed at the target audience
- Keep total length between 400 and 600 words
- Do not add a conclusion section that is not in the outline

Here is the outline to expand into a full blog post:

<outline>
{{outline}}
</outline>

Write the complete blog post now.
```

**Input variable:** `outline` (String) — wired from OutlineGenerator's output.

**Expected output:** A 400–600 word blog post that follows the outline's section structure and includes every key point.

---

## Test Inputs

**Input 1:**

```
We want to write about why companies should move from shared passwords to SSO for internal tools. Target audience is IT managers at mid-size companies. Tone should be practical and slightly urgent without being alarmist.
```

**Input 2:**

```
We want to write about the hidden cost of manual onboarding for SaaS companies — specifically the time spent by engineers setting up accounts and permissions manually. Audience is heads of engineering at growth-stage startups.
```
