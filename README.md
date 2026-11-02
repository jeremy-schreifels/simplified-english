```markdown
# Simplified Technical English (ASD-STE100) Skill

An AI agent skill that rewrites dense, ambiguous, or hedged English into unambiguous *Simplified Technical English (ASD-STE100)*.

ASD-STE100 is a controlled-language specification originally created by the aerospace and defense industry to prevent maintenance technicians from misreading complex instructions. It eliminates the two primary causes of misinterpretation: **words with multiple meanings** and **sentences with ambiguous grammar**.

This skill adapts that discipline for AI outputs (e.g., tool descriptions, error messages, runbooks, and inter-agent instructions) that must be parsed reliably by LLMs, translation pipelines, or engineers without a human in the loop to clarify intent.

---

## Features
* **Two Operating Modes:**
  * **`Strict` Mode:** Enforces hard sentence-length caps, active voice, zero phrasal verbs, zero nominalizations, and strict one-word-one-meaning consistency. Ideal for system prompts, tool definitions, error strings, and runbooks.
  * **`STE-Flavored` Mode:** Enforces all structural clarity rules while treating strict vocabulary lockdowns as advisory. Ideal for READMEs, PR descriptions, changelogs, and technical prose.
* **Modality & Precision Guardrails:** Strips throat-clearing and stacked qualifiers (`"it is worth noting that this might potentially..."`) while strictly preserving factual epistemic hedges (`"may"`, `"could"`). Never upgrades a possibility into a false certainty just to shorten a sentence.
* **Drop-In Output by Default:** Returns only the rewritten text for immediate copy-pasting, or generates a full *Before/After Rule Violations Table* on request.

---

## Repository Structure

```text
simplified-english/
├── SKILL.md                   # Core agent instructions and rewrite state machine
├── README.md                  # Human documentation, background, and CLI usage
```

---

## Usage with AI LLMs

### Trigger Phrases

Once installed in your agent's skill directory, invoke the skill naturally:

* *"Apply Simplified Technical English to this tool description..."*
* *"Disambiguate this system prompt so a downstream agent cannot misread it."*
* *"STE100 rewrite (Strict mode): [paste text]"*
* *"Rewrite this README in STE-flavored mode and show the rule table."*

### Example Transformation

**Input:**

> *"It is important to note that the agent task queue priority handler may potentially spin up a background worker if the payload has not been processed; however, files not backed up will be lost during a restart."*

**Default Output (`Strict` Mode):**

```text
The handler that sets task-queue priority may start a background worker if the system did not process the payload. If you do not back up a file, the system deletes it during a restart.

```

**Diagnostic Output (When asking to *"show changes"* or *"explain rules"*):**

| Rule Violated | Original | Simplified |
| --- | --- | --- |
| Stacked hedges | "It is important to note that ... may potentially" | "may" *(preserved factual modal)* |
| Noun cluster (4+ words) | "the agent task queue priority handler" | "The handler that sets task-queue priority" |
| Phrasal verb (Rule 9.3) | "spin up" | "start" |
| Passive voice + compound tense | "if the payload has not been processed" | "if the system did not process the payload" |
| Semicolon ban (Rule 8.1) | "...processed; however, files..." | Split into two sentences |
| Ellipsis / omitted actor | "files not backed up will be lost" | "If you do not back up a file, the system deletes it" |

`Mode: Strict | Violations fixed: 6`

```

---

## Source, Scope, and Copyright Notice

This skill encodes the **structural rule categories** of **ASD-STE100 Issue 9 (January 2025)**: 53 writing rules across 9 sections covering word choice, grammar, sentence structure, and style. This skill does not apply all ASD-STE100 rules and does not use the approved word/unapproved word dictionaries. This is not an official ASD product.

---

## Acknowledgments & Credits

* Adapted from [`danyuchn/asd-ste100-skill`](https://github.com/danyuchn/asd-ste100-skill).
* Standard maintained by the Simplified Technical English Maintenance Group (STEMG) under the AeroSpace and Defence Industries Association of Europe (ASD).
