# Anthropic API Proxy for Gemini & OpenAI Models 🔄

**Use Anthropic clients (like Claude Code) with Gemini or OpenAI backends.** 🤝

A proxy server that lets you use Anthropic clients with Gemini or OpenAI models via LiteLLM. 🌉

*This project is forked from: https://github.com/1rgs/claude-code-proxy*

## Quick Start ⚡

### Prerequisites

- OpenAI API key (if using OpenAI) 🔑
- Google AI Studio (Gemini) API key (if using AI Studio) 🔑
- Google Cloud authentication configured via `gcloud auth application-default login` (if using Vertex AI) ☁️
- [uv](https://github.com/astral-sh/uv) installed.

### Setup 🛠️

1. **Clone this repository**:
   ```bash
   git clone https://github.com/1rgs/claude-code-openai.git
   cd claude-code-openai
   ```

2. **Install uv** (if you haven't already):
   ```bash
   curl -LsSf https://astral.sh/uv/install.sh | sh
   ```
   *(`uv` will handle dependencies based on `pyproject.toml` when you run the server)*

3. **Configure Environment Variables**:
   Create a `.env` file with your API keys and model configurations:
   ```bash
   touch .env
   ```
   Edit `.env` and fill in your API keys and model configurations:

   *   `ANTHROPIC_API_KEY`: (Optional) Needed only if proxying *to* Anthropic models.
   *   `OPENAI_API_KEY`: Your OpenAI API key (Required if using OpenAI models).
   *   `GEMINI_API_KEY`: Your Google AI Studio (Gemini) API key (Required if using AI Studio Gemini models).
   *   `PREFERRED_PROVIDER`: Set to `google` (default), `openai`, or `vertex`. This determines the primary backend for mapping Anthropic models. If using `vertex`, ensure you have authenticated with `gcloud auth application-default login`.
   *   `BIG_MODEL`: The model to map `sonnet` requests to. Examples: `gpt-4o`, `gemini-2.5-pro-preview-03-25`.
   *   `SMALL_MODEL`: The model to map `haiku` requests to. Examples: `gpt-4o-mini`, `gemini-2.0-flash`.

   **Mapping Logic:**
   - The proxy maps `claude-3-haiku-...` to `SMALL_MODEL` and `claude-3-sonnet-...` to `BIG_MODEL`.
   - The appropriate provider prefix (`openai/`, `gemini/`, `vertex_ai/`) is added based on the chosen model and `PREFERRED_PROVIDER`.

4. **Run the server**:
   ```bash
   uv run uvicorn server:app --host 0.0.0.0 --port 8082 --reload
   ```
   *(`--reload` is optional, for development)*

### Using with Claude Code 🎮

1. **Install Claude Code** (if you haven't already):
   ```bash
   npm install -g @anthropic-ai/claude-code
   ```

2. **Connect to your proxy**:
   ```bash
   ANTHROPIC_BASE_URL=http://localhost:8082 claude
   ```

3. **That's it!** Your Claude Code client will now use the configured backend models through the proxy. 🎯

### Troubleshooting Connection Issues

Sometimes Claude may continue to use an external service (e.g., Vertex AI) instead of your local server. This happens when your credentials cache is still active.

To force Claude to use your local proxy server, explicitly override the auth token:

```bash
ANTHROPIC_BASE_URL=http://localhost:8082 ANTHROPIC_AUTH_TOKEN="some-api-key" claude
```

This command uses a dummy authentication token and ensures that Claude only connects to your local proxy.

## Model Mapping 🗺️

The proxy automatically maps Claude models to OpenAI or Gemini models based on your configuration:

| Anthropic Model Family | `PREFERRED_PROVIDER` | Mapped To       | Default Target Model                         | Prefix Added |
|------------------------|----------------------|-----------------|----------------------------------------------|--------------|
| `claude-3-haiku-...`   | `google` (default)   | `SMALL_MODEL`   | `gemini-2.0-flash`                           | `gemini/`    |
| `claude-3-sonnet-...`  | `google` (default)   | `BIG_MODEL`     | `gemini-2.5-pro-preview-03-25`               | `gemini/`    |
| `claude-3-haiku-...`   | `openai`             | `SMALL_MODEL`   | `gpt-4o-mini`                                | `openai/`    |
| `claude-3-sonnet-...`  | `openai`             | `BIG_MODEL`     | `gpt-4o`                                     | `openai/`    |
| `claude-3-haiku-...`   | `vertex`             | `SMALL_MODEL`   | `gemini-2.0-flash`                           | `vertex_ai/` |
| `claude-3-sonnet-...`  | `vertex`             | `BIG_MODEL`     | `gemini-2.5-pro-preview-03-25`               | `vertex_ai/` |

*You can override the default target models using the `BIG_MODEL` and `SMALL_MODEL` environment variables.*

### Supported Models

#### OpenAI Models
The following OpenAI models are supported with automatic `openai/` prefix handling:
- o3-mini
- o1
- o1-mini
- o1-pro
- gpt-4.5-preview
- gpt-4o
- gpt-4o-audio-preview
- chatgpt-4o-latest
- gpt-4o-mini
- gpt-4o-mini-audio-preview

#### Gemini Models
The following Gemini models are supported with automatic prefix handling:
- gemini-2.5-pro-preview-03-25
- gemini-2.5-pro-exp-03-25
- gemini-2.0-flash

### Customizing Model Mapping

You can customize which models are used via environment variables in your `.env` file:

```
# For OpenAI models
PREFERRED_PROVIDER=openai
OPENAI_API_KEY=sk-...
BIG_MODEL=gpt-4o
SMALL_MODEL=gpt-4o-mini

# For Gemini models (AI Studio)
# PREFERRED_PROVIDER=google
# GEMINI_API_KEY=your-ai-studio-key
# BIG_MODEL=gemini-2.5-pro-preview-03-25
# SMALL_MODEL=gemini-2.0-flash

# For Vertex AI models
# PREFERRED_PROVIDER=vertex
# GEMINI_PROVIDER=vertex_ai
# # Ensure gcloud auth is configured
# BIG_MODEL=gemini-2.5-pro-preview-03-25
# SMALL_MODEL=gemini-2.0-flash
```

Or set them directly when running the server:
```bash
# Using OpenAI models (with uv)
PREFERRED_PROVIDER=openai OPENAI_API_KEY=sk-... BIG_MODEL=gpt-4o SMALL_MODEL=gpt-4o-mini uv run uvicorn server:app --host 0.0.0.0 --port 8082
```

## How It Works 🧩

This proxy works by:

1. **Receiving requests** in Anthropic's API format 📥
2. **Translating** the requests to OpenAI/Gemini format via LiteLLM 🔄
3. **Sending** the translated request to the selected provider 📤
4. **Converting** the response back to Anthropic format 🔄
5. **Returning** the formatted response to the client ✅

The proxy handles both streaming and non-streaming responses, maintaining compatibility with all Claude clients. 🌊
