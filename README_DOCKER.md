# Minimal Docker image for `podcast_mcp_server.py`

Build and run (CPU-only):

```bash
# Build image
docker build -t bkhk-teacher:latest .

# Run container (exposes port 8080)
docker run --rm -p 8080:8080 bkhk-teacher:latest
```

Notes:
- This Dockerfile creates a small multi-stage image using `python:3.11-slim`.
- The `requirements.txt` is derived from the script header. If you use additional packages, update it.
- The image forces CPU-only behavior via environment variables (`TOKENIZERS_PARALLELISM=false`, `OMP_NUM_THREADS=1`).
