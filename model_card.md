# DocuBot Model Card

This model card is a short reflection on your DocuBot system. Fill it out after you have implemented retrieval and experimented with all three modes:

1. Naive LLM over full docs  
2. Retrieval only  
3. RAG (retrieval plus LLM)

Use clear, honest descriptions. It is fine if your system is imperfect.

---

## 1. System Overview

**What is DocuBot trying to do?**  
Describe the overall goal in 2 to 3 sentences.

>  DocuBot aims to help users answer questions about a set of documentation files by providing relevant information in a concise and understandable format. It uses retrieval and LLM generation to balance accuracy with readability.


**What inputs does DocuBot take?**  
For example: user question, docs in folder, environment variables.

>      User question
       Documentation files in the project folder (SETUP.md, AUTH.md, API_REFERENCE.md, etc.)
       Optional environment variables like GEMINI_API_KEY for LLM access

**What outputs does DocuBot produce?**

>   Textual answers to user queries
    Retrieved snippets (in retrieval-only mode)
    References to source files when applicable

---

## 2. Retrieval Design

**How does your retrieval system work?**  
Describe your choices for indexing and scoring.

- How do you turn documents into an index?
- How do you score relevance for a query?
- How do you choose top snippets?

> Documents are split into small snippets (e.g., by sections or paragraphs) and indexed.
Each snippet is scored for relevance against the query using vector similarity (embedding-based).
Top N snippets (most relevant) are returned to the user or fed into the LLM.

**What tradeoffs did you make?**  
For example: speed vs precision, simplicity vs accuracy.

> Snippet size vs context: smaller snippets improve precision but may lose context.
Simplicity vs accuracy: used straightforward section-based splitting rather than complex chunking to keep code readable.
Speed vs coverage: limited number of top snippets to reduce LLM input size, improving response speed.

---

## 3. Use of the LLM (Gemini)

**When does DocuBot call the LLM and when does it not?**  
Briefly describe how each mode behaves.

- Naive LLM mode:
- Retrieval only mode:
- RAG mode:

>  Naive LLM mode: always calls the LLM on the full documentation corpus; does not use retrieval.
Retrieval only mode: never calls the LLM; returns raw relevant snippets.
RAG mode: calls the LLM only on retrieved snippets to generate a concise, grounded answer.

**What instructions do you give the LLM to keep it grounded?**  
Summarize the rules from your prompt. For example: only use snippets, say "I do not know" when needed, cite files.

> Use only the provided retrieved snippets to answer the question.
If snippets do not contain enough information, respond: “I do not know based on the docs I have.”
Cite the source file(s) used in the answer.

---

## 4. Experiments and Comparisons

Run the **same set of queries** in all three modes. Fill in the table with short notes.

You can reuse or adapt the queries from `dataset.py`.

| Query | Naive LLM: helpful or harmful? | Retrieval only: helpful or harmful? | RAG: helpful or harmful? | Notes |
|------|---------------------------------|--------------------------------------|---------------------------|-------|
| Example: Where is the auth token generated? | | | | |
| Example: How do I connect to the database? | | | | |
| Example: Which endpoint lists all users? | | | | |
| Example: How does a client refresh an access token? | | | | |

| Query                                      | Naive LLM: helpful or harmful?                                     | Retrieval only: helpful or harmful?                              | RAG: helpful or harmful?                                  | Notes                                              |
| ------------------------------------------ | ------------------------------------------------------------------ | ---------------------------------------------------------------- | --------------------------------------------------------- | -------------------------------------------------- |
| Where is the auth token generated?         | Helpful but overgeneralized; mentions IdP, JWT, backend frameworks | Helpful; shows exact snippet from `AUTH.md`                      | Helpful; concise and grounded in `AUTH.md`                | RAG balances readability and factual grounding     |
| How do I connect to the database?          | Helpful but may hallucinate steps not in docs                      | Helpful; shows `SETUP.md` snippet with DATABASE_URL instructions | Helpful; gives concise steps with reference to `SETUP.md` | RAG avoids extra speculation                       |
| Which endpoint lists all users?            | Helpful; may mention endpoints not in docs                         | Helpful; raw snippet from `API_REFERENCE.md`                     | Helpful; concise, cites `API_REFERENCE.md`                | RAG produces user-friendly summary                 |
| How does a client refresh an access token? | Helpful; explains concept but adds extra general info              | Helpful; shows snippet about `/api/refresh`                      | Helpful; concise, cites snippet                           | RAG avoids hallucinating refresh logic beyond docs |


**What patterns did you notice?**  

- When does naive LLM look impressive but untrustworthy?  
- When is retrieval only clearly better?  
- When is RAG clearly better than both?

> Naive LLM looks impressive but can hallucinate details or include extra unrelated information.
Retrieval only is accurate and fully grounded but harder to read and interpret.
RAG consistently provides readable answers that are grounded in documentation, making it the most reliable for end users.

---

## 5. Failure Cases and Guardrails

**Describe at least two concrete failure cases you observed.**  
For each one, say:

- What was the question?  
- What did the system do?  
- What should have happened instead?

> Question: “Who can access the /api/projects endpoint?”
What the system did: Naive LLM claimed only admins can access, even though docs specify users with access may vary.
Correct behavior: Should return either the relevant snippet from API_REFERENCE.md or say “I do not know based on the docs I have.”

> Question: “What is the maximum token lifetime?”
What the system did: RAG returned nothing because snippet threshold excluded small TOKEN_LIFETIME_SECONDS mention.
Correct behavior: Should include snippet and note default of 3600 seconds from AUTH.md.

**When should DocuBot say “I do not know based on the docs I have”?**  
Give at least two specific situations.

> When retrieved snippets do not provide enough evidence to answer the question.
When a user asks about undocumented features or scenarios not in the corpus.

**What guardrails did you implement?**  
Examples: refusal rules, thresholds, limits on snippets, safe defaults.

> Threshold on snippet relevance to avoid using unrelated context.
Mandatory refusal if no useful snippets are retrieved.
LLM instructed to cite source files and avoid inventing information.

---

## 6. Limitations and Future Improvements

**Current limitations**  
List at least three limitations of your DocuBot system.

1. Small snippet size may omit context.
2. LLM may still infer slightly beyond snippets if instructions are misinterpreted.
3. Retrieval only answers are less readable and require user interpretation.

**Future improvements**  
List two or three changes that would most improve reliability or usefulness.

1. Adaptive snippet chunking for better context coverage.
2. Enhanced prompt design to reduce LLM hallucinations further.
3. Include confidence scoring to highlight uncertain answers.

---

## 7. Responsible Use

**Where could this system cause real world harm if used carelessly?**  
Think about wrong answers, missing information, or over trusting the LLM.

> Users relying on Naive LLM may act on inaccurate or incomplete instructions.
Sensitive configurations (e.g., AUTH_SECRET_KEY) may be misrepresented if hallucinated.

**What instructions would you give real developers who want to use DocuBot safely?**  
Write 2 to 4 short bullet points.

- Always verify critical instructions against official documentation.
- Prefer RAG mode for production use, not Naive LLM.
- Do not trust raw LLM output for security-related or operational tasks.

---
