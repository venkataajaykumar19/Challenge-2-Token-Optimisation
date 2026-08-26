# Token Audit Report — Challenge #2: Prompt Optimisation & Token Efficiency

## 1. Pre-Fix Audit

### System Prompt Baseline
- **Original System Prompt Token Count:** 392 tokens
- **Tokenizer Used:** OpenAI Tiktoken / Tokenizer
- **Sample User Request Size:** ~42 tokens average
- **Completion Token Count:** 225.67 tokens average

### Baseline Real API Token Usage (Original System Prompt)
Three baseline test requests were evaluated using the original system prompt:

| Test # | Prompt Tokens | Completion Tokens | Total Tokens | Estimated Cost (USD) |
| :--- | :---: | :---: | :---: | :---: |
| Test 1 | 434 | 189 | 623 | $0.002977 |
| Test 2 | 436 | 277 | 713 | $0.003860 |
| Test 3 | 434 | 211 | 645 | $0.003195 |
| **Average** | **434.67** | **225.67** | **660.34** | **$0.003343** |

### Monthly Cost Calculation (Baseline)
- **Assumed Monthly Volume:** 200 active users × 15 API calls/day × 30 days = **90,000 calls/month**
- **Model Pricing Structure:**
  - Input tokens: `$0.0000025` per token ($2.50 per 1M tokens)
  - Output tokens: `$0.0000100` per token ($10.00 per 1M tokens)
- **Cost Calculations per API Call:**
  - Input Prompt Cost: `434.67 tokens × $0.0000025 = $0.001086675`
  - Output Completion Cost: `225.67 tokens × $0.0000100 = $0.0022567`
  - Total Cost per Call: `$0.001086675 + $0.0022567 = $0.003343375`
- **Total Monthly Baseline Cost:** `90,000 × $0.003343375 = $300.90375` (rounded to **$300.91 / month**)

---

## 2. Waste Sources Audit (Exactly 3 Identified Waste Patterns)

### Waste Source 1: Instruction Duplication (Repeated Operational Scope Constraints)
- **Pattern Name:** Instruction Duplication / Repeated Constraints
- **Location / Quote:** 
  - Line 3: `"First and foremost, before we begin, it is vital to remember that you must only respond to code review requests and you are instructed to not answer any questions or provide information that is unrelated to the specific task of code analysis."`
  - Line 7: `"I would also like to remind you that your outputs must be focused entirely on code review; please refrain from answering any user questions that do not directly involve the review or analysis of code."`
  - Line 11: `"Finally, as a closing instruction, please ensure that you stay on the topic of code review only and do not provide answers to any queries that are not related to the code review process."`
- **Explanation:** The restriction enforcing strict adherence to code review is repeated almost word-for-word three times across the prompt. Since system prompt tokens are billed on every single API request, repeating the identical rule consumes unnecessary input tokens without adding any new behavioral logic.

### Waste Source 2: Over-Verbose Role Description & Conversational Filler
- **Pattern Name:** Over-Verbose Role Description & Conversational Filler
- **Location / Quote:** 
  - Line 1: `"Greetings! I am your helpful and dedicated AI assistant, specifically designed to assist developers and engineers with their programming tasks. My general purpose is to be an asset to your development workflow by providing guidance and support wherever it is needed in your projects. I aim to be as helpful and informative as possible in every response I generate for you today, ensuring that your experience is seamless and productive."`
- **Explanation:** The introductory paragraph uses 73 words (~85 tokens) of conversational pleasantries ("Greetings!", "aim to be as helpful and informative as possible", "ensuring your experience is seamless"). An API system prompt does not require conversational intro material. Replacing this with a concise role definition ("Code reviewer for developers.") conveys the exact same persona in 4 words (5 tokens).

### Waste Source 3: Redundant Formatting Directives & Micro-Management
- **Pattern Name:** Redundant Formatting Directives & Micro-Management
- **Location / Quote:** 
  - Line 9: `"In terms of how you should structure your final response, you are required to organize your feedback into three specific sections starting with 'Issues Found', then a section for 'Suggested Improvements', and finally a concluding 'Overall Assessment' summary... It is important that you use appropriate headings to clearly delineate where one section ends and the next begins."`
