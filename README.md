<!-- SPDX-License-Identifier: Apache-2.0 -->
<!-- Copyright 2026 GMG Technology LLC -->
# GMG AI Agent for ServiceNow (open source)

An AI chat panel for ServiceNow that works with the LLM provider you choose (Anthropic Claude or any OpenAI-compatible API). It reads what the signed-in user can read and, after the user clicks Accept, changes only the record they have open.

- `update_sets/gmg_ai_agent_global.xml`: global part, 3 updates (2 script includes and the gmg_ai_agent_panel UI script).
- `update_sets/gmg_ai_agent_scoped.xml`: the x_gmg_ai application, 318 updates (tables, ACLs, script includes, REST API, settings, modules). No AI provider records or keys.

Install the global file first, then the scoped one. See [INSTALL.md](INSTALL.md) for steps and checksums and [FEATURES.md](FEATURES.md) for what the agent does.

## Tested version
v1.0.1, built from source commit bc0499b. v1.0.0 (source 1a7ecb1) was install-tested on glide-australia-02-11-2026__patch7-09-17-2026_10-06-2026_0905 (ServiceNow Australia release, Patch 7), 2026-10-20. The v1.0.1 live install test is pending.

## Important: changes are live
**Changes go live once the user confirms.** The agent writes only to the record you have open, and only after you click Accept. If that record is active code, the confirmed change goes live immediately. The update set tracks the change; it does not hold it back for review. Review the diff, and test in a sub-production instance first.

## Accept and ownership
- **Accept ownership.** Accept on a card owned by another user returns **HTTP 403**. HTTP 409 is returned only for a card that was already applied. Admins can Accept any user's pending card (admin override).

## Data notice
The agent sends prompts and the record data it reads directly from your instance to the AI provider you configure; GMG Technology LLC does not receive it. You are responsible for your provider agreement and applicable law. Full text: [DATA_NOTICE.md](DATA_NOTICE.md).

## Known issues
1. **Double Accept at once.** In a rare race, two Accept clicks on the same card landing at the same moment may both be processed.
2. **Admins can Accept any card.** Admins can Accept any user's pending card (admin override); it is not limited to the card owner.
3. **Slow-page history overwrite.** On a slow page, panel state held in memory before the user id loads can overwrite that user's saved chat history.

## License
Licensed under Apache-2.0 (see LICENSE and NOTICE). Provided AS IS, without warranties (Apache-2.0 sections 7 and 8).

ServiceNow is a trademark of ServiceNow, Inc. GMG Technology LLC is not affiliated with or endorsed by ServiceNow, Inc. Anthropic/Claude, OpenAI and xAI/Grok are trademarks of their owners, used only to identify compatibility. See [TRADEMARKS.md](TRADEMARKS.md).
