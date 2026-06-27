---
layout: post
title: "Give Your Credentials to AI"
date: 2026-06-27
tags: ai agents engineering
description: "The leap with AI comes from access, not a cleverer model. Why giving your agent real credentials is the barrier that matters, and how to do it with a kill switch."
---

Hand over your passwords to your AI.

Claude Code was born when Boris gave an AI free run of his terminal. OpenClaw was born when Peter gave an AI unrestricted control of his computer. Both came from access, not from a cleverer model.

And it's the exact thing we're trained never to do.

As engineers, guarding secrets is drilled into us. Least privilege, rotate the keys, never hand over the whole machine. For good reasons. The model itself has absorbed the same instinct: throw a real credential at it and it gets visibly nervous and starts telling you to rotate it.

Now the uncomfortable part. If Boris and Peter had started by designing the permission model, they'd still be designing the permission model. The breakthrough needed someone willing to hand over the keys and watch what happened.

So if you actually want to see what an agent can do, stop shielding it from your own systems. Forget the training. Trust the model. Give it real access — your repos, your data, your tools — now, not after you've built the perfect sandbox.

This is what the access buys you. Without it, the agent is all brain and no hands. It figures out exactly what needs to happen and then leaves the doing to you. So you become its hands and legs, executing the decisions of a brain that isn't yours. Give it the keys and it gets its own hands and legs. It opens the PR and merges it, sends the email, runs the server. The distance between deciding something and it being done, the distance that used to be you, mostly disappears. You get your head back for the part that was always yours: choosing what's worth doing, and judging whether it came back right.

And no, I'm not saying paste your production secrets into a chat box and walk away. Give the agent a machine of its own. Hetzner will rent you a real physical server for less than half the price of an equivalent cloud VPS. A whole machine, not a thin slice of someone else's. On a box like that the agent has room to actually work: spin up dozens of worktrees that mirror production, run hundreds of processes at once, drive a handful of browsers in parallel. No tiptoeing around a cramped VPS.

Never set up a server in your life? You don't need to learn now. Hand the agent the SSH key, or just the root password, and it stands the whole thing up for you, which is the same move this whole post is about. It stays on. If it ever goes sideways, you wipe the box and start fresh. That's your kill switch.

The ones who pull ahead over the next two years won't be the best prompters. They'll be the ones who gave their agent real access and let it do the work.
