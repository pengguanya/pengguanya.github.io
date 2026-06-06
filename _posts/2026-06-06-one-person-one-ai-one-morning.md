---
title: "One Person, One AI, One Morning: The New Shape of Knowledge Work"
date: 2026-06-06
author: peng
categories: [AI & ML]
tags: [ai-agents, claude-code, productivity, workflow, automation, enterprise-ai, knowledge-work]
math: false
---

I needed to launch a training pilot. The task sounded simple: find the right audience across two overlapping teams, get their contact details, announce the program, and set up a way to collect feedback. In a normal week, this would mean asking someone to pull member lists, cross-referencing them in a spreadsheet, chasing email addresses through the company directory, drafting an announcement, getting it reviewed, and scheduling a kickoff meeting. Three people, two days, a dozen context switches.

I did it in one morning with an AI assistant. Not by working faster --- by working differently.

---

## The Organizational Maze

If you've worked in a global organization, you know the feeling. Teams span continents and timezones. People belong to multiple groups, workstreams, and communication channels. Organizational charts are approximations at best. The actual structure lives in chat spaces, mailing lists, shared drives, and the informal knowledge of people who've been around long enough to know who does what.

Our training curriculum was ready --- eight courses covering everything from prompt engineering fundamentals to regulatory compliance. The pilot audience had a clear definition: people who belong to both the broader technology organization and the specific product team building our AI system. These are the developers closest to the work, the ones whose feedback would be most valuable before a broader rollout.

But "people in both groups" is a deceptively hard query to answer in a large organization. The two groups live in separate chat spaces. The membership models are different --- one space has direct members, the other routes most people through a group alias. Neither space exposes a clean member list with names and email addresses. There's no single system of record that knows who is in both.

Normally, you'd ask a team lead to forward a list, cross-check it against another team lead's list, email a few people to confirm, discover half the addresses are wrong, and iterate. By the time you have a verified recipient list, you've spent a day on what should have been a five-minute task.

---

## The AI Workflow

Instead of navigating this maze manually, I described the goal to an AI assistant and let it work through the problem systematically. The entire workflow looked like this:

![AI-driven training pilot workflow: context sources feed into AI assistant, which drives four phases --- identify audience, communicate, collect feedback, organize meetings](/assets/img/2026-06-06-one-person-one-ai-one-morning/ai-workflow-diagram.svg)

Each phase built on the previous one, all within a single conversation.

### Phase 1: Identify the Audience

The AI connected to our internal messaging platform and pulled the membership data from both spaces. When the direct membership list came back incomplete --- most people had joined one space through a group rather than individually --- it adapted. Instead of stopping at the incomplete list, it fetched the message history from the space and built a more complete participant set from three sources: direct members, people who had sent messages, and people who had been mentioned by others.

With both participant sets assembled, it cross-referenced them to find the overlap. Then it queried the company directory to resolve each person's verified contact information. This step caught something I would have missed: several team members had email addresses on a different corporate domain than the standard convention suggested. Deriving addresses from names would have produced bounced emails.

The result: a verified contact list, exported as a CSV, ready to use --- in minutes rather than days.

### Phase 2: Communicate

With the audience identified, the AI drafted two communications: a detailed email introducing the curriculum (with links to the learning platform and supplementary materials) and a shorter chat message explaining why this specific group was selected for the pilot. I reviewed both, made minor adjustments, and sent them.

The drafts weren't generic templates. Because the AI had access to the chat history where the curriculum had been discussed and refined over months, it understood the context --- the phased structure, the distinction between auditable and supplementary training, the feedback we'd already received from stakeholders. The communications reflected that context without me having to brief the AI separately.

### Phase 3: Collect Feedback

This is the part that changed how I think about coordination. Instead of scheduling a large meeting or building a survey, I posted a simple message in the chat:

> For feedback --- just drop your thoughts in this chat whenever they come to mind. Don't worry about structure or format. I have an AI assistant connected to this space that handles the organizing and summarizing. Once I've collected enough feedback, I'll set up small group or 1:1 follow-ups as needed.

The usual feedback cycle --- schedule meeting, prepare agenda, wrangle calendars, take notes, summarize, distribute --- collapses into something much simpler. People contribute when a thought strikes them, in the moment they're actually experiencing the curriculum. The AI reads the chat history, aggregates responses, categorizes them by theme, and surfaces the patterns. No one has to structure their feedback. No one has to attend a meeting where three people talk and eleven listen.

