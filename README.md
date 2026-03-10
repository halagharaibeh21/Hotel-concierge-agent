# Hotel Concierge AI Agent

A conversational AI agent for The Amman Hotel, built with n8n. The agent answers guest questions about the hotel, checks restaurant availability through a dedicated tool, and escalates requests it cannot handle to human support staff.

---

## Architecture
```
[Guest Chat Interface]
         |
         | POST { chatInput: "..." }
         |
[Webhook Trigger]
         |
[Input Guardrail]
         |
[Hotel Concierge Agent] <---> [Tool: Check Restaurant Availability]
         |
[Escalation Router]
    |           |
  true        false
    |           |
[Format      [Send Guest Reply]
 Escalation
 Ticket]
    |
[Escalation Reply]
```

### Node Reference

| Node | Type | Role |
|---|---|---|
| Guest Chat Input | Webhook | Receives POST requests from the chat interface |
| Input Guardrail | Code | Rejects empty or oversized inputs before they reach the agent |
| Hotel Concierge Agent | AI Agent | The reasoning core — decides whether to answer, call a tool, or escalate |
| LLM: OpenRouter | Chat Model | Language model powering the agent |
| Conversation Memory | Simple Memory | Maintains context across turns within a session |
| Check Restaurant Availability | Tool (Code) | Simulates a restaurant booking API with date-aware availability logic |
| Escalation Router | IF | Checks whether the agent output contains the ESCALATE: keyword |
| Format Escalation Ticket | Code | Structures escalation data for downstream handling |
| Send Guest Reply | Set | Returns the agent response to the chat interface |
| Escalation Reply | Set | Returns a handoff message to the guest when escalated |

---

## Reasoning Approach

The agent follows the ReAct pattern (Reason, Act, Observe):

1. **Reason** — the model reads the guest message alongside a system prompt containing hotel information, then determines whether to answer directly, invoke a tool, or escalate.
2. **Act** — if the query involves restaurant availability, the agent calls the `check_restaurant_availability` tool, passing the date, time, and party size extracted from the conversation.
3. **Observe** — the agent incorporates the tool response into its reply before sending it to the guest.
4. **Escalate** — if the query falls outside the agent's knowledge or authority, the model outputs `ESCALATE: [reason]`. The IF node detects this string and routes the message to the escalation branch rather than the standard reply path.

---

## Design Decisions

**Structured escalation via keyword detection**
Rather than using a secondary classifier to determine when to escalate, the system prompt instructs the agent to output a specific string (`ESCALATE:`) when it cannot confidently handle a request. This approach is deterministic and requires no additional model call. The IF node performs a simple string-contains check to route accordingly.

**Input validation as a separate node**
The Input Guardrail node runs before the AI Agent node, ensuring the model is never invoked on empty or malformed input. Separating this concern into its own node makes the validation logic easy to inspect, modify, and extend independently of the agent.

**Tool implemented as a Code node**
The restaurant availability tool is implemented in JavaScript within n8n rather than as an external HTTP call. It applies date-aware logic: weekend evenings during peak hours return unavailable, while off-peak slots confirm a booking. This allows the full workflow to run without external API dependencies while demonstrating the tool-calling mechanism accurately.

**Conversation Memory**
The Simple Memory node preserves message history within a session, allowing the agent to handle follow-up questions without requiring the guest to repeat context.

---

## Setup

1. Import `workflow.json` into your n8n instance
2. Add your OpenRouter API credentials to the LLM node
3. Activate the workflow and copy the Webhook URL
4. Open `amman-hotel-concierge.html` in a browser
5. Paste the Webhook URL into the configuration field at the top of the interface

---

## Test Cases

| Input | Expected Behaviour |
|---|---|
| "What time does the pool close?" | Agent answers from the hotel knowledge base |
| "Is there a table for 2 on Saturday at 8pm?" | Agent calls the restaurant tool and returns availability |
| "I have a complaint about my bill" | Agent triggers escalation branch, guest receives handoff message |
| Empty message | Guardrail node blocks the request before it reaches the agent |

---

## Repository Contents
```
├── workflow.json                 # n8n workflow export
├── amman-hotel-concierge.html    # Guest-facing chat interface
└── README.md
```
