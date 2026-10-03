---
name: fm-control-freak
description: Firstmate layer for tasks the captain drives hands-on - the worker orients and waits, the captain codes in its pane from a tmux session rooted in the task worktree. Use when the captain invokes /fm-control-freak or asks to work a task hands-on, and before cleaning up any task that has an fm-<task-id> tmux session.
user-invocable: true
metadata:
  internal: true
---

# fm-control-freak

A hands-on task keeps firstmate's bookkeeping (backlog, isolated worktree, validation, PR tracking, cleanup) and hands the implementation to the captain.
The worker is a _copilot_: it orients, then follows the captain.
Everything below layers on firstmate's normal lifecycle in `AGENTS.md`; that contract still governs intake, delivery mode, merge authority, and teardown.

## Start a hands-on task

1. **Intake as usual.**
   Resolve the project, delivery mode, and yolo posture, file the backlog item, and scaffold the brief with `bin/fm-brief.sh`.
   Record `hands-on` in the backlog item note.
   Pass `--branch-prefix rem0te/` to both `bin/fm-brief.sh` and `bin/fm-spawn.sh` in place of the project's registered prefix, so the branch is named for the captain who writes it; work firstmate or a worker implements keeps the registered prefix.
2. **Fill the brief for a copilot.**
   `## Captain's intent` carries the captain's ask as usual.
   `## Firstmate spec` carries this block, with the files to read filled in:

   > This is a captain-driven session. The captain works hands-on in this pane and may edit files in this worktree from their own editor.
   > - After the isolation check and branch step, orient cheaply: read <the files the ask names or clearly implies>, and nothing broader.
   > - Print a summary of at most eight lines in the pane: the relevant code paths, where the change goes, and what test would pin it. Append `paused [at=<epoch>]: oriented, waiting for the captain to work hands-on in this pane` to the status file and stop.
   > - From then on, take direction from the captain's input in the pane; it is authoritative. Make only the changes the captain asks for.
   > - When the captain says the work is done, make sure it is committed on your branch with a regression test where behaviour changed, then follow the Definition of done.

   Spawn at `--effort low` unless the captain says otherwise, because the captain supplies the reasoning.
3. **Spawn, then open the captain's session.**
   After `bin/fm-spawn.sh` succeeds, read `window=` and `worktree=` from `state/<id>.meta` and run:

   ```sh
   tmux new-session -d -s fm-<id> -c <worktree>
   tmux link-window -s <window> -t fm-<id>:
   ```

   `link-window` shows the same window in both sessions.
   Firstmate addresses the worker by the recorded `window=` value, so the window stays linked in place.
   The step is done when `tmux display -p -t fm-<id>: '#{session_path}'` prints the worktree and `tmux list-windows -t fm-<id>` shows the worker window.
4. **Hand over.**
   Tell the captain `tmux switch -t fm-<id>`, the worktree path, and the branch.
   The session's working directory is the worktree, so the captain's prefix+g lazygit popup opens on the task branch.

## While the captain works

The worker's `paused:` line tells monitoring the idle pane is expected.
A worker idle long enough to raise a stale alert while the captain is driving is healthy; confirm the pane is waiting on the captain and move on.
When the worker reports `done:`, start validation exactly as for any other task.

## Before cleanup

Cleanup force-kills every process whose working directory is inside the worktree, including anything the captain has open there.
Before `bin/fm-teardown.sh` on a task with an `fm-<id>` session:

1. List what is running in the session: `tmux list-panes -s -t fm-<id> -F '#{window_name} #{pane_current_command}'`.
2. List everything else rooted in the worktree: `lsof -d cwd -Fpcn 2>/dev/null` and keep the processes whose `n` path is inside the worktree.
3. Anything beyond the worker's own window and idle shells (an editor, lazygit, a running command, a shell the captain is typing in) belongs to the captain: name it, ask the captain to close it, and wait for the captain's go.
4. With the captain's go, or when only idle shells and the worker remain, run `tmux kill-session -t fm-<id>`, then clean up as usual.
