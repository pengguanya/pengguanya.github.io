---
title: "Claude as a Meta-Tool, Through the Loop"
date: 2026-06-22
author: peng
categories: [AI & ML]
tags: [claude-code, ai-agents, agentic-ai, loop-engineering, learning, jupyter, aws, meta-tooling]
math: false
---

Claude isn't just a tool you operate. It's a meta-tool: give it an outcome and it builds its own tools to reach it --- a script, a skill, a small pipeline --- for a single task or a stack of them at any level of complexity.

What makes that work is the loop, and the loop is self-contained. The agent figures out a path, builds and runs the tools to walk it, then checks the result against the goal. When it falls short it doesn't just patch the last step --- it can prompt itself to throw out the whole approach, find a completely new path, and reshape its own structure, then run that. You don't hand it steps. You hand it an outcome, and it keeps reshaping itself until the gap closes. That is the power of the loop.

This shift has a name now: people are calling it *loop engineering*. But the easiest way to see why it matters is to watch one loop do something useful. Here is the example that made it concrete for me.

![Claude as a meta-tool through the loop: an outcome goes into a self-reshaping loop --- find or rethink a path, build and run its own tools, check against the goal, and when it misses, throw the approach out to find a new path and reshape itself --- which produces a reusable machine. Mapped to one 6AM AWS workshop: the outcome "turn this firehose into a course I can learn from" becomes a personal course with an AI tutor, a hands-on notebook, and a scored test.](/assets/img/2026-06-22-claude-meta-tool-workshop-to-personal-course/meta-tool-loop.svg)
_You hand it an outcome; the loop finds a path, runs its own tools, and when it misses, reshapes its whole approach until the result holds. The same shape turned a 6AM workshop into a course built for one._

---

## The Workshop That Wouldn't Slow Down

The workshop started at 6AM my time, run out of India, deep into the AWS stack: agentic AI evaluation and observability. Two hours in we were still in setup --- config flags, version pins, IAM roles --- and the concepts I'd shown up for were buried under all of it. Live workshops move at the instructor's pace, don't pause when you're lost, and test nothing. You leave having attended, not having learned.

So I didn't take notes. The task wasn't "summarize the workshop." It was "build me something that turns this firehose into a course I can actually learn from." I gave the agent that outcome and let it find the path. It split the work in two on its own: capture the live session well enough to reconstruct it, then rebuild it into something I could work through on my own time.

---

## Capture: Listen, Transcribe, Snapshot

The live half had to run unattended while I half-listened and drank my coffee. Three jobs.

**Listen and transcribe.** The agent captured the meeting audio and turned voice into text as it went. A running transcript is the backbone; everything else hangs off it. Speech-to-text is good enough now that the transcript needed light cleanup rather than reconstruction.

**Snapshot the screen, selectively.** A transcript loses everything visual: the architecture diagram, the dashboard with the latency histogram, the exact CLI command the instructor typed. So the agent took screenshots. The trick was *not* grabbing one every few seconds, which would have buried me in noise. Instead it watched the transcript for signals that something on screen mattered, like "the important part here," a switch to a new tool, a command being run, a metric being explained. When the context flagged a moment, it grabbed a frame and tied it to that point in the transcript.

**Keep them aligned.** Transcript text and screenshots stayed linked by timestamp, so a line about "the trace waterfall view" sat next to the screenshot of that view. That alignment is what makes the later reconstruction faithful instead of vague.

By the end of the workshop I didn't have notes. I had a timestamped, screenshot-annotated transcript: raw material, not a finished thing. The rebuild was where it got interesting.

---

## Rebuild: A Course Built Around My Gaps

After the workshop ended, I asked the agent to turn that raw capture into a personalized course. Not a summary. A course I could work through at my own pace, one that pushed back when I was wrong.

It produced three things.

**An AI tutor that answers my questions.** Grounded in the transcript and the snapshots, the tutor knew what *this* workshop had actually covered, not the generic version from its training data. When I asked why they'd used a sampling-based evaluator instead of running every trace through the judge, it answered from what the instructor had said and why, and I could keep digging until it clicked, without raising a hand or waiting for the rest of the room.

**A hands-on Jupyter notebook.** This is the part that made it stick. Watching someone configure an observability pipeline teaches you nothing durable. Rebuilding it yourself does. The notebook walked me through reproducing the workshop's prototype step by step, with runnable cells, real output, and room to break things and see what happened. The concepts stopped being slides and turned into code I'd typed.

**A scored understanding test.** At the end it tested me: real questions, graded, with a score and a map of where I was weak. The test wasn't there to hand me a grade; it was there to find my gaps so I'd know what to revisit. It turned a vague "I think I got most of that" into a specific "you don't actually understand how sampling affects evaluation cost."

Put together, the workshop I half-absorbed at 6AM became a course built for one student. It ran at my pace, answered my questions, made me rebuild the thing myself, and told me where I was still weak. And notice what I actually did to get it: I never wrote the steps. I described an outcome and let a loop close the gap.

---

## From Prompting to Loop Engineering

For about two years, getting value out of an AI meant writing a good prompt. You typed a thing, read what came back, typed the next thing. The model was a tool and you held it the whole time, one turn after another.

The people building these tools have started doing something different. Boris Cherny, who leads Claude Code at Anthropic, put it bluntly in a recent interview:

> I don't prompt Claude anymore. I have loops running that prompt Claude and figuring out what to do. My job is to write loops.
>
> --- Boris Cherny, Claude Code, Anthropic

