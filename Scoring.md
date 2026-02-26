# Scoring (v0)

**Detection Quality Score** ranges from 0–100. Higher is better.  
A score of 100 means a rule fires accurately, closes fast, and rarely produces false positives.  
A score of 0 means a rule is producing pure noise.

---

## Inputs

Normalized alert schema (produced by `normalization.py`):

| Field                  | Type    | Notes                                      |
|------------------------|---------|--------------------------------------------|
| `rule_id`              | string  | Unique rule identifier                     |
| `rule_name`            | string  | Human-readable rule name                  |
| `severity`             | string  | `low / medium / high / critical` — not used in v0 math, reserved for v1 |
| `disposition`          | string  | Canonical label — see table below          |
| `time_to_close_minutes`| float   | 0 = instant close, 240+ = high friction   |
| `event_signature`      | string  | Used to detect repeated/duplicate alerts   |
| `confidence`           | float   | 0.0–1.0. Default: 0.5 if not provided     |

### Disposition Labels

| Canonical Value   | Aliases accepted on ingest                        |
|-------------------|---------------------------------------------------|
| `true positive`   | `true_positive`, `confirmed`, `malicious`         |
| `false positive`  | `false_positive`, `benign`, `expected`, `no_issue`|
| `benign`          | `bp`, `benign positive`                           |
| `undetermined`    | `unknown`                                         |
| `open`            | `new`                                             |

---

## v0 Score Formula
```
score = 100
      - 60  * fp_rate            # FP penalty       (dominant signal)
      - 20  * avg_dup_factor     # Duplication penalty
      - 15  * avg_ttc_factor     # Analyst friction penalty
      + 10  * (avg_conf - 0.5)  # Confidence nudge  (-5 to +5)

score = clamp(score, 0, 100)
```

### Factor Definitions

**`fp_rate`** — proportion of alerts for this rule with a false positive disposition:
```
fp_rate = FP_alerts / total_alerts    range: 0.0–1.0
```

**`avg_dup_factor`** — how often the same `event_signature` repeats for a rule:
```
sig_repeats    = (count of (rule_id, event_signature) pair) - 1
dup_factor     = clip(sig_repeats / 10, 0, 1)
avg_dup_factor = mean(dup_factor) across all alerts for this rule
```
10+ repeated signatures scores a full penalty of 1.0.

**`avg_ttc_factor`** — analyst time-to-close as a friction proxy:
```
ttc_factor     = clip(time_to_close_minutes / 240, 0, 1)
avg_ttc_factor = mean(ttc_factor) across all alerts for this rule
```
240+ minutes to close scores a full penalty of 1.0.

**`avg_conf`** — mean confidence across alerts for this rule. At 0.5 (default) this term
contributes 0. High confidence rules earn up to +5; low confidence rules lose up to -5.

---

## Tuning Priority (P0–P3)

| Priority | Condition                                          | Action                        |
|----------|----------------------------------------------------|-------------------------------|
| **P0**   | Score < 40 **or** FP rate > 60%                   | Immediate tuning or disable   |
| **P1**   | Score < 60 **or** (dup > 0.4 and FP rate > 30%)   | Tune within current sprint    |
| **P2**   | Score < 75                                         | Tune within current quarter   |
| **P3**   | Score ≥ 75                                         | Healthy — monitor only        |

---

## Score Interpretation

| Score     | Signal quality         |
|-----------|------------------------|
| 80–100    | Healthy                |
| 60–79     | Moderate noise         |
| 40–59     | High noise             |
| 0–39      | Critical — pure noise  |

---

## Limitations & Roadmap

**v0 known gaps:**
- `severity` is not yet factored into the score — a high-severity rule firing with 50% FP rate
  should be penalized differently than a low-severity one
- Duplication scaling (denominator of 10) is a heuristic — high-volume environments may need
  this tuned upward
- `confidence` is self-reported or defaulted — it is a weak signal until sourced from a real
  confidence model

**Planned for v1:**
- Severity-weighted FP penalty
- Per-environment duplication baseline (rather than a fixed denominator)
- Trend scoring — track score changes over time to measure tuning ROI
- MITRE ATT&CK coverage weighting (penalize noise less on high-coverage techniques)
