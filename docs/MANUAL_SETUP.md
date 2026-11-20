# Manual Setup Explanation

This guide explains how all the parts of the **Free AI Agent Stack** are configured manually.

Rather than giving you a script of terminal commands to copy and paste blindly, this document explains **what** needs to be installed, **where** each setting lives, and **why** each configuration section is structured the way it is. If you prefer an interactive setup performed by an already-installed coding agent, see [`AI_SETUP_PROMPT.md`](../AI_SETUP_PROMPT.md) and **[Getting Started](GETTING_STARTED.md)**. For a conceptual overview of the request flow, see **[How It Works](HOW_IT_WORKS.md)**.

---

## 1. Components You Need Installed

A complete working setup involves three layers on your computer:

1. **Node.js and npm**: The JavaScript runtime and package manager required to install and run OmniRoute. Check the [official OmniRoute repository](https://github.com/diegosouzapw/OmniRoute) for current Node.js version requirements.
2. **OmniRoute**: The local routing service. Once installed through npm and started, OmniRoute runs a local background server that provides two things on your machine (by default on port `20128`):
   * A **local Web Dashboard** (`http://127.0.0.1:20128`) where you manage provider connections, model catalogs, priority combos, local API keys, and request logs.
   * A **local OpenAI-compatible API endpoint** (`http://127.0.0.1:20128/v1`) that your AI coding agent connects to.
3. **An AI Coding Agent Harness**: At least one client tool—such as [OpenCode](https://opencode.ai/) or [Hermes Agent](https://hermes-agent.nousresearch.com/)—installed according to its official documentation.

OmniRoute stores its local state (registered providers, combos, settings, and logs) in a local SQLite database file inside your user profile directory (by default under `~/.omniroute/storage.sqlite` on Linux and macOS, or under your application data folder on Windows). Because this database holds your local configuration and provider connection records, it should remain private to your user account and must never be committed to version control.

---

## 2. Obtaining Provider API Keys

Before configuring OmniRoute, you need API credentials from one or more of the three providers in scope. Free usage limits vary by provider and can change over time, so check each provider's current documentation for the latest terms and quotas.

### NVIDIA NIM (`nvidia`)

* **Where to Register**: [https://build.nvidia.com/](https://build.nvidia.com/)
* **How Credentials Work**: After creating a free developer account and completing any required account verification, you can generate an API key from the model catalog or account settings page. NVIDIA NIM keys use Bearer authentication (typically prefixed with `nvapi-`) and connect to NVIDIA's hosted inference endpoint (`https://integrate.api.nvidia.com/v1/chat/completions`).

### OpenCode Zen (`opencode-zen`)

* **Where to Register**: [https://opencode.ai/zen](https://opencode.ai/zen)
* **How Credentials Work**: After signing in to the OpenCode Zen portal, you generate a Bearer API key from your Zen dashboard. OpenCode Zen exposes an OpenAI-compatible gateway (`https://opencode.ai/zen/v1`) that includes rotating free-tier models (typically ending in `-free`) alongside paid models.

### OpenRouter Free (`openrouter`)

* **Where to Register**: [https://openrouter.ai/](https://openrouter.ai/)
* **How Credentials Work**: After signing in, open your account's Keys page to create a Bearer API key (typically prefixed with `sk-or-`). OpenRouter serves free models identified by the `:free` suffix through its API (`https://openrouter.ai/api/v1/chat/completions`).
* **Important Privacy Setting**: Some free models on OpenRouter require enabling data-logging or training opt-in settings in your OpenRouter privacy preferences before requests will succeed. Always check your OpenRouter privacy settings so you understand what data handling applies to your account.

---

## 3. Connecting Providers and Creating a Local Key in OmniRoute

Configuration inside OmniRoute happens in two stages: connecting your upstream providers, and creating a local key for your coding agent.

### 3.1 Entering Upstream Provider Keys

In the OmniRoute Web Dashboard (`http://127.0.0.1:20128`), the **Providers** section lists supported upstream services. For each of the three providers you want to use:

1. Select the matching provider entry (`nvidia` for NVIDIA NIM, `opencode-zen` for OpenCode Zen, or `openrouter` for OpenRouter).
2. Enter the API key you obtained from that provider.
3. Run OmniRoute's built-in connection test and model sync action. Testing confirms that the upstream service accepts your API key, and syncing pulls the provider's current list of available model identifiers into OmniRoute.

### 3.2 Creating a Local OmniRoute API Key

Your AI coding agent should not use your NVIDIA, OpenCode Zen, or OpenRouter keys directly. Instead, open the **API Keys** section of the OmniRoute dashboard and generate a **local OmniRoute API key**.

Store this local key in an environment variable in your shell profile (commonly named `OMNIROUTE_API_KEY`) rather than typing it directly into your coding agent's configuration files. Referencing an environment variable keeps sensitive credentials out of plain configuration files.

---

## 4. Selecting and Verifying Models

As explained in **[How It Works](HOW_IT_WORKS.md)**, OmniRoute identifies every upstream model by combining the provider ID and the upstream model ID (`<provider-id>/<upstream-model-id>`).

Because free model catalogs change frequently, you should distinguish between three categories of models when planning your configuration:

### 4.1 Models Tested in This Project's Reference Setup

The following models were tested and confirmed to return successful responses (`HTTP 200`) through OmniRoute during the development of this project:

* **NVIDIA NIM (`nvidia`)**:
  * `nvidia/nvidia/nemotron-3-ultra-550b-a55b`
  * `nvidia/mistralai/mistral-large-4-1t-a52b`
  * `nvidia/nvidia/nemotron-3-nano-omni-30b-a3b-reasoning`
  * `nvidia/google/gemma-4-31b-it`
  * `nvidia/poolside/laguna-xs-2.1`
  * `nvidia/nvidia/nemotron-3.5-lightning-30b-a3b`
  * `nvidia/nvidia/nemotron-3.5-super-vl-120b-a12b`
  * `nvidia/stepfun-ai/step-5-preview`
* **OpenRouter Free (`openrouter`)**:
  * `openrouter/nvidia/nemotron-3-ultra-550b-a55b:free`
  * `openrouter/cohere/north-mini-code:free`
  * `openrouter/nvidia/nemotron-3-nano-omni-30b-a3b-reasoning:free`
  * `openrouter/nvidia/nemotron-3.5-lightning:free`
  * `openrouter/nvidia/nemotron-3.5-super-vl:free`

### 4.2 Listed or Rotating Catalog Models (Require Live Verification)

Some models appear in provider catalogs or earlier configurations but rotate frequently or enforce client-specific restrictions:

* **OpenCode Zen (`opencode-zen`)**: Models such as `opencode-zen/mimo-v2.5-free` (and other `-free` offerings listed in the Zen catalog) rotate periodically and may return authentication, quota, or harness-restriction errors depending on your account state and client headers.
* **OpenRouter Free (`openrouter`)**: Rotating `:free` entries such as `openrouter/dots-studio/dots-3-note-preview:free`, `openrouter/recursive/flint-v1.0:free`, or `openrouter/xingchen-agi/xing4.0-29b-a4b:free` have appeared in the catalog and returned responses in past tests, but may be retired (`404`) by their upstream hosts at any time.

### 4.3 Discovering Currently Available Models

Never assume a static list in documentation reflects today's catalog. Before adding any model to a routing combo, inspect the live models list in the OmniRoute dashboard (or via OmniRoute's `/v1/models` catalog endpoint) after syncing your connected providers, and only include models that are currently active and free on your accounts.

---

## 5. Organizing Priority and Fallback Combos

In OmniRoute's **Combos** section, a combo groups an ordered list of `<provider>/<model>` identifiers under a single virtual model name.

### 5.1 Structuring Your Combos

A clean way to organize your setup is to create one or more combos using the **Priority** strategy—for example:

* `best-free-coder`: Ordered for code generation and file editing.
* `best-free-agent`: Ordered for multi-step tool use and reasoning.
* `best-free-ai`: Ordered for general questions or lighter background tasks.

Within each priority combo:

1. **Place your most reliable, tested models at the top** (for example, a verified NVIDIA NIM model or a tested OpenRouter `:free` coding model).
2. **Alternate across providers where possible** so that if your primary provider reaches an account-wide rate limit, the next entry in the list routes to a different provider rather than failing on the same account.
3. **Keep unverified or retired models out of the list** so OmniRoute does not waste time trying models that no longer exist.

For details on what kinds of upstream errors trigger fallback and what kinds do not, refer to **[How It Works — Priority-Based Routing and Fallback](HOW_IT_WORKS.md#6-priority-based-routing-and-fallback)**.

---

## 6. Configuring Your AI Coding Agent Harness

Once OmniRoute is running and your priority combos exist, you configure your coding agent harness to treat OmniRoute as an OpenAI-compatible provider.

### 6.1 OpenCode Configuration (`opencode.json`)

[OpenCode](https://opencode.ai/) reads its provider and model settings from a JSON configuration file (`~/.config/opencode/opencode.json` on Linux, `~/Library/Application Support/opencode/opencode.json` on macOS, or `%APPDATA%\opencode\opencode.json` on Windows).

Here is an illustrative example showing how the OmniRoute section is structured inside `opencode.json`:

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
        },
        "best-free-agent": {
          "name": "Combo: Best Free Agent",
          "tool_call": true
        }
      }
    }
  },
  "model": "omniroute/best-free-coder"
}
```

#### What Each Section Does

* **`provider.omniroute.npm`**: Tells OpenCode to use the standard `@ai-sdk/openai-compatible` adapter to talk to OmniRoute.
* **`provider.omniroute.options.baseURL`**: Points OpenCode to your local OmniRoute `/v1` API endpoint.
* **`provider.omniroute.options.apiKey`**: Uses OpenCode's `{env:OMNIROUTE_API_KEY}` syntax to read your local OmniRoute key from the environment at runtime instead of storing a secret in the file.
* **`provider.omniroute.models`**: Registers the names of your OmniRoute priority combos (or specific models) so they appear in OpenCode's model selector. You can also include optional capability flags (such as `tool_call`, `reasoning`, or `attachment`) and token `limit` definitions (`context`, `input`, `output`) matching the models in your combo.
* **`model`**: Sets the default model or combo OpenCode selects on startup (`omniroute/<combo-name>`).

### 6.2 Hermes Agent Configuration (`config.yaml`)

[Hermes Agent](https://hermes-agent.nousresearch.com/) stores its configuration in `~/.hermes/config.yaml` (along with environment variables in `~/.hermes/.env`).

Here is a simplified snippet illustrating the relevant sections of `config.yaml`:

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
    models:
      - best-free-coder
      - best-free-agent
```

#### What Each Section Does

* **`model`**: Defines the default model identifier (`best-free-agent`), the active provider name (`omniroute`), and the local OmniRoute `base_url`.
* **`providers.omniroute`**: Enables the OmniRoute provider integration in Hermes Agent.
* **`custom_providers`**: Registers the local OmniRoute endpoint and lists the combo names you created in OmniRoute so you can switch between them inside Hermes Agent. Authentication is read from your environment or Hermes `.env` configuration rather than being committed in plaintext.

### 6.3 Other OpenAI-Compatible Agents

If you use another coding agent that supports custom OpenAI-compatible endpoints, look for its custom provider settings and supply the same three values: the local Base URL (`http://127.0.0.1:20128/v1`), the local OmniRoute API key (via environment variable where supported), and the combo name as the model identifier. Keep in mind that compatibility depends on the specific API parameters and tool-calling format the client requires.

---

## 7. Verifying That Everything Is Connected

After configuring your components, verify the setup in layers rather than starting a complex coding task immediately:

1. **Confirm OmniRoute Is Running**: Open the local OmniRoute dashboard (`http://127.0.0.1:20128`) or check OmniRoute's service status to confirm the local server is listening on port `20128`.
2. **Confirm Provider Health**: In the dashboard's **Providers** view, check that each of your connected providers (`nvidia`, `opencode-zen`, `openrouter`) passes its connection test and shows synced models.
3. **Test Each Priority Combo**: Use the test chat feature in the OmniRoute dashboard (or send a minimal one-line test prompt from your coding agent) targeting your combo name.
4. **Inspect the Request Log**: Open the **Logs** view in the OmniRoute dashboard after sending your test message. Confirm that the request arrived at OmniRoute, resolved your combo to an active upstream model, and returned a successful (`200`) response.

---

## 8. Common Configuration Mistakes and How to Understand Them

When something does not work on the first try, the error status in OmniRoute's logs almost always points to the exact layer that needs attention:

### Missing or Unloaded Environment Variable (`401 Unauthorized`)

If your coding agent reports an authentication failure when connecting to `127.0.0.1:20128/v1`, check two separate places:

* First, verify that the `OMNIROUTE_API_KEY` environment variable is actually set in the terminal session where you launched your coding agent. *(Security tip: check whether the variable is defined or check its character length—never run commands that print or echo the raw API key into your terminal log.)*
* Second, if OmniRoute itself returns `401` from an upstream provider, the upstream API key entered for NVIDIA NIM, OpenCode Zen, or OpenRouter is missing, invalid, or expired. Re-test that provider connection in the OmniRoute dashboard.

### Model Name Mismatch or Retired Free Model (`404 Not Found` / `410 Gone`)

This happens when a combo references a model identifier that no longer exists on the upstream provider, or when the provider prefix was omitted (for example, writing `cohere/north-mini-code:free` inside an OmniRoute combo instead of `openrouter/cohere/north-mini-code:free`). Re-sync the provider's model catalog in OmniRoute and replace any retired model IDs with currently listed ones.

### Client or Harness Restriction (`403 Forbidden`)

Some upstream free endpoints inspect request headers and restrict a model so it can only be called from specific approved coding clients, or require opting into provider privacy settings (such as OpenRouter's free-model data policy settings) before serving requests. If a model consistently returns `403`, check your account's privacy/terms settings on that provider or remove that model from general-purpose combos.

### All Models in a Combo Are Rate-Limited (`429 Too Many Requests`)

Seeing occasional `429` entries in OmniRoute's logs is normal when using free tiers—that is what triggers OmniRoute to try the next model in a priority combo. However, if your coding agent itself receives a final `429` error, it means **every** model in your combo is currently rate-limited or out of free capacity at the same time. Adding models from a second or third provider in scope (across NVIDIA NIM, OpenCode Zen, and OpenRouter Free) reduces the chance that all targets are cooling down simultaneously.

### Connection Refused on Localhost (`ECONNREFUSED`)

If your coding agent cannot connect to `127.0.0.1:20128/v1` at all, either the OmniRoute server process is not currently running, or it was started on a non-default port that does not match the `baseURL` in your agent's configuration file.
