# Willy Pools demo receptionist

First client build for the AI tradie receptionist. Goal: a demo number Willy can ring himself, using his real answers, within 3 days of the discovery call.

| File | What it's for |
|---|---|
| `01-discovery-call-sheet.md` | The 10-minute call with Willy. Every answer fills a gap in the KB. |
| `02-knowledge-base.md` | What the agent knows. Prefilled; gaps marked `[ASK WILLY]` / `[CONFIRM]`. |
| `03-agent-system-prompt.md` | Paste-ready prompt, call flows, post-call fields, test calls. |
| `04-summary-templates.md` | SMS and email to Willy (and the caller), plus how to wire them. |
| `05-eval-suite.md` | 18 pass/fail test calls to rerun after every change. |

Nothing in here is a confirmed price for the client. Internal only.

---

## Running cost check (checked 5 Nov 2026)

All platform prices are in **USD**. AUD figures assume about 1.5 AUD per USD (not checked today; look up the rate before pricing).

### Unit prices from the official pages
| Item | Price (USD) | Source |
|---|---|---|
| Retell voice infra | $0.055/min | retellai.com/pricing |
| Retell TTS (Retell voices / ElevenLabs Flash) | $0.015 / $0.040 per min | retellai.com/pricing |
| Retell LLM: GPT 5 mini / Claude 5.5 Haiku / GPT 4.1 | $0.008 / $0.0032 / $0.045 per min | retellai.com/pricing |
| Retell example all-in | ~$0.11/min | Retell's own calculator |
| Retell number | $2/mo (or bring a Twilio number) | retellai.com/pricing |
| Retell free credit | $10 | |
| Vapi hosting | $0.05/min + model, transcriber and voice at cost | vapi.ai/pricing |
| Vapi typical all-in | $0.08 to $0.13/min (their 1,000 min calculator: $82 to $129) | vapi.ai/pricing |
| Vapi free credit | $5 | |
| Twilio AU local number | $3.00/mo (AU mobile number $8.25/mo; page is inconsistent, verify) | twilio.com voice pricing AU, Aug 2026 |
| Twilio inbound to AU local number | $0.01/min | same |
| Twilio SMS to AU mobile | $0.0515 per segment; inbound SMS $0.0075; MMS $0.35 each way | twilio.com SMS pricing AU, Oct 2026 |

### Willy in peak (~600 AI minutes, ~250 calls a month)
| Line | Lean build (Retell + small LLM + Retell voice) | Typical build (~$0.11/min) |
|---|---|---|
| Voice AI minutes | 600 × ~$0.075 = $45 | 600 × $0.11 = $66 |
| Twilio number + inbound minutes | $3 + $6 = $9 | $9 |
| SMS (summary to Willy + some caller texts, ~1.5 per call) | ~$20 | ~$20 |
| Make/Zapier for summaries | $0 to $10 | $10 to $20 |
| **Total USD** | **~$75 to $85** | **~$105 to $115** |
| **Total AUD (×1.5)** | **~$115 to $130** | **~$160 to $175** |

### What this means for the $40 to $90/mo estimate and the $99 price
- The $40 to $90 estimate only holds if it's in USD and on the lean build. In AUD, peak running cost is more like **$115 to $175/mo**.
- At $99/mo founding price, **Willy runs at a small loss in peak** (Oct to Mar) and a profit in winter. That's acceptable for a case study, but don't repeat $99 for client #2.
- Ways to close the gap: use a small LLM (GPT 5 mini or Claude 5.5 Haiku) and Retell's own voices; send Willy one SMS per call (not caller texts); use a photo upload form instead of MMS; consider a minute cap in the agreement (e.g. 400 minutes included).
- The standard price of $250 to $350/mo holds up comfortably against these costs.

### Still to verify (not on the pages, from general knowledge)
- Twilio AU numbers need a **regulatory bundle** (business name, ABN or ID, and an Australian address) before purchase. Approval can take a day or more, so start it **before** the discovery call.
- Whether Willy's mobile plan includes call forwarding to another AU number at no extra cost.
- Victoria's recording rules: the "this call may be recorded" line in the opening message covers notice. Keep it.

---

## Checked against atlas-agents best practice
Reviewed against github.com/BryanField15/atlas-agents (ch02 prompt A/B, ch13 voice agent, ch14 guardrails, ch18 eval harness). Changes made:
- **Evals (ch18):** the 10 test calls became an 18-case pass/fail suite with must/must-not and expected post-call fields, plus a score log. Rerun on every change.
- **Guardrails (ch14):** prompt now refuses role-change and "system prompt" requests, never takes payment details, never invents facts. Summary check that no card numbers leak into SMS/email/sheet. Optional post-call hallucination check against the KB.
- **Voice (ch13):** spoken answers capped at three sentences, stop when interrupted, no "certainly/absolutely/of course".
- **Prompt A/B (ch02):** compare prompt versions by running both through the eval suite, not by feel.
- **Not adopted:** the ch13 custom Twilio-to-Realtime bridge. Retell/Vapi already handle audio, barge-in and turn-taking; building our own adds hosting and maintenance for no gain at this size. Revisit only if per-minute cost becomes the problem at 10+ clients.

## Build order for the demo (target: 3 days)
1. **Day 0 (now):** create Twilio account, submit AU regulatory bundle, buy a local Melbourne (03) number. Create Retell account.
2. **Day 1:** discovery call. Fill `02-knowledge-base.md`.
3. **Day 1 to 2:** create the Retell agent with the prompt in `03`, attach the KB, connect the Twilio number. Run the 10 test calls.
4. **Day 2:** wire post-call webhook → Make/Zapier → SMS + email + Google Sheet (templates in `04`).
5. **Day 3:** give Willy the number to ring. Don't set up forwarding on his mobile until he's happy.
6. **Trial start:** set his mobile to "forward when busy / unanswered / unreachable" to the Twilio number.
