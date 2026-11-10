# Post-call summaries: SMS and email templates

Fields in `{{ }}` come from the post-call analysis in `03-agent-system-prompt.md`.

## How it's wired (no code needed for v1)
1. Retell (or Vapi) sends a post-call webhook when each call ends.
2. A Make.com or Zapier scenario catches it, picks the template by `call_type`, then:
   - sends the SMS to Dean via Twilio,
   - sends the email via Gmail/Outlook,
   - for `repair_hot_lead`, also texts the caller the photo-upload link,
   - logs a row in a Google Sheet (date, call type, suburb, urgency, booked Y/N). That sheet is the case-study tracker.
3. Skip `spam_or_wrong_number` calls except for the sheet log.

Keep SMS under 160 characters where possible: each extra segment costs another ~US$0.05.
The one-tap callback link is just the number written as `tel:` in email and plain in SMS (phones make it tappable).

---

## Missed-call text-back (what Dean asked to start with)
Dean said he'd rather start with a text-back than an AI voice (10 Nov 2026). When a call to his mobile goes unanswered, his Telstra "forward when unanswered/busy" sends it to the Twilio number, which plays a short message and texts the caller.

Short voice message on the Twilio number (recorded or TTS, then hang up):
```
Hi, you've reached Willy Pools. Dean's on a job. We'll text you now so you can tell us what you need.
```

SMS to the caller (2 segments, keep it):
```
Hi, it's Willy Pools. Sorry we missed you, Dean's on a job. Reply with your name, suburb and what you need (and a photo if it's equipment) and he'll get back to you within a day or two.
```

SMS to Dean when they reply:
```
TEXT-BACK: {{caller_number}} replied: "{{reply_text}}"
```
Same Make/Zapier scenario and Google Sheet as below. Twilio's inbound SMS webhook triggers it instead of the Retell post-call webhook. A photo reply is MMS (~US$0.35 each).

---

## SMS to Dean

**New service**
```
NEW SERVICE: {{caller_name}}, {{suburb}}. {{pool_type}}, {{sanitiser}}, wants {{service_frequency}}. Call {{callback_number}}
```

**Equipment replacement, repair or robot purchase (hot lead)**
```
🔥 HOT LEAD: {{problem}} ({{equipment}}). {{suburb}}, urgency {{urgency}}. {{caller_name}} {{callback_number}}
```

**Quick question answered (no callback wanted)**
```
FYI answered: {{one_line_summary}} ({{callback_number}}). No callback needed.
```
(Dean may find these noisy. Offer to switch them to a daily digest after week one.)

**Message / callback**
```
CALL BACK: {{caller_name}}, {{one_line_summary}}. {{callback_number}}
```

## SMS to caller (repair flow only)
```
Hi {{caller_name}}, it's Willy Pools. Reply to this text with a photo of the equipment (and the label if you can) so Dean can bring the right parts. He'll be in touch soon.
```
Note: replying with a photo is MMS, which Twilio charges at ~US$0.35 inbound in AU. A cheaper option is a link to a simple upload form (e.g. a Tally or Google Form with file upload). Decide before go-live.

## SMS to caller (new service, optional)
```
Thanks {{caller_name}}, Willy Pools here. Got your details for a regular service in {{suburb}}. Dean will be in touch within a day or two to set a time.
```
Optional because every caller text adds cost. Good for trust; try it in the trial.

---

## Email to Dean

**Subject:** `{{call_type_label}}: {{caller_name}}, {{suburb}}` (add `[URGENT] ` prefix when `urgency = high`)

```
{{one_line_summary}}

Call back: {{caller_name}} on {{callback_number}}   [Tap to call](tel:{{callback_number}})

Details
- Type: {{call_type_label}}
- Suburb / address: {{suburb}}, {{address}}
- Pool: {{pool_or_spa}}, {{pool_volume}}, {{pool_type}}, {{sanitiser}}
- Email: {{email}}
- Service wanted: {{service_frequency}}
- Problem: {{problem}}
- Equipment: {{equipment}}
- Urgency: {{urgency}}

Call time: {{start_time}} · Length: {{duration}}
Recording: {{recording_url}}
Transcript:
{{transcript}}

Booked it? Reply "booked" and we'll count it in your monthly results.
```

Leave a field's line out when it's empty (Make/Zapier filter, or just accept blanks for the demo).
`call_type_label` mapping: new_service → "New service enquiry", repair_hot_lead → "Repair / equipment", faq → "Question answered", message → "Callback request".
