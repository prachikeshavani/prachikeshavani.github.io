---
title: "UTM Link Generator"
description: "A Wrike-based automation that turns a form submission into consistent, ready-to-use UTM tracking links — no manual URL building, no typos, no waiting on an analyst."
summary: "A Wrike-based automation that turns a form submission into consistent, ready-to-use UTM tracking links — no manual URL building, no typos, no waiting on an analyst."
date: 2026-11-02
draft: false
params:
  seo:
    title: ""
    description: ""
    canonical: ""
    robots: ""
---

A Wrike automation that builds marketing tracking links automatically.
A campaign manager submits a short form; seconds later, fully formed
UTM links are sitting in their task, ready to use. Built at Hyster-Yale
using Wrike Forms and Wrike Integrate.

![Workflow diagram showing the six stages of the UTM link generator: form submission, recipe trigger, guard checks, parsing multi-selects into arrays, building one link per combination, and writing the links back to the task](/images/utm-workflow-diagram.svg)

*The workflow at a glance. Amber is a human action, blue is the Wrike
Integrate recipe doing its thing, grey is a system step. The detail
below walks through each stage.*

## The problem

UTM parameters are the tags appended to a URL that tell your analytics
platform where a visitor came from: which channel, which campaign,
which creative. They are how marketing teams find out what is actually
working. A tracking link looks like this:

```text
https://example.com/page?utm_source=facebook&utm_medium=social-organic&utm_campaign=year_end_upgrade&utm_content=banner
```

Building these by hand is fiddly and error-prone. People typo
utm_source, mix their casing (Facebook, facebook, FACEBOOK), forget the
question mark, use the wrong separator, or leave parameters out
entirely. Every inconsistency quietly breaks a slice of reporting, and
every link was costing an analyst a few minutes of assembly time that
added up across a global marketing team.

## What I built

Two pieces working together:

- **A Wrike request form** that captures exactly what is needed: the
  destination URL, the source and medium (both multi-select), the
  campaign name, and a content descriptor, along with date, purpose and
  theatre for context. The form itself enforces good habits: campaign
  names are prompted as lowercase with no spaces, with an example shown
  underneath.

![The UTM request form in Wrike, showing fields for date, purpose, theatre, URL, multi-select source and medium, campaign with a lowercase example, and content](/images/utm/utm-form.png)

- **A Wrike Integrate recipe** (Wrike's built-in automation layer) that
  takes those values, assembles them into correctly structured tracking
  links, and writes the results back to the task.

## How it works

**1. The recipe fires only when it should.** The trigger watches for
new or updated tasks in the UTM request folder, and only runs when no
UTM link exists yet on the task. That second condition matters: it means
the automation never overwrites existing links, so reprocessing a task
is safe. One piece of internal configuration (the folder ID) is
redacted in the screenshot.

![The recipe trigger and first steps: new or updated task in the UTM folder with no existing UTM link, retrieve the task, and check that a URL is present](/images/utm/utm-recipe-trigger.png)

**2. It guards before it builds.** Before any link construction, the
recipe checks that a URL is actually present. No URL, no run.

**3. Multi-selects become lists.** Source and medium are multi-select
fields, and a multi-select arrives from Wrike as a list rather than a
single value. The recipe parses these into arrays it can loop through.
This is the step that makes the whole thing work: instead of one long
ambiguous string, the automation can handle every selected option
individually.

![The JSON parsing step, showing the source values list (eloqua, facebook, google, instagram, linkedin and others) being converted into an array the recipe can loop through](/images/utm/utm-recipe-multiselect.png)

**4. One link per combination.** Two FOR EACH loops, one over the
selected sources and one over the selected media, concatenate the URL
and the UTM parameters into a result variable: for every source and
medium combination, one complete, correctly structured link.

![The two FOR EACH loops and the link concatenation step, followed by updating the task, notifying the requester in a comment, and a custom step that completes the UTM task](/images/utm/utm-recipe-loops-complete.png)

**5. Results go back where people are looking.** The generated links
are written onto the task, the requester gets a comment notification,
and a custom step completes the task. The links also land in the team's
UTM register, so there is a searchable record of every tracking link
the organisation has generated.

![The Wrike task activity stream after a successful run: the automation announces the request and posts the generated tracking links for the requester](/images/utm/utm-task-result.png)

![The UTM register row for the same request, showing the URL, the selected sources and medium, campaign and content values, and the resulting tracking links](/images/utm/utm-register-output.png)

For example, a request with two sources selected (facebook, linkedin)
and one medium (social-organic) produces two links:

```text
https://www.yale.com/en-us/north-america/pallet-jacks/nested-delivery-solution/?utm_source=facebook&utm_medium=social-organic&utm_campaign=routerunner&utm_content=routerunner

https://www.yale.com/en-us/north-america/pallet-jacks/nested-delivery-solution/?utm_source=linkedin&utm_medium=social-organic&utm_campaign=routerunner&utm_content=routerunner
```

One link per combination, each cleanly attributable in analytics. (Names
of colleagues are redacted in the screenshots; the links are real.)

## The bit I find most interesting

The multi-select handling is a small version of a bigger idea: data
shapes decide what automation is possible. As long as sources and media
were single-select, one task meant one link. The moment the fields
became multi-select, one task could legitimately mean several links,
and the recipe had to understand lists to keep up. Getting the data
structure right upstream made the automation downstream almost trivial.

## Why it matters

- **Consistency by construction.** The recipe adds the question marks,
  ampersands and parameter names. Humans only supply values, and the
  form nudges them toward clean naming. The structure cannot be typo'd
  because no human types it.
- **Speed.** Links that took an analyst minutes to assemble, plus the
  back-and-forth of a request queue, now arrive in seconds with the
  requester notified automatically.
- **Trust through guard rails.** The never-overwrite condition and the
  URL check mean the automation is safe to leave running. It passed a
  hundred successful runs in its first day without anyone babysitting
  it.
- **Zero training.** Requesters keep using a form. Nothing about their
  day changed except that the links are right and instant.

## Honest notes

This one is low-code, not raw Python like my [translation workflow
automation](/projects/translation-workflow-automation/). That was a
deliberate tool choice: the entire UTM generator lives inside Wrike, so
there is no separate code base to maintain and nothing extra for the
team to host or secure. Same instinct, lighter machinery: find the
repetitive work, understand the data, automate the predictable parts,
and leave people with the decisions.

## Stack

Wrike Forms · Wrike Integrate (trigger, conditions, JSON parsing, FOR
EACH loops, variables) · UTM conventions · Wrike comments and custom
item types
