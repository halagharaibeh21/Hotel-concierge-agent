# Hotel Concierge AI Agent

A conversational AI agent for The Amman Hotel, built with n8n. The agent answers guest questions, checks restaurant availability through a tool, and escalates requests it cannot handle to human support staff.

---

## Architecture
```
[Guest Chat Input]
         |
[Input Guardrail]
         |
[Hotel Concierge Agent] <---> [Tool: Check Restaurant Availability]
         |
[Escalation Router]
      |         |
    true      false
      |         |
[Format      [Send Guest Reply]
 Escalation
 Ticket]
      |
[Escalation Reply]
```

| Node | Role |
|---|---|
| Guest Chat Input | Built-in n8n chat trigger — receives messages from the chat interface |
| Input Guardrail | Rejects empty or oversized inputs before reaching the agent |
| Hotel Concierge Agent | The reasoning core — answers, calls tools, or escalates |
| LLM: OpenRouter | Language model powering the agent |
| Conversation Memory | Maintains context across turns within a session |
| Check Restaurant Availability | Simulates a restaurant booking API with date-aware logic |
| Escalation Router | Routes the conversation based on whether escalation was flagged |
| Format Escalation Ticket | Structures escalation data for support staff |
| Send Guest Reply | Returns the agent response to the guest |
| Escalation Reply | Returns a handoff message when the request is escalated |

---

## Reasoning Approach

The agent follows the ReAct pattern — Reason, Act, Observe. It reads the guest message and decides whether to answer directly from its hotel knowledge base, call the restaurant availability tool, or escalate. If a tool call is needed, it extracts the relevant parameters from the conversation, calls the tool, and incorporates the result into its reply. If the request falls outside its knowledge or authority, it outputs `ESCALATE: [reason]`, which the IF node catches to route the message to the escalation branch.

---

## Design Decisions

**Structured escalation via keyword detection.** The system prompt instructs the agent to output a specific string (`ESCALATE:`) when it cannot handle a request. This is deterministic and requires no additional model call — the IF node performs a simple string-contains check.

**Input validation as a separate node.** The guardrail runs before the agent, so the model is never invoked on empty or malformed input. Keeping it as its own node makes it easy to extend without touching the agent logic.

**Tool implemented as a Code node.** The restaurant tool applies date-aware logic in JavaScript — weekend peak hours return unavailable, off-peak slots confirm availability. This allows the workflow to demonstrate tool-calling without requiring an external API.

**Conversation Memory.** The Simple Memory node preserves message history within a session so the agent can handle follow-up questions without the guest repeating context.

---

## Setup

1. Import `workflow.json` into your n8n instance
2. Add your OpenRouter API credentials to the LLM node
3. Activate the workflow
4. Click the chat icon in the bottom right of the n8n canvas to open the built-in chat interface

---

## Repository Contents
```
├── workflow.json       # n8n workflow export
└── README.md
```
