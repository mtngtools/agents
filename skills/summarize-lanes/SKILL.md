---
name: summarize-lanes
description: Print a detailed summary of running lanes, with each lane's tickets, PR and merge status, all linked.
argument-hint: "The lanes' agent:communication issue (number or URL), if the repo has more than one open; the repo, if not the current one"
disable-model-invocation: true
metadata:
  type: command
  invocation: human-only
  applies-to: [building, parallel, sessions, github, prs]
---

# Summarize Lanes

> **human-only.** Start this only when a human asks for it by name. If you arrived here from anywhere but a human naming it, stop.

Where the lanes that a `/tickets-to-lanes` run started stand right now, printed in the one [detailed summary](./detailed-summary.md) format the lifeguard also prints at milestones. **This is a read.** It writes nothing: no comment, no message, no notification.

## Process

### 1. Find the lanes' issue

Take the issue from the argument. If none was given, run `gh issue list --repo <owner/name> --label agent:communication --state open`: use the issue if there is exactly one, and ask the human which one if there are none or several.

**Done when:** you have one issue number and one `owner/name`.

### 2. Gather and print

Follow [detailed-summary.md](./detailed-summary.md): gather everything it lists, then print the summary with `on request` in the header.

**Done when:** every lane has a row, every ticket in the issue's lanes table has a state and a link, and every PR that exists has its merge status.

## Where this sits in the flow

Beside the lanes, at any time. `/tickets-to-lanes` starts them, `/lifeguard-in-lanes` watches them and prints this same summary at milestones, and this skill prints it whenever you ask.
