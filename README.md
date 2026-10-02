# fusion-ai-agent

An automation agent that keeps GitHub tickets in sync with the development process. It reacts to pushes and to status changes on the project board, so ticket updates don't depend on remembering to do them by hand.

Built for the lab "Créer son premier workflow" on [name of internal automation platform].

## Why

My daily process is: pick a ticket, branch from `dev`, push commits, merge into `dev`, then into `test`. Keeping the ticket status and comments up to date along the way is repetitive and easy to forget. This agent does it automatically.

## Workflows

| # | Workflow | Trigger | What it does |
|---|----------|---------|--------------|
| 2 | **Push Tracker** | GitHub `push` webhook | Posts one comment per push on the matching ticket and detects the `[last]` commit |
| 3 | **Status Reactor** | Project item status changed | Posts a recap on Done, and creates a new round branch on rework |

### Push Tracker

- **Input:** GitHub push event
- **Processing:** verify the signature, ignore bot pushes and deleted branches, extract the ticket ID from the branch name, and build a summary of the commits
- **Output:** one comment on the ticket. If a commit message contains `[last]`, the ticket moves to **Ready for Test**

### Status Reactor

- **Input:** project item status change
- **Processing:** ignore changes made by the bot, then switch on the new status
- **Output:**
  - **Done:** a recap comment (commits, PRs, branches, tester notes)
  - **Back to In Progress (rework):** a new branch `feature/<id>-<title>-r2` (`-r3`, and so on) and a comment with the tester's feedback

## Conventions

**Branch names:** `feature/<ticket-id>-<short-title>`, for example `feature/12-login-fix`

**Rework branches:** same name with a round suffix, for example `feature/12-login-fix-r2`

**Commit tag:** include `[last]` in the commit message when the work is finished, for example `fix login redirect [last]`

**Statuses:** `Backlog` → `Todo` → `In Progress` → `Ready for Test` → `In Testing` → `Done`

## Setup

1. Create a webhook trigger node in the platform and copy its URL
2. In the repo or organization webhook settings, set:
   - Payload URL: the node URL
   - Content type: `application/json`
   - Secret: stored in the platform
   - Events: `push` (Push Tracker), `projects_v2_item` (Status Reactor)
3. Store the GitHub token and webhook secret as platform
