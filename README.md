# Prompt Optimisation and Token Efficiency — Challenge #2

## Objective
This project audits and optimizes the **Code Buddy** AI system prompt to eliminate token waste, reduce input token overhead, implement precise token usage logging, and lower monthly API operating costs while fully preserving all functional review instructions and quality.

---

## Completed Work

- **System Prompt Audit & Measurement:** Measured original system prompt token count (392 tokens) using an actual tokenizer.
- **Identified Waste Sources:** Identified exactly 3 sources of token waste in the original prompt (Instruction Duplication, Over-Verbose Role Description, Redundant Formatting Directives).
- **Prompt Optimization:** Rewrote system prompt from 392 tokens down to 57 tokens — achieving an **85.46% token reduction** (exceeding the required >= 40% reduction target).
- **Preserved All Instructions:** Mapped every original prompt instruction to ensure complete behavior preservation.
- **Token & Cost Logging:** Added real-time token usage logging in `src/callAI.js` to log prompt tokens, completion tokens, total tokens, and estimated call cost in USD.
- **API Testing:** Tested the Express server `/review` endpoint with three actual test code snippets and verified `[TOKEN LOG]` console outputs.
- **Cost Analysis & Audit Documentation:** Fully documented all metrics, waste sources, instruction mappings, and cost comparisons in `token-audit.md`.

---

## Key Metrics Summary

| Metric | Original Prompt | Optimized Prompt | Savings / Reduction |
| :--- | :---: | :---: | :---: |
| System Prompt Size | 392 tokens | 57 tokens | **-85.46%** |
| Avg Total Tokens / Call | 660.34 tokens | 364.00 tokens | **-44.88%** |
| Monthly Operating Cost (90k calls) | $300.91 | $266.85 | **-$34.06 / month (-11.31%)** |

---

## Key Files

- `prompts/system-prompt.txt` — Optimized system prompt (57 tokens)
- `src/callAI.js` — OpenRouter / Gemini API client integration with real-time `[TOKEN LOG]` logging
- `src/index.js` — Express API server hosting the `POST /review` route
- `token-audit.md` — Detailed audit report with waste analysis, token counts, instruction preservation mapping, and financial projections

---

## Git Branch
- **Active Branch:** `token-fix`

---

## Evidence & Verification

- **Tokenizer Measurement:** Measured with actual tokenizer (392 baseline vs 57 optimized).
- **Real-Time Token Logs:** Server output verified during testing:
  ```text
  [TOKEN LOG] prompt_tokens: 83 | completion_tokens: 207 | total: 290 | est_cost_usd: $0.0023
  [TOKEN LOG] prompt_tokens: 97 | completion_tokens: 281 | total: 378 | est_cost_usd: $0.0031
  [TOKEN LOG] prompt_tokens: 90 | completion_tokens: 334 | total: 424 | est_cost_usd: $0.0036
  ```
- **Endpoint Verification:** Three POST requests sent to `http://localhost:3000/review` and received valid 3-part code review feedback structures.

---

## Setup & Running Locally

1. Clone the repository:
   ```bash
   git clone https://github.com/venkataajaykumar19/Challenge-2-Token-Optimisation.git
   cd Challenge-2-Token-Optimisation
   ```
2. Install dependencies:
   ```bash
   npm install
   ```
3. Configure environment variables:
   ```bash
   cp .env.example .env
   # Add your OPENROUTER_API_KEY into .env
   ```
4. Start the server:
   ```bash
   npm start
   ```
5. Test code review endpoint:
   ```bash
   curl -X POST http://localhost:3000/review \
     -H "Content-Type: application/json" \
     -d '{"code": "function add(a, b) { return a + b; }"}'
   ```