And because the AI can identify which topics generate the most discussion or disagreement, it can propose targeted follow-up conversations --- small groups organized around specific themes rather than one large meeting covering everything.

### Phase 4: Organize Follow-ups

When it's time to schedule those small-group sessions, the same coordination challenge reappears: a dozen people across multiple timezones, each with their own calendar constraints. The AI can query availability, find common slots, and create calendar invites with pre-populated agendas drawn from the feedback themes. What would normally take several rounds of "does Thursday work for everyone?" emails becomes a single step.

---

## What Actually Changed

Let me be specific about what the AI did and didn't do.

**What it did:**
- Queried multiple internal systems and navigated their different data models
- Cross-referenced data sets that existed in separate platforms with different schemas
- Identified edge cases I would have missed (incorrect email addresses)
- Drafted professional communications with appropriate context and tone
- Designed a feedback collection workflow that eliminates coordination overhead
- Can organize follow-up meetings across timezones

**What it didn't do:**
- Decide who should be in the pilot group (I defined the criteria)
- Choose the curriculum content (that was months of human work)
- Send the communications (I reviewed and sent them myself)
- Replace my judgment about tone, timing, or organizational dynamics

The AI didn't replace a team --- it replaced the *mechanical work* that a team would spend most of its time on. The strategic decisions, the relationship management, the organizational awareness --- those stayed with me. The data wrangling, the cross-referencing, the drafting, the scheduling --- all of that happened in conversation while I focused on the decisions that actually require human judgment.

---

## The Multiplier Effect

The roles this work would traditionally require aren't specialized. They're coordination roles.

A **project coordinator** would track the member lists, chase down email addresses, schedule meetings. A **data analyst** would cross-reference the two spaces, identify overlaps, clean the data. A **communications person** would draft the announcement, calibrate the tone, manage the distribution. Each role might spend half a day on their part, with handoff delays between them.

With an AI assistant, one person does all three --- not because the AI makes you faster at each task, but because it eliminates the handoffs. There's no briefing document to write for the analyst. No email thread with the comms team about tone. No waiting for the coordinator to get back to you with the member list. The entire workflow stays in one conversation, with one context, building on itself.

This is the multiplier. It's not "AI makes you 3x faster." It's "AI removes the coordination cost that made this a three-person job."

---

## Unstructured In, Structured Out

The feedback workflow deserves special attention because it inverts a pattern most knowledge workers take for granted.

Traditional approach: schedule a meeting, ask everyone to prepare thoughts, spend 45 minutes in a room where the loudest voices dominate, write up minutes, distribute, get corrections, finalize. Calendar overhead: two to three weeks. Actual insight captured: whatever people remembered to say in the meeting.

AI-augmented approach: open a persistent channel, tell people to post raw thoughts anytime, let the AI aggregate and structure them asynchronously. No scheduling. No preparation. No meeting minutes. People contribute when something occurs to them --- in the moment they're actually experiencing the curriculum --- rather than trying to reconstruct their reactions days later.

The key insight: **the AI handles the part humans are worst at** (organizing unstructured input from multiple sources over time) **and leaves people free to do the part humans are best at** (having genuine reactions and articulating them naturally). You don't need to impose structure on the input when the AI can extract structure from the output.

---

## What This Means for How We Work

I'm not claiming this is revolutionary technology. Every piece of what happened this morning --- data queries, cross-referencing, email drafting --- could have been done with scripts and spreadsheets. The difference is in the interaction model.

With scripts, you need to know the system in advance, write the code, handle the edge cases, debug the failures. With an AI assistant, you describe the goal and iterate. When one approach hits a dead end, you try another --- all within the same conversation, with full context preserved. That kind of exploratory, adaptive problem-solving is exactly what makes cross-system coordination tasks so time-consuming for humans working alone.

The pattern I see emerging is this: **one person with domain knowledge plus an AI with technical reach can do the work of a small coordination team**. Not because either party is exceptional, but because the combination eliminates the communication overhead that makes small teams slow.

This morning's task wasn't complex. It was *dispersed* --- spread across multiple systems, requiring multiple skill sets, producing outputs in multiple formats for multiple channels. That dispersion is what creates coordination costs. And coordination costs are what AI assistants are quietly, undramatically eliminating.

The future of knowledge work isn't AI replacing people. It's one person, with the right tools, doing what used to require a room full of people --- and doing it before lunch.