Peter Steinberger, who built the OpenClaw project, posted the same reminder:

> You shouldn't be prompting coding agents anymore. You should be designing loops that prompt your agents.
>
> --- Peter Steinberger

That is the shift. You stop being the person who writes each prompt and become the person who designs the system that writes them. The leverage moves up one floor --- from the prompt to the loop around it.

It helps to see it as a ladder. At the bottom, a person and a computer. Add a chatbot and you can ask questions. Add an agent and it can act --- run code, edit files, call tools. Add a loop and that agent runs on its own: it finds the work, does it, checks itself, and decides what to do next without you holding it each turn. The top of that ladder is where you stop typing prompts and start describing outcomes. The workshop tool lives there. I never told it *how* --- I told it *what*, and the loop did the rest.

This is also why prompt engineering was always a ceiling. A better prompt gets you a better first answer. But in real work the important signal shows up *after* the first answer: the failing test, the type error, the screenshot that exposes a broken layout, the metric that proves the approach was wrong. A single-shot prompt can't see any of that. A loop can, because it acts, observes the result, and feeds that back into the next move.

---

## Loops Stack

The most useful idea in all of this --- swyx calls it "loopcraft," the art of stacking loops --- is that loops compose. A serious agentic system is not one loop; it's several, nested. The team at LangChain lays out four levels, and they map cleanly onto what a meta-tool actually does.

**The agent loop.** A model calls tools in a cycle until a task is done: reason, act, observe, repeat. This is the core. My workshop tool lived here --- transcribe, snapshot, then build the tutor, the notebook, and the test, checking each against the goal.

**The verification loop.** Wrap the agent in a grader that checks the output against a rubric and sends it back when it falls short. The grader can be deterministic (do the tests pass, do the links resolve) or another model acting as judge. This matters because of a problem Addy Osmani named well: *"the model that wrote the code is way too nice grading its own homework."* The fix is to split the maker from the checker --- a second agent, often a different model, whose only job is to try to refute the first. Claude Code's `/goal` does a version of this under the hood: a separate model decides whether the stopping condition is actually met, not the one that did the work.

**The event-driven loop.** The agent stops being something you invoke and becomes something that runs on a trigger --- a schedule, a webhook, a new file landing. The next workshop on my calendar could capture itself without me starting anything.

**The hill-climbing loop.** The outer loop. Every run leaves a trace of what worked and what didn't, and an analysis pass over those traces rewrites the harness itself --- better prompts, better tools, tighter stopping rules. This is the level where the system doesn't just do work, it gets better at doing work. The return arrow doesn't loop back to the top; it reaches inside and improves the inner loops.

The first two loops automate the work. The last two are where value compounds, because the machine starts improving itself between runs. Most people, myself included, spend most of their time in loops one and two. The interesting frontier is three and four.

---

## The Meta-Tool Principle

Step back from the workshop and the pattern is the general one.

I never gave Claude the steps. I gave it an outcome --- a course I could learn from --- and it found a path, ran it, and checked the result against that goal. When a path didn't work, it didn't just tweak the last move; it reframed the problem and built a different one. That self-contained loop, the freedom to reshape its own approach until the outcome is right, is the whole point. It isn't a tool that waits for instructions; it's a tool that builds and runs other tools, rethinks them when they miss, and hands you the machine that finally worked.

That is what turns AI from a tool into a *meta-tool*: a tool whose main job is to build other tools. The loop is the engine, and the machine it produces is reusable. The next workshop, the next dense recorded lecture: same pipeline, new input. The setup cost amortizes across every firehose I'll sit through later.

Once you see it this way, your question to the AI shifts from "can you do this for me?" to "can you build me the thing that does this --- and the loop that keeps it honest?" One gets you an answer. The other gets you a capability you keep.

---

## A Loop Still Needs an Engineer

None of this removes you from the work. It moves you. And three things get *harder* as the loop gets better, not easier.

**Verification is still yours.** A loop running unattended is also a loop making mistakes unattended. The whole reason you split the checker from the maker is to make "it's done" mean something --- and even then, "done" is a claim, not a proof. The faster the loop ships work you didn't write, the more your job becomes confirming it actually works.

**Cost is real.** Subagents and stacked loops burn tokens, because each one does its own reasoning and tool calls. Spend them where a second opinion is worth paying for, and put floors under your loops --- a turn budget, a spend cap, a stopping rule --- so a runaway loop doesn't quietly drain a day's tokens chasing a goal it can't reach.

**Comprehension can rot.** The smoother the loop, the wider the gap between what exists and what you actually understand. Osmani makes a point that stuck with me: two people can build the exact same loop and get opposite results. One uses it to move faster on work they already understand. The other uses it to avoid understanding the work at all. The tool is identical; the difference is who's running it.

So design the loop. Just design it the way you would anything you'll be on the hook for, and stay close enough to catch it when it's wrong.

---

## The Takeaway

The workshop was never really the problem. Live, single-pass, instructor-paced learning just isn't built for the way any one person actually absorbs a technical topic. What changed wasn't the workshop. It was that I stopped being a passive attendee and built a loop that met me where I was.

The leverage has moved. It used to live in the prompt; now it lives in the loop you build around the agent. So the next time you're about to ask an AI to answer the question in front of you, consider designing the loop that answers the next ten --- a loop that builds its own tools, checks its own work, and reshapes itself until the outcome is real. Then keep it, because the next firehose is already on your calendar.
