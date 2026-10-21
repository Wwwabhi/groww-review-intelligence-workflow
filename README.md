# Groww Review Intelligence and Support Workflow

This project turns a CSV of recent Groww app reviews into a weekly product pulse and a customer-facing brokerage explanation. It uses two n8n workflows: one to analyze reviews and request human approval, and an MCP server workflow to write the approved report to Google Docs and create a Gmail draft.

## Workflow files

- `Groww Review Pipeline(1).json` — CSV intake, review analysis, report generation, approval, and MCP calls.
- `Groww Publishing MCP.json` — MCP tools for reading the target Doc, appending the approved report, and creating an email draft.

## Input CSV

Upload a `.csv` file with these columns:

```text
review_id,date,rating,text,review_url
```

Dates should use `YYYY-MM-DD`; ratings should be integers from 1 to 5. The cleaning step removes blank or very short reviews, invalid dates/ratings, duplicate review text, and reviews outside the configured date window.

## What the workflow does

1. Cleans and normalizes the uploaded reviews.
2. Uses an LLM to cluster reviews into no more than five themes and assign each review one theme and a fee stance.
3. Calculates theme counts, selects the top three themes, extracts supporting quotes, and checks for recurring fee confusion.
4. Generates a weekly product pulse and a concise fee explanation using Groww pricing information.
5. Sends the generated report for human review by email. The workflow continues only when the reviewer approves.
6. After approval, the MCP client appends the report to Google Docs and creates a Gmail draft. It does not send the final report email.

## Approval gate

The `Send a message` node uses Gmail's **Send and Wait for Response** operation. The `If` node checks that `data.approved` is `true`. Only the true branch reaches the MCP publishing actions: `append_to_doc`, followed by `create_email_draft`. The false branch does not publish.

## Setup and run

1. Import both JSON workflow files into n8n.
2. Configure the required credentials: OpenAI, Gmail, Google Docs, and MCP Bearer authentication. Configure the HTTP request nodes to access the official Groww pricing and brokerage-help sources.
3. In the MCP server workflow, select the target Google Doc for `read_doc` and `append_to_doc`, and connect `create_email_draft` to the Gmail account that should hold the draft. Publish the MCP server workflow.
4. In the main workflow, set the MCP endpoint and matching Bearer credential on both MCP Client nodes. Set the approval email recipient, then publish the main workflow.
5. Open the workflow form, upload a review CSV, and submit it.
6. Review the approval email. Approving it triggers the Doc append and Gmail draft creation. Check the target Doc and Gmail Drafts afterward; do not send the draft as part of this workflow.

## Fee finding and source notes

The supplied review sample contains two unverified brokerage reports: one describes varying brokerage amounts and another alleges a duplicate charge. These reports do not prove that a charge was incorrect, and they do not establish recurring fee misunderstanding. Treat the finding as a brokerage concern for investigation, not as a verified billing error.

The brokerage-help article text was supplied to the fee-explainer prompt; the workflow should not be represented as having successfully fetched that article. The pricing page is fetched separately. The generated explanation notes a difference in how the pricing page and help text describe the stock-brokerage minimum, so the applicable current terms should be verified before publishing customer guidance.

Official Groww sources:

- Pricing: <https://groww.in/pricing>
- Brokerage help: <https://groww.in/help/stocks%2C-f%26o-%26-ipo/searchable/how-much-is-the-brokerage>

Review evidence:

- Review `r93`: <https://play.google.com/store/apps/details?id=com.nextbillion.groww&reviewId=8004dbbf-58ed-44da-82f4-24e03a6afc6c>
- Review `r291`: <https://play.google.com/store/apps/details?id=com.nextbillion.groww&reviewId=56a483ac-134a-46bd-8170-88255778c1f2>

## Before publishing the workflow exports

The exported workflows may contain personal email addresses, Google Doc identifiers or URLs, MCP endpoint paths, and account-specific settings. Replace those with placeholders or remove them before making the repository public. Do not commit API keys, access tokens, or credential secrets.
