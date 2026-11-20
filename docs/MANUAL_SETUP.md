# Manual Setup Guide

This guide explains how to configure the **Free AI Agent Stack** yourself.

Instead of giving you terminal commands to copy and paste without explanation, this page explains **what** needs to be installed, **where** each setting is configured, and **why** each part matters.

* If you want to understand the overall architecture first, read **[`docs/HOW_IT_WORKS.md`](HOW_IT_WORKS.md)**.
* If you already have an AI coding agent installed and want it to configure the setup with you, use **[`AI_SETUP_PROMPT.md`](../AI_SETUP_PROMPT.md)**.

---

## A. What Needs to Be Installed

Before configuring the routing stack, four things are needed:

### 1. Node.js and npm

OmniRoute is built with JavaScript and distributed through **npm** (the Node.js package manager). You need a current version of **Node.js** and **npm** installed on your computer whenever required by the current OmniRoute release. Check the [official OmniRoute repository](https://github.com/diegosouzapw/OmniRoute) for the exact Node.js version currently recommended.

### 2. OmniRoute

**[OmniRoute](https://github.com/diegosouzapw/OmniRoute)** is the local routing server. Once installed and started according to its official documentation, it runs on your computer (by default on port `20128`) and gives you:

* A **local Web Dashboard** (`http://127.0.0.1:20128`) where you manage providers, models, and combos in your browser.
* A **local OpenAI-compatible API endpoint** (`http://127.0.0.1:20128/v1`) where your AI coding agent sends requests.

### 3. An AI Coding Agent

You need at least one AI coding agent installed on your computer. The two examples demonstrated in this project are:

* **[OpenCode](https://opencode.ai/)**
* **[Hermes Agent](https://hermes-agent.nousresearch.com/)**

Follow the official website for your chosen agent to install it on your operating system.

### 4. Provider Accounts and API Keys

You need a free account and an **API key** from at least one—and ideally all three—of the supported model providers:

* **[NVIDIA NIM](https://build.nvidia.com/)**
* **[OpenCode Zen](https://opencode.ai/zen)**
* **[OpenRouter](https://openrouter.ai/)**

You do not need all three providers to start. Even if one provider is unavailable, you can configure OmniRoute with the remaining providers.

---

## B. Provider Configuration

In OmniRoute, you connect your upstream providers inside the **Providers** section of the local web dashboard (`http://127.0.0.1:20128`).

### What the API Keys Are Used For

There are two different kinds of API keys in this setup:

1. **Provider API keys** (from NVIDIA NIM, OpenCode Zen, and OpenRouter): Entered inside OmniRoute so OmniRoute has permission to call those cloud services on your behalf.
2. **A local OmniRoute API key**: Created inside OmniRoute's **API Keys** section and given to your AI coding agent so your agent can talk to OmniRoute on your computer.

### 1. Connecting NVIDIA NIM

* **Where to get an API key**: Sign in at [https://build.nvidia.com/](https://build.nvidia.com/), complete any required account verification, and generate an API key from your account settings or any hosted model page.
* **What to configure in OmniRoute**: Choose the **NVIDIA NIM** (`nvidia`) provider in the OmniRoute dashboard and enter your NVIDIA API key.
* **Model ID format**: Inside OmniRoute, models from NVIDIA NIM use the prefix `nvidia/` followed by the provider's model name (`nvidia/<model-name>`).
* **Making it available**: Use OmniRoute's test and sync option for that provider connection so OmniRoute checks your key and loads NVIDIA's current model list.

### 2. Connecting OpenCode Zen

* **Where to get an API key**: Sign in at [https://opencode.ai/zen](https://opencode.ai/zen) and create an API key in your Zen dashboard.
* **What to configure in OmniRoute**: Choose the **OpenCode Zen** (`opencode-zen`) provider in the OmniRoute dashboard and enter your Zen API key.
* **Model ID format**: Inside OmniRoute, models from OpenCode Zen use the prefix `opencode-zen/` followed by the model name (`opencode-zen/<model-name>`). Free models in Zen typically end with `-free`.
* **Making it available**: Test the connection and sync the provider catalog in OmniRoute so you can see which `-free` models are currently active.

### 3. Connecting OpenRouter Free

* **Where to get an API key**: Sign in at [https://openrouter.ai/](https://openrouter.ai/) and create a new key on your account's Keys page. Also check your account's Privacy Settings page, as some free models on OpenRouter require specific data-policy settings to be enabled before they will answer requests.
* **What to configure in OmniRoute**: Choose the **OpenRouter** (`openrouter`) provider in the OmniRoute dashboard and enter your OpenRouter API key.
* **Model ID format**: Inside OmniRoute, models from OpenRouter use the prefix `openrouter/` followed by the model name (`openrouter/<model-name>`). To stay at $0 inference cost, only choose models that end with the `:free` suffix.
* **Making it available**: Test the connection and sync the catalog in OmniRoute to pull the current list of `:free` models.

---

## C. Model Selection

### Finding Currently Available Models

Free model catalogs change often. Providers add new models, rename existing ones, or retire old free endpoints. Because of this, you should never copy a model ID from an old article without checking whether it still exists.

After you connect and sync your providers in OmniRoute, look at the **Models** list inside the OmniRoute dashboard (or inspect the live catalog returned by OmniRoute's `/v1/models` endpoint). Only pick models that appear in your live synced list and are clearly marked as free on your provider account.

### Tested Models vs. Listed Catalog Models

When selecting models, it helps to separate two things:

* **Listed models**: Any model that appears in a provider's catalog after you sync. Being listed in a catalog does not always mean the model will work for your coding agent right now—some listed models may be temporarily overloaded, restricted to specific clients, or require different account settings.
* **Tested models**: Models that you have actually sent a test message to through OmniRoute and confirmed that they return a valid answer.

For example, during the testing of this project's setup, several models on **NVIDIA NIM** (such as Nemotron, Mistral Large, and Gemma variants) and `:free` models on **OpenRouter** returned working responses, while some listed `-free` or `:free` models rotated out or returned client restriction errors over time. Always test a candidate model in your own OmniRoute dashboard before relying on it as your primary coding model.

---

## D. Priority and Fallback

Once you know which free models work on your connected providers, you can group them into a **Combo** inside OmniRoute's **Combos** section.

### What a Priority Combo Does

A combo gives a single name—such as `best-free-coder` or `best-free-agent`—to an ordered list of models. When you select the **Priority** strategy for a combo, OmniRoute always tries the models in the exact order you listed them.

### Simple Example of a Priority List

Suppose you create a combo named `best-free-coder` with three entries:

1. **First priority**: A tested coding/reasoning model from **NVIDIA NIM** (`nvidia/...`).
2. **Second priority**: A tested `:free` coding model from **OpenRouter** (`openrouter/...:free`).
3. **Third priority**: A tested `-free` model from **OpenCode Zen** (`opencode-zen/...-free`).

Under normal conditions, every coding request goes to your first priority on NVIDIA NIM. If that model becomes temporarily unavailable or hits a usage limit, OmniRoute's fallback feature can try your second priority on OpenRouter, and then your third priority on OpenCode Zen.

### Realistic Expectations for Fallback

Keep in mind that fallback is helpful, not magic. Whether OmniRoute switches to the next model depends on its current implementation, your configuration, and when the error happens. For example, if a failure occurs after a model has already started streaming an answer back to your agent, or if every model in your combo is rate-limited at the same time, fallback cannot prevent the error. Putting tested models from different providers near the top of your combo gives you the best chance of smooth fallback.

---

## E. AI Agent Configuration

After your priority combos are ready in OmniRoute, the next step is configuring your AI coding agent to send its requests to OmniRoute's local endpoint (`http://127.0.0.1:20128/v1`).

Before editing any existing configuration file for your coding agent, always save a backup copy of the file so you do not lose your previous settings.

### 1. Configuring OpenCode (`opencode.json`)

**[OpenCode](https://opencode.ai/)** stores its configuration in a local JSON file (`opencode.json` inside your user configuration directory). Check the official OpenCode documentation for the exact file location on your operating system.

Below is a small example showing the structure of the OmniRoute provider section inside `opencode.json`:

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

#### What Each Field Means

* **`$schema`**: Points to OpenCode's official configuration schema so editors can validate the file format.
* **`provider.omniroute`**: Creates a custom provider entry named `omniroute`.
* **`npm` (`"@ai-sdk/openai-compatible"`)**: Tells OpenCode to use its standard OpenAI-compatible adapter when talking to OmniRoute.
* **`options.baseURL`**: The local address where OmniRoute listens for OpenAI-compatible API requests (`http://127.0.0.1:20128/v1` by default).
* **`options.apiKey` (`"{env:OMNIROUTE_API_KEY}"`)**: Tells OpenCode to read your local OmniRoute API key from the `OMNIROUTE_API_KEY` environment variable instead of storing a secret key directly in the JSON file.
* **`models`**: Lists the OmniRoute combo names (or specific model IDs) that you want to appear inside OpenCode. Setting `"tool_call": true` tells OpenCode that the target supports agent tool calls (such as reading and editing files). You can also define optional token `limit` settings if required by your OpenCode version.
* **`model`**: Sets the default model (`omniroute/best-free-coder`) that OpenCode selects when it starts.

### 2. Configuring Hermes Agent (`config.yaml`)

**[Hermes Agent](https://hermes-agent.nousresearch.com/)** stores its settings in a local YAML file (`~/.hermes/config.yaml`).

Below is a small example showing the relevant sections in `config.yaml`:

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

#### What Each Field Means

* **`model.default`**: The name of the OmniRoute combo (or model) Hermes Agent should use by default.
* **`model.provider`**: Selects `omniroute` as the active provider.
* **`model.base_url`**: Points Hermes Agent to your local OmniRoute API endpoint (`http://127.0.0.1:20128/v1`).
* **`providers.omniroute.enabled`**: Turns on the OmniRoute provider integration.
* **`custom_providers`**: Registers the local OmniRoute endpoint and lists the combo names you created in OmniRoute so you can select them in Hermes Agent. Store your local API key in your environment or Hermes `.env` file rather than writing plaintext keys into `config.yaml`.

---

## F. Verification and Troubleshooting

### How to Verify Your Setup

Check your setup in three simple steps before starting a real coding task:

1. **Check that OmniRoute is running**: Open the local OmniRoute web dashboard (`http://127.0.0.1:20128`) in your browser. If the page loads, the local server is running.
2. **Check your providers and combos**: In the dashboard, confirm that your connected providers (`nvidia`, `opencode-zen`, `openrouter`) show a healthy status and that your priority combo contains only tested, currently available models.
3. **Send a small test message from your agent**: Start OpenCode or Hermes Agent, ask it a short one-line question, and then check the **Logs** tab in the OmniRoute dashboard. The log will show whether OmniRoute received the request from your agent and which upstream provider answered it.

### Common Configuration Mistakes

* **OmniRoute is not running (`Connection Refused`)**: If your coding agent says it cannot connect to `127.0.0.1:20128`, OmniRoute is either stopped or running on a different port than the one written in your agent's `baseURL` setting.
* **Missing or wrong API key (`Authentication / Unauthorized Error`)**:
  * If the error happens between your agent and OmniRoute, make sure the `OMNIROUTE_API_KEY` environment variable is set in the same terminal window where you started your agent. *(Never run commands that print your secret key onto the screen; only check whether the variable is set.)*
  * If the error comes from an upstream provider in OmniRoute's logs, re-check the provider API key you entered for NVIDIA NIM, OpenCode Zen, or OpenRouter in the OmniRoute dashboard.
* **Missing provider prefix or retired model (`Model Not Found`)**: Inside an OmniRoute combo, every model ID must include its provider prefix (such as `nvidia/...`, `opencode-zen/...`, or `openrouter/...`). If the prefix is correct and you still see a "not found" error, the provider has likely removed or renamed that free model—re-sync the provider in OmniRoute and pick a currently active model.
* **Account privacy or client restriction (`Forbidden Error`)**: Some free models on OpenRouter fail until you enable the required free-model data settings on OpenRouter's website, and some models on OpenCode Zen only accept requests from specific coding clients. If one model repeatedly fails with a permission error, remove it from your combo or adjust your account settings on that provider's website.
* **All models in the combo are busy (`Rate Limit Error`)**: It is normal to see occasional rate-limit warnings in OmniRoute's logs when one model is busy and OmniRoute switches to the next model. If your coding agent itself shows a final rate-limit failure, it means every model in your combo is rate-limited at the same time. Wait a moment for the limit to reset, or add verified free models from another supported provider to your combo.
