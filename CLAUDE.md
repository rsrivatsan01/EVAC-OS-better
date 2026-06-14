# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Environment Setup

Python 3.13. Install PyTorch **before** `requirements.txt` — it is absent from that file and requires a special index URL:

```
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu
pip install -r requirements.txt
```

## Run Commands

| Command | Purpose |
|---|---|
| `python app.py` | Flask dashboard at http://localhost:5000 |
| `python test_pipeline.py` | End-to-end integration test (CLI, not pytest) |
| `python tests/phase6_test.py` | LSTM model validation |
| `python diag_lstm.py` | LSTM inference diagnostics → writes `diag_out.txt` |
| `python Tools/define_zones.py` | Interactive OpenCV GUI to draw exit zones → `data/zones.json` |
| `python Tools/define_graph.py` | Interactive OpenCV GUI to build venue graph → `data/venue_graph.json` |

`test_pipeline.py` accepts flags: `--no-display`, `--no-log`, `--video`, `--conf`, `--skip`, `--max-frames`, `--source`.

## Environment Variables

- `VIDEO_SOURCE` — defaults to `data/videos/sample_crowd.mp4`. Set to `0` for webcam or an RTSP URL for live feed. `python-dotenv` is installed but no `.env` file is committed; create one locally.

## Binary Assets

`data/videos/sample_crowd.mp4` and `models/` weights are **not committed** and have no settled external source yet. Without `sample_crowd.mp4` the pipeline fails at startup. Fallbacks: if LSTM weights are missing the system uses rule-based congestion status; if `yolov8_person.pt` is missing ultralytics auto-downloads `yolov8n.pt`.

## Critical Gotchas

- **Flask debug mode must stay off** — `FLASK_DEBUG = False` in `config.py`. Debug mode spawns a second process which starts a second pipeline thread.
- **`app.py` starts the ML pipeline at module level** — importing it triggers background video processing immediately.
- **LSTM warm-up** — predictions switch from rule-based to LSTM-based only after 30 frames per zone (`LSTM_SEQUENCE_LENGTH` in config). Expect rule-based output during warm-up.
- **`floormap.png`** — if `static/images/floormap.png` is missing, the pipeline substitutes a blank grey image rather than crashing.
- **`data/logs/`** — created automatically by the logger on startup; no manual setup needed.

## Code Style

No linter or formatter is configured. Existing conventions to preserve:
- Alignment-heavy variable assignment: pad with spaces so `=` signs align column-wise.
- File-level comment banners: `# ─────────────────────────────────────────────`
- Section dividers: `# ── Section Name ────────────────────────────`
- Type hints on all function signatures; f-strings throughout (no `%` or `.format()`).
