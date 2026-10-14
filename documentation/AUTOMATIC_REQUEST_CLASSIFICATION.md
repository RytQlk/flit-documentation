# 05 — Automatic Request Classification

Flit's classification engine determines the optimal model tier for prompts sent to the `auto` (Automatic) route. It optimizes inference costs while maintaining high response quality.

---

## The Three Classification Modes

Administrators can configure the gateway in one of three active modes:

```
                            Incoming Request (model = "auto")
                                           |
                                           v
                             +---------------------------+
                             |   Active Mode Selected?   |
                             +-------------+-------------+
                                           |
                +--------------------------+--------------------------+
                |                          |                          |
                v                          v                          v
       +-----------------+        +------------------+       +------------------+
       |   REGEX MODE    |        |   HYBRID MODE    |       |  CLASSIFIER LLM  |
       +--------+--------+        |  (Recommended)   |       +--------+---------+
                |                 +--------+---------+                |
                v                          |                          v
         Evaluate Rules                    v                   Structured LLM
        (Microsecond I/O)           Evaluate Regex               Evaluation
                |                          |                          |
     +----------+----------+        Match? | No Match                 v
     |                     |         +-----+-----+             Assigned Tier:
     v                     v         |           v            LOW / MED / HIGH
  Matched?             No Match      v       Fallback to
LOW/MED/HIGH          Default Tier  Assigned Classifier LLM
                                    Tier         |
                                                 v
                                           Assigned Tier
```

### 1. Regex Mode (Zero-IO, Microsecond Latency)
- **Mechanism**: Evaluates the user prompt against compiled regular expression patterns.
- **Performance**: Executes completely in-process within the gateway proxy worker in under 0.1ms with zero external API calls or network latency.
- **Evaluation Order**: Rules are evaluated strictly by priority order. The first rule that matches assigns the target tier (`LOW`, `MEDIUM`, or `HIGH`). If no rules match, the gateway applies the fallback tier default.

### 2. Classifier LLM Mode
- **Mechanism**: Passes the prompt to an internal, lightweight language model (e.g., `gpt-4o-mini` or `claude-3-5-haiku`).
- **Isolation**: A dedicated internal credential isolates the classifier LLM to prevent recursive loops, external billing attribution, or prompt tampering.
- **Accuracy**: Delivers high semantic understanding for ambiguous prompts, complex instructions, or nuanced enterprise intent.

### 3. Hybrid Mode (Recommended)
- **Mechanism**: Two-phase evaluation pipeline combining microsecond Regex efficiency with LLM intelligence.
- **Phase 1**: The prompt is checked against configured Regex patterns. If a pattern matches, the tier is immediately returned.
- **Phase 2**: If no rule matches (`NO_REGEX_RULE_MATCH`), the prompt falls back to the Classifier LLM.
- **Cost Savings**: Handles 60–80% of routine corporate volume in-process at zero inference cost, reserving LLM evaluations only for novel prompts.

---

## Production Regex Rule Patterns

Below are production-ready regular expression rules implemented in Flit:

### 1. Routine Greetings & Pleasantries (`LOW` Tier)
Redirects casual conversational pleasantries to the lowest-cost model pool:
```regex
(?i)^\s*(hi|hello|hey|good\s+(morning|afternoon|evening)|howdy|thanks|thank\s+you|bye|goodbye)\b.*$
```

### 2. Entity Extraction & Data Transformation (`LOW` Tier)
Assigns deterministic extraction or parsing tasks to high-throughput models:
```regex
(?i)\b(extract\s+(the\s+)?(names?|dates?|numbers?|email|entities)|convert\s+to\s+json|format\s+as\s+csv|output\s+only\s+json)\b
```

### 3. Long-Form Document Summarization (`MEDIUM` Tier)
Routes standard synthesis, outlining, and summary requests to balanced models:
```regex
(?i)^\s*(summarize|tl;?dr|give\s+me\s+the\s+key\s+takeaways|bullet\s+points\s+of|outline\s+the\s+following)\b
```

### 4. Code Synthesis, Refactoring & Debugging (`HIGH` Tier)
Directs complex software engineering, architecture, and debugging to advanced models:
```regex
(?i)\b(refactor|optimize\s+this|write\s+a\s+(python|typescript|javascript|rust|go|c\+\+|sql|bash)\s+script|debug\s+this|fix\s+the\s+bug|algorithm|unit\s+test|dockerfile)\b
```

### 5. Mathematical Proofs & Formal Reasoning (`HIGH` Tier)
Ensures analytical calculations, theorems, and multi-step math tasks use deep reasoning models:
```regex
(?i)\b(prove\s+that|mathematical\s+proof|derivative\s+of|integral\s+of|eigenvalue|differential\s+equation|linear\s+algebra|matrix\s+multiplication)\b
```

---

## Configuring Classifier Rules in Admin Console

1. Navigate to **Routing > Classifiers** in the admin sidebar.
2. Under **Regex classifier**, click **Add Rule**.
3. Provide:
   - **Rule Name & Description**: e.g., "Code Generation Filter".
   - **Pattern**: Valid ECMAScript / Python regular expression.
   - **Target Tier**: `low`, `medium`, or `high`.
   - **Priority Order**: Lower numeric order values evaluate first.
4. Use the **Test Pattern** interactive simulator to paste sample user prompts and verify match behavior.
5. Select the **Active Mode** (Regex, Classifier LLM, or Hybrid) and click **Save & Publish**. Rules are compiled and activated with zero gateway downtime.

---

## Next Steps

- Review [VIRTUAL_KEYS_USERS_AND_TEAMS.md](VIRTUAL_KEYS_USERS_AND_TEAMS.md) to manage access policies.
- Explore [GUARDRAILS_AND_DATA_PROTECTION.md](GUARDRAILS_AND_DATA_PROTECTION.md) for DLP filters.
