# How It Works

This document explains how the components in **Free AI Agent Stack** work together.

---

## 1. The Role of OmniRoute

**[OmniRoute](https://github.com/diegosouzapw/OmniRoute)** runs locally on your computer as an OpenAI-compatible routing layer.

Instead of connecting your AI coding agent to one provider at a time, you connect your agent to OmniRoute's local endpoint (`http://127.0.0.1:20128/v1` by default). OmniRoute stores your provider connections, manages ordered model lists (**combos**), and forwards requests to the selected provider.

---

## 2. The Three Supported Providers

This setup uses free offerings from three providers:

* **[NVIDIA NIM](https://build.nvidia.com/)** (`nvidia`): Developer access to hosted AI models, subject to account terms and rate limits.
* **[OpenCode Zen](https://opencode.ai/zen)** (`opencode-zen`): A model gateway for coding agents that includes rotating free models (often ending in `-free`) alongside paid options.
* **[OpenRouter Free](https://openrouter.ai/)** (`openrouter`): An API aggregator where models ending in `:free` cost $0 per token, subject to daily and per-minute request limits.

Inside OmniRoute, each model is addressed as `<provider>/<model>` so the router knows which provider to call. Because providers update their free catalogs regularly, always choose models from OmniRoute's live synced model list rather than relying on fixed lists.

---

## 3. How Requests Flow

When you use an AI coding agent such as **[OpenCode](https://opencode.ai/)** or **[Hermes Agent](https://hermes-agent.nousresearch.com/)** with OmniRoute:

1. You give a task to your coding agent.
2. The agent sends an OpenAI-compatible request to OmniRoute on your computer, targeting a specific model or combo name.
3. OmniRoute selects an available provider and model and forwards the request over the internet.
4. The provider processes the request on its servers and returns the response through OmniRoute to your agent.

---

## 4. Priority and Fallback

In OmniRoute, a **Combo** lets you group multiple models from different providers under one virtual model name (for example, `best-free-coder`).

* **Priority**: You order the models in the combo from first choice to last choice. OmniRoute tries the highest-priority model that is currently available.
* **Fallback**: If the primary model cannot answer—for example, when a provider returns a rate-limit or temporary availability error before streaming starts—OmniRoute can temporarily cool down that option and try the next model in your combo.

Fallback depends on OmniRoute's current implementation, your configuration, and the type of failure. Not every error triggers fallback: invalid request parameters, mid-stream disconnects after tokens have already started streaming, or simultaneous rate limits across all models in your combo will still cause a request to fail.

---

## 5. Local Routing vs. Local Inference

* **Local routing** means the router (**OmniRoute**) runs on your computer and decides where requests go.
* **Local inference** means the AI model itself runs on your computer's hardware.

**This setup uses local routing, not local inference.** Prompts and any code context included in a request are sent to **NVIDIA NIM**, **OpenCode Zen**, or **OpenRouter Free** for processing. See **[`docs/PROS_AND_CONS.md`](PROS_AND_CONS.md)** for the trade-offs and privacy details.
