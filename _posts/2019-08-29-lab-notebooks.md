---
title: Lab Notebooks
layout: post
excerpt_separator: <!--more-->
---
*Originally published: August 26, 2019 - [LightStep](https://lightstep.com/blog/how-we-write-code-at-lightstep-lab-notebooks/)*

Many engineering teams create some form of a Design Document in the early stages of a new project. These documents are useful because they help the team debate and arrive at a high-level architecture and implementation plan to move forward with.

At LightStep we do things a bit differently. Rather than Design Documents, we’ve created “Lab Notebooks.”
<!--more-->

## Lab Notebooks vs. Design Documents
One common problem with design documents is that they are static. Let’s face it, the software we ship rarely is an exact implementation of the thing we initially designed. This isn’t a failure, but rather a reality of developing software for a business that is constantly changing.

Over the lifecycle of a project, there are countless decisions, big and small, that need to be made in order to drive the project forward. These decisions are incredibly important in explaining the design of a system but often happen at a whiteboard and are then forgotten.

Lab Notebooks allow us to be much more flexible in delivering on multiple projects across many teams — at the same time.

## How Lab Notebooks Work
A lab notebook replaces an engineering design doc (as well as possibly other docs). It is not meant to serve as documentation of the end result of a project (for example, product documentation, oncall training), but rather it captures the evolution of a project throughout its different stages. It records changes in scope, shifts in priority, trade-offs considered, and decisions made.

You can find a more complete explanation of what a Lab Notebook is in our [Lab Notebook template](https://go.lightstep.com/rs/260-KGM-472/images/lab-notebook-template.pdf):

> Commander’s Intent
> 
> - Encourage incremental approaches to project design and implementation
> - Include stakeholders and experts from across the company in the discussion without a rigid sign-off process
> - Support retrospectives and postmortems by recording important decisions for later
> 
> A successful project lab notebook accomplishes these goals by:
>
> 1. Recording questions, hypotheses, assumptions, experiments, and important decisions related to the project as they occur, as well as the results of these experiments and decisions
> 2. Serving as a log of interactions with – and future questions for – people that are not members of the project
> 3. Acting as a resource for future team members (as well as our future selves) to help us recall and understand why we built the things we did
Feel free to use this template with your team! (And let us know how it goes!)

## How Lab Notebooks Have Helped Our Team
We’ve found the introduction of Lab Notebooks to be really helpful in managing the multiple in-flight projects that we have going on. Documenting decisions is both cathartic, and useful. Rather than having to rehash the same conversation over and over, you can point to a specific entry in the Lab Notebook that includes a discussion about the trade-offs made and the reason for the decision. Having a written “history” of a project is also helpful for onboarding new team members on a project. They can read through the Lab Notebook and follow the project’s trajectory from inception to the present — providing a strong understanding of the project in its context.

If you’re interested in seeing what a real Lab Notebook looks like, check out [this one](https://go.lightstep.com/rs/260-KGM-472/images/lab-notebook-saas-pool-upgrade.pdf) around how we improved our ability to scale our hosted Satellite pool (caveat: we removed some internal details).

## Why This Matters
Thinking about the design of your system holistically in the early stages of the project helps to de-risk the project overall because many “gotchas” can be caught early — particularly when the document is socialized across the entire engineering organization.

In addition to getting valuable input from people with different domain specializations, you also get the benefit of increasing the awareness of your project across the organization. This can often lead to synergies with other teams that would be hard to identify otherwise.

With Lab Notebooks, we gain the ability to look at design over time. Rather than creating static documents, we come closer to recording the full story. In a way, Lab Notebooks can be the telemetry for your project; important events are logged and you get an end to end trace.
