# Google Gemini Configuration Guide

This document explains the comprehensive Google Gemini API configuration for the LiveKit voice agent.

## Overview

The agent is fully optimized to use Google's Gemini ecosystem:
- **LLM**: Gemini 2.0 Flash (fastest model for real-time applications)
- **STT**: Google Speech-to-Text (streaming with latest_long model)
- **TTS**: Google Text-to-Speech (neural voice synthesis)

## Environment Setup

### Required API Key

The agent uses the **Google Gemini API** (not VertexAI by default). Set your API key in `.env.local`:

```bash
GOOGLE_API_KEY=AIzaSyDBy68Ok0KlLgp85EQuwjMbfvwcP8QYsRI
```

### Optional: Using VertexAI Instead

If you prefer to use Google Cloud VertexAI:

1. Set up Google Cloud credentials:
   ```bash
   GOOGLE_GENAI_USE_VERTEXAI=true
   GOOGLE_CLOUD_PROJECT=your-project-id
   GOOGLE_CLOUD_LOCATION=us-central1
   GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
   ```

2. Or use Application Default Credentials:
   ```bash
   gcloud auth application-default login
   ```

## Current Gemini LLM Configuration

Location: [src/agent.py:368-374](src/agent.py#L368-L374)

```python
llm=google.LLM(
    model="gemini-2.0-flash-001",  # Fastest Gemini model for real-time voice
    temperature=0.8,                # Balanced creativity and consistency
    max_output_tokens=200,          # Limit for faster responses in voice context
    top_p=0.95,                     # Nucleus sampling for quality
    tool_choice="auto",             # Enable automatic function calling
)
```

### Available Gemini Models

- `gemini-2.5-pro-preview-05-06` - Most capable, slower
- `gemini-2.5-flash-preview-04-17` - Preview flash model
- `gemini-2.5-flash-preview-05-20` - Latest preview flash
- `gemini-2.0-flash-001` - **RECOMMENDED** - Fastest for real-time
- `gemini-2.0-flash-lite-preview-02-05` - Ultra-fast, lighter version
- `gemini-2.0-pro-exp-02-05` - Experimental pro model
- `gemini-1.5-pro` - Previous generation, high quality

### LLM Parameters Explained

- **model**: Choose based on latency vs. capability tradeoff
- **temperature** (0.0-1.0): Higher = more creative, lower = more deterministic
- **max_output_tokens**: Limit response length for faster generation
- **top_p** (0.0-1.0): Nucleus sampling - controls diversity
- **top_k**: Alternative to top_p for sampling (not set by default)
- **presence_penalty**: Reduces repetition of concepts
- **frequency_penalty**: Reduces repetition of words
- **tool_choice**: "auto", "required", or "none" for function calling
- **thinking_config**: Enable chain-of-thought reasoning (experimental)
- **automatic_function_calling_config**: Auto-execute functions

## Google STT Configuration

Location: [src/agent.py:376-385](src/agent.py#L376-L385)

```python
stt=google.STT(
    languages="en-US",              # Primary language
    detect_language=False,          # Disable for speed
    interim_results=True,           # Enable for lower perceived latency
    punctuate=True,                 # Better transcription quality
    model="latest_long",            # Best quality for conversations
    sample_rate=16000,              # Standard telephony quality
    min_confidence_threshold=0.65,  # Balanced accuracy
    use_streaming=True,             # Critical for real-time performance
)
```

### Available STT Models

- `latest_long` - **RECOMMENDED** - Best for conversations
- `latest_short` - For short utterances
- `long` - Standard long-form
- `short` - Standard short-form
- `telephony` - Optimized for phone calls
- `medical_dictation` - Medical terminology
- `medical_conversation` - Medical conversations
- `chirp` - Universal Speech Model
- `chirp_2` - Updated USM

### STT Parameters Explained

- **languages**: Language code (e.g., "en-US", "hi-IN", "es-ES")
- **detect_language**: Auto-detect language (adds latency)
- **interim_results**: Stream partial results before final
- **punctuate**: Add punctuation to transcripts
- **spoken_punctuation**: Transcribe spoken punctuation marks
- **enable_word_time_offsets**: Get word-level timestamps
- **enable_word_confidence**: Get per-word confidence scores
- **model**: Choose based on use case
- **location**: "global" or specific region
- **sample_rate**: Audio sample rate (8000, 16000, 24000, 48000)
- **min_confidence_threshold**: Filter low-confidence results
- **keywords**: Boost recognition of specific words
- **use_streaming**: Enable streaming (essential for low latency)

### Supported Languages (170+ languages)

English variants: `en-US`, `en-GB`, `en-AU`, `en-CA`, `en-IN`, etc.
Other major languages: `hi-IN`, `es-ES`, `fr-FR`, `de-DE`, `ja-JP`, `zh-CN`, etc.

Full list: See [SpeechLanguages type](https://cloud.google.com/speech-to-text/docs/speech-to-text-supported-languages)

## Google TTS Configuration

Location: [src/agent.py:387-395](src/agent.py#L387-L395)

```python
tts=google.TTS(
    language="en-US",               # Match STT language
    gender="female",                # For "Riya" persona
    speaking_rate=1.1,              # Slightly faster for efficiency
    pitch=0,                        # Natural pitch
    sample_rate=24000,              # High quality audio
    use_streaming=True,             # Stream for lower latency
    volume_gain_db=2.0,             # Slight boost for clarity
)
```

### TTS Parameters Explained

- **language**: Language/accent (e.g., "en-US", "en-GB", "hi-IN")
- **gender**: "male", "female", "neutral"
- **voice_name**: Specific voice (e.g., "en-US-Neural2-C")
- **voice_cloning_key**: Custom voice clone key
- **speaking_rate** (0.25-4.0): Speed multiplier (1.0 = normal)
- **pitch** (-20.0 to 20.0): Pitch adjustment in semitones
- **sample_rate**: Output quality (8000, 16000, 24000, 48000)
- **volume_gain_db** (-96.0 to 16.0): Volume adjustment
- **effects_profile_id**: Audio effects (e.g., "telephony-class-application")
- **use_streaming**: Stream audio chunks (recommended)
- **enable_ssml**: Support SSML markup in text
- **custom_pronunciations**: Custom word pronunciations

### Voice Selection

To use a specific voice instead of gender:

```python
tts=google.TTS(
    voice_name="en-US-Neural2-F",  # High-quality female voice
    language="en-US",
    # gender is ignored when voice_name is set
)
```

Available voice types:
- **Standard voices**: Basic TTS
- **WaveNet voices**: Higher quality neural voices
- **Neural2 voices**: Latest generation, most natural
- **Studio voices**: Professional quality
- **News voices**: Optimized for news reading
- **Chirp voices**: Universal speech model

Browse voices: https://cloud.google.com/text-to-speech/docs/voices

## Function Calling with Gemini

The agent supports function calling (tool use) with Gemini. Example in [agent.py:333-345](src/agent.py#L333-L345):

```python
@function_tool
async def lookup_weather(self, context: RunContext, location: str):
    """Use this tool to look up current weather information.

    Args:
        location: The location to look up weather information for
    """
    logger.info(f"Looking up weather for {location}")
    return "sunny with a temperature of 70 degrees."
```

Gemini automatically:
- Detects when to call functions based on user input
- Parses function arguments
- Incorporates results into responses

## Performance Optimization Tips

### For Ultra-Low Latency

1. **Use Gemini 2.0 Flash Lite**:
   ```python
   model="gemini-2.0-flash-lite-preview-02-05"
   ```

2. **Reduce max_output_tokens**:
   ```python
   max_output_tokens=150  # Even shorter responses
   ```

3. **Increase speaking_rate**:
   ```python
   speaking_rate=1.2  # Faster speech
   ```

4. **Use shorter STT model**:
   ```python
   model="latest_short"  # For quick queries
   ```

### For Higher Quality

1. **Use Gemini 2.5 Pro**:
   ```python
   model="gemini-2.5-pro-preview-05-06"
   ```

2. **Increase max_output_tokens**:
   ```python
   max_output_tokens=500  # Longer, more detailed responses
   ```

3. **Use WaveNet or Neural2 voices**:
   ```python
   voice_name="en-US-Neural2-F"
   ```

4. **Higher sample rates**:
   ```python
   sample_rate=48000  # Studio quality
   ```

## Multilingual Support

To support multiple languages, enable language detection:

```python
stt=google.STT(
    languages=["en-US", "hi-IN", "es-ES"],  # Multiple languages
    detect_language=True,                    # Auto-detect
    # ... other settings
)
```

Match TTS language dynamically based on detected language.

## Cost Optimization

Gemini API pricing (as of 2025):
- **Gemini 2.0 Flash**: Very cost-effective for real-time
- **Google STT**: Pay per 15 seconds of audio
- **Google TTS**: Pay per character

Tips to reduce costs:
1. Use shorter max_output_tokens
2. Cache common responses
3. Use streaming efficiently
4. Consider Gemini Flash over Pro for most use cases

## Monitoring and Debugging

### Enable Debug Logging

```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

### Check Metrics

The agent collects metrics on:
- STT latency
- LLM generation time
- TTS synthesis time
- Total response time

Monitor in console logs during operation.

### Common Issues

1. **API Key Error**: Verify `GOOGLE_API_KEY` is correct
2. **Quota Exceeded**: Check Google Cloud quotas
3. **Network Latency**: Test with different regions
4. **Audio Quality**: Adjust sample_rate and use_streaming

## Testing the Configuration

Run the agent:

```bash
# Console mode (interactive testing)
python src/agent.py console

# Dev mode (with frontend)
python src/agent.py dev

# Production mode
python src/agent.py start
```

Test specific features:
- Ask questions → Tests LLM
- Speak naturally → Tests STT
- Listen to responses → Tests TTS
- Request weather → Tests function calling

## Advanced Features

### Thinking Mode (Experimental)

Enable chain-of-thought reasoning:

```python
llm=google.LLM(
    model="gemini-2.0-flash-001",
    thinking_config={
        "enabled": True,
        "max_thinking_tokens": 100,
    }
)
```

### Automatic Function Execution

Configure auto-execution of safe functions:

```python
llm=google.LLM(
    automatic_function_calling_config={
        "enabled": True,
        "max_iterations": 3,
    }
)
```

### Custom Voice Cloning

Use Chirp3 instant voice cloning:

```python
tts=google.TTS(
    voice_cloning_key="your-voice-clone-key",
    # Created at: https://cloud.google.com/text-to-speech/docs/chirp3-instant-custom-voice
)
```

## Resources

- [Gemini API Documentation](https://ai.google.dev/docs)
- [Google Cloud Speech-to-Text](https://cloud.google.com/speech-to-text/docs)
- [Google Cloud Text-to-Speech](https://cloud.google.com/text-to-speech/docs)
- [LiveKit Agents Documentation](https://docs.livekit.io/agents/)
- [Google AI Studio](https://aistudio.google.com/) - Test Gemini models

## Summary

Your agent is now configured with:
- ✅ Gemini 2.0 Flash LLM (ultra-fast)
- ✅ Google STT with streaming (low latency)
- ✅ Google TTS with female voice (high quality)
- ✅ Function calling enabled
- ✅ Optimized for real-time voice conversations
- ✅ Amazon customer care context

All settings are tuned for the best balance of speed, quality, and cost for a production voice AI agent.
