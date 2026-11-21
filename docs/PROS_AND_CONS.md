# Pros and Cons

This page summarizes the main benefits, limitations, and privacy considerations of the **Free AI Agent Stack**.

---

## Advantages

* **$0 API cost for experimentation**: Students and developers can test AI coding agents without paying monthly subscriptions or per-token bills.
* **One endpoint for three providers**: Your coding agent connects once to OmniRoute while you manage **NVIDIA NIM**, **OpenCode Zen**, and **OpenRouter Free** in one dashboard.
* **Task-specific model lists**: You can create different combos in OmniRoute for coding, multi-step agent tasks, or quick questions.
* **Priority and fallback**: Ordering models across multiple providers reduces interruptions when a single free provider hits a temporary rate limit.
* **Easier catalog updates**: When a free model is added or retired, you update your combo in OmniRoute instead of editing every coding agent's config file.
* A useful project for learning how different model providers can work together through one router.

---

## Limitations

* **Strict usage quotas**: Free tiers limit how many requests you can send per minute or per day. Heavy coding sessions can still exhaust limits across all connected providers.
* **Changing model catalogs**: Providers can remove free models or change usage rules at any time without notice.
* **Capability differences**: Free models are useful for learning and everyday coding tasks, but they are not equivalent to top paid frontier models on complex debugging or large codebases.
* **Variable speed and reliability**: Shared free servers can be slow or overloaded during peak hours.
* **Inconsistent behavior on fallback**: Different models handle instructions and tool calls differently, so switching to a fallback model mid-session can change response quality.
* **Provider restrictions**: Some free models restrict which client tools can call them or require specific account privacy settings to function.
* **Setup and maintenance**: Managing a local router and multiple provider accounts takes more effort than using a single paid API key.

---

## Privacy and Security

OmniRoute runs on your computer, but the AI models run on external cloud servers.

* **Requests are processed remotely**: Whenever your coding agent sends a request, the prompt and any code context included in that request are transmitted to the selected provider (**NVIDIA NIM**, **OpenCode Zen**, or **OpenRouter Free**).
* **Data policies vary**: Providers have different rules for logging, data retention, and model training. Some free tiers require opting into prompt logging or data sharing, while others have separate developer terms. Neither zero data retention nor automatic training applies to all providers—check each provider's current policy and your account settings.
* **Protect sensitive code and credentials**: Do not use free cloud endpoints with confidential code, private user data, or secrets unless the provider's terms are acceptable for your work. Keep your local OmniRoute database, environment variables, and configuration files out of Git repositories.

---

## Who Should Use This Setup?

This stack is well suited for students, hobbyists, and developers working on personal projects or learning how AI coding agents work. If you work on confidential software, need consistent frontier-model performance, or want zero maintenance, paid APIs or local models running on your own hardware are a better fit.
