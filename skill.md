---
name: simplified-english
description: Use when English text must be parsed without a human to resolve ambiguity — tool descriptions, error messages, inter-agent instructions, system prompts, status reports — and misreading has a real cost, or when text reads as dense, hedged, or easy to misparse. Triggers: disambiguate, STE100 rewrite, apply Simplified Technical English, plain-language rewrite, controlled-language rewrite, rewrite so an agent cannot misread this. Not for creative or marketing copy."
source: based on github.com/danyuchn/asd-ste100-skill
version: 0.1.0
---

# Simplified Technical English (ASD-STE100)
Rewrite technical and operational English to eliminate two failure modes: **words with multiple meanings** and **sentences with ambiguous structure**.

> **Scope Note:** This skill enforces ASD-STE100 Issue 9 structural rules deterministically and applies lexical rules as a plain-language principle (the proprietary ~900-word ASD dictionary is not bundled). 

## 1. Mode Selection
Select a mode silently before rewriting based on the input text type (unless the user specifies one):

- **`Strict` Mode** *(Default for prompts, tool/function descriptions, inter-agent instructions, error messages, procedures, safety text)*: Enforce all **Structural**, **Tense**, and **Lexical** rules strictly, including hard word-count limits and strict term consistency.
- **`STE-Flavored` Mode** *(Default for READMEs, PR descriptions, changelogs, architectural prose)*: Enforce all **Structural** and **Tense** rules. Treat **Lexical** rules (one-word-one-meaning, strict part-of-speech lockdown) as advisory so prose remains natural.

