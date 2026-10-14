# Review Insights Pipeline: ChatGPT (Google Play, India)

An n8n workflow that turns raw Google Play reviews into a weekly-format product pulse and a support-ready fee explanation. Both are logged to a Google Doc and saved as a Gmail draft only after a human approves.

## What it does
1. **Review intelligence:** an LLM groups reviews into at most 5 themes, picks the top 3, extracts 3 verbatim user quotes, and flags 1 recurring fee/charge pain point.
2. **Weekly product pulse (≤250 words):** top themes, supporting quotes, key observation, 3 action ideas.
3. **Fee explainer (≤6 bullets):** neutral, facts-only, written only from official OpenAI text, with 2 source links and a "Last checked" date.
4. **Approval-gated actions:** after approval, it appends an entry to a Google Doc and creates a Gmail draft. Nothing is auto-sent.

## Data
- **Source:** ChatGPT Android app reviews, Google Play, India, English (`com.openai.chatgpt`)
- **Window:** Aug 17 – Oct 12, 2026 (8 weeks)
- **Collection:** scraped about 410,000 reviews with `google-play-scraper`, randomly sampled up to 400 per week (3,600 rows), then cleaned to **1,167 rows** by removing blanks, duplicates and reviews under 15 characters
- **File:** `chatgpt-reviews-clean.csv`

## Workflow (n8n)
Form upload → Extract From File (CSV) → Code (keeps all 1–3★ reviews plus 150 random 4–5★ reviews, 323 in total) → Basic LLM Chain (themes, quotes, fee issue) → Basic LLM Chain1 (weekly pulse) → Basic LLM Chain2 (fee explainer) → Merge Payload → **Approval Gate** → If → Google Docs (append) + Gmail (create draft)

Model: Groq `openai/gpt-oss-120b`, low temperature.

## How to run
1. In n8n, import `workflow.json` (Workflows → Import from file).
2. Add your own credentials: Groq API key, Google Docs OAuth and Gmail OAuth. Set your Google Doc ID in the Docs node.
3. Click **Execute workflow**, open the form's Test URL, and upload `chatgpt-reviews-clean.csv`.
4. The run pauses at the **Approval Gate**. Approve to continue, or reject to stop.
5. Check the Google Doc (new entry at the bottom) and your Gmail drafts (new draft).

## Where MCP-style approval happens
The **Approval Gate** node sits before every write action. Nothing is appended to the Doc and no draft is created until it is approved. If it is rejected, the If node ends the run with no changes anywhere. A rejected run is shown in `screenshots/`.

## Fee issue identified from reviews
**Free-tier usage limits vs. the cost of paid plans (ChatGPT Plus).**
- Users complain about image-generation limits on the free tier, ask why they need to upgrade, and ask for lower prices.
- Evidence is limited: about 16 of 1,167 cleaned reviews mention pricing or payment, and none mention refunds, unexpected charges, renewals or cancellation. This is a real but small pattern about plan limits and cost, not a surprise-billing issue.
- Theme percentages in the pulse are LLM estimates from a 323-review sample and are approximate.

## Fee explainer sources
The explainer is written only from official OpenAI text that I pasted into the prompt on 2026-10-14. It was not fetched automatically. The India prices (Plus ₹1,999 and Go ₹399 per month, both inclusive of GST) were checked against the official pricing page on 2026-10-14.

## Source list
1. https://help.openai.com/en/articles/9275245 – free-tier and usage limits
2. https://openai.com/chatgpt/pricing – India plan prices
3. https://help.openai.com/en/articles/6950777 – Plus billing and cancellation
4. https://play.google.com/store/apps/details?id=com.openai.chatgpt – source of the reviews

## Limitations
- Clustering is LLM-based, so themes, quotes and percentages vary slightly between runs. Quotes were checked against the CSV for the submitted run.
- The pulse title's date window is set in the prompt, not computed from the data.
- Multilingual reviews (Hindi, Bengali and others) are in the input, but quotes are chosen in English.
- Prices and limits change over time, which is why the explainer carries a "Last checked" date.
- The pulse is a weekly-format note generated from the latest 8 weeks of reviews. Running the workflow each week with a fresh CSV produces the next pulse.




