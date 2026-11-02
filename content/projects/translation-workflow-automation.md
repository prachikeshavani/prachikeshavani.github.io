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

Python automation that connects Wrike (intake) and XTM (translation
management) through their APIs, eliminating most of the manual
administration around global marketing translations at Hyster-Yale.

## The problem

Three analysts were spending around eight hours a week, collectively,
on translation administration: re-creating projects by hand, moving
files through Box, and tracking everything in a years-old Excel
workbook that one person could use at a time. The work was essential,
but the people doing it were moving information between systems instead
of analysing anything.

## What I built

A set of Python scripts that talk to the Wrike and XTM APIs and handle
the repetitive parts of the workflow:

- Transfer request details and source files from a new Wrike task into
  XTM automatically (the Wrike task ID becomes the XTM project name, so
  every project traces back to its request).
- Retrieve translated files once an analyst triggers the retrieval
  script, and upload them back to the original Wrike task.
- Export month-end cost reporting for roughly 800 to 900
  technical-publication projects to CSV, replacing 2+ days of manual
  extraction and reconciliation.

## The design principle: humans keep the decisions

The automation deliberately left three checkpoints with people: analyst
review of agency assignments and costs before purchase orders go out,
a human trigger before completed files return (translation projects can
look finished while linguists still have open questions), and a final
file-and-language-suffix check before a task closes. The company spends
around 2 million euros a year on translation, so automating the spend
itself was never the goal.

## Results

| Area | Before | After |
|---|---|---|
| Main translation admin | ~8 analyst-hours/week across 3 analysts | ~10–15 minutes each morning for one analyst |
| Month-end technical-publication reporting | ~2–2.5 days for one analyst | ~2 hours |

## Stack

Python (REST APIs) · Wrike API · XTM API · Azure DevOps (version
control) · managed secrets for credentials · Copilot as a learning and
debugging aid

Built alongside my normal role as a marketing intelligence analyst,
with no prior API experience, while working through internal
cybersecurity approvals and an XTM Cloud migration.

[Read the full story, including the mistakes and the firewall
curveball →](/log/2026-11-02/)
