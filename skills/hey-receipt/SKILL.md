---
name: hey-receipt
description: Free month-end helper. Matches a card statement to receipts found in your own email and uploads, and builds a packet your bookkeeper can review. Bookkeeping and reporting only. Use when the user has a card statement and wants receipts gathered and reconciled.
---

# hey-receipt

DRAFT. Not installed. Written for a generic user; nothing here is specific to one person, card, mailbox or machine.

## What this is, and is not

Bookkeeping and reporting. It finds receipts, matches them to card statement lines, and lists what could not be matched. It does not prepare, file or advise on tax returns. It never says what is deductible, never maps spend to tax form lines, never says "tax-ready", "audit-proof", "IRS-compliant" or "maximize deductions". Categories are plain bookkeeping labels from a fixed list (see Step 1, question 5), always changeable by the user, never advice. The user and their preparer decide everything tax-related.

If the user asks a tax question, say it is outside this skill and point them to their preparer.

## Data rules (non-negotiable)

- The skill's own files (ledger, packet, settings) stay on the user's machine. No hosted storage, no upload to any service owned by the skill's author. Email content is still seen by the user's mail provider and by whatever AI model runs this skill, and on Route A also by Composio. Say so plainly at setup.
- Output goes to a local folder the user chooses (Step 1, question 1). Nothing is sent to anyone. The user attaches and sends the packet themselves.
- Never print or store full card numbers. Last 4 digits only. Masking rule, the one rule for digit runs everywhere in this skill: keep the printed vendor text as it appears on the statement, but write any run of 8 or more digits (digits side by side, or four-digit groups joined by single spaces or hyphens, such as 1234 5678 9012 3456) as X for every digit except the last 4. A reference ending 884100331755 becomes XXXXXXXX1755. The masked form is used in every output and saved file the skill writes: ledger, totals, worksheet, cover, email text, notes, file names and `hey-receipt.json`. Tell the user once per run that a run was masked, and on which line. Vendor documents (receipts) are attached verbatim and never edited, so masking does not apply inside them. Instead each receipt is scanned for a run of 13 to 19 digits that passes the Luhn check, which is how a real card number behaves (Step 2 item 3). Order and invoice numbers that fail the check are not flagged. If a card-like number is found, the user is asked before that file goes into the packet.
- Do not store credentials in the output folder or the packet. On Route B there are no credentials at all.
- Never invent receipt data: no made-up invoice numbers, dates, or amounts. If a receipt is missing it stays missing and is listed as "Receipt not found". Vendor documents are attached verbatim, unedited.
- The skill cannot control whether the user's AI model provider retains or trains on content it reads. Tell the user to check their provider's data settings before they hand over email.

## Output folder layout

Everything lives under the folder chosen in question 1. This is the one place the layout is defined; every other section points here.

```
<output folder>/
  hey-receipt.json
  inbox/receipts/
  inbox/statements/
  found/
  work/
  runs/MM-YYYY-LAST4/
    ledger.csv
    totals.txt
    packet/
    email-body.txt
    MM-YYYY-LAST4-receipts.zip
```

- `hey-receipt.json`: saved answers (output folder path, entity name, bookkeeper name, cards, category set, route, that the data-handling notice was okayed, the receipt-to-card answers from Step 2 item 3, and which file the user kept when two receipt files were duplicates, Route B step 9). Personal data. Never inside `packet/`, never in the zip.
- If the user chose a folder other than the default (a folder named `hey-receipt` in their Documents folder), also write a one-line file `hey-receipt-folder.txt` holding that path into the default folder (create it if needed), so Step 1 can find the real folder next time. Write it once, at question 8, not earlier. It holds a path only, and it is the one file the skill writes outside the chosen folder.
- `inbox/receipts/`: receipt files the user drops in (.eml, .pdf, .png, .jpg, .html, .txt).
- `inbox/statements/`: card statements (.pdf, .csv, or .txt: text copied out of a PDF statement and pasted into a file).
- `found/`: documents the skill pulled out for matching, as-is, including any near-miss candidate.
- `work/`: scratch only: the reconciliation check script and PDF renderings before they are copied into `packet/`. Never sent.
- `runs/MM-YYYY-LAST4/`: one folder per run. One run is one card and one statement month, so a second month or card gets its own folder and never overwrites another. MM-YYYY is the month of the statement's closing date (for a CSV, the month of the last post date), so a statement running 10/15 to 11/14 is 11-YYYY even if the user called it the October statement. Tell the user which month the folder uses.
  - `ledger.csv`: transcribed statement lines.
  - `totals.txt`: the statement's printed net and section subtotals (PDF, or pasted text that shows them; see Step 2 item 1).
  - `packet/`: the only folder whose contents the bookkeeper receives.
  - `email-body.txt`: the email text for the user to paste. It sits beside `packet/`, not in it.
  - `MM-YYYY-LAST4-receipts.zip`: a zip of the contents of `packet/`, written beside `packet/`, never inside it.

## Step 1: Onboarding

