# ai-stack

A self-hosted AI stack. Its first component is an OpenAI-compatible gateway that routes each request to a local model or a cloud API and records tokens, latency and cost per request.

## Status

Early setup: project scaffolding is in place; the gateway is not implemented yet.

## Requirements

- [git](https://git-scm.com/)
- [uv](https://docs.astral.sh/uv/) (Python 3.12 is required; uv downloads it automatically if it isn't already installed)

## Setup

```bash
git clone https://github.com/Tudix/ai-stack.git
cd ai-stack
uv sync
```