## 2. Rewrite Specification
### A. Structural Rules (Enforce in Both Modes)
| Rule | Requirement | Bad (Don't) | Good (Do) |
|---|---|---|---|
| **Active voice** | Subject performs the action (unless actor is unknown/irrelevant). | "The file is deleted by the agent." | "The agent deletes the file." |
| **One instruction per sentence** | Split compound instructions into separate sentences or numbered steps. | "Open the file and read line 3, then check if it matches." | "1. Open the file.\n2. Read line 3.\n3. Check if line 3 matches." |
| **Sentence length caps** | **≤20 words** for instructions/procedures.<br>**≤25 words** for descriptions. | Long multi-clause sentences. | Split into two sentences or convert conditions into a list. |
| **No semicolons (Rule 8.1)** | Ban `;` completely. Replace with a period or list. | "Connection failed; retry in 5s." | "The connection failed. Retry in 5 seconds." |
| **Noun clusters (≤3 words)** | Unpack stacked noun phrases exceeding 3 words with prepositions or relative clauses. | "high pressure fuel pump inlet valve assembly" | "the inlet-valve assembly for the high-pressure fuel pump" |
| **No ellipsis / omission** | Keep subjects, verbs, and articles explicit. | "Files not backed up will be lost." | "If you do not back up a file, the system deletes it." |
| **Lists for sequences** | Use numbered/bulleted lists for 3+ steps, conditions, or options. | Burying 3+ items inside prose. | Vertical list with parallel grammar. |
| **Paragraph limits** | One topic per paragraph; **≤6 sentences** maximum. | Dense multi-topic walls of text. | Short, single-topic paragraphs. |

### B. Tense & Modality Rules (Enforce in Both Modes)
| Rule | Requirement | Bad (Don't) | Good (Do) |
|---|---|---|---|
| **Simple tenses only** | Use imperative, infinitive, simple present, simple past, or simple future. Avoid compound tenses *unless* present relevance is critical to status reporting. | "We have received the payload." | "We received the payload." *(Or keep "The job has completed" if current state matters, and note it).* |
| **Preserve factual modality** | **Never upgrade a hedge into a fact** or invent certainty/causation not in the source. | Rewriting "The request may have timed out" as "The request timed out." | "The request may have timed out." |
| **Strip stacked hedges** | Remove throat-clearing and redundant qualifiers while keeping single factual modals. | "It is worth noting that this might potentially fail." | "This operation might fail." |

### C. Lexical & Anti-Bloat Rules (Strict = Enforce; STE-Flavored = Advisory)
| Rule | Requirement | Bad (Don't) | Good (Do) |
|---|---|---|---|
| **No phrasal verbs (Rule 9.3)** | Replace multi-word idiomatic verbs with single plain verbs. | spin up, take off, reach out, dive into, kick off | start, remove, contact, examine, begin |
| **Verb over nominalization (Rule 3.7)** | Replace noun-forms of actions with active verbs. | "Perform an analysis of", "provides assistance to" | "Analyze", "helps" |
| **No synonym rotation** | Use the exact same noun for the same entity and the same verb for the same action throughout. | Mixing "user / client / caller" or "check / verify / confirm" | Pick "user" and "check"; use them every time. |
| **One part of speech per word** | Do not use the same word as both a noun and a verb in the same text. | "Oil the valve with clean oil." | "Apply clean oil to the valve." |
| **Zero marketing adjectives** | Delete unverifiable quality claims or replace with exact metrics. | seamless, robust, cutting-edge, blazing-fast | Delete, or state the metric ("responds in <50ms"). |
| **Domain jargon** | Keep necessary technical terms, but define uncommon ones on first use. | Undefined internal acronyms or overloaded terms. | Define inline on first use. |

## 3. Execution Workflow
1. **Assess Input:**
   - If the input is creative, marketing, or persuasive copy where tone/nuance is the primary goal, decline the rewrite and state why.
   - If the input has zero substantive content (pure fluff), state that the source text contains no concrete claims to preserve rather than outputting hollow sentences.
   - If the input already complies with all rules, state that it is already compliant and make no forced edits.
2. **Read for Invariants:** Identify every fact, condition, exception, metric, and epistemic modal ("may", "could", "sometimes") that **must** survive the rewrite.
3. **Run Linter (When Working with Files or Long Specs):**
   - If shell execution is available and the input is a file or multi-paragraph specification, run:
     `python3 scripts/ste-lint.py <input>` (use `--json` for structured diagnostics).
   - Flags: `--baseline N` (tolerate N hard violations), `--disable rule1,rule2` (silence specific rules).
4. **Rewrite Sentence by Sentence:**
   - Apply Mode rules from Section 2.
   - **Precision Guardrail:** Never drop a safety condition, scope qualifier, or factual hedge just to hit the 20/25-word sentence cap. Either split the sentence cleanly or keep the longer sentence and declare it under `Kept as-is:`.
5. **Verify Output:**
   - Re-verify against the 6 common failure habits: synonym rotation, stacked hedges, nominalizations, marketing adjectives, semicolons/run-ons, and phrasal verbs.
   - If `scripts/ste-lint.py` was used in Step 3, run it against your rewritten draft to confirm 0 hard violations before responding.

## 4. Output Format
### Default Output (Drop-In Replacement)
Output **only** the rewritten text so the user can copy-paste it directly into code, prompts, or documentation.
- **No preamble**, no mode announcement, no summary of changes, and no closing pleasantries.
- **Sole Exception (`Kept as-is:`):** If you deliberately violated a length or tense rule to preserve critical precision or modality, append a single line after a blank line:
  `Kept as-is: "<exact phrase>" — <1-sentence reason precision/modality required it>.`

### Diagnostic Output (Only When Requested)
If the user asks for an explanation, diff, or before/after breakdown ("show changes", "which rules broke", "before/after"), output:
1. The complete **Rewritten Text** block first.
2. A **Rule Violations Table**:
   ```markdown
   | Rule Violated | Original | Simplified |
   |---|---|---|
   | Present perfect tense | "We have received your request." | "We received your request." |
   | Noun cluster (4+ words) | "the agent task queue priority handler" | "the handler that sets task-queue priority" |

   Mode: <Strict STE-Flavored |> | Violations fixed: <N>
3. Any Kept as-is: notes (if applicable).
