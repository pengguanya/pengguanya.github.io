---
title: "The last mile nobody demos: shipping agentic AI in regulated pharma"
date: 2026-06-24
author: peng
categories: [AI & ML]
tags: [agentic-ai, genai, llm, regulated-ai, gxp, drug-discovery, evaluation, mlops]
math: false
---

I've watched the same meeting play out more than once. An agentic system reads a batch of messy clinical trial data, reasons about what looks wrong, and drafts the questions a human reviewer would otherwise write by hand. The demo lands. People lean in. Then someone who has shipped software in this industry before asks the question that settles it: "Great. How does this pass validation, and how does it run in our GxP environment?"

The room goes quiet, because that's where most agentic AI dies.

The gap between a demo that impresses a room and a system that runs inside a regulated pharma environment is a different kind of problem from the demo itself, and it's the one I find most interesting, because it's where almost all the practical value lives. A demo proves a model *can* do something. A deployment proves an organization can trust it to keep doing that, on real data, under audit, for years.

---

## Why regulated pharma is the hardest last mile

Pharma runs on GxP, the family of "good practice" regulations covering how drugs are developed, manufactured, and brought to patients. Any software that touches GxP data has to be validated: you document what it's supposed to do, you produce evidence that it does exactly that, and you keep that evidence current for the life of the system. The discipline has a name, Computerized System Validation, and a well-worn playbook (ISPE's GAMP 5, and more recently the move toward risk-based Computer Software Assurance).

That playbook rests on a quiet assumption: software is deterministic. Same input, same output, every time. You write a test, it passes, and it passes again tomorrow for the same reason. Validation is really just pinning software to a specification and proving it stays pinned.

Generative and agentic systems break that assumption on contact. They're probabilistic by construction. The same prompt can give you different text on two runs. The model can produce something fluent, confident, and wrong (a hallucination), with no exception thrown and no log line to catch. Often there's no single correct output to test against, just a range of acceptable ones that an expert recognizes when they see it. And the behavior drifts as the data underneath it shifts, long after the system passed its tests.

So you've got a probabilistic technology meeting an assurance regime designed for deterministic software. That's the real last mile. The regulators aren't standing still either: the EU AI Act now sorts systems into risk tiers with obligations attached, the FDA has published guiding principles for good machine learning practice and draft guidance on using AI in regulatory decisions, and the WHO has weighed in on the ethics of large multi-modal models in health. None of that dissolves the core tension. It just raises the cost of getting the engineering wrong.

The mistake I see most often is treating validation as paperwork you generate after the system already works. Really it's an architecture constraint. Build the demo first and bolt on deployment later, and you'll build the thing twice. The systems that survive are the ones you designed to be provable from the start.

---

## What changes when you design for production validation

Here's what "designed to be provable" means in practice. None of it is exotic. It's the set of choices that separate a thing that demos from a thing that deploys.

Non-determinism becomes something you test, not hide. You can't promise identical output every time, so stop pretending you can. Pin down the randomness you can (sampling settings, fixed seeds where that makes sense), then test for *consistency*: run the same input many times and check that the system lands on an acceptable answer often enough for its risk level. You end up characterizing a distribution instead of checking a single value.

No ground truth means you build the judgment in. A lot of what these systems produce (a drafted query, a summary, a ranked shortlist) has no single right answer to diff against. So evaluation leans on the people who'd otherwise do the work: experts scoring outputs against a rubric, pass/fail or on a scale. Where that won't scale, you let a model judge a model, but only after you've checked the judge against human scores. The evaluation harness is part of the system here, and usually the hardest part to get right.

Hallucination is a failure mode you engineer against, not a quirk you apologize for. You ground the model in real sources instead of its own memory. You write prompts that let it say "I don't know" instead of inventing. You add guardrails that keep it inside the validated scope. And the pattern I keep coming back to is letting the system critique its own output before a human ever sees it. A second pass that asks "is this actually supported?" catches a surprising amount.

Auditability means tracing everything. When a multi-step agent does something, a regulator or your own quality team will eventually ask *why*. So every step (each model call, tool invocation, handoff, retrieved document) has to be captured as a trace you can replay. You don't bolt this on afterwards. You instrument it from the start, because you can't defend a decision you can't reconstruct.

Human-in-the-loop is a design choice tied to risk. Not every output needs a human gate, and gating everything kills the value. The job is to map where humans review, override, or are just kept informed, and tie that to what a wrong answer actually costs. A low-stakes suggestion can flow straight through. A decision anywhere near patient safety doesn't move without a person. That mapping is part of the architecture.

Drift monitoring starts on day one. A model that passed validation in March can quietly rot by September because the incoming data moved. So the evaluation you used to qualify it keeps running in production, against live traffic, with thresholds that trip when quality slips. You don't add monitoring after launch; having it is part of what lets you launch.

Put those together and they're really one idea: the distance between a model and a *system*. The model is maybe ten percent of the work. The rest is the scaffolding that makes it trustworthy enough to run somewhere that answers to regulators.

![A small green "The model" node at the center, ringed by six neutral boxes: consistency testing, grounding and guardrails, self-critique pass, trace every step, human-in-the-loop, and drift monitoring. The core is labelled "the easy ten percent" and the surrounding scaffolding "the other ninety percent that earns trust in production."](/assets/img/2026-06-24-last-mile-nobody-demos-agentic-ai-gxp/model-vs-system.svg)
_The model is the easy part. The scaffolding around it is what makes it deployable._

---

## The same discipline, different altitudes

What took me a while to appreciate is that this scaffolding isn't one fixed checklist. It's a dial. How far you turn it depends on where you're standing in the pharma value chain and what a mistake costs there. The architect's job isn't to crank every control to maximum. It's to match the controls to the stakes. A few places I've watched the same discipline land at very different settings:

