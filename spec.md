# spec.md — S12: S/4HANA Migration Planner

## 1. Problem
- **Who:** SAP Basis/ABAP migration leads and technical architects planning ECC-to-S/4HANA conversions.
- **What pain:** Manually cross-referencing custom Z/Y ABAP objects against S/4HANA simplification items, usage logs, and effort buckets is slow and inconsistent.
- **How often:** Every S/4HANA readiness assessment and wave-planning cycle (weekly/per migration sprint).

## 2. Success
- Cut custom-code triage and wave-planning time by **70%** (from ~5 days to under 4 hours per 500 objects) with **0 unclassified objects** and **>= 90%** agreement with architect labels on evals.

## 3. Users
- **Uses it:** Migration analysts and ABAP consultants generating wave recommendations.
- **Approves:** Lead SAP Solution Architect reviews and approves the final migration plan and any `retire` / `adapt` overrides.

## 4. Data
- **Source:** Synthetic/anonymized custom code inventory JSON (`data/custom_inventory.json`) and sample S/4HANA simplification rules (`data/simplification_rules.json`).
- **Sample size:** 30 benchmark objects for evals + sample inventory for live demo.
- **No real data:** Strictly synthetic object names (`Z_*`, `Y_*`), dummy usage counts, and non-production client data only.

## 5. Tools
1. `load_inventory` — **Input:** `inventory_path` (str) · **Output:** list of custom objects with type, LOC, and last-used metrics.
2. `match_simplification` — **Input:** `object_record` (dict), `rules_path` (str) · **Output:** matching simplification impact notes and obsolete/changed table/API flags.
3. `classify_object` — **Input:** `object_record` (dict), `simplification_matches` (list) · **Output:** label (`retire` | `adapt` | `keep`) with explicit technical rationale.
4. `estimate_effort` — **Input:** `classification` (str), `complexity_score` (int/float) · **Output:** estimated effort (story points / hours) and migration wave (`Wave 0 Retire` | `Wave 1 Quick Keep` | `Wave 2 Adapt`).
5. `write_plan` — **Input:** `classified_items` (list), `output_path` (str) · **Output:** structured migration plan report (Markdown/JSON) + summary counts.

## 6. Guardrails
- **Approval point:** Human architect sign-off required before marking objects for deletion/retirement in production cutover scope; every classification must include a non-empty reason.
- **Limits:** Max 500 objects per batch run; read-only analysis on inventory files; never fabricate simplification rule IDs not present in `simplification_rules.json`.
- **Audit log:** Append every tool call, input hash, classification decision, and approval state to `audit.jsonl` with timestamp.

## 7. Evals
- **30 cases:** 30 custom objects pre-labeled by domain logic into `retire`, `adapt`, or `keep` (covering unused reports, obsolete BAPI/table references, and clean custom enhancements).
- **How scored:** Exact match on classification label (`retire`/`adapt`/`keep`) + presence of valid rationale + effort/wave consistency.
- **Target %:** **>= 90%** label accuracy (27/30) and **100%** schema/audit compliance.

## 8. Attacks
1. Prompt injection hidden inside a custom object description/comment field trying to force `keep` on deprecated code.
2. Empty or malformed inventory JSON file.
3. Object with zero usage (`last_used_days > 365` and `exec_count == 0`) that also matches a simplification item.
4. Unknown object type (e.g., custom enhancement or smartform variant) missing complexity metrics.
5. Simplification rule file missing or containing duplicate rule IDs.
6. Attempt to call `write_plan` with missing mandatory `reason` field on `adapt` or `retire`.
7. Oversized batch exceeding configured object limit.
8. Path traversal attempt in `inventory_path` or `output_path`.
9. Ambiguous/conflicting signals (e.g., high execution count but references a removed table).
10. LLM hallucinating an SAP simplification note ID not in the dataset.

## 9. Out of scope
- Direct RFC/OData connection to a live SAP ECC or S/4HANA system.
- Automated ABAP code refactoring or ATC transport generation.
- Real customer/client production codebases or HR/financial PII.
- Multi-user web UI or role-based SSO authentication.

## 10. Demo
- **Minute 0:00–1:00:** State the problem, metric target, and guardrail (architect approval + mandatory reason).
- **Minute 1:00–2:30:** Run `load_inventory` and `match_simplification` on sample objects.
- **Minute 2:30–3:45:** Show `classify_object` and `estimate_effort` separating unused objects as `retire` and grouping active objects into migration waves.
- **Minute 3:45–4:30:** Generate plan via `write_plan` and inspect `audit.jsonl` + refusal/fallback on an edge case.
- **Minute 4:30–5:00:** Share eval score on the 30-object benchmark and next MCP build steps.