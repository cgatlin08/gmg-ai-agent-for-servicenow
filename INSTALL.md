<!-- SPDX-License-Identifier: Apache-2.0 -->
<!-- Copyright 2026 GMG Technology LLC -->
# Installing GMG AI Agent for ServiceNow (open source)

You need a ServiceNow instance and an account with the admin role. Install it on a sub-production instance first.

## Files

| Order | File | Update set after import | Updates | sha256 |
|---|---|---|---|---|
| 1 | update_sets/gmg_ai_agent_global.xml | GMG AI Agent open source (global) | 3 | a202e99bd33c027b2b6f209e04b896ddbb3262c8cfcfbea8965a9ad6fe0913f9 |
| 2 | update_sets/gmg_ai_agent_scoped.xml | GMG AI Agent open source (scoped) | 318 | 2afe3e66337f3d5f1987d6d9b28db64a610bd70a02247548b6e1c6383c425876 |

Built from source commit 1a7ecb1. Tested on glide-australia-02-11-2026__patch7-09-17-2026_10-06-2026_0905 (ServiceNow Australia release, Patch 7), 2026-10-20. To check a download, compare its sha256 (`sha256sum <file>`) with this table or SHA256SUMS.

**Changes go live once the user confirms.** The agent writes only to the record you have open, and only after you click Accept. If that record is active code, the confirmed change goes live immediately. The update set tracks the change; it does not hold it back for review. Review the diff, and test in a sub-production instance first.

## Install
Do these steps for the global file first, then the same steps for the scoped file:
1. Go to **System Update Sets > Retrieved Update Sets** and choose **Import Update Set from XML**. Upload the file.
2. Open the retrieved update set it creates (names above) and click **Preview Update Set**. Resolve any problems before you go on.
   In our test install both sets previewed with 0 problems and 0 collisions.
3. Click **Commit Update Set**.

## After install
1. **Give users the role x_gmg_ai.user.** Only users with this role see the GMG AI Agent menu and the chat panel.
   The Providers and Settings modules are admin-only.
2. **Add an AI provider** under **GMG AI Agent > Providers**. None are installed.
   - Type **anthropic** for Claude, or **openai_compatible** for OpenAI, xAI Grok and other OpenAI-compatible APIs.
   - Fill in name, base URL, model and API key. For openai_compatible the agent calls `<base URL>/chat/completions`.
     For anthropic, leave the base URL blank to use https://api.anthropic.com/v1.
   - To see costs, enter prices in USD per million tokens (price_input_per_m, price_output_per_m). Until you do, every cost shows as 0.
     Admins can also open the provider list from the **Edit prices & budgets** link on the **Usage** page.
   - Optional spend caps in USD: daily_budget and monthly_budget on the provider, and x_gmg_ai.user_daily_cap_usd per user (0 = no cap).
3. **Review the settings** under **GMG AI Agent > Settings** (27 system properties). Key defaults:
   - `x_gmg_ai.panel_access` = everyone. The other values are admins and off.
   - `x_gmg_ai.allow_non_admin_writes` = true. Set it to false and the agent is read-only for anyone without the admin role:
     their proposals and Accept are refused with "Updating records is turned off for your role." Admins are not affected.
   - `x_gmg_ai.web_search.enabled` = false. Web search works with Claude only.
4. **Chat history retention.** The scheduled job **GMG AI - chat history retention** ships inactive. Chat history is not purged, and
   `x_gmg_ai.context.retention_days` has no effect, until you set the job's run time and activate it.

## What gets installed
- The application **GMG AI Agent** (scope x_gmg_ai) with role x_gmg_ai.user.
- Tables: x_gmg_ai_provider, x_gmg_ai_pending_action, x_gmg_ai_log, x_gmg_ai_conversation, x_gmg_ai_message.
- ACLs on pending actions, logs and chat history: users read only their own rows; create, write and delete are admin-only.
- The chat panel UI script `gmg_ai_agent_panel` (global). It is the only active UI script.
- The Scripted REST API `/api/x_gmg_ai/agent`, the Usage page and the menu modules.
- No AI providers, API keys, prices or budgets.

## If the chat history ACLs are missing
The agent checks for its 8 chat-history ACLs (x_gmg_ai_conversation and x_gmg_ai_message, read, write, create and delete).
If any are missing, every /api/x_gmg_ai/agent call answers HTTP 503 "install incomplete: conversation ACLs missing".
Fix: re-commit the scoped update set, or elevate to security_admin and recreate the missing ACLs in scope x_gmg_ai.

See FEATURES.md for what the agent does, and LICENSE / NOTICE for licensing. Provided AS IS under Apache-2.0.

## Accepting cards
- **Accept ownership.** Accept on a card owned by another user returns **HTTP 403**. HTTP 409 is returned only for a card that was already applied. Admins can Accept any user's pending card (admin override).
