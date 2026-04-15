---
layout: post
title: "Teaching an AI How We Build Operators"
description: ""
category:
read: 13
tags: [openstack, kubernetes, ai, claude, operators, devops]
comments: true
---

It's been a while since my last post -- life got busy, as it usually does when
you're deep in the weeds of operator development. Between reconciliation loops,
CRD schemas, and the occasional `CrashLoopBackOff` that keeps you company on a
Friday evening, finding time to write about what you're doing (rather than just
doing it) has always been the hardest part.

This project didn't start from a grand plan. It started from frustration --
specifically, the friction of the ASDLC shift. AI-assisted development was
arriving whether we liked it or not, and I wanted to understand how an agentic
model would actually fit into my daily work. Not in theory, not in a demo --
in the real processes I'm supposed to follow day by day: planning features from
Jira tickets, reviewing code against team conventions, debugging operators in a
live cluster, running the right `make` targets in the right order. The result is
**[openstack-k8s-agent-tools](https://github.com/fmount/openstack-k8s-agent-tools)**,
a [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that
encodes the collective knowledge of how we develop operators in the
[openstack-k8s-operators](https://github.com/openstack-k8s-operators/)
ecosystem.

## The Problem With Tribal Knowledge

Every mature project accumulates a body of conventions that don't live in any
single document. The kind of knowledge that experienced contributors carry
around without thinking about it -- the patterns that make the difference
between code that works and code that fits.

In Kubernetes operator development, this is especially pronounced. The
controller-runtime framework gives you the building blocks, but the way a team
assembles them -- how finalizers are scoped, where status conditions are set,
when to return versus fall through -- is learned through practice and
reinforced through code review. In the
[openstack-k8s-operators](https://github.com/openstack-k8s-operators/)
ecosystem, where I spend most of my time, these conventions include things
like:

- Finalizers go on the top-level CR, not sub-resources
- `Status` conditions must have `Severity` and `Reason`, and
  `ObservedGeneration` must be set
- `Webhooks` validate at `.Spec` level, not `.Status`
- [lib-common](https://github.com/openstack-k8s-operators/lib-common) already
  has helpers for what you're about to write from scratch

These patterns are scattered across
[dev-docs](https://github.com/openstack-k8s-operators/dev-docs), code review
comments, Slack threads, and the accumulated scar tissue of debugging sessions.
New contributors (and sometimes experienced ones) rediscover these rules the
hard way. But this isn't unique to openstack-k8s-operators -- any project with
enough history has its own version of this list.

I wanted a tool that already knew all of this.

## What It Actually Is

[openstack-k8s-agent-tools](https://github.com/fmount/openstack-k8s-agent-tools)
is a Claude Code plugin -- a set of **skills** and **agents** that you invoke
with slash commands while working inside an operator repository. Nine skills,
three agents, roughly 73 commits over three weeks of iteration.

The architecture separates concerns cleanly:

- **Skills** (`/feature`, `/code-review`, `/task-executor`, etc.) are thin entry
  points. They parse your input, check for an existing state, and dispatch work.
- **Agents** are the workers. They contain the actual domain knowledge -- the
  11 criteria review checklist, the cross-repo analysis methodology, the
  planning principles.
- **Self-contained skills** (`/debug-operator`, `/test-operator`, `/code-style`,
  `/analyze-logs`, `/explain-flow`) handle their logic inline without needing an
  agent.

```
User: /feature OSPRH-2345
  |
  v
Skill (input parsing, state check, dispatch)
  |
  v
Agent (cross-repo analysis, planning checklist, strategy evaluation)
  |
  v
Plan file --> /task-executor --> implementation with checkpoints
```

The key insight is that skills should orchestrate and agents should execute. A
skill knows *how to start*; an agent knows *how to think*.

## The Evolution

Looking at the git history tells the story more honestly than any architecture
diagram.

**Week 1 -- Bootstrap and basic skills.** The initial commit laid out the
structure: debug-operator, explain-flow, analyze-logs, code-style. These were
self-contained skills -- markdown files that describe how to approach specific
tasks. No agents, no state management. Feature planning arrived as
`plan-feature` with a companion agent, and the `task-executor` followed
immediately with `checkpointing` and `resume` support.

**Week 2 -- The agent pattern emerges.** This was the most productive period.
The skill-agent separation was refactored properly, with `subagent_type`
dispatch replacing the earlier `read AGENT.md` approach. The **code-review** skill
learned to accept **PR numbers** and fall back from `gh` CLI to WebFetch. The Jira
skill was born out of necessity -- I needed hierarchy validation (**Feature ->
Epic -> Story**) and rules about where to produce and post **outcome**
comments. **OpenCode** support landed, with a **Makefile-driven** install that
converts agent frontmatter between platforms. The code-style skill grew from a
basic linter wrapper into eight sections covering error handling, naming,
interface design, and context management -- drawing from Effective Go, Go Code
Review Comments, and the Google Go Style Guide.

**Week 3 -- Memory and collaboration.** The most interesting architectural
evolution. Plans stored at `~/.openstack-k8s-agents-plans/<operator>/` gained a
shared `MEMORY.md` that persists across sessions -- a project-level memory
containing active work, discoveries (e.g., "lib-common already has
`TopologyHelper` -- don't reimplement"), architectural decisions, and blockers.
Every `/feature` and `/task-executor` session reads it at start and updates it
during work, so context accumulates instead of being lost between sessions.

A `state.json` tracks active tasks, worktrees, and completed plans, enabling
two things that matter for real-world development: parallel execution and
cross-plan dependencies. Each plan runs in its own git worktree, so two
features can execute simultaneously without file conflicts. And tasks can
declare dependencies on other plans -- `Depends on: OSPRH-6789/Task 1.1` --
or on external PRs -- `External dep: lib-common PR #789` -- with the
task-executor resolving them at runtime and stopping when blocked.

```
~/.openstack-k8s-agents-plans/<operator>/
|
+-- MEMORY.md                            Shared project memory
|     read at start of every /feature and /task-executor session
|     updated after planning and after task completion
|
+-- state.json                           Active work tracking
|     tracks worktrees, active tasks, completed plans
|     enables cross-plan dependency resolution
|
+-- 2026-04-11-OSPRH-2345-plan.md        Plan files
+-- 2026-04-11-OSPRH-6789-plan.md        (one per feature/bug)
+-- decisions/                           Decision records (optional)

<operator-repo>/
|
+-- .worktrees/                          Git worktrees (gitignored)
      +-- OSPRH-2345/                    One per active plan
      +-- OSPRH-6789/                    Isolated from main branch
```

The **/backport-review** skill arrived via external contribution (PR #1 from
[Alfredo Moralejo](https://github.com/amoralej)), adding downstream-vs-upstream
patch comparison with **Gerrit** integration.

Each phase built on the previous one. The progression from "teach Claude some
patterns" to "give Claude a working memory and execution environment" happened
organically.

## What I Actually Use Daily

The daily workflow comes down to three commands:

```bash
# While coding
/test-operator quick

# Before opening a PR
/code-review my-feature-branch

# When something breaks in deployment
/debug-operator glance-operator openstack
```

The feature planning workflow (`/feature` -> `/task-executor`) is for larger
pieces of work where I want structured thinking before I start writing code. The
code-style skill catches things I'd normally only notice during review.

## Teaching Domain Knowledge Through Markdown

The most counterintuitive aspect of this project is that all the domain
knowledge lives in markdown files. No code. No configuration DSL. Just
structured prose that tells the AI how to approach a problem.

Take the **code-review** agent. Its `AGENT.md` defines **11 review** criteria:

1. Reconciliation patterns (finalizers, deferred status, return-after-update)
2. Status conditions (severity, reason, ObservedGeneration)
3. Webhook conventions
4. API design (override patterns, probes, topology/affinity)
5. Testing (EnvTest, TestVector pattern)
6. RBAC markers
7. Error handling and logging
8. lib-common usage
9. Security
10. Documentation
11. Complexity (readability, over-engineering, nested conditionals)

Each review point has specific rules, examples of what's correct, and examples of
what's wrong. The agent doesn't just flag issues -- it classifies severity,
labels minor findings as `Nit:` / `Optional:` / `FYI:`, and renders a
structured outcome. And importantly, it **preloads** the code-style skill so it
knows about modernize patterns before it starts reviewing.

This is where the skill-agent separation pays off. The skill handles "what are
we reviewing?" (a PR number? a branch diff? specific files?). The agent handles
"how do we review it?" (against which criteria, the severity thresholds).

## Collaboration Across Boundaries

One thing that consistently slows down operator development is the need to check
what lib-common already provides, what peer operators do for similar problems,
and what dev-docs says about conventions. Humans are good at this once they've
been around long enough. New contributors are not.

The `/feature` skill addresses this directly. When you invoke
`/feature OSPRH-2345`, it:

1. Fetches the Jira ticket (if Atlassian MCP is configured)
2. Walks the full hierarchy -- Story -> Epic -> Feature -- collecting sibling
   stories, spike outcomes, linked PRs, and relevant comments
3. Analyzes your operator's codebase
4. Cross-references lib-common for reusable helpers
5. Checks peer operators (how did manila-operator handle a similar problem?)
6. Runs an 11-principle planning checklist
7. Proposes 2-3 implementation strategies with trade-offs
8. Produces a task breakdown grouped by functional area

The plan gets saved to disk as a markdown spec file -- and this is where the
human takes over. The engineer reviews the proposed strategies, chooses the one
that best aligns with the feature's goals, and can modify the plan or drop the
proposed design entirely in favor of a better approach. Each task in the plan
is scoped to carry a small amount of code and logical work, so nothing moves
forward without review. Then `/task-executor` picks it up, executes tasks
sequentially with checkpointing, enforces code quality at each step, and can
resume if you close your terminal and come back the next day.

## Guarding the Reasoning

The deep power of an LLM is inference -- the ability to connect patterns, fill
gaps, and produce plausible output at a speed no human can match. But plausible
is not the same as correct, and speed without direction is just fast failure.
The real work in building this plugin wasn't teaching the AI what to do. It was
canalizing that inference into a consistent model made of rules and practices,
without losing the end goal.

A few examples from practice:

The code-review agent kept flagging types and functions as "missing" or
"undefined" when they came from a `Depends-On` PR that hadn't merged yet. It
saw the usage, couldn't find the definition locally, and raised a finding. A
human reviewer wouldn't make this mistake: checking the dependency chain is
intrinsic to how we read code. We don't think about it; we just follow the
`replace` directive or glance at the linked PR. **The fix was to make that
implicit knowledge explicit**: the skill now resolves dependency context before
dispatching the review agent, and the agent has a hard rule: **"do NOT flag
usage of types, functions, or helpers that come from those dependencies as
'missing.'"**

Similarly, the feature agent would propose implementing helpers that lib-common
already provides, things like `condition.CreateList()` or
`endpoint.CreateOrUpdate()`. It read the operator's code, found no local
definition, and assumed it needed to be written. A human wouldn't do
this. We carry a _mental index_ of shared patterns and know to check lib-common
before writing anything new. The guard: **"Never propose reimplementing what
lib-common already provides. Check first."**

These are examples of encoding intuition: turning what lives in a developer's
head as second nature into explicit rules that prevent an otherwise capable
model from confidently producing wrong output.

But not all guards are about what the AI knows. Some are about keeping the
human in the loop. The task-executor, for instance, would charge ahead on a
plan written days earlier, even when the underlying files had been moved or
rewritten in the meantime. The fix: **"Before executing any task, check that
referenced files still exist and that the codebase hasn't drifted from the
plan's assumptions. If it has, it stops and asks."** This is human steering --
the ability to correct, align, and review before the tool runs ahead of you.

##### This matters more than it might seem.

The general ask from the ASDLC shift is to be _the next level of speed_. But
speed is the easy part. The hard part is quality, and quality requires a human
with limited processing bandwidth to actually review what the machine produces.
Rules, small tasks with breakpoints, explicit checkpoints before continuing --
these aren't constraints on productivity. They're what make the
human-in-the-loop viable. Without them, you're not collaborating with an AI.

The skill files, in this sense, are less about teaching the AI and more about
preserving the _contract_ between human judgment and machine execution. They
accumulate operational wisdom: not just "what went wrong" but "what to check
before assuming things went right."

## Considerations

The most valuable takeaway from this project is the **skill/agent model** and
the deliberate braindump of domain knowledge that makes it work.

[Skills](https://agentskills.io/what-are-skills) describe the *what* -- processes, workflows, entry points. They tell the
LLM what needs to happen: "we're reviewing a PR," "we're planning a feature
from this Jira ticket," "we're debugging a failing operator."
[Agents, or sub-agents](https://code.claude.com/docs/en/sub-agents), handle
the *how*. They implement the behavior defined for a
particular role: the reviewer who checks 11 criteria, the planner who
cross-references lib-common and peer operators, the executor who works through
tasks with checkpoints. Agents can access skills, knowledge, tools, and any
asset that helps them carry out the work -- but the separation is what keeps the
system composable rather than monolithic.

The real critical part, though, is not the architecture. It's bringing humans to
the same table. The value of an AI coding assistant isn't in the AI being smart
-- it's in the AI being *informed*. A general-purpose model can write perfectly
valid Go, pass a linter, and still produce code that violates every convention
your team has spent years establishing. It doesn't know that
openstack-k8s-operators uses controller-runtime with specific patterns for
status updates and finalizer management. It doesn't know that lib-common has
`condition.CreateList()` or that RBAC markers need to match watched resources.
It doesn't know that sub-tasks are not used in our Jira workflow and that
outcome comments go on Stories, never Epics.

None of that knowledge is exotic. It's all written down somewhere -- scattered
across dev-docs, review comments, Slack threads, and the accumulated memory of
people who've been around long enough to have made the mistakes. The problem is
that "somewhere" isn't anywhere a tool can reach unless you deliberately put it
there. And that deliberate act -- collaborating to standardize rules,
contributing to a shared set of processes -- is where the human role becomes
essential. The AI doesn't decide what the conventions are. People do. The AI
just applies them consistently, in a way that humans -- with our context
switches, our Friday afternoon fatigue, our tendency to skip the checklist
"just this once" -- sometimes don't.

There's a broader lesson here for any team that maintains a complex project with
non-obvious conventions. If your onboarding involves a senior developer sitting
next to a new contributor saying "no, not like that -- like this," you have
encodable knowledge. If your code reviews keep flagging the same patterns, you
have encodable knowledge. The question isn't whether AI can help. It's whether
your team is willing to sit down, agree on what the rules actually are, and
write them in a form that's structured enough to be useful -- for humans and
machines alike. This project is built on Claude Code, but the patterns --
encoding conventions as structured prose, separating orchestration from
execution, keeping the human in the review loop -- are transferable to any
agentic tooling.

The project is open source, MIT licensed, and installable as a Claude Code
plugin:

```bash
claude plugin marketplace add \
  https://github.com/fmount/openstack-k8s-agent-tools
claude plugin install openstack-k8s-agent-tools
```

It also works with [OpenCode](https://github.com/opencode-ai/opencode) via
`make install-opencode`.

If you work on openstack-k8s-operators -- or on any Kubernetes operator project
with enough conventions to encode -- I'd be curious to hear how it works for
you.
Contributions are welcome; the backport-review skill already came from an
external contribution.

The tools we build to help us build are, in the end, a reflection of what we've
learned. Might as well write it down where something else can remember it too.
