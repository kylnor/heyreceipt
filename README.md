# HeyReceipt

A free helper for the end of the month. You give it your card statement and the receipts from your own email. It matches each charge to a receipt, lists the charges it could not match, and builds a packet that you or your bookkeeper can review.

It is made for people who run their own work, such as realtors and freelancers, and who dread the monthly receipt hunt.

We do not prepare, file, or advise on tax returns.

## What it does

- Reads one card statement (a CSV, or text pasted from a PDF) for one month.
- Matches each charge to a receipt by vendor, exact amount and date.
- Lists possible matches for you to confirm and never counts them on its own. A charge with no receipt is listed as "Receipt not found".
- Builds a packet folder, a zip, and a short email text that you paste and send yourself.

It is bookkeeping and reporting only. It does not say what is deductible and it does not give tax advice. Ask your preparer anything about taxes.

## How it gets your receipts

**Route B (the default).** You save receipt emails into a folder on your own computer. No account is connected and no third party is involved. The skill reads only the files you put there.

**Route A (not recommended yet).** A free Composio account reads your Gmail for you. This hands a copy of your mail-reading permission to a third party, and some safety questions about it are still unanswered. Use Route B for now.

## Install it as a Claude skill

1. Get this folder onto your computer.
2. Copy the folder `skills/hey-receipt` into your Claude skills folder, for example `~/.claude/skills/`.
3. Start a new Claude session and ask it to help with your month-end receipts. It will walk you through setup one question at a time.

Your files stay on your machine. Your mail provider and your AI model still see any email content they handle, so check your AI provider's data settings first.

## Known limits (v0.1)

- This skill was tried on made-up data only. It has not been tested on real statements.
- It handles the statement shapes it lists: a CSV export, or text pasted from a PDF statement. A PDF you drop in directly is read too, but an unusual statement can be misread.
- If it misreads a line, it says so by asking you to check the parsed lines and the sum against your own statement. Please do check them.
- It is bookkeeping only. It never gives tax advice. Ask your preparer anything about taxes.
- Route A (connecting your mail through Composio) is not recommended until its open safety questions are answered. Use Route B.

## License

MIT. See LICENSE and NOTICE.md.
