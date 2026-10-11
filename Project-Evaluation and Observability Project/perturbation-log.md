# Perturbation Log — AI Engineering Systems Evaluation

## System 1: Validated, Routed Pipeline (Policy Extractor)
- **Experiment / Change:** Evaluated fallback and retry behavior on malformed/missing required fields using the test suite (`pytest tests/test_us01_retry.py -v`) and calibration routing analysis.
- **Observed Behavior:** When a required field was missing or returned `null`, the validation error trigger correctly initiated a single-turn retry / fallback escalation path with exactly one API call, preventing infinite loops while preserving audit visibility.
- **Contrast with Unperturbed Input:** Under normal valid inputs, the pipeline routes records deterministically (`auto_approve`, `spot_check`, or `human_review`) without triggering error-feedback loops.

## System 2: Schema-Enforced Two-Pass Extraction (Mortgage Extractor)
- **Experiment / Change:** Ran the extraction pipeline across fixture files exercising different safeguards: `appraisal_informal_sqft.txt` (unit normalization), `income_missing_bonus.txt` (null handling), and `income_sum_mismatch.txt` (arithmetic consistency).
- **Observed Behavior:** 
  - `appraisal_informal_sqft.txt`: Informal text like "about 2,400 sq ft" was successfully normalized to an integer value.
  - `income_missing_bonus.txt`: Missing bonus fields correctly returned `null` instead of hallucinating values.
  - `income_sum_mismatch.txt`: The validator explicitly flagged a discrepancy between the line-item sum and the stated total.
- **Contrast with Unperturbed Input:** Clean documents pass all arithmetic checks and unit validations without raising consistency flags or returning error metadata.

## System 3: Multi-Source Synthesis (Supply Chain Risk)
- **Experiment / Change:** Executed the investigation on supplier `meridian` with and without the source-failure simulation flag (`--simulate-timeout`).
- **Observed Behavior:** 
  - *Normal run:* Successfully integrated sources, exposing metrics under Well-Established, Contested (`on_time_delivery_rate` conflict between `supplier_audit` at 95.0% and `logistics` at 78.0%), and Incomplete.
  - *Timeout run (`--simulate-timeout`):* The `logistics` source timed out (`Sources unavailable: logistics unavailable (timeout)`), but the system did not crash. Instead, it gracefully completed the run, moving dependent metrics (like `late_shipment_count`) into the **Incomplete** section annotated with `missing source: timeout reading logistics`.
- **Contrast with Unperturbed Input:** Without timeout simulation, all sources respond concurrently, allowing full cross-source corroboration and reducing the volume of incomplete/missing metrics.