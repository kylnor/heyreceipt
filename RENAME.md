# Renaming the product

The product name appears as "HeyReceipt" (display) and "hey-receipt" (slug, folder, file names). To rename, search and replace both forms, then rename the folder `skills/hey-receipt`. Also check the skill's output file names it writes at runtime (`hey-receipt.json`, `hey-receipt-folder.txt`, a default folder named `hey-receipt`); they are inside SKILL.md and covered below.

Search command (run from the repo root):

    grep -rnE 'HeyReceipt|hey-receipt|Hey Receipt|heyreceipt' --exclude-dir=.git .

Domain and repo name `heyreceipt` are outside the files and need changing separately.

Path:
- skills/hey-receipt/ (folder name)

Occurrences (file:line, number of matches on that line):

- site/index.html:6 (1)
- site/index.html:27 (1)
- site/index.html:55 (1)
- NOTICE.md:1 (1)
- skills/hey-receipt/SKILL.md:2 (1)
- skills/hey-receipt/SKILL.md:6 (1)
- skills/hey-receipt/SKILL.md:20 (1)
- skills/hey-receipt/SKILL.md:31 (1)
- skills/hey-receipt/SKILL.md:44 (1)
- skills/hey-receipt/SKILL.md:45 (2)
- skills/hey-receipt/SKILL.md:59 (6)
- skills/hey-receipt/SKILL.md:61 (1)
- skills/hey-receipt/SKILL.md:63 (1)
- skills/hey-receipt/SKILL.md:67 (2)
- skills/hey-receipt/SKILL.md:74 (1)
- skills/hey-receipt/SKILL.md:103 (1)
- skills/hey-receipt/SKILL.md:194 (2)
- skills/hey-receipt/SKILL.md:241 (1)
- skills/hey-receipt/SKILL.md:271 (1)
- README.md:1 (1)
- README.md:28 (1)
