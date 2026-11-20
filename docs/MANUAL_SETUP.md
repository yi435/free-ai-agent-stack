# Manual Setup Guide

This guide is a configuration reference for setting up **Free AI Agent Stack** manually.

For architectural background, see **[`docs/HOW_IT_WORKS.md`](HOW_IT_WORKS.md)**. For agent-assisted setup, use **[`AI_SETUP_PROMPT.md`](../AI_SETUP_PROMPT.md)**.

---

## 1. Required Tools and Links

Install the following tools using their official documentation:

* **Node.js and npm**: Required to install and run OmniRoute (check the OmniRoute repo for current version requirements).
* **OmniRoute**: [https://github.com/diegosouzapw/OmniRoute](https://github.com/diegosouzapw/OmniRoute)
* **An AI Coding Agent** (demonstrated examples):
  * **OpenCode**: [https://opencode.ai/](https://opencode.ai/)
  * **Hermes Agent**: [https://hermes-agent.nousresearch.com/](https://hermes-agent.nousresearch.com/)

Create free accounts on one or more of the three supported model providers:

* **NVIDIA NIM**: [https://build.nvidia.com/](https://build.nvidia.com/)
* **OpenCode Zen**: [https://opencode.ai/zen](https://opencode.ai/zen)
* **OpenRouter**: [https://openrouter.ai/](https://openrouter.ai/)

---

## 2. Configuring API Keys in OmniRoute

When OmniRoute is running locally, open its Web Dashboard (`http://127.0.0.1:20128` by default). You will configure two types of keys:

1. **Provider API Keys** (**Providers** section in the dashboard):
   * **NVIDIA NIM** (`nvidia`): Generate an API key at [build.nvidia.com](https://build.nvidia.com/) and save it under the `nvidia` provider.
   * **OpenCode Zen** (`opencode-zen`): Generate an API key at [opencode.ai/zen](https://opencode.ai/zen) and save it under the `opencode-zen` provider.
   * **OpenRouter Free** (`openrouter`): Generate an API key at [openrouter.ai](https://openrouter.ai/) and save it under the `openrouter` provider. Check your OpenRouter privacy settings, as some `:free` models require data-policy opt-in to work.
   * After adding each provider key, run the **Test / Sync** action in the dashboard to verify the connection and fetch its current model catalog.
2. **Local OmniRoute API Key** (**API Keys** section in the dashboard):
   * Create a local key for your coding agent to authenticate with OmniRoute.
   * Store this key in an environment variable named `OMNIROUTE_API_KEY` in your shell profile rather than hardcoding it in config files.

---

## 3. Selecting Models

OmniRoute prefixes every model ID with its provider ID: `<provider>/<model>`.

* **Check live availability**: Free model catalogs and usage limits change often. After syncing your providers in OmniRoute, browse the live **Models** list in the dashboard (or the `/v1/models` endpoint) to see what is currently active.
* **Filter to free models**: On OpenRouter, select only models ending in `:free`. On OpenCode Zen, select only free models (typically ending in `-free`).
* **Test before relying on a model**: A model listed in a synced catalog may still be overloaded, restricted to specific clients, or retired upstream. Send a quick test message in the OmniRoute dashboard to confirm a model returns a valid response before adding it to your primary combo.

---

## 4. Configuring Priority and Fallback

In the **Combos** section of the OmniRoute dashboard, create a combo (such as `best-free-coder`) using the **Priority** strategy.

Add your verified working models in priority order—for example:

1. A tested model from **NVIDIA NIM** (`nvidia/...`)
2. A tested `:free` model from **OpenRouter** (`openrouter/...:free`)
3. A tested `-free` model from **OpenCode Zen** (`opencode-zen/...-free`)

Ordering models from different providers allows OmniRoute to try the next provider in the list when your first choice is rate-limited or temporarily unavailable.

---

## 5. Connecting an AI Coding Agent

Point your coding agent at OmniRoute's local OpenAI-compatible endpoint (`http://127.0.0.1:20128/v1`). Always back up your existing configuration file before editing it.

### OpenCode (`opencode.json`)

See the [OpenCode documentation](https://opencode.ai/) for your OS config path (typically `~/.config/opencode/opencode.json` on Linux). Add the `omniroute` provider using `@ai-sdk/openai-compatible` and reference your combo name:

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
          "tool_call": true
        }
      }
    }
  },
  "model": "omniroute/best-free-coder"
}
```

* `{env:OMNIROUTE_API_KEY}` loads your local router key from the environment at runtime.
* Add any optional model capability or token `limit` fields supported by your installed OpenCode version.

### Hermes Agent (`~/.hermes/config.yaml`)

See the [Hermes Agent documentation](https://hermes-agent.nousresearch.com/) for full configuration details. Point the `omniroute` provider to your local endpoint and list your combo names:

```yaml
model:
  default: best-free-coder
  provider: omniroute
  base_url: http://127.0.0.1:20128/v1

providers:
  omniroute:
    enabled: true

custom_providers:
  - name: OmniRoute (Local)
    base_url: http://127.0.0.1:20128/v1
    models:
      - best-free-coder
```

Keep your API key in your environment or Hermes `.env` file rather than hardcoding it in `config.yaml`.

---

## 6. Verification and Troubleshooting

To verify the setup:

1. Confirm the OmniRoute dashboard (`http://127.0.0.1:20128`) loads in your browser.
2. Send a short test prompt from your coding agent using your combo.
3. Check the **Logs** view in the OmniRoute dashboard to confirm the request arrived and was answered by one of your connected providers.

### Common Issues

* **Connection refused**: OmniRoute is not running, or its port does not match the `baseURL` in your agent config.
* **Unauthorized / authentication error**: Check that `OMNIROUTE_API_KEY` is set in the terminal where you start your agent (check variable presence without printing the secret value to the terminal), and re-test your upstream provider keys in the OmniRoute dashboard.
* **Model not found**: Ensure every model in your combo includes its provider prefix (`nvidia/...`, `opencode-zen/...`, `openrouter/...`). If a free model was retired upstream, re-sync the provider catalog and replace it.
* **Forbidden / permission error**: Some OpenRouter `:free` models require enabling free-model data settings on OpenRouter's website, and some OpenCode Zen models restrict which client tools may call them.
* **All models rate-limited**: If every model in a combo is rate-limited at once, wait for the cooldown to reset or add verified free models from another supported provider.
