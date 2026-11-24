# AI Insights (advisory)

**A11y: Generate AI Insights (advisory)** sends a redacted summary of an audit run to an AI model you choose: with your own Anthropic or OpenAI API key, or through GitHub Copilot with no key. The model explains the automated findings: the likely shared root cause, who is affected, what to fix first and how to verify the fix. The result appears in the A11y Results view and as an extra section in the HTML report.

AI Insights are **advisory**. They never change a finding, a severity, a WCAG mapping, a checklist rating or the VPAT. Every recommendation is marked for human review. Nothing in the section is a conformance claim.

- [What it does](#what-it-does)
- [Setup](#setup)
- [GitHub Copilot](#github-copilot)
- [Running it](#running-it)
- [What is sent, and what never is](#what-is-sent-and-what-never-is)
- [Cost and limits](#cost-and-limits)
- [Partial results and retries](#partial-results-and-retries)
- [Where results are kept](#where-results-are-kept)
- [Settings](#settings)
- [Troubleshooting](#troubleshooting)
- [Limitations](#limitations)

## What it does

1. **Groups findings into clusters.** Findings with the same rule, criteria, component (or normalized selector), state and viewport form one cluster, across pages. A logo without `alt` on 40 pages is one cluster, not 40 findings. Clusters are ordered most severe first.
2. **Impact.** For each cluster, the model gives a root-cause hypothesis, the affected user groups, the task impact, why the severity matters, and a confidence. It must cite the evidence IDs it used.
3. **Remediation.** For each cluster, the model gives a recommended action, verification steps and a confidence. With `a11y.ai.allowCodeFixSuggestions` on, it may also suggest a code change, but only where the evidence includes the element's whole HTML.
4. **Summary.** One more call turns the cluster analyses into an executive summary and a prioritized work list. This call gets the analyses only, no page evidence.

Each answer is checked before it is accepted:

- It must match a strict JSON schema.
- It may cite only evidence that was sent.
- It may not contain a status field.
- It may not contain anything that looks like a credential.
- Its confidence must be backed by evidence.

An answer that fails these checks fails only its own cluster, which is then listed under **Not analyzed (needs human review)**.

The severity shown next to each cluster always comes from the deterministic audit, never from the model.

## Setup

1. Choose a provider: **Settings**, search `a11y.ai.provider`, pick `anthropic` or `openai`. For `copilot`, see [GitHub Copilot](#github-copilot): it needs no key and no model name.
2. Enter a model name in `a11y.ai.model`, spelled the way the provider spells it. There is no default: you pick the model you pay for. Use a model that supports structured outputs (JSON schema).
3. Run **A11y: Set AI API Key**, pick the same provider and paste the key.
   - The key is stored in VS Code SecretStorage (OS keychain), never in a file, a setting or the workspace.
   - It is sent only to the provider's fixed endpoint: `https://api.anthropic.com/v1/messages` or `https://api.openai.com/v1/responses`. The endpoint cannot be changed.
   - If you save a key for a provider other than the one in `a11y.ai.provider`, the command offers to switch the setting.
4. Optional: set your prices (`a11y.ai.inputPricePerMTok`, `a11y.ai.outputPricePerMTok`) to see dollar estimates and enforce the cost limit.

All `a11y.ai.*` settings are application-scoped: they live in your user settings, and a workspace or a repository you open cannot change them.

**A11y: Clear AI API Keys** deletes the keys for both providers.

## GitHub Copilot

With a GitHub Copilot plan you can use AI Insights without an API key. Requests go through the VS Code Language Model API (`vscode.lm`) to the models your Copilot plan offers.

1. Install **GitHub Copilot Chat** and sign in. Recent VS Code versions include it; sign in from the Accounts menu or the Chat view. A11y Audit does not depend on it; without it the other providers work as before.
2. Set `a11y.ai.provider` to `copilot`. Leave `a11y.ai.model` empty.
3. Run **A11y: Generate AI Insights (advisory)**. The first time, a list of the Copilot models available to you opens (name, family, input limit). The model you pick is saved in `a11y.ai.model`. If that model later disappears, the list opens again.
4. On the first request, VS Code asks whether A11y Audit may use the language model. Allow it. You can withdraw it later with **Manage Language Model Access…** in the Accounts menu.

### Trying it out (for testers)

1. Run **A11y: Run Audit** on a test site, or open a workspace that already has results in `.a11y/results/`.
2. In **Settings**, set `a11y.ai.provider` to `copilot`. Optionally turn on `a11y.ai.usageLog` to see the tokens per run in `.a11y/ai/usage.jsonl`.
3. Run **A11y: Generate AI Insights (advisory)** from the Command Palette or the ✨ button in the A11y Results view.
4. Pick a model, check the dialog (clusters, number of requests, privacy mode) and click **Send**. Allow model access when VS Code asks.
5. When it finishes, click **Export Report**. The AI part is section 18 of the report.

If something does not work:

- **"No GitHub Copilot models are available"** right after VS Code starts: open the Chat view once, so Copilot finishes loading, then run the command again.
- **The run ends as partial with `budget_exhausted`**: larger audits can need more than the default 10,000 output tokens. A run of 25 clusters used about 9,800. Raise `a11y.ai.maxOutputTokens`, then **Retry failed clusters**.
- **Small models** such as `gpt-4o-mini` accept about 12,000 input tokens per request and tend to report `high` confidence everywhere. A model with a larger input limit usually gives more careful answers.

When you report a problem, include the message text, the model and the line from `usage.jsonl`. Do not send the `.a11y/` folder: it contains page content from the audited site.

What is different from the API-key providers:

- **Same redaction.** Copilot receives the same redacted evidence as Anthropic and OpenAI. Nothing extra is sent.
- **Requests, not dollars.** The dialog shows no cost. It shows how many requests the run sends, for example `about 7 (up to 67 if batches must be split)` for 30 clusters. Depending on the model and your plan, each request may count as a premium request.
- **Tokens counted locally.** The Language Model API does not report usage. The token limits use the model's own token counter on the prompt and the answer instead.
- **The output limit is checked between requests.** Copilot cannot cap one answer, so a single answer may go a little past `a11y.ai.maxOutputTokens`. The next request is then not sent.
- **No effort, no JSON schema mode.** `a11y.ai.effort` and the prices are ignored. The JSON schema is part of the prompt, and every answer goes through the same local checks; an answer that is not valid JSON fails only its cluster.
- **No automatic retries on errors.** A failed request is not retried, because each attempt may count as a premium request. Use **Retry failed clusters**. Only a Copilot rate limit is waited out, at most twice and for at most a minute, because a rate-limited request is not served.
- **Your plan and your organization decide.** What Copilot keeps and which models you may use are set by your Copilot plan and your organization's policies, not by this extension.

## Running it

1. Run **A11y: Generate AI Insights (advisory)** from the Command Palette or the ✨ button on the A11y Results view.
2. Pick an audit run when there is more than one.
3. A confirmation dialog shows the following. Nothing is sent before you click **Send**.
   - the provider, model and effort (for Copilot: the request count instead of the cost);
   - how many clusters will be analyzed;
   - the estimated input tokens;
   - the cost: an estimate with prices set, otherwise `Cost: unknown (no prices set), limit: 50k in / 10k out tokens`;
   - whether privacy mode is on.
4. **Change effort…** in the dialog switches between `low`, `medium` and `high` and shows the estimate again.
5. Progress shows each request. **Cancel** stops the run. Requests already answered are kept; the rest are listed as cancelled.
6. When it finishes, the message says how much was analyzed, for example `7 of 10 clusters analyzed`. **Export Report** writes the report with the AI section.

When results already exist for this run and these settings, the command offers **Export Report** or **Regenerate all**. If the earlier result was partial, it offers **Retry failed clusters** instead.

### Effort

`a11y.ai.effort` sets how much the model reasons before answering. Reasoning tokens count toward the output limit and are billed as output.

- `low` (default): fastest and cheapest. It is enough for most audits.
- `medium`: more reasoning per cluster.
- `high`: the most reasoning. It reaches the output limit soonest.

Anthropic receives `effort` in `output_config`. OpenAI receives `reasoning.effort` for reasoning models only (`o`-series and `gpt-5` family); other OpenAI models ignore the setting.

Effort is part of the cache key, so changing it starts a new analysis rather than overwriting the old one.

## What is sent, and what never is

**Sent**, after redaction:

- The audited site's base URL and page paths.
- For each finding: rule ID, WCAG criteria, help text, summary, selectors and page path.
- Evidence for each element:
  - the HTML snippet, capped at 1,500 characters;
  - the failure summary;
  - the measured facts (focus before and after, ARIA states, sizes);
  - accessibility-tree data where the audit recorded it.

Page content is wrapped as untrusted data. The model is told never to follow instructions found in it. Text on the page cannot close that wrapper.

**Always removed**, whatever privacy mode is set to:

- Values typed into fields. HTML snippets are recorded without them in the first place.
- API keys (`sk-…`, `sk-ant-…`, `sk-proj-…`), JWTs, Bearer tokens and PEM private keys.
- `Authorization` and `Cookie` values.
- Session IDs, including `session=`, `sid=`, `JSESSIONID` and similar.
- Passwords and other secret, token or API-key values.
- The values of your `A11Y_*` environment variables.
- Storage contents, cookies and network payloads recorded as evidence facts.

**Privacy mode** (`a11y.ai.privacyMode`, on by default) also removes:

- query strings and fragments from URLs;
- internal host names, replaced with `internal.invalid`:
  - `localhost`, private IP addresses and single-label names;
  - names ending in `.local`, `.internal`, `.lan`, `.corp`, `.intranet`, `.home` or `.localdomain`;
- email addresses;
- phone numbers.

**Never sent:**

- **Screenshots.** Version 0.5 does not send images at all.
- **Your checklist answers and VPAT.**
- **Anything for a run you did not confirm.**

The redaction is pattern-based. It catches the common forms of the items above, not every possible secret. If client data must not leave the machine, do not use AI Insights for that client.

OpenAI requests are sent with `store: false`. What the provider keeps and for how long is set by your agreement with the provider, not by this extension.

## Cost and limits

Limits are checked **before every request**, not after the run:

| Setting | Default | |
|---|---|---|
| `a11y.ai.maxInputTokens` | 50,000 | Input tokens for one run. |
| `a11y.ai.maxOutputTokens` | 10,000 | Output tokens for one run, reasoning included. Each request may use only what is left. |
| `a11y.ai.maxEstimatedCostUsd` | 1 | Dollars for one run. Applies only when both prices are set. |

A request that would go over a limit is not sent; its clusters are listed as not analyzed (`budget_exhausted`).

Clusters are analyzed in groups of up to 10, most severe first. For each group, impact and remediation are done together before the next group starts. When the limits stop a run early, what was analyzed is therefore complete, and it is the most severe work.

There are **no default prices**. Prices change, and they differ by account and contract. Without prices, cost is shown as unknown and the token limits are the only limit.

## Partial results and retries

A cluster can fail on its own: an invalid answer, the output limit, a refusal, a timeout, rate limiting or the budget. The other clusters are kept, and the result is saved as **partial**:

- The report section and the tree list the failed clusters under **Not analyzed (needs human review)**, with the reason.
- **Retry failed clusters** sends only those clusters. Analyses that succeeded are reused, not paid for again.
- **Regenerate all** starts over.

If **no** cluster succeeds, nothing is saved, the reason is shown, and **Try Again** is offered. A rejected key or an account without credit stops the run at the first request.

Rate limits (HTTP 429) and server errors are retried twice with backoff, honoring `retry-after`. A refusal from the model is shown as a failure for that cluster; the extension does not switch to another model.

## Where results are kept

- **Cache:** `.a11y/ai/<auditId>.<cfgHash>.json`.
  - `auditId` is a hash of the results file's content, so renaming the results file keeps its AI results.
  - `cfgHash` covers the provider, model, effort, privacy mode, code-fix setting and prompt version.
  - `.a11y/` is git-ignored.
- **Report:** **A11y: Export Report** adds the newest AI result for that run as section 18, *AI Insights (advisory)*, after the VPAT section. Every value from the model is escaped. The model's HTML is never rendered, and code suggestions are shown as text. Without an AI result, the report is byte-identical to one from a version without AI Insights. Delete the cache file to remove the section.
- **Tree:** the A11y Results view shows an *AI Insights (advisory)* node for the loaded run.
- **Usage log** (opt-in, `a11y.ai.usageLog`): `.a11y/ai/usage.jsonl` gets one line per run:
  - time, provider and model;
  - input and output tokens;
  - cost, when prices are set.

  It never holds the key or any content, and nothing in it leaves the machine.

## Settings

| Setting | Default | Description |
|---|---|---|
| `a11y.ai.provider` | none | `anthropic`, `openai` or `copilot`. |
| `a11y.ai.model` | none | Model name. Required for `anthropic` and `openai`; for `copilot`, leave empty to pick from a list. |
| `a11y.ai.effort` | `low` | `low`, `medium` or `high`. Not used by Copilot. |
| `a11y.ai.privacyMode` | `true` | Remove query strings, internal hosts, emails and phone numbers. |
| `a11y.ai.maxInputTokens` | `50000` | Input token limit per run. |
| `a11y.ai.maxOutputTokens` | `10000` | Output token limit per run. |
| `a11y.ai.maxEstimatedCostUsd` | `1` | Cost limit per run (needs prices). |
| `a11y.ai.inputPricePerMTok` | none | Your input price, USD per million tokens. |
| `a11y.ai.outputPricePerMTok` | none | Your output price, USD per million tokens. |
| `a11y.ai.allowCodeFixSuggestions` | `false` | Allow code suggestions where the whole element HTML was sent. |
| `a11y.ai.usageLog` | `false` | Write `.a11y/ai/usage.jsonl`. |

## Troubleshooting

| Message | Cause and fix |
|---|---|
| `Choose a provider…` / `Enter a model name…` | Set `a11y.ai.provider` and `a11y.ai.model`. Nothing was sent. |
| `no API key for anthropic` (or `openai`) | The key was saved for the other provider, or not at all. Run **A11y: Set AI API Key** and pick the provider shown in the message. |
| `The API key was rejected (HTTP 401)` | The key is wrong, revoked, or belongs to another provider. |
| `No credit left on this API account…` | The provider account has no credit. Add credit with the provider, then **Try Again**. |
| `HTTP 400: …` naming the model or `output_config` / `text.format` | The model name is wrong, or the model does not support structured outputs or effort. Pick another model. |
| Clusters `token_limit` | The request or the answer did not fit the model's token limit. Raise `a11y.ai.maxOutputTokens` or lower the effort, then **Retry failed clusters**. With Copilot, a batch too big for the model's input limit is split automatically; a single cluster that is still too big needs a model with a larger input limit. |
| `No GitHub Copilot models are available…` | GitHub Copilot Chat is not installed or has not finished loading, you are not signed in, or your organization does not allow extensions to use Copilot models. Open the Chat view once, then run the command again. |
| `Copilot access was not allowed…` | You declined the VS Code prompt. Run the command again and allow it. |
| `Copilot quota reached…` | Your premium request quota is used up. Wait for the reset shown in the message, or pick a model that does not use premium requests. |
| `Copilot blocked the request (quota or policy)…` | Copilot blocked A11y Audit for a while after too many requests, or your organization's policy blocked it. Try again later. |
| `This Copilot model is no longer available…` | Clear `a11y.ai.model` and pick another model. |
| Clusters `refusal` with Copilot | Copilot's content filter stopped the answer for that cluster. **Retry failed clusters**, or try another model. |
| Clusters `budget_exhausted` | The run reached a limit. Raise the limit, then **Retry failed clusters**. |
| Clusters `invalid_output` | The answer failed validation (schema, unknown evidence, a credential). Retry, or try another model. |

## Limitations

- **Advisory only.** The model can be wrong, including when its confidence is high. Verify each recommendation before acting on it.
- **Automated findings only.** The model sees what the automated audit found. It knows nothing about criteria that need manual testing, and it cannot find issues the audit missed.
- **Pattern-based redaction.** Unusual secret formats can get through. Use test accounts and test data in audited apps.
- **Retries are per cluster.** A cluster whose remediation failed pays for its impact assessment again on retry.
- **Approximate estimate.** The estimate uses about 3.5 characters per token. Real counts differ by model. The limits are enforced with the provider's reported usage (with Copilot, the model's local token count).
- **Text only.** Screenshots are not sent in this version.
