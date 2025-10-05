<a href="https://livekit.io/">
  <img src="./.github/assets/livekit-mark.png" alt="LiveKit logo" width="100" height="100">
</a>

# LiveKit Voice AI Agent - Powered by Google Gemini

A production-ready voice AI assistant built with [LiveKit Agents for Python](https://github.com/livekit/agents) and **Google Gemini**. Features "Riya", an intelligent Amazon Customer Care agent with comprehensive customer service capabilities.

## What's Inside

🎯 **Amazon Customer Care Agent** - "Riya", a fully-featured AI customer service representative
- Natural conversation flow with contextual understanding
- Comprehensive Amazon service knowledge (Prime, orders, returns, etc.)
- Objection handling and escalation procedures
- Professional tone and empathy

⚡ **Google Gemini Integration** - Complete AI pipeline
- **LLM**: Gemini 2.0 Flash (ultra-low latency)
- **STT**: Google Speech-to-Text (streaming, 170+ languages)
- **TTS**: Google Text-to-Speech (natural neural voices)
- **Function Calling**: Automatic tool use with Gemini

🔧 **Performance Optimized**
- ~1-2 second total response time
- Streaming enabled on all components
- VAD optimized for 100ms speech detection
- Preemptive generation for lower latency

📊 **Testing & Evaluation**
- Complete test suite with pytest
- LLM-as-judge evaluations
- Function calling tests
- Safety and refusal testing

## Quick Start (5 Minutes)

### Prerequisites

- Python 3.9+ installed
- [Google Gemini API key](https://aistudio.google.com/app/apikey) (free tier available)
- [LiveKit Cloud account](https://cloud.livekit.io/) (free tier available)

### Setup

**1. Install Dependencies**

```bash
cd livekit-agent

# Option A: Using uv (recommended)
uv sync

# Option B: Using venv
python -m venv .venv
.venv\Scripts\activate  # Windows
# source .venv/bin/activate  # macOS/Linux
pip install -r requirements-dev.txt
```

**2. Configure API Keys**

Copy the environment template:

```bash
cp .env.example .env.local
```

Edit `.env.local` with your credentials:

```bash
# LiveKit Configuration
LIVEKIT_URL=wss://your-project.livekit.cloud
LIVEKIT_API_KEY=your-api-key
LIVEKIT_API_SECRET=your-api-secret

# Google Gemini API Key (PRIMARY)
GOOGLE_API_KEY=your-gemini-api-key
```

**Get your keys:**
- **Gemini API**: https://aistudio.google.com/app/apikey (free tier: 15 requests/min)
- **LiveKit**: https://cloud.livekit.io/projects (free tier: 50 concurrent connections)

**3. Download Required Models**

```bash
# Using uv
uv run python src/agent.py download-files

# Using venv
python src/agent.py download-files
```

This downloads VAD and turn detection models (one-time setup, ~2-3 minutes).

### Run the Agent

**Option A: Console Mode** (Interactive testing in terminal)

```bash
# Using uv
uv run python src/agent.py console

# Using venv
python src/agent.py console
```

Now speak! The agent will respond as "Riya" from Amazon Customer Care.

**Option B: Dev Mode** (For frontend integration)

```bash
# Using uv
uv run python src/agent.py dev

# Using venv
python src/agent.py dev
```

Then connect from:
- **Web**: https://agents-playground.livekit.io/
- **Mobile**: Use LiveKit example apps (see below)

**Option C: Production Mode**

```bash
# Using uv
uv run python src/agent.py start

# Using venv
python src/agent.py start
```

## Testing Your Agent

### Quick Voice Test

1. Run: `python src/agent.py console`
2. Say: **"Hello, I need help with my order"**
3. Agent responds as Riya with customer care assistance

### Test Function Calling

1. Say: **"What's the weather in Tokyo?"**
2. Agent automatically calls the `lookup_weather` function
3. Returns result naturally in conversation

### Run Test Suite

```bash
# Using uv
uv run pytest

# Using venv
pytest

# With verbose output
pytest -v
```

Tests include:
- Greeting and assistance behavior
- Function calling (weather tool)
- Error handling
- Grounding (refusing unknown info)
- Safety (refusing harmful requests)

## Configuration

### Current Setup (Google Gemini)

The agent is fully configured with Google's AI stack:

**LLM** - Gemini 2.0 Flash
```python
model="gemini-2.0-flash-001"    # Ultra-fast for real-time
temperature=0.8                  # Balanced creativity
max_output_tokens=200            # Optimized for voice
```

**STT** - Google Speech-to-Text
```python
model="latest_long"              # Best for conversations
languages="en-US"                # English (170+ languages available)
use_streaming=True               # Real-time transcription
```

**TTS** - Google Text-to-Speech
```python
gender="female"                  # For "Riya" persona
speaking_rate=1.1                # Slightly faster
use_streaming=True               # Low latency
```

📖 **Full configuration guide**: [GEMINI_CONFIGURATION.md](GEMINI_CONFIGURATION.md)

### Customization Examples

**Change Model for Higher Quality**
```python
# In src/agent.py, line 369
model="gemini-2.5-pro-preview-05-06"  # More capable, slower
```

**Change Model for Lower Latency**
```python
model="gemini-2.0-flash-lite-preview-02-05"  # Ultra-fast
max_output_tokens=100
```

**Add Multiple Languages**
```python
# In src/agent.py, line 377
languages=["en-US", "hi-IN", "es-ES"]
detect_language=True
```

**Adjust Speaking Speed**
```python
# In src/agent.py, line 390
speaking_rate=1.3  # Faster (range: 0.25-4.0)
```

**Use Specific Voice**
```python
# In src/agent.py
voice_name="en-US-Neural2-F"  # High-quality neural voice
```

## Available Models

### Gemini LLM Models

- `gemini-2.0-flash-001` - **CURRENT** - Ultra-fast for real-time (recommended)
- `gemini-2.5-pro-preview-05-06` - Most capable, higher latency
- `gemini-2.0-flash-lite-preview-02-05` - Lightest, lowest latency
- `gemini-1.5-pro` - Previous generation, high quality

### Google STT Models

- `latest_long` - **CURRENT** - Best for conversations
- `latest_short` - For short utterances
- `telephony` - Optimized for phone calls
- `chirp_2` - Universal speech model

### Supported Languages (170+)

English: `en-US`, `en-GB`, `en-AU`, `en-IN`, etc.
Others: `hi-IN`, `es-ES`, `fr-FR`, `de-DE`, `ja-JP`, `zh-CN`, and more

Full list: [Google Speech languages](https://cloud.google.com/speech-to-text/docs/speech-to-text-supported-languages)

## Frontend Integration

Connect your voice agent to a frontend application:

| Platform | Repository | Description |
|----------|-----------|-------------|
| **Web** | [agent-starter-react](https://github.com/livekit-examples/agent-starter-react) | React & Next.js web app |
| **iOS/macOS** | [agent-starter-swift](https://github.com/livekit-examples/agent-starter-swift) | Native Swift app |
| **Android** | [agent-starter-android](https://github.com/livekit-examples/agent-starter-android) | Kotlin & Jetpack Compose |
| **Flutter** | [agent-starter-flutter](https://github.com/livekit-examples/agent-starter-flutter) | Cross-platform app |
| **React Native** | [voice-assistant-react-native](https://github.com/livekit-examples/voice-assistant-react-native) | Expo mobile app |
| **Web Embed** | [agent-starter-embed](https://github.com/livekit-examples/agent-starter-embed) | Embeddable widget |

### Telephony Integration

Add phone calling capabilities:
- **Inbound**: Receive calls to your agent
- **Outbound**: Agent makes calls
- **SIP Integration**: Connect to phone systems

📖 [Telephony Documentation](https://docs.livekit.io/agents/start/telephony/)

## Adding Custom Functions

Create new capabilities by adding functions:

```python
# In src/agent.py, add to the Assistant class:

@function_tool
async def check_order_status(self, context: RunContext, order_id: str):
    """Check the status of an Amazon order.

    Args:
        order_id: The order number (e.g., "112-7890123-4567890")
    """
    # Your logic here - call database, API, etc.
    return f"Order {order_id} is out for delivery, arriving today"
```

Gemini automatically:
- Detects when to call your function
- Extracts parameters from user speech
- Incorporates results into natural responses

More examples: [Function Tool Documentation](https://docs.livekit.io/agents/build/function-calling/)

## Performance

### Expected Latency

With current Gemini configuration:

| Component | Latency |
|-----------|---------|
| Voice Activity Detection | ~100ms |
| Speech-to-Text | 150-300ms |
| LLM Generation | 400-800ms |
| Text-to-Speech | 200-400ms |
| **Total Response Time** | **~1-2 seconds** |

### Optimization Tips

For ultra-low latency (< 1 second):
1. Use `gemini-2.0-flash-lite-preview-02-05`
2. Reduce `max_output_tokens` to 100
3. Increase `speaking_rate` to 1.3
4. See [LATENCY_OPTIMIZATIONS.md](LATENCY_OPTIMIZATIONS.md)

## Project Structure

```
livekit-agent/
├── src/
│   ├── agent.py                  # Main agent implementation (Riya)
│   └── __init__.py
├── tests/
│   └── test_agent.py             # Test suite
├── .env.example                  # Environment template
├── .env.local                    # Your credentials (gitignored)
├── pyproject.toml                # Python dependencies
├── requirements.txt              # Pip dependencies (venv)
├── requirements-dev.txt          # Dev dependencies (venv)
├── Dockerfile                    # Container deployment
├── README.md                     # This file
├── CLAUDE.md                     # Claude Code guide
├── GEMINI_CONFIGURATION.md       # Complete Gemini config guide
├── QUICKSTART_GEMINI.md          # 5-minute setup guide
├── LATENCY_OPTIMIZATIONS.md      # Performance tuning
└── SETUP_SUMMARY.md              # Setup changes summary
```

## Environment Variables

### Required

```bash
LIVEKIT_URL=          # wss://your-project.livekit.cloud
LIVEKIT_API_KEY=      # From LiveKit Cloud dashboard
LIVEKIT_API_SECRET=   # From LiveKit Cloud dashboard
GOOGLE_API_KEY=       # From Google AI Studio
```

### Optional (VertexAI)

```bash
GOOGLE_GENAI_USE_VERTEXAI=true
GOOGLE_CLOUD_PROJECT=your-project-id
GOOGLE_CLOUD_LOCATION=us-central1
GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
```

### Alternative Providers

```bash
# Not currently in use, but supported:
OPENAI_API_KEY=       # For OpenAI LLM
DEEPGRAM_API_KEY=     # For Deepgram STT
CARTESIA_API_KEY=     # For Cartesia TTS
```

## Development Commands

```bash
# Run agent in console mode
python src/agent.py console

# Run agent in dev mode (with frontend)
python src/agent.py dev

# Run agent in production mode
python src/agent.py start

# Download models (one-time setup)
python src/agent.py download-files

# Run tests
pytest

# Run tests with verbose output
pytest -v

# Run linter
ruff check .

# Format code
ruff format .
```

## Troubleshooting

### API Key Errors

**Issue**: `google.api_core.exceptions.Unauthenticated`

**Fix**: Verify your `GOOGLE_API_KEY` in `.env.local`

```bash
python -c "import os; from dotenv import load_dotenv; load_dotenv('.env.local'); print(os.getenv('GOOGLE_API_KEY'))"
```

### Import Errors

**Issue**: `ModuleNotFoundError: No module named 'livekit'`

**Fix**: Reinstall dependencies

```bash
# Using uv
uv sync

# Using venv
pip install -r requirements-dev.txt
```

### Model Download Issues

**Issue**: "Models not found" or VAD errors

**Fix**: Re-download models

```bash
python src/agent.py download-files
```

### Connection Issues

**Issue**: LiveKit connection timeout

**Fix**: Verify credentials and network

```bash
# Test LiveKit connection
python -c "import os; from dotenv import load_dotenv; load_dotenv('.env.local'); print('URL:', os.getenv('LIVEKIT_URL')); print('Key:', os.getenv('LIVEKIT_API_KEY')[:10] + '...')"
```

More troubleshooting: [QUICKSTART_GEMINI.md](QUICKSTART_GEMINI.md#troubleshooting)

## Deploying to Production

### Docker Deployment

```bash
# Build image
docker build -t livekit-agent .

# Run container
docker run -e LIVEKIT_URL=$LIVEKIT_URL \
           -e LIVEKIT_API_KEY=$LIVEKIT_API_KEY \
           -e LIVEKIT_API_SECRET=$LIVEKIT_API_SECRET \
           -e GOOGLE_API_KEY=$GOOGLE_API_KEY \
           livekit-agent
```

### LiveKit Cloud Deployment

```bash
# Deploy to LiveKit Cloud
lk deploy create

# Follow prompts to configure deployment
```

📖 Full deployment guide: [LiveKit Deployment Docs](https://docs.livekit.io/agents/ops/deployment/)

## Documentation

- **[GEMINI_CONFIGURATION.md](GEMINI_CONFIGURATION.md)** - Complete Gemini API reference
- **[QUICKSTART_GEMINI.md](QUICKSTART_GEMINI.md)** - 5-minute quick start
- **[LATENCY_OPTIMIZATIONS.md](LATENCY_OPTIMIZATIONS.md)** - Performance tuning
- **[SETUP_SUMMARY.md](SETUP_SUMMARY.md)** - What was configured
- **[CLAUDE.md](CLAUDE.md)** - Guide for Claude Code

## Resources

### Google/Gemini
- [Google AI Studio](https://aistudio.google.com/) - Get API keys, test models
- [Gemini API Documentation](https://ai.google.dev/docs)
- [Google Cloud Speech-to-Text](https://cloud.google.com/speech-to-text/docs)
- [Google Cloud Text-to-Speech](https://cloud.google.com/text-to-speech/docs)

### LiveKit
- [LiveKit Agents Documentation](https://docs.livekit.io/agents/)
- [LiveKit Cloud Dashboard](https://cloud.livekit.io/)
- [LiveKit Discord Community](https://livekit.io/discord)

### Examples & Templates
- [Frontend Examples](https://docs.livekit.io/agents/start/frontend/)
- [Telephony Integration](https://docs.livekit.io/agents/start/telephony/)
- [Function Calling Guide](https://docs.livekit.io/agents/build/function-calling/)

## Support

- **Discord**: https://livekit.io/discord
- **GitHub Issues**: Report problems in your fork
- **Documentation**: https://docs.livekit.io/

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## What Makes This Agent Special

✅ **Production-Ready** - Full customer service agent with instructions, objection handling, escalation
✅ **Gemini Optimized** - Complete Google AI stack integration (LLM + STT + TTS)
✅ **Low Latency** - ~1-2 second responses with streaming enabled throughout
✅ **Function Calling** - Automatic tool use with Gemini's native function calling
✅ **Multilingual** - 170+ languages supported by Google Speech
✅ **Tested** - Complete test suite with LLM-as-judge evaluations
✅ **Documented** - Comprehensive guides for all features
✅ **Flexible** - Easy to customize, extend, or swap providers

**Start building voice AI in 5 minutes!** 🎙️
