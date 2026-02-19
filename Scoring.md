# Scoring (v0)

**Detection Quality Score** ranges from 0–100.

## Inputs
Normalized alert schema:
- rule_id
- rule_name
- severity (not yet used in v0 math)
- disposition (used for FP/TP)
- time_to_close_minutes
- event_signature (used for redundancy)
- confidence (0..1)

## v0 Score Formula
Start at 100, subtract penalties:
- FP penalty: 60 * FP_rate
- Duplication penalty: 20 * avg_dup_factor
- TTC penalty: 15 * avg_ttc_factor
- Confidence adjustment: + 10 * (avg_conf - 0.5)

Clamped to 0..100.

## Why this is useful
- Lets SOC leaders quantify "noise"
- Prioritizes tuning work with P0–P3
- Creates measurable improvement over time
