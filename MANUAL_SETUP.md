# Manual Setup Guide

This guide walks you through installing [OmniRoute](https://github.com/diegosouzapw/OmniRoute), connecting free model offerings from the providers in scope, configuring priority fallback combos, and connecting AI coding agents such as [OpenCode](https://opencode.ai/) and [Hermes Agent](https://hermes-agent.nousresearch.com/).

> [!CAUTION]
> **Privacy & Security Reminder**: OmniRoute runs on your local machine (`127.0.0.1:20128`), but **model inference is remote**. Every prompt, file context, and tool output sent through this router is transmitted to the active upstream cloud provider. Never send passwords, API keys, `.env` secrets, customer PII, or confidential source code to free-tier endpoints without verifying each provider's current data-retention and training policies.

---

## Table of Contents

1. [Install and Start OmniRoute](#1-install-and-start-omniroute)
2. [Create Provider Accounts and Obtain Credentials](#2-create-provider-accounts-and-obtain-credentials)
3. [Connect Providers and Generate a Local Router Key](#3-connect-providers-and-generate-a-local-router-key)
4. [Select and Verify Free Models](#4-select-and-verify-free-models)
5. [Configure Priority Combos and Fallback](#5-configure-priority-combos-and-fallback)
6. [Connect an AI Coding Agent](#6-connect-an-ai-coding-agent)
7. [Verify Connectivity](#7-verify-connectivity)
8. [Troubleshooting](#8-troubleshooting)

---

## 1. Install and Start OmniRoute

### 1.1 Check Node.js Version

OmniRoute requires Node.js 20 LTS or newer (Node.js 22+ is recommended because it includes the built-in `node:sqlite` driver).

```bash
node --version
npm --version
```

### 1.2 Install OmniRoute

Install OmniRoute globally via `npm`:

```bash
npm install -g omniroute
```

> [!TIP]
> **npm 11+ Lifecycle Scripts**: Newer versions of `npm` (v11+) block native `postinstall` build scripts by default. If your installation skips native dependencies (`keytar`, `esbuild`, `@swc/core`), install with explicit script permissions:
>
> ```bash
> npm install -g --allow-scripts=omniroute,keytar,onnxruntime-node,@parcel/watcher,@swc/core,protobufjs,esbuild omniroute
> ```

**Alternative (Local Directory Install)**: If you prefer not to install global npm packages, install OmniRoute inside a dedicated directory (for example, `~/Applications/omniroute`):

```bash
mkdir -p ~/Applications/omniroute
cd ~/Applications/omniroute
npm install omniroute
```

### 1.3 Start the Router

Run OmniRoute in your terminal:

```bash
# Global install
omniroute

# Or headless without auto-opening a browser window
omniroute serve --no-open --port 20128
```

If you installed OmniRoute into `~/Applications/omniroute`, you can create a small startup script (`~/start_omniroute.sh`):

```bash
#!/usr/bin/env bash
cd "$HOME/Applications/omniroute" && ./node_modules/.bin/omniroute "$@"
```

Make it executable (`chmod +x ~/start_omniroute.sh`) and run `~/start_omniroute.sh`.

### 1.4 Default Endpoints and Data Locations

Once running, OmniRoute listens on `127.0.0.1:20128` by default:

| Resource | Default Location |
| :--- | :--- |
| **Web Dashboard** | `http://127.0.0.1:20128` |
| **OpenAI-Compatible Base URL** | `http://127.0.0.1:20128/v1` |
| **Models Catalog Endpoint** | `http://127.0.0.1:20128/v1/models` |
| **Chat Completions Endpoint** | `http://127.0.0.1:20128/v1/chat/completions` |
| **Local SQLite Database (Linux/macOS)** | `~/.omniroute/storage.sqlite` (or `$XDG_CONFIG_HOME/omniroute/storage.sqlite` if `~/.omniroute` does not exist; override via `DATA_DIR`) |
| **Local SQLite Database (Windows)** | `%APPDATA%\omniroute\storage.sqlite` |

> [!WARNING]
> Treat `~/.omniroute/storage.sqlite` like a credential vault—it stores your upstream provider keys and local OmniRoute API keys. Never commit `*.sqlite` files to Git.

---

## 2. Create Provider Accounts and Obtain Credentials

You do not need every provider to get started, but connecting at least two or three providers gives OmniRoute fallback options when one provider hits a rate limit.

### 2.1 NVIDIA NIM (`nvidia`)

* **Portal**: [https://build.nvidia.com/](https://build.nvidia.com/)
* **How the Free Offering Works**: NVIDIA provides free developer API access to hosted NIM endpoints, subject to developer rate limits (typically ~40 requests per minute) and account terms.
* **How to Get Your API Key**:
  1. Sign up or log in at [build.nvidia.com](https://build.nvidia.com/) and complete phone/email verification if prompted.
  2. Open any hosted model page and click **Get API Key** (or go to your account's API Keys settings).
  3. Generate a key beginning with `nvapi-` and copy it to a secure location.
* **Upstream Endpoint Used by OmniRoute**: `https://integrate.api.nvidia.com/v1/chat/completions`

### 2.2 OpenRouter Free (`openrouter`)

* **Portal**: [https://openrouter.ai/](https://openrouter.ai/)
* **How the Free Offering Works**: OpenRouter hosts a rotating catalog of models ending in `:free` that cost **$0/token**. Accounts with zero credit balance are subject to strict rate limits (commonly **20 requests/minute** and **200 requests/day** across `:free` models).
* **How to Get Your API Key**:
  1. Sign in at [openrouter.ai](https://openrouter.ai/) and navigate to **Keys** (`https://openrouter.ai/settings/keys`).
  2. Create a new key (begins with `sk-or-v1-...`).
  3. Check your **Privacy Settings** (`https://openrouter.ai/settings/privacy`). Note that some `:free` upstream providers on OpenRouter require prompt-logging/training opt-in to serve free requests; review those settings carefully before sending any code.
* **Upstream Endpoint Used by OmniRoute**: `https://openrouter.ai/api/v1/chat/completions`

### 2.3 OrcaRouter (`orcarouter`)

* **Portal**: [https://www.orcarouter.ai/](https://www.orcarouter.ai/)
* **How the Free Offering Works**: OrcaRouter exposes `$0/token` models suffixed with `-free` backed by a shared free capacity pool. Under high traffic, upstream servers frequently return `429` (`free_rate_limited: Free capacity is currently busy`), making it best suited as a secondary step in a priority combo.
* **How to Get Your API Key**:
  1. Create an account at [orcarouter.ai](https://www.orcarouter.ai/).
  2. Generate a Bearer API key (begins with `sk-orca-...`).
* **Upstream Endpoint Used by OmniRoute**: `https://api.orcarouter.ai/v1/chat/completions`

### 2.4 OpenCode Zen (`opencode-zen`)

* **Portal**: [https://opencode.ai/zen](https://opencode.ai/zen)
* **How the Free Offering Works**: OpenCode Zen provides a curated gateway for coding agents that includes rotating free-tier models (suffixed with `-free` at $0/token) alongside paid models. Free models enforce usage caps (`429 Free usage limit reached`) and rotate periodically.
* **How to Get Your API Key**:
  1. Sign in at [opencode.ai/zen](https://opencode.ai/zen).
  2. Create an API key from your Zen dashboard.
* **Upstream Endpoint Used by OmniRoute**: `https://opencode.ai/zen/v1`

### 2.5 HuggingChat (`huggingchat`) — Optional / Experimental

* **Portal**: [https://huggingface.co/chat/](https://huggingface.co/chat/)
* **How the Free Offering Works**: Unlike the four API-key providers above, OmniRoute's `huggingchat` integration talks to HuggingChat's web chat endpoint (`https://huggingface.co/chat/conversation`) using your browser session cookies.
* **How to Get Credentials**:
  1. Sign in to [huggingface.co/chat](https://huggingface.co/chat/) in your web browser and start a chat.
  2. Open Browser Developer Tools (`F12` $\rightarrow$ **Network** tab), send a message, and inspect the `POST /chat/conversation` request.
  3. Copy the request's `Cookie` header value (including `hf-chat=...` and any associated session/WAF tokens) and paste it as the credential for the `huggingchat` provider in OmniRoute.
* **Important Reliability Warning**: Because web session cookies expire and Hugging Face employs anti-bot challenges (AWS WAF), `huggingchat` connections frequently fail with `502 Bad Gateway` (`Received HTML instead of JSON`). Keep HuggingChat out of your primary automated combos unless you actively refresh its cookies.

---

## 3. Connect Providers and Generate a Local Router Key

### 3.1 Add Providers in the OmniRoute Dashboard

1. Open `http://127.0.0.1:20128` in your browser.
2. Navigate to **Providers**.
3. Add each provider you registered in Step 2 using its OmniRoute provider ID:
   * **NVIDIA NIM** (`nvidia`) $\rightarrow$ Paste your `nvapi-...` key.
   * **OpenRouter** (`openrouter`) $\rightarrow$ Paste your `sk-or-...` key.
   * **OrcaRouter** (`orcarouter`) $\rightarrow$ Paste your `sk-orca-...` key.
   * **OpenCode Zen** (`opencode-zen`) $\rightarrow$ Paste your Zen API key.
   * **HuggingChat** (`huggingchat`, optional) $\rightarrow$ Paste your browser `Cookie` string.
4. Click **Test / Sync** on each provider connection to verify authentication and fetch its live model catalog.

### 3.2 Create a Local OmniRoute API Key

Your coding agents authenticate to OmniRoute (`127.0.0.1:20128/v1`) using a local router key rather than your upstream provider keys.

1. In the OmniRoute dashboard (`http://127.0.0.1:20128`), open **API Keys** (or **Settings $\rightarrow$ API Keys**).
2. Generate a new local API key (typically prefixed with `sk-...` or `or-...`).
3. Export it in your shell profile (`~/.bashrc` or `~/.zshrc`) with restricted file permissions so it is never hardcoded in configuration files:

```bash
export OMNIROUTE_API_KEY="your-local-omniroute-key-here"
```

---

## 4. Select and Verify Free Models

In OmniRoute, every model is addressed using the format:

```text
<provider-id>/<upstream-model-id>
```

For example, NVIDIA's `nvidia/nemotron-3-ultra-550b-a55b` is referenced in OmniRoute as `nvidia/nvidia/nemotron-3-ultra-550b-a55b`, and OpenRouter's `cohere/north-mini-code:free` is referenced as `openrouter/cohere/north-mini-code:free`.

### 4.1 Verified Free Models Used in This Stack

The table below lists the verified models used in this stack across the active API providers:

| Provider ID | OmniRoute Model Identifier (`<provider>/<model>`) | Upstream Model ID | Notes |
| :--- | :--- | :--- | :--- |
| `nvidia` | `nvidia/nvidia/nemotron-3-ultra-550b-a55b` | `nvidia/nemotron-3-ultra-550b-a55b` | High-capacity reasoning & coding |
| `nvidia` | `nvidia/mistralai/mistral-large-4-1t-a52b` | `mistralai/mistral-large-4-1t-a52b` | General & agentic tasks |
| `nvidia` | `nvidia/nvidia/nemotron-3-nano-omni-30b-a3b-reasoning` | `nvidia/nemotron-3-nano-omni-30b-a3b-reasoning` | Fast reasoning fallback |
| `nvidia` | `nvidia/google/gemma-4-31b-it` | `google/gemma-4-31b-it` | Instruction-following fallback |
| `nvidia` | `nvidia/poolside/laguna-xs-2.1` | `poolside/laguna-xs-2.1` | Code-focused model |
| `nvidia` | `nvidia/nvidia/nemotron-3.5-lightning-30b-a3b` | `nvidia/nemotron-3.5-lightning-30b-a3b` | Low-latency MoE |
| `nvidia` | `nvidia/nvidia/nemotron-3.5-super-vl-120b-a12b` | `nvidia/nemotron-3.5-super-vl-120b-a12b` | Multimodal / general |
| `nvidia` | `nvidia/stepfun-ai/step-5-preview` | `stepfun-ai/step-5-preview` | General preview model |
| `openrouter` | `openrouter/nvidia/nemotron-3-ultra-550b-a55b:free` | `nvidia/nemotron-3-ultra-550b-a55b:free` | `$0/token` `:free` variant |
| `openrouter` | `openrouter/cohere/north-mini-code:free` | `cohere/north-mini-code:free` | `$0/token` coding model |
| `openrouter` | `openrouter/nvidia/nemotron-3-nano-omni-30b-a3b-reasoning:free` | `nvidia/nemotron-3-nano-omni-30b-a3b-reasoning:free` | `$0/token` reasoning |
| `openrouter` | `openrouter/dots-studio/dots-3-note-preview:free` | `dots-studio/dots-3-note-preview:free` | `$0/token` preview |
| `openrouter` | `openrouter/nvidia/nemotron-3.5-lightning:free` | `nvidia/nemotron-3.5-lightning:free` | `$0/token` fast MoE |
| `openrouter` | `openrouter/nvidia/nemotron-3.5-super-vl:free` | `nvidia/nemotron-3.5-super-vl:free` | `$0/token` multimodal |
| `openrouter` | `openrouter/recursive/flint-v1.0:free` | `recursive/flint-v1.0:free` | `$0/token` general/code |
| `openrouter` | `openrouter/xingchen-agi/xing4.0-29b-a4b:free` | `xingchen-agi/xing4.0-29b-a4b:free` | `$0/token` MoE |
| `orcarouter` | `orcarouter/tencent/hy4-preview-free` | `tencent/hy4-preview-free` | Shared-pool `-free` tier |
| `opencode-zen` | `opencode-zen/mimo-v2.5-free` | `mimo-v2.5-free` | Rotating Zen `-free` tier |

### 4.2 Always Verify Live Catalog Availability

Because free models rotate frequently, verify which models are currently live on your router before creating combos:

```bash
curl -s http://127.0.0.1:20128/v1/models \
  -H "Authorization: Bearer $OMNIROUTE_API_KEY" | grep '"id"'
```

If an upstream provider has retired a model, omit it from your combos and replace it with a currently listed free model from that provider.

---

## 5. Configure Priority Combos and Fallback

A **Combo** in OmniRoute is a virtual model name (such as `best-free-coder`) backed by an ordered list of `provider/model` targets. When your coding agent requests `"model": "best-free-coder"`, OmniRoute tries the first model in the list and advances down the chain if an eligible upstream error occurs.

### 5.1 Create Priority Combos via CLI (or Dashboard)

You can create combos in the Dashboard (**Combos $\rightarrow$ Create Combo**, Strategy: `priority`) or directly from your terminal using `omniroute combo create`:

```bash
# 1. Coding-focused priority combo
omniroute combo create best-free-coder \
  --strategy priority \
  --models "nvidia/nvidia/nemotron-3-ultra-550b-a55b,openrouter/cohere/north-mini-code:free,nvidia/mistralai/mistral-large-4-1t-a52b,openrouter/nvidia/nemotron-3-ultra-550b-a55b:free,opencode-zen/mimo-v2.5-free,nvidia/poolside/laguna-xs-2.1,openrouter/recursive/flint-v1.0:free,orcarouter/tencent/hy4-preview-free"

# 2. Autonomous multi-step agent priority combo
omniroute combo create best-free-agent \
  --strategy priority \
  --models "nvidia/nvidia/nemotron-3-ultra-550b-a55b,nvidia/mistralai/mistral-large-4-1t-a52b,openrouter/nvidia/nemotron-3-ultra-550b-a55b:free,openrouter/cohere/north-mini-code:free,nvidia/nvidia/nemotron-3-nano-omni-30b-a3b-reasoning,openrouter/nvidia/nemotron-3-nano-omni-30b-a3b-reasoning:free,opencode-zen/mimo-v2.5-free,orcarouter/tencent/hy4-preview-free,nvidia/google/gemma-4-31b-it"

# 3. General-purpose free priority combo
omniroute combo create best-free-ai \
  --strategy priority \
  --models "nvidia/nvidia/nemotron-3-ultra-550b-a55b,nvidia/mistralai/mistral-large-4-1t-a52b,openrouter/nvidia/nemotron-3-ultra-550b-a55b:free,orcarouter/tencent/hy4-preview-free,nvidia/nvidia/nemotron-3-nano-omni-30b-a3b-reasoning,openrouter/nvidia/nemotron-3-nano-omni-30b-a3b-reasoning:free,opencode-zen/mimo-v2.5-free,nvidia/google/gemma-4-31b-it,openrouter/dots-studio/dots-3-note-preview:free"
```

Verify your configured combos at any time with:

```bash
omniroute combo list
```

### 5.2 How Priority Routing and Fallback Work

Understanding what triggers fallback prevents surprises during long coding sessions:

1. **Errors That Trigger Fallback**:
   * **HTTP `429` (Rate Limit / Free Capacity Busy)**: Temporarily cools down the rate-limited model/account and tries the next model in the combo.
   * **HTTP `402` (Payment Required / Quota Exhausted)**: Marks that specific model as out of quota for its cooldown window and advances to the next model.
   * **HTTP `500`, `502`, `503`, `504` (Transient Server Errors)** and **NVIDIA NIM `DEGRADED` state**: Backs off and advances to the next healthy model.
   * **HTTP `404` / `410` (Model Not Found / Shut Down)**: Because `nvidia`, `openrouter`, `orcarouter`, and `opencode-zen` are configured in OmniRoute as multi-model catalog providers (`passthroughModels: true`), a `404` or `402` on one model locks out **only that specific model** rather than disabling your entire provider account.
2. **Errors That Do Not Automatically Fall Back**:
   * **Client `400 Bad Request`**: If a request contains malformed JSON or parameters rejected before routing, it may return immediately to the client.
   * **Mid-Stream Interruptions**: Once an upstream model begins streaming SSE tokens back to your coding agent, a mid-generation disconnect cannot be transparently re-run on another model without the agent retrying the turn.

---

## 6. Connect an AI Coding Agent

OmniRoute exposes a standard OpenAI-compatible `/v1` API, so you can connect any compatible coding agent. Below are the configurations for **OpenCode** and **Hermes Agent**, which are two harnesses used by the project author, followed by generic settings for other tools.

### 6.1 Option A: OpenCode (`https://opencode.ai/`)

#### Automated Setup via OmniRoute CLI

OmniRoute includes a built-in helper that writes `~/.config/opencode/opencode.json` for you:

```bash
omniroute setup-opencode --port 20128
```

#### Manual Configuration (`~/.config/opencode/opencode.json`)

If you prefer full control over which combos and models appear in OpenCode, create or edit `~/.config/opencode/opencode.json` (on macOS: `~/Library/Application Support/opencode/opencode.json`; on Windows: `%APPDATA%\opencode\opencode.json`):

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "omniroute": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "OmniRoute (Local)",
      "options": {
        "baseURL": "http://127.0.0.1:20128/v1",
        "apiKey": "{env:OMNIROUTE_API_KEY}"
      },
      "models": {
        "best-free-coder": {
          "name": "Combo: Best Free Coder",
          "attachment": true,
          "reasoning": true,
          "tool_call": true,
          "limit": {
            "context": 131072,
            "input": 131072,
            "output": 8192
          }
        },
        "best-free-agent": {
          "name": "Combo: Best Free Agent",
          "attachment": true,
          "reasoning": true,
          "tool_call": true,
          "limit": {
            "context": 131072,
            "input": 131072,
            "output": 8192
          }
        },
        "best-free-ai": {
          "name": "Combo: Best Free AI",
          "attachment": true,
          "reasoning": true,
          "tool_call": true,
          "limit": {
            "context": 131072,
            "input": 131072,
            "output": 8192
          }
        }
      }
    }
  },
  "model": "omniroute/best-free-coder",
  "small_model": "omniroute/best-free-ai"
}
```

> [!IMPORTANT]
> * Keep `"apiKey": "{env:OMNIROUTE_API_KEY}"` so your key is read from your environment at runtime and never stored in plaintext inside `opencode.json`.
> * Always include the `"limit"` block (`context`, `input`, `output`) on custom models in `opencode.json` so OpenCode can calculate context compaction accurately.

### 6.2 Option B: Hermes Agent (`https://hermes-agent.nousresearch.com/`)

#### Install Hermes Agent

Follow the official installation command from [hermes-agent.nousresearch.com](https://hermes-agent.nousresearch.com/):

```bash
# Linux / macOS / WSL2
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

#### Configure `~/.hermes/config.yaml`

Point Hermes Agent at your local OmniRoute endpoint by editing `~/.hermes/config.yaml`:

```yaml
model:
  default: best-free-agent
  provider: omniroute
  base_url: http://127.0.0.1:20128/v1

providers:
  omniroute:
    enabled: true

custom_providers:
  - name: OmniRoute (Local)
    base_url: http://127.0.0.1:20128/v1
    api_key: "" # Or pass via environment / ~/.hermes/.env
    models:
      - best-free-coder
      - best-free-agent
      - best-free-ai
```

Restrict permissions on your Hermes configuration directory:

```bash
chmod 600 ~/.hermes/config.yaml
```

### 6.3 Other OpenAI-Compatible Coding Harnesses

Any other tool that accepts an OpenAI-compatible API endpoint (such as Aider, Cline, Continue, Roo Code, or Goose) can connect using these three parameters:

* **API Provider Type**: `OpenAI Compatible`
* **Base URL**: `http://127.0.0.1:20128/v1`
* **API Key**: Your `$OMNIROUTE_API_KEY` value
* **Model ID**: `best-free-coder`, `best-free-agent`, or `best-free-ai`

---

## 7. Verify Connectivity

Before launching your coding agent, run these three checks in a terminal to confirm the stack works end to end.

### 7.1 Check Router Status

```bash
omniroute status
omniroute doctor
```

### 7.2 Test the `/v1/models` Endpoint

```bash
curl -sS http://127.0.0.1:20128/v1/models \
  -H "Authorization: Bearer $OMNIROUTE_API_KEY" | head -n 30
```

### 7.3 Send a Test Chat Completion Through a Combo

```bash
curl -sS http://127.0.0.1:20128/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OMNIROUTE_API_KEY" \
  -d '{
    "model": "best-free-coder",
    "messages": [
      {"role": "user", "content": "Reply with the single word READY."}
    ],
    "max_tokens": 16
  }'
```

If you receive a valid JSON completion response, inspect the **Logs** tab in the OmniRoute dashboard (`http://127.0.0.1:20128`) to see which upstream provider and model fulfilled the request.

---

## 8. Troubleshooting

| HTTP Code / Symptom | Typical Upstream Message | Root Cause | Fix |
| :--- | :--- | :--- | :--- |
| **`401 Unauthorized`** | `No credentials for provider: ...` or `Invalid API key` | Missing/expired upstream API key in OmniRoute, or missing `OMNIROUTE_API_KEY` in your agent's environment. | Verify `echo $OMNIROUTE_API_KEY` in your shell and re-test the upstream provider key in the OmniRoute Dashboard. |
| **`403 Forbidden`** | `This model is only available in approved coding harnesses` | Certain upstream free endpoints inspect client headers or restrict which harnesses may call a specific model. | Remove that model from generic combos or place another provider (`nvidia/...` or `openrouter/...:free`) ahead of it. |
| **`404 Not Found`** or **`410 Gone`** | `Not found: model '...' not found` or `Model is shut down` | The upstream provider renamed or retired a free model. | Re-sync the provider in the OmniRoute Dashboard, inspect `GET /v1/models`, and replace the retired model ID in your combo. |
| **`429 Too Many Requests`** | `free_rate_limited: Free capacity is currently busy` / `Free usage limit reached` | You hit the provider's RPM/RPD limit or shared free pool capacity limit. | Normal on free tiers. Ensure your combo contains models across **multiple distinct providers** (`nvidia`, `openrouter`, `opencode-zen`, `orcarouter`) so OmniRoute can fall back across providers. |
| **`502 Bad Gateway`** (on `huggingchat`) | `HuggingChat error: Received HTML instead of JSON` | Browser session cookie (`hf-chat`) expired or triggered a Hugging Face WAF/CAPTCHA challenge. | Remove `huggingchat` models from automated priority combos, or refresh the `Cookie` header from a live browser session. |
| **`ECONNREFUSED 127.0.0.1:20128`** | Connection refused | OmniRoute is not running or is listening on a different port. | Run `omniroute status` or start the router with `omniroute serve --port 20128`. |
