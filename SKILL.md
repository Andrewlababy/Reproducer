---
name: reproducer
description: >-
  Helps a product manager triage user bug reports: reproduces each report in a browser
  with playwright-cli and classifies it as a defect, likely user error, or insufficient information.
  Produces numbered reproduction records (W1, W2, ...) as a list plus detail pages with
  step-by-step screenshots, then pushes them to the tool the user chooses (Notion, Jira,
  Linear, Slack, and so on) or keeps them local. Use when the user says reproducer,
  reproduce user feedback, user feedback, bug report triage, report mode, assistant mode,
  or reproduction assistant.
---

# reproducer

A product manager receives user bug reports and uses this skill to check each one before handing it to engineering. Reproduce only the product behavior that the report describes (a click does nothing, data is wrong, a flow cannot be completed). Do not write exploit steps, build attack payloads, or demonstrate remote code execution.

This skill defines only the content and structure of the reproduction records. Which tool receives them, and how to call that tool, is up to the executing agent with the integrations it already has.

Use only the global `playwright-cli` command for the browser.

## Configuration

Read `config.md` in this skill directory.

- `default_mode` is `report` or `assistant`: use that mode, tell the user in one sentence, and do not ask again.
- Empty: before any analysis, ask the user to choose report mode or assistant mode.
- The user sets a default (for example "always use report mode"): write it to `default_mode` and use it for this run.
- For `destination` and `destination_ref`, see "Push".

The two modes differ only in what the user wants: in assistant mode the user clicks through the steps personally (more confidence, more accuracy); in report mode the agent runs the whole check. Do not choose a mode for the user based on how clear the report is.

## Before you start

1. List what you still need, then ask. Usually this is the URL and login credentials. Do not ask for anything the report already gives.
2. Use credentials only to log in during this run. Do not write them to records or to `config.md`. Do not run `state-save` on the session.
3. If one message contains several reports, split them first and handle them one at a time. Assign record numbers only when writing records (see "Numbering"). While splitting, refer to them as "report 1", "report 2".

## Screenshots

- Use `playwright-cli screenshot --filename=<path>`. The path is `reproducer-reports/<batch date>/<report index>/step-<two-digit number>.png` under the current workspace. Create the folder if it does not exist.
- Before a screenshot, if a username or password is visible in plain text, blur that input first: `playwright-cli eval "el => el.style.filter='blur(6px)'" <ref>`. Do not skip the screenshot for that step to hide the credentials.

## Capture rules (both modes)

- One step contains exactly one visible action (open a URL, click, type into one field, select, scroll, open a menu).
- Do the action first, then take the screenshot. The screenshot must show the screen after that action.
- The next step starts from the screen in the previous screenshot. Split login too: open the login page, type the username, type the password, click log in. Each is one step.
- Do not write steps that were not performed. If you cannot continue, stop. In the last step, state where you got stuck and what the screen shows.

## Verdict (choose one)

Write the verdict in the detail page. It is not a list field.

| Verdict | When to use | Status |
|---|---|---|
| Defect | Following normal usage, the reported behavior appears | Reproduced |
| Likely user error | Following normal usage, the behavior does not appear, and the report matches a usage mistake (wrong click, missing input, wrong permissions) | Not reproduced |
| Insufficient information | All known steps are done, the behavior does not appear, and there is no sign of a usage mistake | Not reproduced |

If a different behavior appears instead of the reported one: set the status to Not reproduced and describe what you actually saw.

## Report mode

1. `playwright-cli open <url>` (headless is fine).
2. Follow the capture rules from login until the reported behavior appears or you cannot continue.
3. `playwright-cli close`.
4. Write the record, then push it.

## Assistant mode

1. Turn the fragmented report into steps the product manager can follow. Step 1 is the first action after login is complete. One action per step, no skipped steps.
2. `playwright-cli open --headed <url>`. You must pass `--headed`, or the user cannot see the window.
3. Follow the capture rules for each login step and take a screenshot after each one. Stop on the screen after login is complete, which is where step 1 starts. Do not perform step 1.
4. Send the user all steps starting from step 1, and tell them to continue in the browser window that is already open. Keep the browser open.
5. After the user tells you what they actually saw, run `playwright-cli close`, then choose the verdict, write the record, and push it. The capture contains the screenshots of each login step. Write the remaining steps as given, mark each one "Performed manually by the product manager", and end with the result the user confirmed. Do not write the record before the user replies.

## Reproduction record

A record has two levels: the **list** has one row per report, and the **detail page** is what you see when you open a row.

The list has exactly these four fields:

| Field | Value |
|---|---|
| ID | W1, W2, ... |
| Title | One sentence |
| Status | Reproduced, Not reproduced |
| URL | The address where the problem occurs |

Write the whole record in the user's language: field names, status and verdict labels, headings, and body text. The tables and template in this skill are in English only as a reference; translate them and keep the same structure. Do not add recommendations or severity.

```markdown
# W1 Title

## Verdict
Defect

## Description
The behavior in the original report, plus the behavior observed in this run.

## Impact
Who is affected and which task they cannot complete.

## Capture
1. Open the login page.
![Step 1](W1/step-01.png)
2. Type the username in "Username".
![Step 2](W1/step-02.png)
3. Type the password in "Password".
![Step 3](W1/step-03.png)
4. Click "Log in".
![Step 4](W1/step-04.png)
5. Click "Some button".
![Step 5](W1/step-05.png)
```

### Local records

Whatever the destination, first write the local records under `reproducer-reports/<batch date>/`:

- `index.md`: the list as a table. Each title links to its detail file.
- `W<n>.md`: the detail page. Reference screenshots by relative path as `W<n>/step-xx.png`. When writing the record, rename the screenshot folder from the report index to `W<n>`.

### Numbering

Before writing records, find the highest existing ID in the destination (when `destination` is `local`, check every `index.md` under `reproducer-reports/`) and continue from the next number. If there is none, start at W1. When writing several records at once, increment for each.

## Push

1. `destination` is `local`: stop after the local records and tell the user the path.
2. `destination` has another value: use the matching integration that is already connected to push the records to `destination_ref`.
3. Empty: ask the user which tool and which location to push to; the user can also choose local only. After the user agrees, write `destination` and `destination_ref`.

Keep the structure when pushing:

- Map the list to the tool's list unit (database row, issue, ticket, message thread, and so on). All four fields must be present. Put the status in the closest field the tool offers.
- Map the detail page to the body of that item. Put each screenshot as an image or attachment under its step.
- If the destination has no place for the list yet (for example, no database exists): ask the user before creating one, then write it to `destination_ref`.
- If a screenshot fails to upload: write its local file path in that step, and tell the user which screenshots were not uploaded.
- If the tool the user chose is not connected in the current environment: stop at the local records and tell the user to connect that tool first.
