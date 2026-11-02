---
title: "Translation Workflow Automation"
description: "Python scripts connecting Wrike and XTM to automate a global translation workflow — from 8 analyst-hours a week of admin to a 10-minute morning routine."
summary: "Python scripts connecting Wrike and XTM to automate a global translation workflow — from 8 analyst-hours a week of admin to a 10-minute morning routine."
date: 2026-11-02
draft: false
params:
  seo:
    title: ""
    description: ""
    canonical: ""
    robots: ""
---

Python automation that connects Wrike (request intake) and XTM
(translation management) through their APIs, eliminating most of the
manual administration around global marketing translations at
Hyster-Yale. Built by a marketing analyst with no prior API experience,
alongside a full-time role.

![Workflow diagram showing the nine steps of the translation process, with system steps, Python-automated steps and human checkpoints colour-coded](/images/translation-workflow-diagram.svg)

*The full workflow at a glance. Grey boxes are system steps, blue boxes
are automated by Python, amber boxes are human checkpoints that were
deliberately kept manual. Each one is explained below.*

## The problem

Translation sits underneath a surprising amount of global marketing
work: website localisation, SEO-focused content, product manuals, blog
posts and brand material, all of it needing to exist in multiple
languages. My team ran this through a process that had been in place for
years and never substantially revisited.

A requester submitted a form in Wrike with their details, region, team,
source files and target languages. From there, an analyst carried the
project by hand:

1. Review the request and its files.
2. Create the matching project in XTM.
3. Select the project type and upload the source files.
4. Wait for XTM to analyse the project and generate costs.
5. Review costs and purchase orders.
6. Send the purchase orders to the right agencies.
7. Monitor while agencies completed the translations.
8. Download the finished files and return them to the Wrike task.
9. Check the files and close the request.

On top of that, files travelled through Box, and everything was tracked
in a large, password-protected Excel workbook that only one person could
use at a time. Three analysts collectively spent around eight hours a
week on this administration. That figure came from the analysts' own
time records and was confirmed by my manager, not estimated after the
fact.

The workflow was essential. What was wasted was skilled analyst time
being spent moving information between systems.

## The systems involved

| System | Role in the workflow |
|---|---|
| Wrike | Request intake, task tracking, and the place requesters receive completed files |
| XTM | Translation project management, analysis, cost generation, purchase orders, agency workflow |
| Python | The glue: API integration, file transfer, cost reporting |
| Azure DevOps | Version control for all scripts |
| Box | Part of the former manual file-transfer process (made obsolete) |
| SAP | Legacy system for final month-end purchase-order processing (kept manual) |

One small design decision mattered a lot: the Wrike task ID becomes the
XTM project name. Every translation project is therefore traceable back
to its original request without any extra tracking. The XTM project ID
stays a separate identifier and is what the file-retrieval script uses
later.

## How the automation works

The solution is a set of scripts, not one monolith, so each stage can be
run, checked and debugged independently. The flow matches the diagram
above:

1. **Intake stays in Wrike.** Requesters keep using the same form. No
   behaviour change on their side, which mattered for adoption.
2. **Python moves the request into XTM.** The script reads the new Wrike
   task, creates the corresponding XTM project and uploads the source
   files and reference material.
3. **XTM analyses and prices the project.** Costs generate per target
   language.
4. **An analyst reviews before anything is committed.** Agency
   assignments, project settings and costs get a human check. Projects
   above €1,000 are flagged to the manager. Only then are purchase
   orders released.
5. **Purchase orders go out through XTM's own functionality.** I used
   the platform's built-in PO workflow rather than building a separate
   emailing integration.
6. **Agencies do the translation in XTM** as they always have.
7. **An analyst triggers retrieval when the work is actually done.** The
   script only runs when someone enters the XTM project ID.
8. **Python downloads the translated files and uploads them** to the
   original Wrike task, so everything lands where the requester is
   looking.
9. **An analyst verifies and closes.** Files are checked, language-code
   suffixes on filenames confirmed, and only then is the Wrike task
   marked complete, which notifies the assignees.

## Why three steps stayed human

The organisation spends around €2 million a year on translation, so
"the API can do it" was never a good enough reason on its own. The three
checkpoints protect three different kinds of risk:

- **Financial, before agency handoff.** Automating project creation
  should not mean automatically committing spend. Cost review and the
  €1,000 manager-awareness threshold stay with people.
