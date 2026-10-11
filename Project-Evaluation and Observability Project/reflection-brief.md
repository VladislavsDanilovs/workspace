# Reflection Brief — AI Engineering Project Evaluation

## 1. Validated, Routed Pipeline (System 1)
- **Human Review Routing & Independent Signal:** In our routing decisions, records requiring manual verification were flagged based on low confidence scores or policy risk triggers (e.g., umbrella policies with complex exclusion fields). The independent signal driving human review is the model's epistemic uncertainty paired with strict schema validation rules.
- **Calibration Insight:** As demonstrated in our calibration report, an aggregate Brier score can often mask performance issues. Specifically, individual cells (such as umbrella policy exclusions) showed an observed accuracy of `0.00` despite a predicted confidence of `0.93`, exposing hidden vulnerabilities that a single overall metric fails to reveal.

## 2. Schema-Enforced Two-Pass Extraction (System 2)
- **Handling Missing Data vs. Fabrication:** When processing `income_missing_bonus.txt`, the system returned `null` for missing fields rather than inventing values. This behavior is enforced by structuring the JSON schema with explicit nullable union types (`["<base>", "null"]`) and keeping optional fields out of the strict per-document `required` lists, effectively removing the model's license to fabricate when documents are silent.
- **Arithmetic Consistency:** When running `income_sum_mismatch.txt`, the second-pass validator caught a mathematical discrepancy between line items and the stated total, proving that deterministic post-processing safeguards are essential because LLMs cannot reliably perform raw arithmetic under unstructured text constraints.

## 3. Multi-Source Synthesis & Conflict Resolution (System 3)
- **Conflict Handling & Attribution:** In our briefing output (`03-briefing-normal.txt`), the metric `on_time_delivery_rate` exhibited a direct conflict between two sources:
  - `95.0 percent` reported by `supplier_audit` (as of 2026-04-10)
  - `78.0 percent` reported by `logistics` (as of 2026-04-05)
  Rather than blindly reconciling or averaging these conflicting values into a single distorted number, the system retained both with precise source attribution and timestamps, ensuring complete provenance and transparency for human decision-makers.
- **Resilience under Source Failure:** When executing with `--simulate-timeout`, the system successfully isolated the failure of the `logistics` source (`timeout reading logistics`), shifted affected metrics into the **Incomplete** section, and generated a partial executive briefing instead of aborting the entire pipeline.

## 4. Synthesis & Reliability Principle
- **Core Principle:** *Deterministic guards must wrap probabilistic generation.* 
- **Application:** Across all three systems, LLMs were treated as untrusted generators of raw text or structured hypotheses, while Python-level schemas, retry policies, calibration bounds, and cross-source synthesis engines served as hard structural boundaries. In our own engineering work, we will apply this principle by never trusting raw model outputs in critical paths without explicit schema validation, provenance tracking, and fallback error handling.
