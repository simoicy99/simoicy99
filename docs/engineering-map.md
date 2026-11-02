# Engineering map

Three selected public prototypes, organized around the systems questions I care about most. These examples are a reading guide to the linked repositories.

Line links are pinned to the commits reviewed when this page was written. They describe the code at those commits, not any later changes.

## 1. Where does deterministic code end and model judgment begin?

**[Dureg](https://github.com/simoicy99/dureg)**, a prototype surveillance system for prediction-market exchanges.

Stage one is plain code. Each detector takes a tenant and a time window and returns a typed [`AlertCandidate`](https://github.com/simoicy99/dureg/blob/f4d04110151f4ab42335b2aed8e6304b619e711a/src/lib/agents/detectors/types.ts#L11-L20). The [registry](https://github.com/simoicy99/dureg/blob/f4d04110151f4ab42335b2aed8e6304b619e711a/src/lib/agents/detectors/registry.ts#L126-L166) runs each detector in its own try block and skips candidates matching an alert already stored for that window.

Stage two is the model. [`validateTriageOutput`](https://github.com/simoicy99/dureg/blob/f4d04110151f4ab42335b2aed8e6304b619e711a/src/lib/agents/surveillance.ts#L330-L353) accepts one of four decisions and requires at least one citation for `escalate` or `urgent`. A failed triage [reopens the alert](https://github.com/simoicy99/dureg/blob/f4d04110151f4ab42335b2aed8e6304b619e711a/src/lib/inngest/surveillance-functions.ts#L88-L97). The [human disposition](https://github.com/simoicy99/dureg/blob/f4d04110151f4ab42335b2aed8e6304b619e711a/src/lib/inngest/surveillance-functions.ts#L226-L273) step records a person's decision beside the agent's and closes the alert.

Reflection proposals are copied into the live `prompt_overrides` and `detector_configs` tables by [`approveReflection`](https://github.com/simoicy99/dureg/blob/f4d04110151f4ab42335b2aed8e6304b619e711a/src/lib/orchestrator/reflections.ts#L326-L390), called from the [admin approval action](https://github.com/simoicy99/dureg/blob/f4d04110151f4ab42335b2aed8e6304b619e711a/app/orchestrator/reflections/actions.ts). Agents and detectors read those tables at run time through [`resolveSystemPrompt`](https://github.com/simoicy99/dureg/blob/f4d04110151f4ab42335b2aed8e6304b619e711a/src/lib/agents/prompts.ts#L20-L62) and [`resolveDetectorConfig`](https://github.com/simoicy99/dureg/blob/f4d04110151f4ab42335b2aed8e6304b619e711a/src/lib/agents/detectors/configs.ts#L94-L103).

**Tradeoffs.** The validator checks that a citation exists, not that it supports the claim, and `benign` or `monitor` results need none. Dedupe only sees the current window, so overlapping windows can repeat an alert. Approval is not one transaction: override rows land before the reflection is stamped, so a failure in between leaves live overrides on a reflection that still reads as pending. The loaders apply any active row, so approval governs reflection proposals rather than every writer.

## 2. What should an agent do when it does not know?

**[Luma Autopilot](https://github.com/simoicy99/luma-autopilot)**, an MIT-licensed browser automation tool.

In the profile, [a `null` fact means "I don't know"](https://github.com/simoicy99/luma-autopilot/blob/075d92159dedb3c775401e19831eb84129965ba9/profile.example.json#L2). The [classifier](https://github.com/simoicy99/luma-autopilot/blob/075d92159dedb3c775401e19831eb84129965ba9/browser/pipeline.js#L273-L320) assigns each event a status, and a required question its rules cannot fill becomes `NEEDS-HUMAN` with the questions attached. The [registration loop](https://github.com/simoicy99/luma-autopilot/blob/075d92159dedb3c775401e19831eb84129965ba9/browser/pipeline.js#L374-L392) submits only `READY` events, waits 20 seconds between them, and stops for the night if Luma's verification gate repeats after one cooldown.

**Tradeoffs.** The [dry-run review](https://github.com/simoicy99/luma-autopilot/blob/075d92159dedb3c775401e19831eb84129965ba9/.claude/skills/luma-autopilot/SKILL.md#L64-L72) is required by the runbook, not enforced in code; the loop does not check that it happened. Answers come from regex rules over form labels, which are easy to read and fix but can misread new phrasing or fill a question the profile does not truly answer. The tool depends on Luma's private web endpoints, and scheduling is macOS only.

## 3. How does money move without being counted twice?

**[Ignite](https://github.com/simoicy99/IGNITE)**, an experimental competitive-gaming MVP. Payment and verification flows include test adapters and stubs.

Balances are not stored; each is [the sum of an account's entries](https://github.com/simoicy99/IGNITE/blob/50a7a60fdd98d8b6b365899d530355fb7054ce67/packages/ledger/src/index.ts#L29-L41). Every entry has a [unique idempotency key](https://github.com/simoicy99/IGNITE/blob/50a7a60fdd98d8b6b365899d530355fb7054ce67/prisma/schema.prisma#L77) and is written as an [upsert with an empty update](https://github.com/simoicy99/IGNITE/blob/50a7a60fdd98d8b6b365899d530355fb7054ce67/packages/ledger/src/index.ts#L60-L81), so replaying the same key adds nothing. [`transfer()`](https://github.com/simoicy99/IGNITE/blob/50a7a60fdd98d8b6b365899d530355fb7054ce67/packages/ledger/src/index.ts#L112-L150) writes a paired debit and credit in one Prisma transaction, and match locks and dispute bonds go through it. Keys come from domain IDs, so a redelivered [payment webhook](https://github.com/simoicy99/IGNITE/blob/50a7a60fdd98d8b6b365899d530355fb7054ce67/services/api/src/routes/webhooks.ts#L63-L80) for the same intent reuses `topup:<intent>`.

Other accounting paths are incomplete: [top-ups](https://github.com/simoicy99/IGNITE/blob/50a7a60fdd98d8b6b365899d530355fb7054ce67/packages/ledger/src/index.ts#L250-L263), [settlement](https://github.com/simoicy99/IGNITE/blob/50a7a60fdd98d8b6b365899d530355fb7054ce67/packages/ledger/src/index.ts#L324-L386), and [withdrawals](https://github.com/simoicy99/IGNITE/blob/50a7a60fdd98d8b6b365899d530355fb7054ce67/packages/ledger/src/index.ts#L487-L505) write one-sided entries, and the platform fee appears only in metadata.

**Tradeoffs.** [`lockFunds`](https://github.com/simoicy99/IGNITE/blob/50a7a60fdd98d8b6b365899d530355fb7054ce67/packages/ledger/src/index.ts#L156-L181) reads the balance outside the transfer transaction, so two concurrent locks could both pass the check; this needs a row lock or serializable isolation. Idempotency covers repeats of the same key only. The webhook returns 200 on internal errors, so recovery depends on reconciliation rather than provider retries.
