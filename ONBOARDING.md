# Studio AI: Alpha Onboarding

Welcome! You're one of the first people outside the core team using Studio AI. This page gets you logged in, oriented, and through your first end-to-end loop.

## 1. The setup: you own Quiz Lab

When you log in, you'll find one small product waiting for you: **Quiz Lab**. It's a working quiz application, deliberately spare. A few questions, a results screen, nothing fancy. It's also, for the next few weeks, **yours**.

You're the product lead, designer, and engineer all at once. What Quiz Lab becomes is up to you:

- Maybe you turn it into a study tool for whatever you're learning right now.
- Maybe it becomes a vibe-check quiz your team can pass around.
- Maybe you decide the quiz framing is wrong and you reshape the codebase into something else entirely.

The destination doesn't matter. The point is to give you a real product (with real users, real code, and real GitHub PRs) to think about, prioritize against, and ship into. That's the surface on which we'll test whether Studio AI actually helps.

There's also a shared **`product-collab`** product alongside Quiz Lab. Use it for joint experiments with the other alpha users when something is more fun built together than alone.

## 2. The pitch: where Studio AI fits

Building products with AI agents has two parts.

**The coding part.** Studio AI's value is to speed up development: working with coding agents, making sure they have the best instructions, and enabling async for deterministic tasks. It does help with coding speed.

**Everything around the code**, where rework hides: turning vague feelings into testable hypotheses, breaking them into specs sharp enough for an agent to implement well, tracking what you tried, closing the loop between *"I had an idea"* and *"the PR is merged"* without dropping the thread along the way.

Studio AI gives you tools for both parts. As you work on Quiz Lab, it gives you:

- **An AI partner that drafts your ideas.** Rough thoughts in, a sharpened hypothesis, success criteria, and a draft technical design out. You stay in control of the words; the AI takes care of the scaffolding.
- **A validation funnel that tracks ideas as they mature.** *Backlog → Building → Design Partner → All Customers.* Failed ideas stay visible too. Killed ideas are data.
- **Small, incremental steps with clear specification and background context** so agents implement with high quality the first time. The result: efficient context management and less rework.
- **Tasks that auto-link to GitHub PRs.** Open a PR on your Quiz Lab repo and it appears on the task. Merge it and the task closes itself. The plumbing stays out of your way.
- **An MCP server** so your coding agent (Claude Code or similar) can pick up tasks, load context, and submit work for review without you copy-pasting between windows.
- **Managed tasks for async work.** For toil and well-specified deterministic tasks where you don't need to pair, hand them off to a managed (cloud) Claude run and stay focused on the high-value judgment calls. See [MANAGED-TASKS.md](./MANAGED-TASKS.md).

The bet: you spend more time on **high-value judgment** (what to build for Quiz Lab, how sharp the hypothesis is, whether the result actually moved the needle) and less time on the toil around it. You're our test pilots for that bet. Tell us where it pays off and where it doesn't. That's the most valuable thing to come out of alpha.

## 3. Logging in

Log in at:

```
https://app.routeburn.org
```

Sign in with **Google OAuth** using the email tied to your alpha invite. After sign-in, you'll land on the product picker. Pick your `product-{your-handle}` product (or the shared `product-collab`) to navigate in.

## 4. What you'll see

Two products under the `demo-alpha-studio` studio:

