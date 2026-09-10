# LLM Architecture & Safety Guardrails — ProAssist AI

ProAssist AI treats Large Language Models as **untrusted, sandboxed reasoning engines**. The LLM is strictly used to parse ambiguous natural language into structured JSON execution plans. It has zero direct execution authority, zero database access, and cannot bypass security or confirmation barriers.

---

## 1. Hybrid Routing: Deterministic First, Cloud Second

```
Incoming User Query
         │
         ▼
  [ TaskRouter ] ──(High-Speed Regex / Keywords)──► Matches Built-in Pattern?
         │                                                      │
        YES                                                     NO
         │                                                      │
         ▼                                                      ▼
Direct Tool Plan                                     Escalate to Cloud LLM
(0ms LLM Latency, 100% Offline)                     (Google Gemini 2.5 Flash)
- "open chrome"                                      - "Find all pdfs in downloads,
- "volume up"                                          rename them, and note it"
- "what is the weather in Delhi"                     - "What is photosynthesis?"
- "show my notes about python"
```

---

## 2. LLM Provider Integration

- **Primary Cloud Provider**: Google Gemini (`core/llm/gemini_provider.py`).
  - Model: `gemini-2.5-flash` (via official REST / SDK client).
  - Temperature: `0.1` (deterministic planning).
  - Request Timeout: `15.0` seconds.
  - Max Tokens: `1024`.
- **Alternative Providers**: `ClaudeProvider` (Anthropic Claude 3.7 Sonnet) and `MockLLMProvider` (offline testing and CI).

---

## 3. Structured Output: `ExecutionPlanSchema`

The LLM is prompted using few-shot structured prompting to output an `ExecutionPlanSchema` JSON object:

```json
{
  "plan_id": "550e8400-e29b-41d4-a716-446655440000",
  "command": "clean up temporary downloads",
  "reasoning": "User requested deleting temporary installer files in downloads",
  "is_conversational": false,
  "conversational_response": null,
  "needs_clarification": false,
  "clarification_prompt": null,
  "steps": [
    {
      "step_id": "step_1",
      "tool_name": "search_files",
      "parameters": {
        "directory": "downloads",
        "pattern": "*.tmp"
      },
      "risk_level": "LOW"
    }
  ]
}
```

---

## 4. Rigorous Plan Validation (`PlanValidator`)

Before any plan is accepted for execution by `ExecutionEngine`, it must pass `PlanValidator.validate()`:

```python
class PlanValidator:
    def validate(self, plan: ExecutionPlan) -> ValidationResult:
        # 1. Step count barrier
        if len(plan.steps) > 10:
            return ValidationResult(is_valid=False, error="Plan exceeds maximum 10 steps")

        for step in plan.steps:
            # 2. Strict Tool Allowlist check
            if step.tool_name not in ToolRegistry.registered_tools():
                return ValidationResult(is_valid=False, error=f"Prohibited tool: {step.tool_name}")

            # 3. Explicit Prohibited Operation Check
            if step.tool_name in FORBIDDEN_OPERATIONS:
                return ValidationResult(is_valid=False, error="Security violation")

            # 4. Mandatory Risk Auto-Elevation
            registered_risk = PermissionManager.get_risk(step.tool_name)
            if registered_risk > step.risk_level:
                step.risk_level = registered_risk # Auto-elevate to prevent downgrade attacks
```

---

## 5. Explicitly Prohibited Operations

The following tools and actions are **strictly forbidden** and actively rejected by the schema and registry:
- `run_python`
- `execute_shell`
- `eval` / `exec`
- `powershell` / `cmd`
- Arbitrary HTTP fetches or web crawling
- Raw SQL execution against `proassist.db`
- Reading Windows Credential Vault directly

---

## 6. Truthful Cloud Indicator on HUD

Whenever a request is handled by the Cloud LLM, the PySide6 UI immediately displays the `[CLOUD]` badge in the Top Telemetry Bar. Local deterministic commands display `[LOCAL]`. The system never lies about where processing occurred.

### Email Prompt Isolation
External email bodies retrieved for LLM context are sanitized and isolated inside `<untrusted_email_content>` containment tags with system instructions to treat the content strictly as data, never as executable instructions.
