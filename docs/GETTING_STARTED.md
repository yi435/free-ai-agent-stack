# Getting Started

Before you can connect an AI coding agent to free model providers through OmniRoute, it helps to separate two distinct steps:

1. **Installing an AI coding agent** on your computer.
2. **Configuring OmniRoute** and connecting your agent to it.

This guide explains what you need in place before starting either the automated setup or the manual configuration.

---

## 1. What Is an AI Coding Agent?

In this project, an **AI coding agent** (often called a **harness**) is the client program you interact with in your terminal or editor.

An AI coding agent does not generate intelligence on its own. Instead, it acts as the workspace coordinator:

* It reads your prompt and inspects relevant files in your project.
* It sends requests to a language model API and receives structured responses.
* With your permission, it can run terminal commands, edit files, and check the results.

Because the agent is only the harness, it needs a model endpoint to talk to. In our setup, that endpoint will be **[OmniRoute](https://github.com/diegosouzapw/OmniRoute)**, which routes your agent's requests to free model offerings from **NVIDIA NIM**, **OpenCode Zen**, and **OpenRouter Free**.

---

## 2. Why You Need an Installed Agent Before Using `AI_SETUP_PROMPT.md`

This repository includes [`AI_SETUP_PROMPT.md`](../AI_SETUP_PROMPT.md), a reusable prompt designed to let an AI agent inspect your system, guide you through installing and configuring OmniRoute, and verify the connection.

> [!IMPORTANT]
> **The automatic setup prompt requires an already-installed AI coding agent.**
>
> A text prompt cannot run on its own. To use [`AI_SETUP_PROMPT.md`](../AI_SETUP_PROMPT.md), you must already have a working AI agent installed that can inspect files, execute terminal commands, and edit configuration files with your approval.

If you are starting from a fresh machine with no AI coding agent installed yet, you have two choices:

* **Option A**: Install an AI coding agent first (using its built-in free tier or a temporary provider key), and then give it [`AI_SETUP_PROMPT.md`](../AI_SETUP_PROMPT.md) to set up OmniRoute.
* **Option B**: Follow **[Manual Setup](MANUAL_SETUP.md)** to configure OmniRoute and your agent yourself without needing an already-working agent.

---

## 3. Demonstrated AI Coding Agents

This project demonstrates two AI coding agent harnesses:

### OpenCode

* **Official Website & Documentation**: [https://opencode.ai/](https://opencode.ai/)
* **OpenCode Zen Portal**: [https://opencode.ai/zen](https://opencode.ai/zen)
* **Overview**: OpenCode is an open-source terminal-based AI coding agent. It supports custom OpenAI-compatible providers through a local JSON configuration file and includes native access to OpenCode Zen, which offers selected free models alongside paid options.

### Hermes Agent

* **Official Website & Documentation**: [https://hermes-agent.nousresearch.com/](https://hermes-agent.nousresearch.com/)
* **Overview**: Developed by Nous Research, Hermes Agent is an autonomous CLI and gateway agent that stores its model and provider settings in a local YAML configuration file.

### What About Other AI Agents?

OpenCode and Hermes Agent are the examples documented in this repository, not the only tools that can talk to OmniRoute. Other coding tools that accept a custom OpenAI-compatible Base URL and API key may also work. However, compatibility depends on the exact API features, streaming format, and tool-calling behavior each client expects, so not every OpenAI-compatible agent is guaranteed to work without adjustments.

---

## 4. Prerequisites Checklist

Before moving on to the setup, check that you have:

1. **Supported Operating System**: Linux, macOS, or Windows (native or WSL2).
2. **Node.js and npm**: Required to install and run OmniRoute locally. Check the [OmniRoute repository](https://github.com/diegosouzapw/OmniRoute) for current runtime requirements.
3. **Provider Accounts**: You will need an account and API key from at least one—and ideally all three—of the providers used in this project:
   * **[NVIDIA NIM](https://build.nvidia.com/)**
   * **[OpenCode Zen](https://opencode.ai/zen)**
   * **[OpenRouter](https://openrouter.ai/)**
4. **An Installed AI Coding Agent** *(if using the automated prompt)*: Install [OpenCode](https://opencode.ai/), [Hermes Agent](https://hermes-agent.nousresearch.com/), or another agent capable of running local commands and editing configuration files with your confirmation.

---

## 5. Choosing Your Setup Path

### Path 1: AI-Assisted Setup (Using `AI_SETUP_PROMPT.md`)

1. Install your preferred AI coding agent using its official documentation.
2. Start the agent in your terminal or editor.
3. Open [`AI_SETUP_PROMPT.md`](../AI_SETUP_PROMPT.md), copy the prompt block, and paste it into your agent.
4. Answer the agent's questions and review every proposed action before approving it. The agent will inspect your environment, help you configure OmniRoute and your provider keys safely, and test the connection.

### Path 2: Manual Setup (Using `docs/MANUAL_SETUP.md`)

If you prefer to understand and configure each part yourself—or if you do not have a working AI agent yet—read:

1. **[How It Works](HOW_IT_WORKS.md)** to understand how requests flow from your agent through OmniRoute to the three providers.
2. **[Pros and Cons](PROS_AND_CONS.md)** to understand the trade-offs and privacy considerations.
3. **[Manual Setup](MANUAL_SETUP.md)** for a complete explanation of how to configure OmniRoute, your providers, priority combos, and your agent harness.
