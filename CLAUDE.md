# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a LiveKit Agents voice AI application built with Python, designed to create conversational voice AI assistants. The current implementation features "Riya", an Amazon Customer Care AI agent with comprehensive customer service capabilities.

## Development Setup

### Environment Setup with venv (Alternative to uv)

The project uses `uv` by default, but you can use Python's built-in `venv` instead:

```bash
# Navigate to the project directory
cd livekit-agent

# Create virtual environment
python -m venv .venv

# Activate virtual environment
# On Windows:
.venv\Scripts\activate
# On macOS/Linux:
source .venv/bin/activate

# Install dependencies from pyproject.toml
pip install -e .
pip install -e ".[dev]"
```

### Environment Variables

Copy `.env.example` to `.env.local` and configure:

**Required:**
- `LIVEKIT_URL`: LiveKit Cloud URL or self-hosted instance
- `LIVEKIT_API_KEY`: LiveKit API key
- `LIVEKIT_API_SECRET`: LiveKit API secret
- `GOOGLE_API_KEY`: **PRIMARY** - Google Gemini API key (get from https://aistudio.google.com/)

**Optional (for VertexAI instead of Gemini API):**
- `GOOGLE_GENAI_USE_VERTEXAI`: Set to `true` to use VertexAI
- `GOOGLE_CLOUD_PROJECT`: Your Google Cloud project ID
- `GOOGLE_CLOUD_LOCATION`: Region (default: us-central1)
- `GOOGLE_APPLICATION_CREDENTIALS`: Path to service account JSON

**Alternative Providers (not currently in use):**
- `OPENAI_API_KEY`: For OpenAI LLM
- `DEEPGRAM_API_KEY`: For Deepgram STT
- `CARTESIA_API_KEY`: For Cartesia TTS

**Current Configuration**: The agent is fully optimized to use Google Gemini ecosystem (Gemini 2.0 Flash LLM, Google STT, Google TTS). See [GEMINI_CONFIGURATION.md](GEMINI_CONFIGURATION.md) for comprehensive configuration details.

### Required Model Downloads

Before first run, download required models:

```bash
# With venv activated:
python src/agent.py download-files
```

This downloads:
- Silero VAD (Voice Activity Detection) models
- LiveKit Turn Detector models

## Common Commands

### Running the Agent

```bash
# Console mode (speak directly in terminal)
python src/agent.py console

# Development mode (for use with frontend/telephony)
python src/agent.py dev

# Production mode
python src/agent.py start
```

### Testing

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_agent.py

# Run with verbose output
pytest -v
```

### Code Quality

```bash
# Run linter
ruff check .

# Format code
ruff format .
```

## Architecture

### Core Components

**Main Entry Point**: [src/agent.py](src/agent.py)
- `Assistant` class (lines 29-345): Defines the AI agent with comprehensive Amazon customer care instructions
- `prewarm()` function (lines 348-356): Initializes VAD with optimized low-latency settings
- `entrypoint()` function (lines 358-428): Sets up the agent session and voice pipeline

### Voice Pipeline Configuration

The agent uses a multi-component pipeline:

1. **VAD (Voice Activity Detection)**: Silero VAD with optimized latency settings
   - Configuration at [agent.py:350-355](src/agent.py#L350-L355)
   - Optimized for 100ms speech detection, 300ms silence detection

2. **LLM**: Currently configured for Google Gemini
   - Configuration at [agent.py:368](src/agent.py#L368)
   - Can be swapped with OpenAI, Anthropic, etc.

3. **STT (Speech-to-Text)**: Currently Google STT
   - Configuration at [agent.py:369](src/agent.py#L369)
   - Alternative: Deepgram (see docs)

4. **TTS (Text-to-Speech)**: Currently Google TTS
   - Configuration at [agent.py:370](src/agent.py#L370)
   - Alternatives: ElevenLabs, Cartesia (see docs)

5. **Turn Detection**: Multilingual model for conversation turn detection
   - Configuration at [agent.py:372](src/agent.py#L372)

### Latency Optimizations

The codebase includes comprehensive latency optimizations documented in [LATENCY_OPTIMIZATIONS.md](LATENCY_OPTIMIZATIONS.md):
- Target: ~53% reduction in total pipeline latency (2.25s → 1.05s)
- VAD: 60% faster (250ms → 100ms)
- STT: 50% faster (300ms → 150ms)
- LLM: 50% faster (800ms → 400ms)
- TTS: 60% faster (500ms → 200ms)

### Agent Instructions

The `Assistant` class contains extensive instructions for the "Riya" Amazon customer care agent, including:
- System preamble and agent profile
- TTS compatibility rules (spelling out numbers, dates, etc.)
- Amazon-specific knowledge (Prime benefits, delivery options, return policy)
- Conversation flow guidelines
- Objection handling
- Escalation procedures
- Compliance and security guidelines

## Switching Service Providers

### Using Google Services (Current Configuration)

The agent is currently configured with Google services. Required API keys in `.env.local`:
- Google Cloud credentials (typically via `GOOGLE_APPLICATION_CREDENTIALS` or default credentials)

Reference documentation:
- [Google Gemini LLM](https://docs.livekit.io/agents/models/llm/plugins/gemini/)
- [Google STT](https://docs.livekit.io/agents/models/stt/plugins/google/)
- [Google TTS](https://docs.livekit.io/agents/models/tts/plugins/google/)

### Switching to OpenAI/Deepgram/Cartesia

To switch back to the original configuration, update [agent.py:368-370](src/agent.py#L368-L370):

```python
from livekit.plugins import openai, deepgram, cartesia

session = AgentSession(
    llm=openai.LLM(model="gpt-4o-mini"),
    stt=deepgram.STT(model="nova-2"),
    tts=cartesia.TTS(),
    # ... rest of config
)
```

### Using OpenAI Realtime API

For ultra-low latency, use OpenAI's Realtime API (see commented code at [agent.py:379-383](src/agent.py#L379-L383)):

```python
session = AgentSession(
    llm=openai.realtime.RealtimeModel(voice="marin")
)
```

## Testing Framework

Tests use the LiveKit Agents testing framework with LLM-as-judge evaluations:
- `test_offers_assistance`: Validates friendly greeting behavior
- `test_weather_tool`: Tests function calling and tool integration
- `test_weather_unavailable`: Error handling validation
- `test_unsupported_location`: Edge case handling
- `test_grounding`: Ensures agent refuses unknown information
- `test_refuses_harmful_request`: Safety and refusal testing

## Key Features

- **Metrics Collection**: Built-in usage and performance metrics ([agent.py:394-405](src/agent.py#L394-L405))
- **False Interruption Handling**: Detects and recovers from background noise interruptions ([agent.py:387-390](src/agent.py#L387-L390))
- **Noise Cancellation**: LiveKit Cloud BVC noise cancellation ([agent.py:423](src/agent.py#L423))
- **Function Tools**: Decorator-based function calling (example at [agent.py:333-345](src/agent.py#L333-L345))

## Dependencies

Key dependencies from [pyproject.toml](pyproject.toml):
- `livekit-agents[google,turn-detector,silero]~=1.2`
- `livekit-plugins-noise-cancellation~=0.2`
- `python-dotenv`

Dev dependencies:
- `pytest`
- `pytest-asyncio`
- `ruff`

## File Structure

```
livekit-agent/
├── src/
│   ├── __init__.py
│   └── agent.py              # Main agent implementation
├── tests/
│   └── test_agent.py         # Test suite
├── pyproject.toml            # Python project configuration
├── taskfile.yaml             # Task runner configuration
├── .env.example              # Environment variable template
├── .env.local               # Local environment variables (gitignored)
├── Dockerfile               # Container deployment
└── README.md                # Project documentation
```

## Production Deployment

The project includes a production-ready `Dockerfile`. For deployment:
- See [README.md](README.md) section "Deploying to production"
- LiveKit Cloud deployment documentation: https://docs.livekit.io/agents/ops/deployment/

## Important Notes

- The project template excludes `uv.lock` from git, but you should commit it for production projects
- For CI/CD, add API keys as repository secrets (see [README.md:103](README.md#L103))
- The agent uses `preemptive_generation=True` for improved response times
- Metrics are logged to console for monitoring pipeline performance
