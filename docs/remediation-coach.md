# Remediation Tracker

The Remediation Tracker checks whether the recommendations from an [AI Insights](ai-insights.md) run were fixed in later audit runs. It works from the files A11y Audit already saves (`.a11y/results/` and `.a11y/ai/`). It sends nothing anywhere, calls no model and changes no file it reads.

It closes the loop: AI Insights says what to fix, you fix it and run the audit again, and the tracker says which recommendations the automated re-run no longer finds.

## How to use it

1. Run **A11y: Run Audit**.
2. Run **A11y: Generate AI Insights (advisory)** for that run.
3. Fix things, then run **A11y: Run Audit** again. You do not need to generate AI Insights again for this run.
4. Open the **Remediation Tracker** node in the A11y Results view, or run **A11y: Export Remediation Tracker**. The command writes two files to `.a11y/reports/`:
   - `remediation-tracker-<run>.html`, which opens in the browser;
   - `remediation-tracker-<run>.csv`, for a Jira import.

The tracker uses the newest audit run that has AI Insights and a later run of the same site (`baseUrl`) to check them against. That run is the "analysed run". The tracker compares it with the newest later run of the same site, and each later run also gets a row in the run history.

If you generate AI Insights for the re-run as well, the earlier analysis stays tracked until there is an audit run after the newer one. The tree, the HTML and the message then say that the newer AI Insights are tracked after the next audit. Only when no analysed run has a later run is everything "not checked".

- **No AI Insights yet:** nothing is written, and the message offers **Generate AI Insights**.
- **No later run yet:** the files are written with every recommendation marked "not checked", and the message offers **Run Audit**. In the tree, **Run an audit to check the recommendations** is a separate item, so a click on the tracker node itself never starts an audit.

## Statuses

| Status | Meaning |
|---|---|
| fixed | No element of the recommendation is found by the automated re-run. |
| partially fixed | Some of its elements are no longer found (shown as resolved/total). |
| not fixed | Every element is still found. |
| regressed | A new violation in the later run belongs to this recommendation with high or medium confidence (see below). This wins over the other statuses. |
| unknown (not comparable) | Every element is on a page that could not be compared: it failed or the session expired in one of the runs, or it is no longer audited. |
| needs human review | The recommendation covers only "review" findings. They depend on a person's decision, so they are never tracked automatically. |
| not checked | There is no audit run after the analysed run yet. |

Elements are counted per page, state, viewport, rule and selector, as in **A11y: Compare Runs**. Elements on pages that cannot be compared are not part of resolved/total. The HTML and the CSV list them as "not comparable".

Recommendations that the AI run could not analyse are not tracked.

## New violations and confidence

Each violation that the later run finds and the analysed run did not is listed as a new violation. The tracker matches it against the recommendations with the same rule, state and viewport:

| Confidence | Condition | Effect |
|---|---|---|
| high | Same page, and the same element pattern as one of the recommendation's elements | The recommendation becomes **regressed**. |
| medium | Same element pattern, on another page ("likely same cluster") | The recommendation becomes **regressed**. |
| low | No element pattern matches, and only one recommendation has this rule in this state and viewport ("possibly related") | Shown next to the recommendation; its status does **not** change. |
| none | Anything else, including two possible recommendations | Listed on its own. |

The element pattern is the selector with list positions and numbers removed. For example, `ul > li:nth-child(3) > a` and `ul > li:nth-child(1) > a` have the same pattern.

Violations on pages that only the later run audits are not compared, and do not show up as new.

## Limits

- **The status comes from the automated re-run, not from a conformance check.** "fixed" means the automated check no longer finds the violation. It does not mean the page meets WCAG, and it never changes the checklist or the VPAT.
- **Selectors are the only identity.** When markup changes, a fixed element can come back with another selector. It then shows as fixed, plus a new violation (often with low confidence).
- **Generic selectors** such as `button` or `a` make medium matches less meaningful. Low confidence never changes a status for this reason.
- **High and medium need the same element pattern.** axe often names an element by an attribute, such as `a[href="/c"]`, or only by its tag, such as `button`, when it is the only one on the page. A new element of the same rule then gets low confidence, not high, so the recommendation keeps its status. That is on purpose: a new empty button in the footer is a new problem, not proof that the menu button fix failed.
- **What to do with low confidence:** each new violation is listed under the recommendation it is possibly related to, and is exported as a separate Bug. Check whether the same fix applies (often it does, for example a shared component), and fix it the same way.
- **Configuration changes:** if `a11y.config.json` changes between runs (pages, states, viewports), the affected elements become "unknown", not "fixed".
- **AI text:** recommendation titles, rationale and steps are AI-generated and need human review. They are shown escaped and never as links.

## Importing the CSV into Jira

The CSV is in the standard Jira CSV import format. Each recommendation becomes one row with Issue Type `Task`, and each new violation one row with Issue Type `Bug`.

| Column | Content | Map to |
|---|---|---|
| Summary | Recommendation title, or `New: <rule> on <page>` | Summary |
| Description | Rationale, action, verification steps, rule, pages, status, the two runs compared, and a note that the text is AI-generated | Description |
| Issue Type | `Task` or `Bug` | Issue Type |
| Priority | `Highest`, `High`, `Medium`, `Low` or `Lowest`. For a Task it comes from the deterministic severity; for a Bug from the axe impact. Never from the model. | Priority |
| Labels (3 columns) | `a11y`; `a11y-recommendation` or `a11y-regression`; `wcag-<criterion>` | Labels |
| Coach Status | the status above, or `regression` | a custom field, or do not map |
| Elements resolved | for example `1/2 (1 not comparable)` | a custom field, or do not map |
| Regression confidence | `high`, `medium`, `low` or `none` | a custom field, or do not map |
| Cluster ID | the AI Insights cluster | a custom field, or do not map |
| Owner, Target date | empty in this version | Assignee and Due date, once you fill them in |

In Jira, open the CSV importer (**Import issues from CSV**), choose the file and map the columns as above.

- **Labels:** Jira reads several labels from columns with the same name. If your Jira puts all three into one label, map only the first `Labels` column.
- **Priority names** must exist in your project's priority scheme. Otherwise, map the values in the importer.
- **Required custom fields** of your project are not in the CSV. Give them a default value in the importer, or add a column yourself.

This format has not been tested against a specific Jira project. Try a small import first.

## Not in this version

These are planned for a later version:

- owner, target date and Jira key entered in VS Code (the tracker keeps no state of its own yet);
- a direct Jira API integration;
- a tracker section in the main report;
- an AI explanation of why something regressed.
