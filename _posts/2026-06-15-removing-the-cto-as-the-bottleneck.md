---
layout: post
title: "Removing the CTO as the Bottleneck: Building an AI-Native Operating Layer Inside Hero Group"
date: 2026-06-15
tags: ai engineering leadership
description: "A practical note on what changes when AI meets people where they already work: context, tools, memory, review, and artifacts."
image: /assets/removing-the-cto-as-the-bottleneck/stats-hero.png
---

In a small company, the CTO often becomes the point where business
intent gets translated into technical reality.

The visible work is architecture, infrastructure, code, pull requests,
and technical judgment. The less visible work is holding the map of
software, data, product history, customer context, operations, old edge
cases, and hidden constraints.

Someone needs a report, and the CTO knows where the data lives. Someone
asks whether a new idea is simple or expensive, and the CTO translates
the question into schema, permissions, code paths, deployment risk, and
sequencing. Someone sees something odd in the business, and the CTO
becomes the query engine, the debugger, and the explainer.

That is useful for a while. Early on, it gives the company speed. But as
the company grows, it turns technical leadership into a reactive
translation queue, and the CTO becomes a bottleneck precisely because
they have too much context.

The internal AI work I have been doing at Hero Group is my attempt to
move the company toward that kind of AI-native operating model.

One part of that work, and the part I want to focus on here, is the
internal operating layer: how our own team gets answers, creates
artifacts, follows up on decisions, inspects systems, and turns intent
into work without routing every step through me. The goal is not to
remove engineering judgment. The goal is to reserve engineering judgment
for work that actually needs it, instead of spending it on translation
work created by systems that are hard for non-engineers to operate
directly.