- **Operational, before files return.** A project can look finished
  while linguists still have open questions about the source content.
  Returning files too early creates rework and can force a linguist to
  reopen a project. A human decides when "done" is actually done.
- **Quality, before completion.** The final check catches upload errors
  and wrong or missing language suffixes before the requester is
  notified.

This framing was also how I got a sceptical team on board: the
conversation was never "the robots are taking the process", it was
"the boring 80% goes away and you keep every decision that matters".
By the time the analysts saw it working on real projects, the scepticism
was gone.

## Error handling and logging

Every script writes stage-specific logs, so when something fails you can
see which stage succeeded and where it stopped. A requester occasionally
submits a PDF where the workflow expects a translatable source format;
the logs make that diagnosable in minutes rather than hours.

That mattered because the people operating the workflow do not read
Python. The logs let an analyst identify what failed and send me the
relevant lines without needing to understand the code behind them.

Two conventions keep things safe:

- Suspected duplicate project IDs get visible suffixes (`_1`, `_2`) so
  they can be spotted and investigated. This is a visibility convention,
  not full idempotency, and I describe it as what it is.
- All scripts live in Azure DevOps with credentials kept out of the code
  in managed secret configuration.

## How it was tested

No formal automated test suite existed, so I tested end to end with a
fake project: a lorem ipsum document pushed through the entire live
workflow. Before the test purchase order went out, I told the receiving
agency it was a demo that must not appear in monthly reconciliation as a
real project.

The sample ran through both systems, the demo PO landed on the agency's
side, and they confirmed the project and cost information were visible
to them. Costs were naturally tiny because nothing real was translated,
which was the point: the test proved information flowed through
correctly, not that translation was happening.

This is honest manual end-to-end testing with sample projects, and it
worked, but automated tests for language mappings, identifiers, data
fields and business rules are first on my list for the next version.

## Adapting when the world changed

An integration does not run in isolation, and this one got two reminders
of that.

**New languages.** New target languages could be approved and added to
the Wrike form without the person maintaining the scripts being told. I
set up a Wrike notification to alert me when the form changed, then
updated the language mapping in Python. It is a practical workaround
that still needs a human edit; centralised configuration would be
better.

**XTM Cloud migration.** About six months after go-live, the
on-premises XTM was retired and we moved to XTM Cloud. I expected a
base-URL change. Instead the new domain was blocked by the company
firewall and connections failed with an SSL handshake error. Routing
through the company VPN resolved it immediately, which confirmed the
firewall was the blocker. The VPN requirement stayed, but the workflows
carried on working.

## The second script: month-end reporting

The main workflow was not the only time sink. At month-end, one analyst
spent roughly two to two and a half days retrieving and reconciling
cost data across the technical-publication programme: around 800 to 900
projects.

A separate Python script takes a date range, pulls all matching projects
from XTM and exports a CSV with project ID, project name, source and
target languages, total language cost and total project cost. The costs
are the ones XTM already generated; the script does not recalculate or
audit them. The analyst totals the relevant figures, compares them
against the standard-publication reconciliation sheet, and enters two
totals into SAP, where the legacy version makes further automation
impractical.

That process now takes about two hours.

## Results

| Area | Before | After |
|---|---|---|
| Main translation admin | ~8 analyst-hours/week across 3 analysts | ~10–15 minutes each morning for one analyst |
| Month-end technical-publication reporting | ~2–2.5 days for one analyst | ~2 hours |
| File and project movement | Manual downloads, uploads and re-entry | Key transfers handled by Python |
| Oversight | Manual checks throughout | Human approval kept at agency handoff, costs and completion |

One honesty note: the eight hours is a weekly figure across three
people; the 10–15 minutes is the daily routine for one. They are not a
like-for-like percentage, and I do not present them as one.

What this project did not do is just as bounded: it did not improve
translation quality, SEO rankings or revenue, and I have no evidence it
did. What it did was give the team a working day back every week and
make month-end reporting bearable.

## What I would build next

Persistent run state and idempotent retries instead of duplicate
suffixes, status polling to replace the fixed wait for XTM cost
generation, automated tests around mappings and business rules,
proactive failure notifications, a proper runbook with a backup
administrator, and centralised language configuration.

## Stack

Python (REST APIs) · Wrike API · XTM API · Azure DevOps · managed
secrets · Copilot as a learning and debugging aid

[Read the full story, including what it felt like to learn APIs from
scratch, the cybersecurity approval process and the Friday nights →](/log/2026-11-02/)