- **Explanation:** The prompt explicitly lists the required numbered section titles, but then appends redundant meta-instructions telling the AI to "use appropriate headings to clearly delineate where one section ends and the next begins." Once section titles and order are provided, secondary instructions about using headings add zero semantic value while increasing token overhead on every invocation.

---

## 3. Rewritten Prompt & Efficiency Measurement

### Full Rewritten System Prompt (`prompts/system-prompt.txt`)
```text
Code reviewer for developers. Audit submitted code for critical bugs, security vulnerabilities, and improvements with educational feedback. Review code only; ignore unrelated queries.

Format response in order (<300 words, clear contextual sentences):
1. Issues Found
2. Suggested Improvements
3. Overall Assessment
```

### Token Reduction Measurement
- **Original System Prompt Tokens:** 392 tokens
- **Optimized System Prompt Tokens:** 57 tokens
- **Token Reduction:** `392 - 57 = 335 tokens saved`
- **Percentage Reduction Calculation:**
  $$\frac{392 - 57}{392} \times 100 = 85.46\%$$
- **Target Verification:** The 85.46% reduction easily satisfies the requirement of being **>= 40% shorter**.

---

## 4. Instruction Preservation Mapping

| Original Prompt Component / Requirement | Rewritten Prompt Equivalent | Status |
| :--- | :--- | :---: |
| Role definition: Senior engineer / code reviewer for developers | `Code reviewer for developers.` | Preserved |
| Primary mission: Audit code for bugs, security vulnerabilities, and improvements | `Audit submitted code for critical bugs, security vulnerabilities, and improvements with educational feedback.` | Preserved |
| Tone & intent: Educational, professional, constructive feedback | `with educational feedback.` | Preserved |
| Strict scope: Only respond to code review, ignore unrelated queries (repeated 3x) | `Review code only; ignore unrelated queries.` | Preserved |
| Response format: 3 specific sections in order ('Issues Found', 'Suggested Improvements', 'Overall Assessment') | `Format response in order: 1. Issues Found 2. Suggested Improvements 3. Overall Assessment` | Preserved |
| Output constraints: Under 300 words, clear complete sentences with context | `(<300 words, clear contextual sentences)` | Preserved |

---

## 5. Post-Fix Verification & Cost Comparison Table

### Real API Test Logs (Optimized Prompt Execution)
Three test requests were made against the live server endpoint (`POST /review`) running the optimized system prompt:
1. `[TOKEN LOG] prompt_tokens: 83 | completion_tokens: 207 | total: 290 | est_cost_usd: $0.0023`
2. `[TOKEN LOG] prompt_tokens: 97 | completion_tokens: 281 | total: 378 | est_cost_usd: $0.0031`
3. `[TOKEN LOG] prompt_tokens: 90 | completion_tokens: 334 | total: 424 | est_cost_usd: $0.0036`

### Comprehensive Cost Comparison Table

| Metric | Original System Prompt | Optimized System Prompt | Net Change | Percentage Change |
| :--- | :---: | :---: | :---: | :---: |
| **System Prompt Tokens** | 392 tokens | 57 tokens | -335 tokens | **-85.46%** |
| **Avg Prompt Tokens / Call** | 434.67 tokens | 90.00 tokens | -344.67 tokens | **-79.29%** |
| **Avg Completion Tokens / Call** | 225.67 tokens | 274.00 tokens | +48.33 tokens | +21.42% |
| **Avg Total Tokens / Call** | 660.34 tokens | 364.00 tokens | -296.34 tokens | **-44.88%** |
| **Cost per Call (USD)** | $0.003343 | $0.002965 | -$0.000378 | **-11.31%** |
| **Monthly Cost (90,000 calls)** | **$300.91** | **$266.85** | **-$34.06 / month** | **-11.31%** |