That is the lens through which I watched [YC’s recent discussion on
becoming an AI-native
company](https://www.youtube.com/watch?v=B246K_G7mHU&t=360s). What
stayed with me was how familiar the operational problem felt. Their
examples were different, but the shape was similar: give teams a way to
express intent in natural language, connect the agent to real company
context, and build the review boundaries around the places where it can
act.

AI becomes genuinely useful inside a company when it is connected to the
company’s context, tools, permissions, workflows, review loops, and
artifacts. Without that, it is mostly a clever assistant sitting outside
the real work.

![Aggregate Hero Hero usage stats: asks answered, conversations, team size, non-engineering usage, autonomous work hours, longer jobs, pull requests, image and voice note usage, and words written back.](/assets/removing-the-cto-as-the-bottleneck/stats-hero.webp)

*A four-person team, one internal AI agent, and three months of real usage.*

## AI-Native Is An Operating Model

The phrase “AI-native company” is easy to flatten into marketing. It can
mean a product has AI features, employees are encouraged to use ChatGPT,
or a company has a few internal automations running in the background.
The version I care about is operational: a company where internal
systems are redesigned around the assumption that AI can safely sit
close to the work, read context, use tools, create first drafts, inspect
systems, ask for review, produce artifacts, and hand work back to humans
at the right boundary.

That requires more than prompts. It requires architecture.

It also requires meeting people where they already are.

For us, that means Hero Hero, our internal AI agent, lives directly in
the tools the team already uses: Slack for everyday interaction and
Basecamp for the more durable project-management layer. Our team was
already comfortable in those places. If the AI required everyone to move
into a new cockpit, it would have become one more system to remember.
Putting the agent inside the surfaces where work already happens made it
feel less like “using an AI tool” and more like asking for help in the
normal flow of work.

The system still has to know where context lives, which tools exist,
what it can do without approval, what it must escalate, how to produce
outputs people can inspect, and how to leave a trail. But the interface
has to stay natural. AI should adapt to the team more than the team
adapts to the AI.

## Hero Hero Is An Operating Interface

Chat is the surface of an internal AI system, not the substance.

Hero Hero, or HH internally, matters because it sits near the actual
operating context of the company. It can work with internal
documentation, plans, code, read-only data, Slack, Basecamp, email,
browser sessions, Drive, Calendar, call transcripts, and generated
artifacts. More importantly, it has instructions and boundaries around
how to use that context: when to inspect, when to summarize, when to
draft, when to create a pull request, when to ask another AI for review,
when to ask a human, and when to refuse.

That changes the shape of everyday work.

The primary interface is human language. A teammate should be able to
explain what they want in the words they naturally use, not in schema
names, product internals, or ticketing-system language. They should be
able to send a message, screenshot, voice note, video, transcript, or
rough idea and have the system understand enough to move the work
forward. They should also be able to get work back in useful formats: a
report, task, draft email, walkthrough, product note, implementation
plan, generated asset, or pull request.

![Slack exchange where a teammate asks Hero Hero how many customers used the product in the last 30 days, and Hero Hero answers with the count and a clarifying follow-up.](/assets/removing-the-cto-as-the-bottleneck/card-customers.webp)

*A teammate asks a business question in plain English, and Hero Hero fetches the answer.*

![Slack exchange where a teammate asks Hero Hero if it is a dog, and Hero Hero replies that it is an AI agent for the team: no paws, no tail, just text.](/assets/removing-the-cto-as-the-bottleneck/card-dog.webp)

*Not every useful AI interaction has to look formal.*

![Slack exchange where a teammate asks Hero Hero for a Hero Group slogan. Hero Hero gives several slogan options and recommends Tools that help shops win as the strongest brand line.](/assets/removing-the-cto-as-the-bottleneck/card-slogan.webp)

*Hero Hero is useful for creative work too, not only technical or analytical work.*

The important shift is not that AI answers questions. The important
shift is that AI moves intent closer to the systems and artifacts where
real work happens.

## Where The Leverage Shows Up

One of the strongest internal patterns has been read-only access to
production context.

Companies often discuss AI access as if there are only two choices: keep
the model away from real systems, or give it dangerous power too early.
The useful middle is controlled read-only visibility. Let the agent
inspect real context, but make mutation narrow, explicit, auditable, and
human-reviewed.

Read-only context changes the quality of answers immediately. A model
that cannot see the business gives generic advice. A model that can
safely inspect the actual state of the business can answer grounded
questions, find inconsistencies, explain trends, and help people
understand what is happening without waiting for the CTO to manually run
the first query.

The bigger change is that people ask more questions. When asking a data
question requires finding the right person, explaining the context,
waiting for the query, and then asking the follow-up, many questions
never get asked. When the interface is natural language and the system
can fetch the context safely, curiosity becomes cheaper. We ask about
user behavior, product interactions, documents, operations, and internal
workflows because the cost of asking has collapsed.

That does not remove the need for boundaries. Some sensitive data should
remain encrypted so even the agent cannot read it. That is a feature,
not a limitation. A serious internal AI system should create more
visibility where visibility is useful and hard walls where hard walls
are necessary.

Another leverage point is the tool and skill layer. In the [YC
discussion of tool
registries](https://www.youtube.com/watch?v=B246K_G7mHU&t=840s), the
important idea is not the packaging. The important idea is resolution:
can the agent discover what it is capable of doing, understand the right
entry point, and use the right tool without a human manually wiring
every step?

That registry can be code. It can be markdown. It can be YAML. It can be
a set of internal skills, CLIs, scripts, browser tools, and documents.
As models get better, a well-written markdown file can be a perfectly
serious part of the architecture. The question is whether the system is
legible enough for the agent to resolve capabilities correctly and safe
enough for humans to trust the result.

![Slack exchange where a teammate asks Hero Hero to merge a pull request. Hero Hero reports that it merged and deployed the pull request, resolved merge conflicts, and started production deployment.](/assets/removing-the-cto-as-the-bottleneck/card-ship.webp)

*From review to merge to deployment, internal AI can move work across real engineering steps.*

Model routing is part of the same layer. Hero Hero knows that different
models have different strengths and weaknesses. Some are better at
technical architecture, some at design, some at image generation, some
at transcription, some at long-form reasoning, some at fast routine
work, and some as reviewers. A good internal system should not pretend
that one model is always the answer. It should know when to route, when
to review, when to ask another AI for a second opinion, and when to stop
for a human.

![Quote from a stand-up where Shane Bralove says Hero Hero knows more about the company than Replit does, so he cut Replit out of his process.](/assets/removing-the-cto-as-the-bottleneck/quote-replit.webp)

*Company context can matter more than a standalone AI tool.*

Memory and scheduling matter too. Some work is not a one-shot answer.
The agent needs to remember durable context, follow up at the right
time, and connect a conversation today to a task or artifact tomorrow.
That is when AI starts feeling less like a search box and more like
operating infrastructure.

## What Changed In The Team

The best part of this work has been watching the team use it.

People no longer wait for me to answer every product, data, or
implementation question. They ask Hero Hero. They ask in the language
they already use. Sometimes they give it a screenshot. Sometimes they
give it a voice note. Sometimes they ask for a plan, a draft, a report,
a product idea, a launch outline, a social post, or an analysis.
Everyone seems to address it like a real teammate, which says something
important about where the interface is working.

![Slack message from Shane Bralove saying Hero Hero has been the biggest game changer for him in operations. He used to have a 65-person team trained to read a ticket, look up the customer in the system, troubleshoot the issue, and write a response. Hero Hero now does all of this in minutes, and he works in the language of the customer without converting to database terms.](/assets/removing-the-cto-as-the-bottleneck/card-ops.webp)

*The work a 65-person ops team was trained to do — read the ticket, look up the customer, troubleshoot, respond — now happens in natural language, in minutes.*

![Quote from a stand-up: Why scroll when Hero Hero will just pull what I need for me?](/assets/removing-the-cto-as-the-bottleneck/quote-scroll.webp)

*The behavior change is simple: people stop searching manually and ask the internal agent.*

Our COO, who has never been a frontend engineer, is now actively driving
product UI and UX decisions. He talks to users, collects feedback, works
through ideas, and can drive improvements with HH in the loop instead of
waiting for every detail to be translated through engineering first.
Other people are creating launch plans, strategy documents, social media
drafts, product ideas, and internal analysis with a level of
independence that would have been hard to create with traditional tools.

![Quote from a stand-up where Shane Bralove thanks Hero Hero and says it did a good job designing the emails.](/assets/removing-the-cto-as-the-bottleneck/quote-thanks.webp)

*When the agent is in the workflow, people start treating it like a teammate.*

![Quote from a stand-up where Shane Bralove compares separate brand, product, and email design teams at Uber with Hero Hero acting as one consistent source.](/assets/removing-the-cto-as-the-bottleneck/quote-uber.webp)

*One internal agent can create consistency across product, brand, and communication work.*

This is the part that is easy to underestimate from the outside. The
value is not only that work gets done faster. The value is that people
feel more able to contribute. They do not come only with ideas anymore.
More often, they come with a first version, a prototype, a draft, or a
concrete artifact that can be reviewed.

That changes morale. It changes ownership. It changes how much of the
company can move without waiting for one technical person to translate
every next step.

![Quote card. Shane Bralove&#39;s phrase, relayed in a Hero Group stand-up: Hero Hero is becoming the central nervous system of our operation.](/assets/removing-the-cto-as-the-bottleneck/quote-nervous-system.webp)

*Once the agent sits close to enough of the company's work, the team stops calling it a tool and starts calling it core infrastructure.*

## Conversations Become Company Memory

A lot of important work does not begin in a dashboard or a ticketing
system. It begins in conversation.

Slack messages, Basecamp threads, calls, transcripts, screenshots, rough
notes, and half-formed ideas are all part of the company’s memory.
Traditional software loses a lot of this because the conversation and
the system of action are separate. Someone has to remember what was
said, write the task, update the document, notify the right person, and
follow up later.

An internal AI operating layer can reduce that loss if it is built with
the right boundaries. This is not an argument for exposing every private
conversation to every workflow. Scope matters. Consent matters. Context
should be used for the purpose it was collected for, and sensitive
material should be summarized carefully or not used at all.

Within the right boundary, conversations become operational inputs. A
meeting can become a decision record. A transcript can become follow-up
work. A screenshot can become design feedback. A rough conversation can
become a draft, a report, a task, a pull request, or a set of next
steps.

We already record transcripts, and HH can use that context. We are also
building toward a workflow where HH can be addressed directly from a
call and then act once the transcript becomes available. That may sound
like a small interface change, but I think it points to a bigger shift:
the next system of action will often begin inside the conversation that
already happened.

![Quote from a leadership meeting where Morgan Felchner says an AI bot replied on behalf of Kartikey, making it feel like the team was living in the future.](/assets/removing-the-cto-as-the-bottleneck/quote-future.webp)

*The team noticed the system feeling like a different operating mode, not just another tool.*

## The Challenges Are Real

None of this works by magic.

You have to decide what the agent can see, what it cannot see, what it
can do, what it must never do, and when a human has to review the
output. You have to keep context fresh. You have to prevent tool sprawl.
You have to keep improving prompts, skills, routing, and review paths as
models change. You have to accept that the workflow you build this month
may need to be improved next month, or next week.

That pace is uncomfortable, but it is also the nature of the work.
AI-native operations are not a one-time implementation. They are a
living operating layer.

The companies that get good at this will not be the ones that install
one tool and call it done. They will be the ones that keep improving how
context, tools, models, review, and artifacts fit together.

## What This Means For The CTO Role

The CTO role does not disappear in this model. It becomes more
architectural.

Instead of being the person who manually translates every operational
question into a technical action, the CTO designs the system that lets
more people interact with technical context safely. That means building
boundaries, choosing tools, exposing the right context, making review
paths explicit, and deciding where automation should stop.

This is the kind of technical leadership I think will matter more over
the next few years.

Not adding AI as decoration. Not wrapping a model in a thin interface
and calling it transformation. The harder layer is the internal
operating system that lets a company move faster because AI has
controlled access to context, tools, workflows, and review.

That still requires deep engineering. Rails matters. Databases matter.
Infrastructure matters. Logs, queues, permissions, tests, deployment
discipline, and product judgment all matter. AI does not make the
underlying systems less important. It makes weak underlying systems more
dangerous because the model can only reason from the context and tools
you give it.

My own work changed before the company layer changed. I have always
treated my workflow as infrastructure: editor habits, terminal setup,
TDD loops, key bindings, tmux, small scripts, and tiny improvements that
compound over years. AI changed the ingredients, but not the habit. The
workflow now changes faster. Sometimes the improvement is a prompt.
Sometimes it is an agent rule. Sometimes it is a new model route, a
review step, a tool, or a better way to hand work between humans and AI.

Since moving into AI-native coding tools, I have stopped treating the
traditional editor as my main interface. That personal shift made a
larger question unavoidable: if AI can become the interface through
which I build software, why can it not become the interface through
which more of the company operates?

That is the question I have been answering inside Hero Group.

## Why The YC Conversation Mattered

The YC conversation mattered because it gave better language to patterns
I was already seeing from the inside.

The patterns were familiar: company context matters, agents need tools,
read-only access changes what people are willing to ask, tool registries
behave like resolvers, model routing and review matter, conversations
and transcripts can become useful inputs, and outputs should often be
artifacts rather than chat replies.

We are not finished. Some parts are already working well, some parts are
rough, and some parts are clearly on the roadmap, including deeper
automation and analysis around marketing and social distribution. But
the direction feels right.

The larger shift is cultural as much as technical. In AI-native teams,
people do not only bring ideas to the table. They increasingly bring
drafts, prototypes, analyses, plans, and working artifacts. The distance
between “I have a thought” and “here is something we can inspect” gets
much shorter.

That is what I am seeing inside Hero Group.

For me, that is what AI-native means in practice. It is not a slogan and
it is not just a better chatbot. It is a company gradually rebuilding
its internal operating layer around context, tools, safety, artifacts,
and human judgment.

That is the work I am doing.

If you are building in this direction, or trying to make AI useful
inside a real company, I would be happy to compare notes.

Reach me at: tannakartikey@gmail.com

---

*Originally published on [LinkedIn](https://www.linkedin.com/pulse/removing-cto-bottleneck-building-ai-native-operating-layer-tanna-amswf).*
