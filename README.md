# Job Application Tracker — Power Automate Flow

A working, tested Power Automate cloud flow that automatically logs job
application emails from my own Gmail inbox into a live Excel tracker —
built to demonstrate hands-on, practical experience with Microsoft Power
Automate, directly matching job postings that ask for "knowledge of
Power Apps (e.g. Power Automate)."

This isn't a synthetic demo. It watches my real Gmail account
(the one I actually use to apply for jobs) and logs real
application-related emails as they arrive, splitting them into
**Applied** vs **Rejected** based on keyword detection — in both
English and German, since I'm applying to roles in Germany and expect
correspondence in either language.

## What it does

1. **Trigger:** "When a new email arrives" (Gmail connector), filtered
   to only fire on emails whose subject contains application-related
   keywords (`application`, `applied`, `bewerbung`, `absage`,
   `rejection`, `interview`).
2. **Condition:** Checks both the email's **Subject** and **Body** for
   rejection-style keywords in English and German (`unfortunately`,
   `regret`, `leider`, `absage`).
3. **Branching:**
   - If a rejection keyword is found → adds a row to the Excel tracker
     with **Status = "Rejected"**.
   - Otherwise → adds a row with **Status = "Applied"**.
4. Each logged row captures: Date Applied (from the email's received
   time), Company, Position (from the email subject), the original
   Source Email Subject, and Status.

## Why this design

- **Real, ongoing use, not a one-off test.** The flow runs against my
  actual job-search inbox, so it's a genuinely useful tool I built for
  myself, not just a portfolio artifact.
- **Bilingual keyword matching.** Since I'm applying for roles in
  Germany, correspondence can come in German or English — the condition
  checks both languages rather than assuming English-only, which is a
  detail that's easy to miss but matters in this context.
- **Subject *and* body checked.** Many rejection emails don't put
  rejection language in the subject line (e.g. "Update on your
  application to ABB") — the actual "unfortunately..." wording is
  usually in the body. Checking both catches more real cases than
  subject-only matching would.

## A real bug I hit and fixed

During testing, emails with "Unfortunately" (capitalized, as it normally
appears at the start of a sentence) were incorrectly logged as "Applied"
instead of "Rejected." The root cause: Power Automate's `contains`
expression is case-sensitive, and my condition was checking for the
lowercase keyword `unfortunately` — so `Unfortunately, your application...`
never matched. I fixed it by wrapping both the Subject and Body values in
a `toLower()` expression before the comparison, verified in the run
history's Code view, and re-tested to confirm both the "Applied" and
"Rejected" branches now log correctly. Screenshots of both the bug and
the fixed, successful run are in `screenshots/`.

## Known limitations (being upfront about scope)

- The **Company** field is currently a fixed placeholder value rather
  than being extracted automatically from the sender or body — accurate
  company-name extraction would need additional parsing logic (e.g. a
  Compose action with string functions, or an AI-based extraction step)
  that was out of scope for this first version.
- If a company later sends a rejection for an application that was
  already logged as "Applied," this flow **adds a new row** rather than
  updating the original one (so a single application can end up with
  two rows: one Applied, one Rejected). A more advanced version would
  use "Get rows" + "Update a row" matched by company name to update the
  existing entry in place — a deliberate next step, not an oversight.
- Keyword matching is simple string-contains logic, not natural language
  understanding — it will miss rejections phrased without any of the
  listed keywords, and could rarely misfire on a coincidental keyword
  match in an unrelated email.

## Screenshots

See the `screenshots/` folder for:
- The flow's designer view (trigger, condition, both branches fully
  configured)
- A successful test run in the flow's run history
- The Excel Online tracker showing a logged row after a real test

## Tech used

Power Automate (cloud flow), Gmail connector, Excel Online (Business)
connector, Condition control with OR/contains logic.