- **`product-{your-handle}`** is your own product. The other alpha users *can* see it (it's all one studio), but you're the one shipping it. Its GitHub repo is `routeburn-alpha/product-{your-handle}`, and only you (and the system) can push to that repo.
- **`product-collab`** is a shared product all four alpha users can both see and write to. Use it for joint experiments. Its repo is `routeburn-alpha/product-collab`.

Access control today is per GitHub repo, not per product page (see §10 Known limitations).

## 5. Your deployed app

Your product app is auto-deployed to GitHub Pages. After each push to `main` (typically via a merged PR), the live build refreshes at:

```
https://routeburn-alpha.github.io/product-{your-handle}/
```

The shared collab app lives at `https://routeburn-alpha.github.io/product-collab/`.

This is the surface to check after merging a change. No deploy step on your side, no infra to manage.

If a deploy doesn't show up after a few minutes, check the **Actions** tab of your repo (`https://github.com/routeburn-alpha/product-{your-handle}/actions`). Failed deploys show there. If the deploy succeeded but the page is stale, hard-refresh (GitHub Pages aggressively caches).

## 6. How things are organized

Studio AI nests work into three concepts: **organization → studio → product**. Knowing the shape helps you read URLs and ask questions in Slack.

| Concept | What it is | For alpha |
|---|---|---|
| **Organization** | The top-level container. | One shared org (`demo-alpha`) across the whole alpha cohort. |
| **Studio** | A team workspace inside an org. Holds people, agents, and products. | One shared studio (`demo-alpha-studio`) for the whole cohort. |
| **Product** | A line of work inside a studio. Has its own ideas, tasks, and (usually) repo. | Each of you has one (`product-{your-handle}`), plus the shared `product-collab`. |

**Ideas and tasks are numbered per product.** Idea #1 in *your* product is a different idea than idea #1 in `product-collab`. Same for tasks. When you reference a number in Slack, say which product.

If you peek at the URL bar, the path mirrors the hierarchy:

```
/{org}/{studio}/{product}/{feature}
```

So `/demo-alpha/demo-alpha-studio/product-collab/tasks` is the tasks page for the shared collab product.

## 7. Core concepts

Three concepts you'll see everywhere: **ideas**, **tasks**, and the **SDLC** that ties them to GitHub.

### Idea

An **idea** is a product hypothesis you want to validate. It has a name, a hypothesis (what you believe and want to prove), and a technical design (architecture, data model, success criteria). The technical design is required before you start building from the idea.

Ideas move through a validation funnel:

```
draft → backlog → Building → Design Partner → All Customers

Off-ramp: failed
```

The labels map to evidence: **Building** = you're constructing it. **Design Partner** = it's running with a specific user or team. **All Customers** = proven and shipped widely.

No transitions are enforced. You can move an idea anywhere, including backwards. The funnel is a thinking tool, not a gate. Every transition is logged in the idea's status history.

When you're ready to act on an idea, you break it into **tasks** (linked as `ideaSubtask`).

### Task

A **task** is an actionable unit of work. It can be standalone or linked to an idea:

| Type | What it's for |
|---|---|
| `ideaSubtask` | Part of executing an idea. Auto-set when you create a task under an idea (default from the idea page). |
| `nit` | A small standalone improvement. Default type when you create a task from the tasks page. |
| `techDebt` | Standalone cleanup work. |
| `bug` | Something that's broken. |

Tasks move through their own lifecycle:

```
backlog → inProgress → review → done
            ▲   │         │
            │   │         └─→ changesRequested ──┐
            │   └─ release ─→ backlog            │
            │                                    │
            └────────────────────────────────────┘

Any state can move to: deprioritized
```

- Only one agent or person can have a task `inProgress` at a time. If another agent tries to pick it up, the call fails. To re-assign, the current owner releases the task (or asks their agent to) and the next agent calls `work_on_next_task` to claim it.
- `release` is in the diagram for completeness. There's no explicit release UI yet (coming soon); for now you ask the agent to drop the task or update it manually.
- `review` is reached by submitting work **with a PR**. Without a PR, the task goes straight to `done`.
- Every transition is logged in the task's status history.

### SDLC: review workflow

The end-to-end loop from idea to merged code:

1. **Create an idea** in your product. Use the AI loop to sharpen the hypothesis.
2. **Break it into tasks**, one per discrete piece of work.
3. **Pick up a task** by telling your agent in Claude Code: *"Pick up task #N."* The agent calls `work_on_next_task` and the task moves to `inProgress`.
4. **Branch and code** in your `routeburn-alpha/product-{handle}` repo (the agent does most of this).
5. **Open a PR** back to `main`. The GitHub webhook attaches the PR to your task automatically.
6. **Submit for review.** The task moves to `review` and the assigned reviewer gets pinged on the PR.
7. **Reviewer responds:**
   - Approves and merges → task auto-transitions to `done`.
   - Requests changes → task moves to `changesRequested`. Address feedback, push again, re-request review.
8. **Idea moves forward** as its subtasks complete. You decide when it's `All Customers` (proven), `Design Partner`, or `failed`.

The webhook closes the loop between GitHub and Studio AI. If you push a PR and don't see it linked to your task within a minute, that's a bug. Flag it.

## 8. Creating an idea with AI

The headline alpha flow. From your `ideas` page:

1. Click **New idea** and type a rough description, bullet points, half-sentences, whatever's in your head. Don't pre-polish it.
2. Studio AI runs a **grammar + spelling pass** to clean it up without changing meaning. Accept or edit.
3. Optionally trigger the **full spec build**. The AI expands your rough idea into a hypothesis, success criteria, and a draft technical design. You stay in the loop and can edit any section.
4. From the finished idea, generate **tasks** that break the work down. Each task can be picked up by you or an agent.

This is the loop we most want feedback on. Where does the AI help, where does it get in the way, where does it hallucinate?

## 9. Working a task end-to-end

1. Open a task in your product.
2. Pick it up (status → `inProgress`). The task is now assigned to you.
3. Clone your `routeburn-alpha/product-{your-handle}` repo locally and make changes on a branch.
4. Push the branch and open a PR back to `main`. The webhook attaches the PR to the task automatically.
5. Submit for review. Task moves to `review`.
6. Merge the PR. Task auto-transitions to `done` and the loop is complete.

If the webhook ever misses a PR, ping the feedback channel. That's exactly the kind of edge case we want to catch in alpha.

## 10. Studio AI MCP: the agent loop

Studio AI ships an MCP server so you can drive the whole loop from Claude Code (or any MCP-compatible agent). Instead of clicking through the web UI, you tell your agent things like *"work on the next backlog task"* and it talks to Studio AI directly.

### Setup (one-time per machine)

Skip if you've already done these. Otherwise:

1. **Install Claude Code.** Follow the install guide at [claude.com/claude-code](https://claude.com/claude-code). Any other MCP-capable client works too.

2. **Configure the Studio AI MCP server.** Create or edit `.mcp.json` at the root of your product worktree:

   ```json
   {
     "mcpServers": {
       "studio-ai": {
         "type": "http",
         "url": "https://app.routeburn.org/api/mcp",
         "headers": {
           "Authorization": "Bearer <token>"
         }
       }
     }
   }
   ```

   Replace `<token>` with the Bearer token Cassie sends you in Slack. The token is your personal credential.

   **Do not commit this file.** Add `.mcp.json` to your `.gitignore` if it isn't already. Distributing bearer tokens by hand is a known foot-gun, so keep this one local until we ship a real auth flow.

3. **Restart Claude Code** in your worktree directory to load the new server.

Confirm by asking your agent to call `list_products`. A response means you're wired up.

### What each Studio AI MCP tool does

Tools are grouped by what they're for. Your agent picks the right one based on the request.

**Discovery: figure out what's there**

| Tool | Purpose |
|---|---|
| `list_products` | List all products in the studio. Start here if you don't know your productCode. |
| `list_studio_members` | List people in the studio (name, email, role). Use to resolve a name to an email. |
| `list_tags` | List existing tags so you don't duplicate them. |
| `list_agents` | List registered agents for the studio. |
| `list_execution_agents` | List the available agent execution modes (supervised, managedOpus, managedSonnet). |
| `list_environments` | List configured environments for the studio. |
| `load_product_context` | Load strategy, vision, and PRDs for a product. Run at the start of brainstorming to align with product goals. |

**Ideas: the validation layer**

| Tool | Purpose |
|---|---|
| `create_idea` | Create an idea with a hypothesis (what you want to prove) and optional technical design. |
| `update_idea` | Edit an idea's name, hypothesis, technical design, owner, reviewers, or validation status. |
| `get_idea` | Fetch a single idea by number, including its linked tasks. |
| `get_ideas` | List ideas across a product (or the whole studio), grouped by validation status. |
| `delete_idea` | Soft-delete an idea (recoverable by admin). |

**Tasks: the unit of work**

| Tool | Purpose |
|---|---|
| `create_task` | Add a new task to the backlog. Link to an idea with `ideaNumber` to make it a subtask. |
| `update_task` | Edit a task's name, spec, tags, owner, reviewers, PRs, resources, or status. |
| `get_task` | Fetch a single task by number: full spec, comments, status history, PRs. |
| `get_tasks` | List tasks, filterable by status / type / tags / owner / reviewer / agent. |
| `delete_task` | Soft-delete a task. |
| `create_comment` | Post a markdown comment on a task. Good for progress notes. |

**Working a task: the core agent loop**

| Tool | Purpose |
|---|---|
| `work_on_next_task` | Pick up a task (specific number, or oldest backlog). Marks it `inProgress` and assigns to the agent. Use this to start work. |
| `release_task` | Drop a task back to the backlog if you picked it up by mistake. |
| `submit_for_review` | Submit completed work. Include the PR you opened. The task moves to `review` and reviewers get auto-assigned on the PR. Without a PR, it moves to `done`. |

**Search / context**

| Tool | Purpose |
|---|---|
| `search` | Search ideas, tasks, and context docs. Supports filter syntax like `type:idea status:firstLevel`, `taskType:bug`, etc. |
| `search_context` | Get context documents linked to an idea, or search studio-wide docs by query. |

**Agent management**

| Tool | Purpose |
|---|---|
| `register_agent` | Register a new agent with a name, execution type, and capabilities. |
| `deregister_agent` | Remove an agent by name. |

**Auth**

| Tool | Purpose |
|---|---|
| `logout` | Clear stored credentials. You'll re-auth on next request. |

### Typical session

```
You: "What's on my plate?"
Agent: → get_tasks(owner: "me")

You: "Pick up #42."
Agent: → work_on_next_task(taskNumber: 42)

You: "Done, PR is #117."
Agent: → submit_for_review(taskNumber: 42, pullRequests: [...])
```

## 11. Feedback channels

You'll be added to two Slack channels:

- **`#studio-ai-alpha`**: questions, collaboration, real-time chatter. Use this for "is anyone else seeing X?", trading PR reviews, sharing spec-build screenshots, and the Slack rituals from Part E of the walkthrough.
- **`#studio-ai-feedback`**: feedback drop. We'll prompt the channel at the end of each week with a couple of questions. Reply in-thread, or post unprompted whenever something hits you.

Give us *everything*. We mean it:

- **Surprises:** places the product behaved differently than you expected.
- **Friction:** where you got stuck, what you had to re-read, what felt manual.
- **Wishes:** features you reached for that didn't exist.
- **Naming:** if "Studio AI" doesn't land, tell us. If the URL hierarchy feels weird, tell us. If "demo-alpha-studio" sounds like a placeholder we forgot to rename, tell us.
- **Spec-build outputs that made you laugh.** Good *or* bad. Especially bad.
- **Anything that felt like a hack.** Workarounds you invented to get past something rough.
- **What you almost said but didn't.** That's usually the most useful one.

Feedback is a gift and we love gifts. The earlier you tell us what's off, the more time we have to fix it before it ossifies.

## 12. Known limitations

Things you'll likely run into. All intentional for alpha, listed here so you don't waste cycles reporting them:

- **No product-level ACLs.** Access is enforced at the GitHub repo level via outside-collaborator scoping. Anyone with a `product-{handle}` URL can in principle visit it; we're relying on URL obscurity + GitHub repo permissions for the alpha cohort. Don't paste production secrets here.
- **N+1 webhook scan.** PR routing scans repos one-at-a-time. Fast at our current scale; will get slow if we add many more studios or products to this org. Optimization deferred.
- **No invite/welcome email automation.** You got this doc by hand; future alpha users will too until self-serve sign-up lands.
- **Operator-driven onboarding.** Adding a new alpha user runs through a script (`scripts/provision-alpha.ts`), not a UI. Self-serve "create studio" is a known gap.

## 13. Walkthrough: your first agent and your first task

Now you've seen the moving parts. This section ties them together end-to-end. Plan for ~30 minutes for your first agent and first task. Additional agents are a few minutes each.

The unit you'll be working with is an **agent**. An agent is a named Claude Code session (or any MCP-capable client) wired up to Studio AI. Each agent has its own local workspace, its own identity in the studio, and can claim one task at a time. Spinning up multiple agents is how you get parallel work: each agent owns one task in flight.

### Best practices: running parallel agents in the same repo

Multiple agents working on the same repo is the point of this setup, but it's where you'll hit collisions if you're not deliberate. The core question is **what unit you isolate at**: working tree, repo clone, process environment, or full machine. Each layer eliminates a different class of collision; most teams end up combining two.

**The isolation spectrum**

| Layer | When to use | What to watch |
|---|---|---|
| **Git worktrees** (`git worktree add`), recommended default for alpha | Agents on independent branches, shared host environment is fine | Can't check out the same branch in two worktrees; hooks and local git config are shared; `node_modules` is per-worktree but `~/.npm` and `~/.cache` are not |
| **Multiple full clones** | Stronger git isolation, agents need different submodule states or remotes | Disk usage, drift between clones |
| **Containers** (devcontainer or docker per agent) | Agents install packages, modify system state, or need different runtimes | Build time, volume mounts, port mapping |
| **Cloud sandboxes** (Codespaces, ephemeral VMs) | Parallelism exceeds local resources, or you want auditable/replayable environments | Latency for supervision, per-minute cost, secrets management |

A reasonable default for alpha: **worktrees for interactive local parallelism**. 

We're shipping three Claude Code skills into the product repos before alpha launch to make this easier:

| Skill | What it does |
|---|---|
| `/create-agent` | Creates a new agent workspace, sets up a git worktree, and bootstraps the environment |
| `/bootstrap-agent` | Repairs an existing agent workspace, reinstalls deps, and verifies tests pass when something's out of sync |
| `/worktree-status` | Reports on workspace health (clean tree, up-to-date with main, stale branches) |

They're optional. Use whatever workflow works best for managing your agents, whether that's these skills, a custom script, or doing it by hand.


### Part A: Stand up your first agent

> **Optional workflow.** The steps below use the checked-in skills (`/create-agent`, `/bootstrap-agent`, `/worktree-status`) because they're the fastest path. You don't have to use them. Drive your worktrees, clones, or containers however works best for you.

**1. Run `/create-agent`** from a Claude Code session in the product repo. Pick a name when prompted (`felix`, `kevin`, `alice`, anything memorable). The skill creates a git worktree on a new branch and stubs out the environment.

**2. Open Claude Code in the new workspace.** A directory is created for the agent (e.g. `product.quiz-felix`). Open a new Claude Code session there.

**3. Run `/bootstrap-agent`.** Installs dependencies, regenerates config, writes the env file, and verifies the product app runs. Idempotent, so safe to re-run anytime the workspace feels stale.

**4. Register the agent in Studio AI.** Ask your agent in plain language: *"Register me as a supervised agent named `quiz-felix`."* It calls `register_agent` for you. Supervised mode is for the local-agent loop in this part of the walkthrough; for the cloud-run alternative, see [MANAGED-TASKS.md](./MANAGED-TASKS.md). If you want to confirm, ask: *"List the registered agents."*

**5. Sanity-check anytime with `/worktree-status`.** Reports the workspace state (clean, branch, sync with `main`). Useful before picking up a task and after long-running agent runs.

### Part B: Get an idea ready

**6. Pick a starter idea.** Good first ideas for Quiz Lab, small enough to land in one PR:

- **"Show the explanation after each answer."** A toggle that reveals a per-question explanation once you've answered.
- **"Confidence rating per question."** A 1–5 slider before answering, surfaced on the results page.
- **"Topic breakdown on results."** Group the final score by topic instead of one total.
- **"Personalized welcome screen."** Greet the user by name and show their last score.

**7. Create the idea in Studio AI.** From `/demo-alpha/demo-alpha-studio/product-{your-handle}/ideas`, create a new idea. Type a rough hypothesis (bullets are fine), then run the AI grammar pass and optionally the full spec build.

> Example hypothesis: *"Showing the explanation immediately after answering will keep users engaged for more questions per session, because they get learning value even when they get it wrong."*

**8. Break the idea into one or more tasks.** From the idea page, add a task (or two, or three). Each task is a discrete piece of work, small enough that one agent can land it in one PR.

### Part C: Hand the task to your agent

**9. Tell your agent to pick up a task.** From inside Claude Code:

```
"Pick up task #N and work on it."
```

Your agent will call `work_on_next_task(taskNumber: N)`. The task transitions to `inProgress` and is now bound to this agent.

**10. Let the agent work.** It will branch, write code in your worktree, run tests, commit, and push. You stay in the loop, review what it's doing, push back when it's off course, accept when it nails the spec.

**11. Open the PR.** When the agent is ready, it (or you) opens a PR from the feature branch back to `main`. The webhook attaches the PR to the task within seconds.

**12. Submit for review.** Either through the task page or via MCP:

```
submit_for_review(
  taskNumber: N,
  pullRequests: [{
    repoOwner: "routeburn-alpha",
    repoName: "product-{your-handle}",
    prNumber: <pr#>,
    url: "https://github.com/routeburn-alpha/product-{your-handle}/pull/<pr#>",
    status: "ready"
  }]
)
```

**13. Merge.** When the PR is approved and merged, the task auto-transitions to `done`. If the reviewer requests changes, the task moves to `changesRequested`. Push fixes to the same branch and re-request review.

**14. Advance the idea.** Back on the idea page, decide what's next:
- Running with a real user/team → mark it `Design Partner`.
- Proven and ready to ship widely → mark it `All Customers`.
- More subtasks needed → add them under the idea and keep building.
- It didn't work → mark it `failed` and write a sentence about why. Killed ideas are real outcomes.

### Part D: Spin up a second agent for parallel work

Once one agent is humming, you can add more. Each agent claims its own task and works independently.

**15. Run `/create-agent` again** with a different agent name (`iris`, `kevin`, whatever). Same flow as Part A. The skill creates a fresh worktree and stubs the environment.

**16. Bootstrap and register the new agent.** Open Claude Code in the new workspace, run `/bootstrap-agent`, then `register_agent` for the new name. From then on, the new agent can pick up tasks in parallel with the first one.

**17. Avoid collisions.** Before starting both dev servers, walk through *"The collision checklist"* sub-section above. The most common ones in practice: ports (assign each worktree its own `PORT` in `.env.local` at the root of the worktree) and local DB files (give each worktree its own SQLite path or namespace).

**18. Distribute work intentionally.** Only one agent can have a given task `inProgress` at a time. `list_agents` shows what's registered; `get_tasks(status: "inProgress")` shows who's working on what.

### Part E: Collaborate with other alpha members

The alpha cohort is small enough to feel like a working group, not a beta panel. Studio AI is designed to support that. It isn't just a personal todo list with AI on top, it's a shared surface for evaluating each other's thinking. A few patterns to try.

**Inside Studio AI**

- **Cross-review tasks.** Add another alpha user as a reviewer on your task. When you submit for review, they get pinged on the PR. Faster turnaround than waiting on Tim or Cassie, and you learn from seeing how others read a spec.
- **Comment on each other's ideas and tasks.** The `create_comment` MCP tool (or the comment field in the UI) drops a markdown comment with full status-history attribution. Use it to challenge a hypothesis, sanity-check a spec, or share what you tried that didn't work.
- **Browse the studio.** Call `get_ideas` (no `productCode`) or `get_tasks` (no `productCode`) to see what everyone in `demo-alpha-studio` is working on. Use `search "<term>"` to find related thinking before you start your own.
- **Use the shared `product-collab` product.** Drop joint ideas there. Useful for things that benefit from multiple perspectives: rubrics, content sets, shared utilities your individual apps might pull from.

The bet behind alpha: small-group collaboration around AI-assisted product work generates a different kind of feedback than one user talking to one PM ever does. Make the cohort dynamic part of the experiment, not just the tooling.

### Part F: Try a managed task (async, no local agent)

Once you've shipped a task or two with a supervised local agent, try the other lane: **managed tasks**. Instead of running Claude Code in your worktree, Studio AI runs Claude in the cloud against your repo, opens the PR, and submits for review on your behalf. You don't need a worktree, a registered agent, or Claude Code open. You pick the task, pick the model (Opus or Sonnet), and click **Execute**.

**The short version:**

1. Open a task in the backlog with a clear spec.
2. In the **Execution** section, set the agent to **Managed (Opus)** or **Managed (Sonnet)**.
3. Click **Execute**. The task moves to `inProgress` and the run starts.
4. Watch the run in the task's **Executions** tab (live event stream: messages, tool calls, bash, costs).
5. When the agent opens a PR and submits for review, you review and merge as normal.

Good fits: well-specified small features, refactors, doc updates, anything you'd otherwise babysit a local agent through without intervening. Bad fits: exploratory work, fuzzy specs, anything that needs you to course-correct mid-run.

Full walkthrough, model trade-offs, prerequisites, and limits: [MANAGED-TASKS.md](./MANAGED-TASKS.md).

### What you'll have at the end

- A working agent registered in Studio AI, wired up to a worktree with credentials.
- One complete idea → task → PR → merge loop.
- A second agent set up for parallel work.
- A few peer rituals running in Slack so the cohort feels like a working group, not a panel.
- Material for your first feedback message. That's the most valuable thing to come out of alpha.

Thanks for trying this. The earlier you tell us what's broken, the more time we have to fix it before it ossifies.
