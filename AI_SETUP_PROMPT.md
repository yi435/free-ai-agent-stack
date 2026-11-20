# AI-Assisted Setup Prompt

Give the prompt below to an **already-installed AI coding agent** (such as [OpenCode](https://opencode.ai/) or [Hermes Agent](https://hermes-agent.nousresearch.com/)) to inspect your environment, configure [OmniRoute](https://github.com/diegosouzapw/OmniRoute) with the supported free providers, and connect your agent.

> [!IMPORTANT]
> **An AI coding agent must already be installed.** This prompt cannot run by itself. If you do not have an agent installed yet, install one first from **[OpenCode](https://opencode.ai/)** or **[Hermes Agent](https://hermes-agent.nousresearch.com/)**, or follow **[`docs/MANUAL_SETUP.md`](docs/MANUAL_SETUP.md)**.

---

## Copyable Prompt

```text
You are helping me set up the "Free AI Agent Stack" on my computer.

### Goal
Configure OmniRoute (https://github.com/diegosouzapw/OmniRoute) as a local OpenAI-compatible router and connect my installed AI coding agent to it using free model offerings from the supported providers, with priority routing and fallback where supported.

### Supported Providers (Strict Scope)
Only configure the providers I choose from this list:
1. NVIDIA NIM (https://build.nvidia.com/)
2. OpenCode Zen (https://opencode.ai/zen)
3. OpenRouter Free (https://openrouter.ai/ — only `:free` models)

Do not configure or suggest any other providers. If one of these three providers is unavailable or I choose to skip it, explain any impact and continue with the remaining supported providers.

### Expected Result
- OmniRoute is installed and running locally.
- My chosen providers from the three above are connected in OmniRoute.
- Currently available free models are discovered from the live catalog, tested, and organized into priority/fallback combo(s) in OmniRoute.
- My AI coding agent is configured to use the local OmniRoute endpoint without losing my existing settings.
- The setup is tested end to end, with a clear report of what works and what does not.

### Mandatory Rules
1. **Verify an AI coding agent is installed**: Check whether a compatible AI coding agent (such as OpenCode or Hermes Agent) is installed and whether you have local tool access. If no suitable agent is installed—or if you cannot inspect the local environment—stop and tell me to install an agent first from https://opencode.ai/ or https://hermes-agent.nousresearch.com/ (or to use `docs/MANUAL_SETUP.md`).
2. **Inspect first**: Before proposing changes, inspect my OS, installed dependencies, existing OmniRoute installation/version, running processes, and existing agent configuration files.
3. **Ask permission before making changes**: Explain your plan and get my explicit approval before installing software, starting services, or modifying any configuration file.
4. **Preserve existing data**: Always back up existing configuration files before editing them, and merge changes non-destructively.
5. **Verify against current documentation and live catalogs**: Consult official documentation and `--help` output for my installed tool versions. Never invent commands, configuration keys, endpoints, or model IDs. Discover free models from the live synced catalog and clearly distinguish tested working models from untested catalog entries.
6. **Protect API keys and secrets**:
   - Never ask me to paste API keys into the chat; guide me to enter provider keys directly in the local OmniRoute dashboard and store my local OmniRoute key in an environment variable.
   - Never print, echo, or log secret values in terminal output (check whether an environment variable is set without revealing its value).
   - Never hardcode plaintext API keys in configuration files.
7. **Privacy reminder**: Remind me that OmniRoute routes requests locally, but model inference runs on the remote servers of the selected providers, whose data policies vary.
8. **Test and report honestly**: Never claim a provider, model, combo, or agent configuration works before testing it. At the end, report what you found, what was installed or modified, which models were verified by live tests, which tests passed or failed, and anything that still needs my attention.

Start by inspecting my environment (read-only) and summarizing what is currently installed.
```
