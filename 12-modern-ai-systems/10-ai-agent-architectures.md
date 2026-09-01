# AI Agent Architectures: Tool Use & Autonomous Reasoning

AI Agents extend pure LLMs from static text generators into autonomous systems capable of reasoning, planning, executing external APIs, and remembering past interactions.

---

## 1. The ReAct (Reason + Act) Loop

```mermaid
flowchart TD
    UserGoal["User Goal: 'Book a flight to NYC under $300'"] --> Agent[AI Agent Controller]

    subgraph ReActLoop["ReAct Reasoning Loop"]
        Thought["1. Thought: 'I need to check available flights on Friday'"]
        Action["2. Action: Call API search_flights('NYC', '2026-09-05')"]
        Observation["3. Observation: '[Flight DL123: $280, AA456: $350]'"]
        Thought2["4. Thought: 'DL123 is under $300, I should book it'"]
        Action2["5. Action: Call API book_flight('DL123')"]

        Thought --> Action --> Observation --> Thought2 --> Action2
    end
    Agent --> ReActLoop
    Action2 --> FinalResponse["Confirmed: Booked Delta DL123 for $280! ✅"]
```

---

## 2. Core Agent Architectural Primitives
- **Planning & Decomposition**: Breaking large complex goals into ordered sub-tasks.
- **Tool Execution Sandbox**: Executing arbitrary API calls or Python code in ephemeral, isolated sandboxes.
- **Memory Systems**:
  - **Short-Term Working Memory**: In-context conversation history and intermediate tool outputs.
  - **Long-Term Episodic Memory**: Vector database indexing past user sessions and learned preferences.
- **Self-Correction & Reflection**: Validating tool execution outputs; automatically retrying with corrected parameters upon API error.