![A horizontal pharma value chain reading early discovery, preclinical, clinical under GxP, then post-market. A green "autonomy and exploration" band is strongest on the left and fades to the right; an amber "validation rigor and human gating" band is strongest on the right and fades to the left. An early-discovery design loop is tagged "more rope"; clinical data review is tagged "tight gate."](/assets/img/2026-06-24-last-mile-nobody-demos-agentic-ai-gxp/validation-dial.svg)
_Same discipline, different settings: turn autonomy down and rigor up as you move closer to the patient._

Clinical data review, where the dial is turned high. This is the setting I've spent the most time in, and it's where the dial sits near its limit. Reviewing study data for the discrepancies that have to be queried before a database locks is slow, expert work, and a natural fit for an agentic pattern: one agent assembles the relevant records, one grounds its reasoning in the actual study protocol so it's checking against the rules of *this* trial rather than a generic notion of correctness, one drafts the query a human would write, and one acts as a self-critique gate that decides whether a draft is sound before a person ever sees it. Every control from the last section is turned up: full tracing, because any recommendation might be questioned later; a human reviewer at the decision point, because this is GxP; evaluation built against expert-annotated cases, with blinded comparison to what experienced reviewers produce. The metric I cared about most wasn't raw accuracy. It was how often the system wrongly rejected a *sound* analysis, because a reviewer's trust is cheap to lose, and a tool that keeps flagging good work as bad gets switched off, however clever it is. So you optimize for high recall on what's worth surfacing and treat false rejections as the expensive error. The time saved is real, but it only counts because it comes with an audit trail and a human gate.

![A left-to-right agentic flow: assemble records, check against the protocol, draft the query, a highlighted self-critique gate asking "is this supported?", human review, and query raised. The whole flow sits on a band labelled "trace every step, replayable for audit," with a dashed loop underneath labelled "monitor in production, same evaluation, watching for drift."](/assets/img/2026-06-24-last-mile-nobody-demos-agentic-ai-gxp/agentic-review-flow.svg)
_The clinical setting with every control turned up: self-critique before a human sees it, a human at the decision point, full tracing, and drift monitoring after launch._

Early discovery, where the dial sits lower, on purpose. Walk upstream into research and the trade-offs flip. A pattern I keep seeing there is an agentic design loop: agents propose candidate molecules or protein binders, score them against constraints like drug-likeness, synthetic accessibility, and predicted off-target effects, throw out the failures, and iterate before handing a chemist a ranked shortlist. The ground truth here isn't a regulator. It's the assay, the wet lab, the next experiment. Nothing the agent proposes reaches a patient, so you can give it far more rope: a looser gate, more freedom to explore, because a bad suggestion costs a wasted synthesis, not a safety event. The discipline doesn't disappear, though. It changes shape. You still want traceability, because a chemist will ask why a molecule made the list. And you still watch for drift, except here you're watching whether the agent has learned to game its own scoring function instead of finding genuinely good candidates. Same framework, turned down for a lower-stakes, higher-exploration job.

![A cyclic agentic flow for early research: propose candidates, score them against constraints (drug-likeness, synthesizability, off-targets), keep the best, and loop back to propose again, iterating freely. The loop exits with a ranked shortlist to a chemist, where the assay decides. A note reads that a miss costs a synthesis, not a patient, and that the loop is still traceable and still watched so it cannot game its own scoring function.](/assets/img/2026-06-24-last-mile-nobody-demos-agentic-ai-gxp/early-research-design-loop.svg)
_Turned down: the loop iterates on its own, and the human gate is a chemist deciding what to make, not a regulator signing off._

And the long tail in between. Drafting a first-pass regulatory document, triaging post-market safety reports, tuning a manufacturing process from historical runs: each is the same spine at a different setting. The regulatory draft tolerates generation but wants a human author and tight grounding in the source. The safety triage can surface and cluster signals freely, but it must never *decide*. The process model can suggest experiments, but physical validation has the last word. Once you see the dial, you stop asking "is agentic AI allowed in pharma?" and start asking the more useful question: for *this* use case, at *this* altitude, how far do I turn it?

That question, asked well, is most of what separates a proof-of-concept that gets adopted from one that gets admired and quietly shelved.

---

## Where this is heading

The encouraging part is that the scaffolding is turning into a stack. A year ago the pieces were bespoke; now there are real building blocks for production agentic AI: fast inference services to run models behind an API, toolkits for orchestrating agents and their handoffs, safety guardrails as a configurable layer, and evaluation-and-observability tooling that treats tracing and scoring as first-class. The platform vendors building accelerated computing for life sciences are converging on exactly this: not just bigger models, but the production plumbing that gets a customer from a notebook to a validated deployment.

That matters for drug development, because the bottleneck usually isn't the science. It's the time. Every week you take out of cleaning trial data, generating a hypothesis, or screening a library is a week closer to a database lock and a submission. Agentic AI can pull real time out of that pipeline, but only if it can live where the work actually happens, which in this industry means inside a validated, audited, monitored environment.

---

## The takeaway

If there's one thing I'd want a team to take from this, it's that the differentiator isn't the model. Models are becoming a commodity, and the next one will be better than whatever you picked today. The differentiator is judgment: knowing the terrain well enough to design a proof-of-concept that's deployable on day one, with the dial set right for where it lives, traceable and evaluable and monitored, gated for human oversight exactly where the risk demands it and no further.

A demo that wins the room and a system that wins the audit are built differently from the very first decision. The interesting work, and most of the value, is in building the second one. That's the last mile nobody demos, and in this industry it's the only mile the patient ever feels.
