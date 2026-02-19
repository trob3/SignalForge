# SignalForge

SignalForge is an MVP tool for **Detection Quality Scoring** — it helps SOC teams identify noisy detections, quantify signal-to-noise, and prioritize tuning work.

## What it does (MVP)
- Ingest alerts from CSV (e.g., Sentinel/Splunk/Security Hub exports)
- Normalize to a common schema
- Compute a **Detection Quality Score (0–100)** per rule/detection
- Surface top noisy detections and tuning candidates

## Detection Quality Score (simple v0)
Signals are scored using:
- **False Positive Rate** (from disposition history)
- **Duplication / Redundancy** (repeated similar alerts)
- **Time to Close** (proxy for analyst friction)
- **Confidence** (optional field; default mid)

See docs/scoring.md for details.

## Quickstart (local)
```bash
python -m venv .venv && source .venv/bin/activate
pip install -U pip
pip install -e .
uvicorn signalforge.api.main:app --reload
