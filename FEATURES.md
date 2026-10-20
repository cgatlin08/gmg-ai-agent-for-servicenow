<!-- SPDX-License-Identifier: Apache-2.0 -->
<!-- Copyright 2026 GMG Technology LLC -->
# GMG AI Agent for ServiceNow (open source): features

Describes the update sets built from source commit bc0499b (v1.0.1).

## Chat panel
- A floating chat panel on desktop pages (global UI script `gmg_ai_agent_panel`, the only UI script that is active after install).
- It knows the current page. On a form it also reads the table, record sys_id and the field values you can see.
- Users need the role **x_gmg_ai.user** to see the panel and the GMG AI Agent menu.

## Any LLM provider
- Provider types: **anthropic** and **openai_compatible**. OpenAI-compatible covers OpenAI and xAI Grok.
- Add one as an admin under **GMG AI Agent > Providers**: name, type, base URL, model, API key. For openai_compatible the agent calls
  `<base URL>/chat/completions`. For anthropic a blank base URL means https://api.anthropic.com/v1.
- Web search is Claude (anthropic) only and off by default (`x_gmg_ai.web_search.enabled`). Up to `x_gmg_ai.web_search.max_uses` searches (default 3)
  per model call. Each search is charged at `x_gmg_ai.web_search.price_per_1k` (default $10 per 1,000).
- No providers ship with the install.

## Prompt caching
- Claude prompt caching is on by default (`x_gmg_ai.llm.prompt_cache`). Cached tokens that an OpenAI-compatible provider reports are also
  counted. Cache reads and writes are priced with multipliers (`x_gmg_ai.cache.read_multiplier` 0.1, `x_gmg_ai.cache.write_multiplier` 1.25,
  plus per-model overrides).

## Guardrails
- **The agent runs as the calling user.** It reads only what that user's ACLs allow.
- **Writes are proposals.** The agent can change only the open, saved record. Each change appears as a card with a before/after view.
  Nothing is written until the user clicks **Accept**. **Reject** discards it.
- **Where changes are recorded.** Script and configuration changes (records on tables that extend sys_metadata) are captured in a new update set
  whose name starts with "GMG AI – ", and the confirmation names it. Edits to data records (like incidents) are a direct update, made as the
  user, with no update set. They show in the record's history where the platform audits that table.
- **Changes go live once the user confirms.** The update set tracks the change; it does not hold it back for review. Review the diff,
  and test in a sub-production instance first.
- **Live-change notes.** Editing active code warns that the change goes live on Accept. Editing an inactive record notes that it won't run until someone activates it.
  `x_gmg_ai.opensource.refuse_active_script_edits` = true refuses edits to active code instead (default false).
- **What open source can't do:** create records of any kind (including catalog items, variables, flows and business rules), delete records,
  or change any record other than the open one. Those requests are refused.
- **Server seal check.** The server seals every card when it is proposed. Accept refuses a card whose contents no longer match that seal,
  and refuses an edit if the record changed after the card was made.
- **ACL and role tables are refused** for everyone, admins included: sys_security_acl, sys_security_acl_role, sys_user_role, sys_user_has_role,
  sys_user_role_contains, sys_group_has_role. These are allowed only if an admin sets `x_gmg_ai.opensource.acl_elevation_exception` = true (default false)
  AND the user has elevated to security_admin in that session. Otherwise the reply is "Elevate to security_admin to edit ACL or role records."
- **Always-denied tables:** credential, certificate, OAuth and auth-profile tables, and the agent's own x_gmg_ai_* tables. Admins can add more with
  `x_gmg_ai.opensource.denied_tables` (comma-separated, * wildcards).
- **Non-admin writes:** `x_gmg_ai.allow_non_admin_writes` (default **true**). Set it to false and the agent is read-only for anyone without the
  admin role: proposals and Accept are refused with "Updating records is turned off for your role." Admins are not affected.

- **Accept ownership.** Accept on a card owned by another user returns **HTTP 403**. HTTP 409 is returned only for a card that was already applied. Admins can Accept any user's pending card (admin override).

## Chat window access
- `x_gmg_ai.panel_access`: **everyone** (default), **admins** (users with the admin role), or **off**.
- The server enforces it. When a user's access is off, chat, Accept, Reject and GET /health return HTTP 403 with "The GMG AI Agent is turned off for your role."
  The launcher stays hidden unless the server explicitly says the user is allowed.

## Usage, cost and logs
- Every model call is logged in **AI Agent Log** (x_gmg_ai_log; module **Logs**). Each entry records provider, model, duration, input, output and
  cache tokens, web searches, rounds and estimated cost. These usage fields are read-only. Users see only their own log rows.
- **Cost** = tokens × the provider's price per million (price_input_per_m / price_output_per_m), with cache multipliers, plus web search fees.
  It shows **$0** until prices are entered. Admins use the **Edit prices & budgets** link on the Usage page, which opens the provider list.
- Modules: **Usage** (page x_gmg_ai_usage.do) and **Usage by Model** (log list grouped by model).

## Spend caps (USD, on estimated cost; 0 = no cap)
- Per provider: **daily_budget** and **monthly_budget**. Per user: **x_gmg_ai.user_daily_cap_usd**.
- Days start at midnight and months on the 1st, in each user's own time zone, so caps reset automatically. There is no manual reset.
- From **80%** the user sees a warning. At **100%** chat is blocked with a message naming the cap. To unblock, raise the cap.

## Chat history
- Conversations are stored on the server (`x_gmg_ai.context.server_store`, default true). Users read only their own (AI Conversation / AI Conversation Message).
- `x_gmg_ai.context.retention_days`: 5 by default, 5 at most.
- The scheduled job **GMG AI - chat history retention** ships **inactive**. Retention has no effect until an admin sets its run time and activates it.

## Platform pieces
- Application menu **GMG AI Agent**. Modules: Providers (admin), Logs, Pending Actions, Conversations, Usage by Model, Usage, Settings (admin).
- Tables: x_gmg_ai_provider, x_gmg_ai_pending_action, x_gmg_ai_log, x_gmg_ai_conversation, x_gmg_ai_message.
  Pending actions and logs: users read their own rows; create, write and delete are admin only.
- Scripted REST API `/api/x_gmg_ai/agent`: POST /chat, GET /health, POST /action/{id}/{verb}, GET /usage, GET /providers.
- Role: **x_gmg_ai.user**. Settings: 27 x_gmg_ai system properties under **GMG AI Agent > Settings**.

## Install
Global first, then scoped. See INSTALL.md.

---
See TRADEMARKS.md. Licensed under Apache-2.0 (see LICENSE and NOTICE). Provided AS IS, without warranties.

ServiceNow is a trademark of ServiceNow, Inc. GMG Technology LLC is not affiliated with or endorsed by ServiceNow, Inc. Anthropic/Claude, OpenAI and xAI/Grok are trademarks of their owners, used only to identify compatibility.
