# CLAUDE.md — S12: S/4HANA Migration Planner

## Context
Read `spec.md` first. This repository implements the **S/4HANA Migration Planner** agent and MCP tools for classifying custom ABAP/SAP objects into `retire`, `adapt`, or `keep`, estimating effort, grouping into migration waves, and writing an architect-reviewable plan.

## Folder Structure
- `spec.md` — Product, tool, guardrail, eval, attack, and demo specification.
- `CLAUDE.md` — Repository conventions and instructions for Claude Code.
- `data/custom_inventory.json` — Sample custom object inventory.
- `data/simplification_rules.json` — Sample S/4HANA simplification rules.
- `src/mcp_tools/` — MCP tool implementations (`load_inventory`, `match_simplification`, `classify_object`, `estimate_effort`, `write_plan`).
- `evals/` — 30-case evaluation dataset and runner script.
- `audit.jsonl` — Append-only audit log (gitignored).

## Coding & Agent Rules
- Keep Python clean, typed, and modular; separate pure tool logic from MCP server wiring so unit tests and evals can import tools directly.
- Never invent simplification IDs or SAP notes not present in `data/simplification_rules.json`.
- Require a non-empty `reason` on every classification and log actions to `audit.jsonl`.
- Do not commit `.env` or real customer/SAP data.
