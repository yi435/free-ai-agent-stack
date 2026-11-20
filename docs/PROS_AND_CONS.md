# Pros and Cons

This page gives an honest look at the strengths and weaknesses of the **Free AI Agent Stack**. Read this before you start so you know what to expect.

---

## Advantages

### 1. Practice with AI Coding Agents for Free

AI coding agents send many requests during a single coding task. By using free offerings from **NVIDIA NIM**, **OpenCode Zen**, and **OpenRouter Free**, students and developers can practice using AI agents without paying monthly subscriptions or per-token API bills.

### 2. Connect Several Providers Through One Address

Without a router, switching from one provider to another means editing your coding agent's configuration files each time. With OmniRoute, your coding agent connects to one local address, and you manage all three providers in one place.

### 3. Use Different Models for Different Tasks

You can create separate model lists (combos) in OmniRoute for different needs—for example, one combo ordered for writing code and another ordered for quick questions.

### 4. Priority and Fallback Support

By putting models from three different providers into a priority list, you do not rely on a single free service. When one provider is busy or reaches a temporary limit, OmniRoute can try the next available model in your list where fallback is supported.

### 5. Easy to Update When Free Models Change

When a provider removes an old free model or adds a new one, you only need to update your model list inside OmniRoute instead of changing settings in every coding tool you use.

### 6. Learn How Real AI Routing Works

Setting up OmniRoute is a practical way to learn how OpenAI-compatible endpoints, API keys, model identifiers, and multi-provider routing work in real software projects.

---

## Disadvantages

### 1. Free Quotas Are Limited

Free accounts have strict limits on how many requests you can send per minute or per day. During a long coding session, it is still possible to hit usage limits across all of your connected free providers.

### 2. Free Models Can Change or Disappear

Providers regularly change which models are offered for free. A model that works today can be removed or switched to a paid tier without warning, which means you will occasionally need to update your model list.

### 3. Free Models Are Not Equal to Paid Frontier Models

Many free models are good at explaining code, writing small scripts, or helping with everyday bugs. However, they are often less reliable than top paid models on very large codebases, hard debugging problems, or long multi-step tool use.

### 4. Rate Limits and Slow Responses Can Interrupt Work

Because many people share free servers, responses can become slow during busy hours, or requests may fail when a server is overloaded.

### 5. Models Behave Differently When Fallback Happens

Each AI model has its own strengths and habits. If your first-choice model hits a limit and OmniRoute switches to a backup model mid-session, you may notice a change in how well the agent follows instructions or formats code changes.

### 6. Initial Setup and Ongoing Maintenance Take Time

Paying for one official API key is fast and rarely needs maintenance. Combining three free providers behind a local router takes more time to set up and requires occasional maintenance when models change.

### 7. Provider Rules and Restrictions

Some free models have extra rules. For example, a provider may restrict certain free models so they only work with specific coding clients, or require you to enable certain account settings before using free endpoints.

---

## Privacy and Security

> [!IMPORTANT]
> **Local Routing Is Not Local Model Processing**
>
> OmniRoute runs on your computer, but the AI models do **not** run on your computer. Every prompt, file, and code snippet your AI agent reads is sent over the internet to the selected provider (**NVIDIA NIM**, **OpenCode Zen**, or **OpenRouter Free**).

Keep these privacy and security points in mind:

* **Providers have different data policies**: Each provider has its own rules about whether prompts are logged, stored, or used to train future AI models. On services like OpenRouter, privacy rules can also depend on which company hosts the specific `:free` model you use.
* **Check your provider settings**: Some free model tiers require you to allow prompt logging or data sharing in order to use $0 models, while other providers have different developer terms. Do not assume that all providers train on your data, and do not assume that any free provider offers zero data storage unless you have checked its current policy.
* **Do not send secrets or private code**: Avoid using free cloud providers on confidential company code, private customer data, passwords, or `.env` files containing secret keys unless you have read and accepted the provider's data policy.
* **Keep your local credentials private**: OmniRoute stores your configuration and provider connection records in a local database file on your computer. Treat your local configuration files and environment variables carefully, and never upload them to GitHub.

---

## Conclusion: Who Is This Setup For?

* **This setup is a great choice if** you are a student, hobbyist, or developer who wants to learn how AI coding agents work, build personal or school projects, and experiment with multi-provider routing at $0 cost.
* **You may prefer paid APIs or local models (running on your own hardware) if** you work on confidential or commercial codebases, need fast and predictable quality every day, or do not want to spend time maintaining changing free model lists.