First look in the default output folder (a folder named `hey-receipt` in the user's Documents folder). If it holds `hey-receipt.json`, use it. If it holds only `hey-receipt-folder.txt`, read the path in it and use the `hey-receipt.json` there. If neither is there, ask one question: "Have you used hey-receipt before?" If yes, ask separately: "What is the folder path?" If the path holds a `hey-receipt.json`, use it; otherwise run onboarding in full. If the file exists, read it, say back in one line what is saved, and ask only: "Still correct?" If yes, skip to Step 2 and ask only which card and which month, one at a time. If the user says something changed, re-ask just that question. Onboarding runs in full only the first time.

Ask ONE question at a time and wait for the answer. Never bundle two questions in one message. Save answers to `hey-receipt.json` after each one. Never ask for, accept or store passwords, full card numbers, or SSNs. If the user pastes one, tell them not to, and do not write it anywhere.

The data-handling notice comes first, before question 1 (first time only; it is not a numbered question). Say the data-handling rules above in plain words: the skill's files stay on the user's machine; their mail provider and AI model still see any email content; and they should check their AI provider's data settings before handing over email (the skill cannot control retention or training). Then ask only: "Okay to continue?" and wait for yes. Save the yes in `hey-receipt.json` as soon as the folder exists (question 1), and do not repeat the notice when it is saved.

Order of questions:

1. Where should the output folder go? Suggest a default (for example a folder named `hey-receipt` in their Documents folder) and wait for them to accept it or give another path. Create it with `hey-receipt.json`, `inbox/`, `found/`, `work/` and `runs/` from the layout above. Nothing is written before this answer.
2. Card: ask the issuer, then in a separate message the last 4 digits only. After it is saved, ask separately: "Is there another card?" Repeat for each card and save the list. One run covers one card; the skill asks which card at the start of each run and offers to loop for the next (Step 4). Each run has its own run folder and packet.
3. The first run. If more than one card is saved, ask which card this first run is for. Then, in a separate message, ask which statement month (for example last month). The answers are used for the first run only and are not saved; Step 2 does not ask them again. One run covers one card and one month; never batch months or cards.
4. Entity name: the name to print on the cover (for example the business name, or the user's own name for a sole proprietor). Ask it before it is used anywhere. Name only: no tax ID, EIN or SSN.
5. Which of these describes your work best? (a) sole proprietor or single-member LLC that does not own a brokerage, (b) owner of a brokerage (through an LLC or any other entity), (c) W-2 employee with side income, (d) other or not sure. If more than one fits, the user picks; the labels are editable either way. This is about how you work, not a tax question and not a legal-entity question. Say plainly why this is asked: your answer picks which list of category labels is used to tag each charge in the packet. It does not change which charges are reported, it is not tax guidance, and a label is not a claim that anything is deductible. The user may edit any label or paste their own list instead. The lists are below, "Category label sets".
6. Bookkeeper name. Optional; the user can say skip. Used only for the greeting in `email-body.txt`. The skill never sends anything and always makes both the zip and the email text, so it does not ask for the bookkeeper's email address or delivery method.
7. Email access: "Use the default (Route B: you save receipt emails into a local folder, no account connected, no third party), or the optional upgrade (Route A: your own Composio account reads your Gmail for you)? Default is Route B." If they pick A, read them the Route A gate before anything else. If they hesitate, use B.
8. Confirm back in one line: output folder, entity name, category set, source (route), bookkeeper if any, and for the first run its card last 4 and month. Wait for yes. The Route B steps run only after this yes, in the order given there (steps 2 to 4 are skipped when `inbox/` already has files; steps 6 and 9 are policy, not actions). Write `hey-receipt-folder.txt` here if the folder is not the default.

### Category label sets

Labels only. Tag each ledger line with exactly one label based on the vendor text as printed. If unsure, use Uncategorized. Never explain what a label means for taxes, never total by label on the cover. Every set also includes Personal, not business, and Uncategorized. A credit line is labeled Uncategorized.

- (a) Sole proprietor or single-member LLC: Software and subscriptions; Advertising and marketing; Meals; Travel; Office supplies; Professional services; Vehicle; Phone and internet; Education and training; Other.
- (b) Brokerage-LLC owner: Marketing and advertising; Technology and subscriptions; Dues, licenses and association fees; Referral and transaction costs; Meals and client gifts; Travel and vehicle; Office and supplies; Professional services; Education and training; Other.
- (c) W-2 employee with side income: Side work, software and subscriptions; Side work, supplies; Side work, advertising; Side work, travel; Side work, services; Other.
- (d) Other or not sure: use set (a).

## Email access

Route B is the default. Route A is an optional upgrade.

### Route B (default): your own folder, no account connected

No third party is connected and no credential is shared. The skill never logs in to the user's mail. It reads only files the user puts in `inbox/`. Menu names and click paths differ by mail provider and change over time, and the provider help pages were not fetched for this draft, so every provider click path below is UNVERIFIED: the agent should describe the goal and let the user find the button in their own mail, not claim a path it has not seen.

Steps. Do them with the user one at a time. Steps 6 and 9 are policy to follow, not actions to perform; do the rest in order. They are always called "Route B step N" (never "Step N", which means the headings below). If the user already has files in `inbox/`, skip steps 2 to 4, list what is there, and go to step 5.

1. Check the folders. Question 1 made `inbox/`; the agent creates `inbox/receipts/` and `inbox/statements/` only if they are missing, and tells the user where they are.
2. Find the receipts in their mail. Give the user a search to paste into their own mail search box, for example: `receipt OR invoice OR "order confirmation" OR "payment received"`, plus a date range: the statement period plus 7 days on each side, which is how far a receipt can lag (Step 2 item 3). If the statement is not in `inbox/statements/` yet, do step 5 first and come back here. Gmail and Outlook both accept search text like this; exact operators differ (UNVERIFIED). Ask them to also glance at the Promotions, Updates or Other tabs and the spam folder.
3. Optional, set up a label or forwarding address. Ongoing, the user can make a filter in their own mail that applies a label or folder to receipt-looking messages, or forwards them to another mailbox they own (not a mailbox owned by the skill's author, which does not exist). Forwarding to a different address usually needs that address to confirm (UNVERIFIED, depends on the provider). The skill never reads that mailbox. It is only a place for the user to collect mail, so they can do step 4 once per month instead of hunting.
4. Get the messages into `inbox/receipts/`. For each receipt the user either saves the message as an .eml file or prints it to PDF (plain .txt files are also read), then drops the file in `inbox/receipts/`. Attachments (a vendor's own PDF invoice) are saved to the same folder as they are. A receipt that exists only in the message body is saved as .eml or printed to PDF; both count. Vendor portal receipts (behind a login) are downloaded by the user from the vendor site and dropped in the same folder.
5. Give the card statement. Ask: "Drop this month's card statement in `inbox/statements/`. A PDF downloaded from your card issuer, or a CSV export, both work. You can also copy the text out of a PDF statement and paste it into a .txt file there." Wait until the file is there; list the file name back and the number of lines you can see, charges and credits both, after joining any line that wraps onto the next (not counting any header row). If it is a photo, a screenshot or a scan, say transcription from images is less reliable, and ask for the issuer's PDF or CSV first; if they only have the image, proceed but mark every ledger row "from image" so it gets a closer look. Statement check (every run, including each later card or month, uses this one check): compare the card last 4 in the statement against the card chosen for this run (question 2 list). If they differ, stop and ask. If the statement has no card column or the last 4 cannot be seen, ask the user to confirm the last 4 once. Apply the masking rule (Data rules) to any run of 8 or more digits in the statement, and tell the user once. Statements for other months or other cards go in the same folder; run one at a time.
6. If a statement line has no matching receipt, the skill does not guess and does not drop the line. The line is kept, marked "Receipt not found" (or "Paid via vendor portal", only under the rule in Step 2 item 5), listed on the cover and in the email text as charges we could not match, and the user is told in Step 4 exactly which ones to chase. A near-miss is never a match: it follows Step 2 items 4 and 5.
7. Test it. Before the full run, the agent lists the files it read from `inbox/receipts/` (count and type), runs the de-duplication from step 9 first (so a duplicate is never shown twice), then shows the user the vendor, date and amount it extracted from the first three remaining files by file name (alphabetical), one file at a time, and asks the user to confirm each is right. If a file shows no vendor name, ask one question: "I cannot find a vendor name on this file. Which vendor is it from?" and use the answer as that file's vendor for this run; if the user does not know, the file is handled by Step 2 item 5 (receipt that matched no line). If the date or amount is wrong, ask the user to re-save that file in another format (for example print to PDF); after two tries with the same file, stop and tell the user it cannot be read. This tests extraction only. Any file that is blank or cannot be read, at any point, is listed to the user as not read with a request to re-save it; it is never skipped silently, and lines that would have matched it stay "Receipt not found" until it is re-saved.
8. Backfill. For earlier months, the user repeats step 2 and step 4 with that month's date range and drops those receipts in the same folder. The skill matches by date, so one folder can hold several months. Nothing else is needed.
9. How the skill handles the folder (Step 2 uses this): it reads every file in `inbox/receipts/`, and for .eml files reads the headers (sender, date, subject), the message body and attachments, reads .txt files as plain text, and reads .html files as a web page (the body text, including a date printed in the body when there are no headers). De-duplication: two .eml files with the same message ID are one receipt; two files with the same vendor, date and amount are flagged as a possible duplicate and the user is asked which to keep, never silently dropped. Save the answer (kept file name, ignored file name) in `hey-receipt.json`, so later runs skip the ignored file and do not ask again. This is about duplicate receipt files; duplicate statement lines are Step 2 item 3. When a receipt is only a message body, the original .eml is kept in `found/` and a PDF rendering of it goes into the packet, with the source column saying "rendered from email" (a .txt receipt: "rendered from text file"; an .html receipt: "rendered from html file"). The rendering is made by converting the message to HTML (an .html file is used as it is) and printing that HTML to PDF with the headless browser (see Step 3, Rendering).

Cost: $0 beyond what the user already pays for their mail and their AI model. Time cost: one setup, then a few minutes a month to save and drop files.

Revocation: nothing was granted to a third party, so there is nothing to revoke. To stop, the user deletes the filter or forwarding rule in their own mail settings and deletes the output folder.

### Route A (optional upgrade): the user's own Composio account

This route is gated. Whether Google's restricted-scope review has been completed for Composio's managed Gmail app is UNVERIFIED, and whether the user will see an "unverified app" warning is UNVERIFIED (details below). Before the user chooses A, say that to them in plain words. Do not describe Route A as safe or approved. If the consent screen shows a warning or asks for more than reading mail, stop and go back to Route B. Do not remove this gate until a released version of this skill records a verified answer with a URL.

Written from Composio's live docs and Google's own pages, fetched 2026-11-18 (URLs listed under "Sources for route A" below). Composio changes pricing and docs often: before release, re-fetch the cited pages and compare. Anything marked UNVERIFIED was not stated in a page that was fetched. Say that plainly to the user; do not fill it in.

Read this to the user before they start, in plain words:

- Route A hands a copy of their mail-reading permission to Composio, a third party. Composio stores the connection credentials on its servers (C8). Route B (above) avoids that. They can switch back to B at any time.
- Composio's default Gmail connection uses Composio's own Google app, not an app owned by the skill's author (C6, C7).

Steps for the user. The agent runs any terminal command itself and tells the user what it is doing. The user only does the browser steps (signup, sign-in, consent).

1. Create your own Composio account. Go to https://composio.dev and choose Sign up or Get started (C1). The page says the Hobby plan needs no credit card (C2). Which sign-in methods the signup offers (email, Google, GitHub) is UNVERIFIED: the fetched pages do not list them. Use an account you control, not a shared one.
2. Check what plan you landed on. Composio's site shows two free plans in two places, and which one a new account gets was not stated in the fetched pages (UNVERIFIED). The developer platform page lists Hobby, $0, no credit card, 100,000 tool calls a month with up to 20,000 on Composio-managed apps, hard-capped (C2). The consumer pricing page lists Free, $0, no credit card, 10,000 tool calls a month and $2 in one-time credits (C3). Either way, signing up is free. Do not enter a card. If any screen asks for payment, stop and ask the user.
3. Install the Composio command line tool, which is how the agent will talk to Composio. The agent runs: `curl -fsSL https://composio.dev/install | sh`, then the user opens a new terminal window (C4). On Windows the docs mark the installer experimental (C4).
4. Log in. The agent runs `composio login` and the user finishes sign-in in the browser tab that opens (C4). If the machine cannot open a browser, the docs offer `composio login --no-wait`, which prints a link for the user to open, then `composio login --poll --timeout 120` (C5). Do NOT use `composio login --agent`: that creates a separate agent account that cannot see the user's own account or connections (C5).
5. Get an API key only if the agent will use the Composio SDK or web API instead of the command line tool. In the dashboard open Settings, then Project Settings, then API Keys, and copy the project API key (C9). Treat it like a password: never paste it into chat, never put it in the output folder or packet. The key is owned by the user. Save it only as COMPOSIO_API_KEY in the agent's own environment file, which must be outside the output folder and listed in git-ignore if that folder is ever a repository (C5). The command line route in steps 3 and 4 and 6 does not require copying a key by hand (C4); whether the CLI login creates the same project key is UNVERIFIED.
6. Connect Gmail. The agent runs `composio link gmail` (C4). A Google consent page opens for the user. Read it, confirm the Google account at the top is the one with the receipts, and approve. Hosted connect links expire after 10 minutes: if it fails, ask the agent to generate a fresh link and open it right away (C10). Success means the connection status shows ACTIVE (C10).
7. Connect Outlook, if the user has it. Composio's Outlook toolkit supports Composio-managed sign-in (C11), so the agent runs `composio link outlook`. The `link` command taking `outlook` is inferred from the Gmail example in C4, not shown in the docs, so UNVERIFIED until run. Whether a personal Outlook.com account consents without trouble is UNVERIFIED, no fetched page says. On a work or school Microsoft account, the organization may block the consent and need its administrator to approve; Composio's own Outlook notes say this (C11, C12). Do not use an administrator's login to get around it: Composio notes that doing so can connect the administrator's mailbox instead of the user's (C11).
8. Test it. The agent searches the user's mail with a small receipt query (for example the Gmail search tool `GMAIL_FETCH_EMAILS` with a Gmail-style query and a small max_results) and shows the user the subject lines it found, before running the full month (C6, C4). Composio's docs recommend small result sets and `include_payload=false` where supported to keep calls light (C7). How message attachments get downloaded from Composio was not found in the fetched pages (UNVERIFIED): if the agent cannot download an attachment, it lists that receipt as one the user must save by hand into `inbox/receipts/` as in Route B step 4.

What it costs (as read on 2026-11-18):

- Signup: $0, no credit card, on both free plans shown (C2, C3).
- Connecting Gmail: no separate charge was found. On Hobby the first 1,000 connected accounts on Composio-managed apps are free (C2). The consumer page allows up to 10 linked accounts per app on every plan (C3).
- Usage limits: Hobby 100,000 calls a month, of which 20,000 can use Composio-managed apps, hard-capped, so usage pauses at the cap instead of billing (C2). Free (consumer page) 10,000 calls a month, and the page lists $0.005 per extra tool call (C3). Whether that overage can bill a user who never added a card is UNVERIFIED. Paid tiers (Pro $29 a month, Standard $18 per seat a month) are not needed and the user should not upgrade for this skill (C2, C3).
- How many calls a one-person month-end sweep uses has not been measured. Measure it on a real run and stop if it nears a cap.

Whose Google app is used, and warnings the user may see:

- Default: Composio's managed Gmail OAuth app, shared with other Composio customers. The user does not need to create a Google Cloud project or bring their own OAuth client (C6, C13).
- Bringing your own Google OAuth client is optional and technical (Google Cloud project, consent screen, client ID and secret, test users). Composio documents it, and says Google may require app verification depending on configuration (C14). It gives your own app name on the consent screen and your own quota (C13). Do not send a non-technical user down this path.
- Does the user see a Google "unverified app" warning on the default managed app with its default scopes? UNVERIFIED. Composio's docs say Google shows "App is blocked" or an unverified-app screen when scopes beyond what Google approved for Composio's app are added (C6, C15), and that the toolkit's default scopes do not establish what Google approved (C15). They do not state the verification status of the default connection itself. Tell the user a warning screen is possible, and that if one appears with an "unverified" or "blocked" message they should stop and not click through; the agent then goes back to route B.
- Whether Google's restricted-scope review (the security assessment Google requires of apps that read Gmail on servers) has been completed for Composio's app: UNVERIFIED. No fetched page says.
- Default scopes the managed Gmail connection actually requests: UNVERIFIED in the fetched docs (C6, C15 say the default list is not shown). Do not tell the user what the default scopes are. Tell the user to read the consent page and compare it to the next point.

Least-privilege scope to request:

- Gmail: `https://www.googleapis.com/auth/gmail.readonly`, described by Google as "View your email messages and settings" (C16). Google classes it as a restricted scope (C16). Do not request `gmail.metadata` for this skill: it returns headers but not message bodies, and Composio says it cannot be combined with content scopes (C7, C6). Do not request `gmail.modify`, `gmail.send` or `https://mail.google.com/`, which can change, send or permanently delete mail (C16).
- Composio lets you override scopes on its managed app only within what Composio's app is set up to request; anything outside that needs your own Google app, and Composio tells you to confirm exact scope URLs with its support at support@composio.dev first (C15, C13). So whether `gmail.readonly` alone can be requested through the managed app is UNVERIFIED. If the consent page lists more than "view your email", say so to the user and go back to route B.
- Outlook: `Mail.Read`, which Composio's Outlook guide names as the minimum for reading full message content (C14b). Whether `User.Read` and `offline_access` are also needed is UNVERIFIED (not found in the fetched pages); add them only if the connection will not work without them, and tell the user.

How the user revokes access afterward:

1. Fastest, no Composio login needed: open Google's linked apps page, find Composio, and choose Remove access, then Confirm (C17). Google states that after removal the app cannot access the account's data, but data the app already copied is not deleted by that step, and the user may need to ask the developer to delete it (C17).
2. In Composio: delete the connected account. Composio's API supports revoking the grant at the provider and deleting the connected account; its docs say to revoke at the provider first, and that if provider-side revoke is not available, remove access in the provider's connected-app settings (C18, C10). The agent can run this if the user asks. Which dashboard button does the same thing was not found in the fetched pages (UNVERIFIED).
3. Then delete the Composio account or its API key if the user is done with it. The dashboard path for deleting an account or rotating a key was not found in the fetched pages (UNVERIFIED).
4. Outlook: Composio's page and Microsoft's own revocation page were not both read. Composio's revoke or delete (step 2) applies. Microsoft's own support page for removing app access could not be fetched (the URL tried returned 404), so the exact click path is UNVERIFIED. Do not guess it for the user.

Sources for route A. All fetched 2026-11-18. Anything not fetched is listed as such.

- C1 https://composio.dev/pricing/platform , signup entry points and plan details (read, 200)
- C2 https://composio.dev/pricing/platform , Hobby and Pro plan table, FAQ "Is the Hobby plan really free?" (read, 200)
- C3 https://composio.dev/pricing , consumer "for you" plans Free and Standard, overage rate, connection limits (read, 200). CONFLICT with C2 on limits: two different free tiers at two URLs, not reconciled.
- C4 https://docs.composio.dev/docs/cli.md , install, `composio login`, `composio link gmail`, `composio execute` (read, 200)
- C5 https://docs.composio.dev/docs/agent-setup/unattended-authentication.md , human login flow, `--agent` warning, key handling (read, 200)
- C6 https://docs.composio.dev/toolkits/gmail.md , Gmail toolkit: OAuth2, managed auth available, "App is blocked", tools (read, 200)
- C7 https://docs.composio.dev/kb/guide/toolkits-gmail.md , Gmail scopes, `gmail.metadata` limits, fetch tips (read, 200)
- C8 https://docs.composio.dev/docs/authentication.md , credentials held by Composio in default Cloud deployment, Connect Link flow (read, 200)
- C9 https://docs.composio.dev/reference/authenticating-to-composio.md , where to find the project API key (read, 200)
- C10 https://docs.composio.dev/kb/guide/platform-connected-accounts.md , 10 minute link expiry, statuses, revoke at provider (read, 200)
- C11 https://docs.composio.dev/toolkits/outlook.md , Outlook toolkit, managed auth, admin consent notes (partly read: first 100,000 of about 549,000 characters in the summarizer pass, plus a full download searched for scope and consent terms)
- C12 https://docs.composio.dev/kb/guide/toolkits-outlook.md , Outlook admin consent and reconnect notes (read, 200)
- C13 https://docs.composio.dev/docs/authentication/custom-app-vs-managed-app.md , managed versus own OAuth app (read, 200)
- C14 https://composio.dev/auth/googleapps , own Google OAuth app setup, verification may be needed (searched for verification terms only, not read end to end)
- C14b https://composio.dev/auth/outlook , `Mail.Read` minimum for message content (searched for permission terms only, not read end to end)
- C15 https://docs.composio.dev/docs/authentication/controlling-scopes.md , Google scope approvals, "contact support" instruction (read, 200)
- C16 https://developers.google.com/workspace/gmail/api/auth/scopes , Gmail scope list and restricted classification (read, 200)
- C17 https://support.google.com/accounts/answer/13533235 , remove a linked app's access (read, 200)
- C18 https://docs.composio.dev/reference/api-reference/connected-accounts.md , revoke and delete endpoints (grepped for revoke and delete only, not read end to end)
- NOT FETCHED: Microsoft's own page for removing app access (support.microsoft.com URL returned 404); Google's restricted-scope verification requirements page.

## Step 2: Month-end run

Run for one card and one statement month at a time. Unless just answered in Step 1 (question 3), ask which card (from the saved list), then which month, one question at a time. Create `runs/MM-YYYY-LAST4/`. Input: the card statement for that month, collected in Route B step 5 (a PDF or CSV in `inbox/statements/`). If the file is not there yet, ask for it and wait. Do not start without it.

1. Transcribe the statement into `runs/MM-YYYY-LAST4/ledger.csv`. Columns: post, tran, vendor, amount, from_image (yes or blank). One row per charge line. Dates written MM/DD/YYYY, converted from the statement's printed format without changing the date. When a printed date has no year, take the year from the statement period: the closing date's year, except a month later in the calendar than the closing month, which takes the year before (a December charge on a statement closing in January). A two-digit year gets 20 in front. If the statement has only one date column, put it in both post and tran and tell the user. Amount as a number with two decimals, no $ or commas; credits are negative rows. Keep the printed vendor text as it appears, with the masking rule from Data rules applied (a run of 8 or more digits is shown as X's plus its last 4). A line that wraps onto a second text line is one row: join the pieces with one space.
   - Pasted text is a supported statement input. A .txt file in `inbox/statements/` is a statement pasted from a PDF; parse it the same way as a PDF. Then show the user every parsed line (date, vendor, amount, one per row) and their sum, and have them confirm before any matching (see item 2). Pasted text can parse badly (wrapped lines, mixed date formats, trailing references); if it does not look like a statement the skill can parse, say so and ask for the issuer's PDF or CSV instead.
   - If the statement is a PDF, or pasted text that shows a printed net, write the printed net and any printed section subtotals into `totals.txt` in the same folder. A CSV, or pasted text with no printed net, has none: write "net: computed from rows" in `totals.txt` and tell the user so.
2. Check the ledger. For a PDF or pasted text with a printed net, row amounts must sum to the printed net in `totals.txt`, and where printed subtotals exist the charge rows must sum to the purchases subtotal and the credit rows to the credits subtotal; if not, stop and fix the transcription before matching. For a CSV the net is computed from the rows, so the sum cannot prove the transcription and must not be reported as a check of the statement; the item 7 check on a CSV only guards against a wrong status bucket or a missing file. Either way, ask the user to confirm the line count (charges and credits, after joining wrapped lines) against their own statement or their card issuer's website, and wait for the answer. Then, in a separate message, ask them to confirm the net. For pasted text, the parsed lines and sum already shown under item 1 replace these two questions: ask one question, "Do these lines and this sum match your statement?", and wait. Then, in a separate message, show the three largest rows by amount, ignoring sign (vendor, date, amount), and ask whether each matches their statement, because a sum alone cannot catch offsetting errors. If the user says one is wrong, correct it from the statement, redo the sum check, and ask again. For a CSV exported from the issuer, this confirms the transcription only, not the statement itself.
3. Search for each charge: in `inbox/receipts/` (Route B), and if Route A is set up, the connected mailbox too. Use vendor name variants, amount, and a date window of 7 days before and after the transaction date (receipts often lag). Save found documents as-is into `found/`. De-duplicate as in Route B step 9. Scan each document you intend to attach for a run of 13 to 19 digits that passes the Luhn check (Data rules). If one is found, do not copy it anywhere, tell the user which file, and ask whether to send that file as it is. If the user says no, the file is not put in `packet/` and the line is listed as "Receipt not found" with the note "document withheld by user". Never edit the document.

   One receipt, one line. A receipt is never matched to more than one statement line unless the user confirms it covers several (ask once per receipt). Two cases:
   - Duplicate charge: when two or more identical statement lines (same vendor, date and amount) share one receipt, exactly one line, the first in statement order, takes the receipt (HAVE). The others are listed as "Receipt not found" with the note "possible duplicate charge" for the user to resolve. Never count a single receipt as covering a double charge.
   - Two cards: read any card last 4 a receipt shows (for example "Visa ending 3456"). A receipt that shows a last 4 different from this run's card is never attached; neither is one that could match a line on another card (another statement for the same month in `inbox/statements/`, or a receipt already recorded in `hey-receipt.json` against another card), which matches one line only. Ask the user which card it belongs to (last 4 only), and record the answer (file name, card last 4, tran date, amount) in `hey-receipt.json`. If the user says it belongs to this run's card, it can be matched here. If the user does not know, handle it as in item 5 (receipt that matched no line): never attached. Only the card the user names takes it; the other card's line is listed as "Receipt not found" with the note "receipt belongs to the card ending LAST4 per user". If the other card's run is already built, tell the user that run needs redoing for that line. Never count one receipt on two cards.
4. Match each ledger line to evidence. A match needs all three: the vendor (name variants allowed), the exact amount, and a date inside the window. A near-miss is the same vendor with a different amount, or a date outside the window (this includes a prior-year document for the same subscription, and a document in another currency). A near-miss is NEVER counted as a match. A receipt with no vendor name is never matched on its own (item 5). A near-miss line starts as MISSING and item 5 may change it to HAVE. Assign one status and one category label (from the chosen set in Step 1, question 5):
   - HAVE: a receipt or invoice that matches on vendor, exact amount and date is attached; or a near-miss the user confirmed in item 5.
   - ROLLUP: many small charges from one vendor covered by a vendor-issued itemized report; attach the report and list the charges on a schedule.
   - PORTAL: nothing found, and the user (asked in item 5) or a document in hand says the receipt only exists behind a vendor login; list it for the user to download and drop in `inbox/receipts/`. The skill has no built-in vendor list and never assigns PORTAL by guessing from the vendor name; otherwise it is MISSING.
   - MISSING: nothing found, or only a near-miss the user has not confirmed. For a near-miss, copy the candidate into `found/`, put "possible match, unconfirmed: <its date>, <its amount>" in the notes column (no file name, because `found/` is never sent), and show it as a possible match in the could-not-match table. The file name is told to the user in Step 4. It is never counted as attached.
   - CREDIT: statement credit or refund line. It is listed as it is; the skill does not pair a refund with its original charge.
   - SUPPORTING: a document the user says supports the charge but is not a receipt for it (for example a vendor account page, or an order confirmation with no amount), labeled as such. Never used for a near-miss.

   These status names are internal. Every output (cover, worksheet, email, hand-off) shows the plain name instead: HAVE is "Receipt attached", ROLLUP is "Rollup schedule", PORTAL is "Paid via vendor portal", MISSING is "Receipt not found", CREDIT is "Credit", SUPPORTING is "Supporting document only".
5. Confirm possible matches. Before reconciling, take each near-miss one at a time and ask the user to confirm or reject it, showing the statement line (vendor, date, amount) beside the candidate (date, amount). Confirmed: the line becomes HAVE, the file goes in `packet/`, and the difference goes in notes and in the file name suffix. Rejected or unsure: the line stays MISSING. A confirmed document in another currency gets the suffix `_foreign-currency` and its document amount and currency in notes. Next, take each receipt file that matched no line and is not a near-miss candidate (for example a total-only receipt with no vendor), one at a time, and ask: "Which statement line does this belong to?", showing the file name and the amount and date read from it. If the user names a line, treat the user's answer as the vendor; the exact amount and the date window must still fit, otherwise it is a near-miss and goes through the confirm or reject step above. If the user does not know, copy the file into `found/`, list it in the hand-off by file name as "Supporting document only", and never attach it to a line or put it in `packet/`. Such a file is not a statement line, so it is not counted in the summary table. After that, ask the user once: "Do any of the lines still marked not found only exist behind a vendor login?" Lines the user names become PORTAL. Last, show the labels assigned, one line per vendor, and ask once: "Any label you want to change?" Apply the changes.
6. Disclose variances instead of hiding them: date offsets inside the window, card fees added by the vendor, charges billed for someone else, billed after a cancellation (only when the user says so; no step detects it), and any user-confirmed near-miss (amount difference, date difference, prior-year document for the same subscription, foreign currency), and any possible duplicate charge. Each goes in the notes column.
7. Reconcile. Buckets are: attached (HAVE), supporting (SUPPORTING), rollup (ROLLUP), outstanding (PORTAL plus MISSING), credits (CREDIT, negative). "Outstanding" is the internal bucket name; bookkeeper-facing text says "charges we could not match". The five must sum to the statement net, to the penny. Write a small script in `work/` to check it and run it. Also check every named file exists. If either check fails, do not produce the packet.

## Step 3: Packet format

Reproduce this shape and add nothing the bookkeeper did not ask for. Everything is written inside `runs/MM-YYYY-LAST4/` as in the layout above.

Folder `packet/` (this is all the bookkeeper receives, as the zip):

1. `00_COVER_reconciliation.pdf`, one cover document:
   - Title: "Receipt packet reconciliation, MM/YYYY statement"
   - Header line: entity name (from question 4) | issuer card ending LAST4 | statement date (the date printed on the statement; for a CSV, the last post date in the file, labeled "last post date") | N charge lines (every ledger row, credits included), net $X
   - Summary table with line counts and amounts, ending in a statement total that equals the net:
     - Receipts attached: N lines, $
     - Supporting documents only: N lines, $
     - Rollup schedule: N lines, $
     - Charges we could not match (Receipt not found and Paid via vendor portal): N lines, $
     - Credits: N lines, $
     - Statement total: N lines, $
   - Leave out any summary row whose count is 0, except the Statement total, and leave out any table below that would have no rows. The Receipts attached line count can exceed the file count when one confirmed file covers several lines.
   - Table "Receipts attached, N lines totaling $X" with columns TRAN date, VENDOR as printed on the statement, CATEGORY label, AMOUNT, FILE name.
   - Table "Supporting documents only, N lines" with the same columns plus a short NOTE saying what the document is.
   - Table "Charges we could not match" with the reason ("Paid via vendor portal" or "Receipt not found", plus any note such as "possible duplicate charge") and, where there is one, the unconfirmed possible match (its date, its amount; no file name).
   - Credit lines have no document and no file. They appear only in the summary table and the worksheet.
2. Receipt files, one PDF per charge that has a document, named `MM-DD_Vendor_Amount.pdf`. MM-DD is the TRAN date. Vendor is the printed vendor text with each run of characters other than letters and digits turned into one hyphen, case kept as printed, cut at 30 characters. Amount is the statement amount with two decimals and no $, even when the document shows a different amount (for example `03-12_ACME-HOSTING_41.00.pdf`). Cut and trim in this order: turn each run into one hyphen, remove hyphens at the start and end, cut at 30 characters, remove a trailing hyphen again. A file covering two charges, which is allowed only when the user confirmed it covers several (Step 2 item 3), carries both amounts in its name (`03-20_EXAMPLE-SUPPLY_62.10+18.25.pdf`). Notes that explain a variance go in the cover and worksheet, and in the file name only when the file is not the exact document, as a suffix (for example `..._2025-invoice-same-subscription.pdf` or `..._invoice-different-amount.pdf`).
3. Rollup schedule, only if there is at least one line with a "Rollup schedule" status (otherwise none), for example `00_VENDOR_itemization.pdf`: one row per charge with date, what it was for, amount, and the total tying to the statement lines.
4. Worksheet `worksheet.csv` with columns: post, tran, vendor, amount, status, category, file, source, notes. The status column uses only the plain names from Step 2 item 4 ("Receipt attached", "Supporting document only", "Rollup schedule", "Receipt not found", "Paid via vendor portal", "Credit"), never the internal names. Dates written MM/DD/YYYY; amount as in `ledger.csv` (two decimals, credits negative, no $ or commas); `file` is empty for "Receipt not found", "Paid via vendor portal" and "Credit" lines. `source` says where the document came from (for example inbox, rendered from email, rendered from text file, Route A). `notes` holds the variance notes and any unconfirmed possible match (date and amount, no file name).

Beside `packet/` in the same run folder (never inside `packet/`, never in the zip):

5. `email-body.txt`, the email text for the user to paste, with:
   - Greeting (use the bookkeeper name from question 6 if given) and the sentence "All N lines are accounted for in the summary below", followed by "M charges are listed as ones we could not match." (use "1 charge" for one, and "No charges are listed as ones we could not match." for zero; singular and plural of every count agree, for example "1 line", "2 lines"). This sentence says every line is listed, not that every receipt was found. Negative amounts are written "-$35.00" on the cover and in the email.
   - The summary table (the same rows as the cover)
   - "Charges we could not match:" followed by each vendor, date and amount, with any unconfirmed possible match (date and amount, no file name)
   - A one-line note per variance (prior-year invoice, billed for someone else, billed after a cancellation only if the user said so, user-confirmed near-miss, possible duplicate charge)
   - The attached-receipts list (every line in the "Receipts attached" table) and the rollup schedule inline; if there is no rollup, write "No rollup schedule this month"
6. `MM-YYYY-LAST4-receipts.zip`: made only after the path check passes. A zip of the contents of `packet/` only, written beside `packet/`, never inside it. Because only `packet/` is zipped, `hey-receipt.json` and anything from `inbox/`, `found/` and `work/` are never in it.

Rendering: print documents to PDF with a headless browser and a hard timeout (live pages can hang). An .eml or .txt receipt is first converted to HTML, then printed the same way. Use any headless browser that is installed (for example Chrome or Chromium run headless) with a 30 second timeout per file. Turn the browser's page headers and footers off: a default print adds the date and the page address (a local file path) to every page, including the cover. In Chrome or Chromium the flag is `--no-pdf-header-footer` (older versions use `--print-to-pdf-no-header`); for any other browser, finding its equivalent is UNVERIFIED, so check the result below. If the browser will not start as a normal user (for example a sandbox error), do not weaken its security settings on your own; tell the user and treat it as no browser installed. If none is installed, tell the user and ask them to print that message to PDF from their own mail or browser and drop it in `inbox/receipts/`; never skip the document. Render into `work/` and copy the result into `packet/`. Keep vendor PDFs verbatim.

Path check, required before the zip is made: extract the text of every PDF the skill generated (the cover, every rendering, the rollup schedule) and search it for `file://`, `/tmp`, `/home/`, `/Users/`, a drive letter such as `C:\`, and the output folder's own path. If any is found, do not finish: delete that PDF, render it again with headers and footers off, and check again; if it still shows a path, stop and tell the user which file. If no tool to read PDF text is installed, tell the user the packet could not be checked and do not finish. Vendor PDFs are verbatim and are not part of this scan.

The cover is plain reconciliation. Category labels appear only as a per-line label. No category totals, no deduction language.

## Step 4: Hand off

Tell the user: where the run folder is (`packet/`, the zip, and `email-body.txt`), the counts, the charges we could not match (with any unconfirmed possible match, including its file name from `found/`), any "Supporting document only" lines that still lack an actual receipt, any possible duplicate charges to resolve, the "Paid via vendor portal" downloads they need to do by hand, and any receipt files not attached to a line (by file name from `found/`, as "Supporting document only", or as belonging to another card). This run covered one card and one month. Ask: "Do you want to do another statement?" If yes, Step 2 asks which card and then which month, one question at a time; go back to Route B step 5 for the new statement, which repeats the Statement check there (card last 4, digit-run scan, line count), then continue to Step 2 (a new run folder). Add Route B steps 2 to 4 and 7 again only if the user has new receipts to add. The user sends it. The skill does not send email.

## Wording guardrails

Say: "Finds receipts in your email and uploads and matches them to your card statement. Produces an organized report for you or your bookkeeper to review." "Charges we could not match." "We do not prepare, file, or advise on tax returns." Every user-facing and bookkeeper-facing output (cover, worksheet.csv, email, hand-off) uses "charges we could not match", "Receipt not found" and "Paid via vendor portal", never "outstanding" or "missing" or any internal status name from Step 2 item 4; those are internal names.

Do not say: tax-ready, audit-proof, IRS-compliant, maximize deductions, never miss a deduction, we found everything, Schedule C filled in, we recommend you deduct. Do not claim a matched statement line replaces a receipt.

## Known limits (v0.1)

- This skill was tried on made-up data only. It has not been tested on real statements.
- It handles the statement shapes it lists: a CSV export, or text pasted from a PDF statement. A PDF you drop in directly is read too, but an unusual statement can be misread.
- If it misreads a line, it says so by asking you to check the parsed lines and the sum against your own statement. Please do check them.
- It is bookkeeping only. It never gives tax advice. Ask your preparer anything about taxes.
- Route A (connecting your mail through Composio) is not recommended until its UNVERIFIED items are resolved. Use Route B.

## Open items before release

- Route A stays gated: the Gmail verification status of Composio's managed app and the default scopes it requests are UNVERIFIED. Route B is the default until both are confirmed from a fetched source.
- Decisions left open: category set (a) has no dues, licenses or referral label (a realtor who is not a brokerage owner has to use Other or paste a list); the "All N lines are accounted for" sentence in `email-body.txt` reads oddly when many lines are unmatched; "Still correct?" at the start of a later run confirms a several-item read-back in one question; the receipt file name cut at 30 characters can end mid-word.
- Trademark search for HeyReceipt and the HEY mark; not cleared.
- No statement parser exists; the ledger is transcribed from the statement by the model. Step 2 asks the user to confirm line count, net and the largest rows.
- Counsel should review the Step 1 question 5 category sets and the cover wording before release.
- Provider click paths for Route B (Gmail, Outlook search, filter and forwarding menus) are UNVERIFIED and need to be checked against the providers' own help pages.
