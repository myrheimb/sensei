---
description: Review the task you finished, run its checks, and tick it when it holds
argument-hint: "[task number in the approved phase]"
---

Load the `sensei:sensei` skill with the Skill tool and follow it. It holds the file formats and the rules.

1. Find the phase with status `approved` with `grep -l '^Status: approved' docs/plan/phases/*.md`. If there is none, tell the user to run `/sensei:plan` and stop.
2. Print its outline with `grep -n -E '^(## |### Task |- \[[ x]\] [0-9]+\.)' <phase file>`. Take task number $ARGUMENTS of that phase, or its first unticked task if no number was given. If every task is ticked, suggest `/sensei:close` and stop. With the Read tool and the outline's line numbers, read the Milestone, this task with its "Done when", its log entry if it has one, and Decisions.
3. Run the checks for each "Done when" item and note the command and its exit code. A check you cannot run goes to the user to run, with the command. Do not fix anything that fails. Then snapshot the working tree with `git stash create`, and if it prints nothing, as on a clean tree, use `git rev-parse HEAD`. If it fails, as in a sandbox that blocks writes inside `.git/`, use `none`. Keep the hash for the `Review:` line.
4. Hand the task to the `reviewer` agent as the path of the phase file, the task number, the check results and, on a later round, the task's earlier `Review:` line, which holds the earlier snapshot hash. If you cannot start it, say so and stop. Do not review it yourself. A reply that does not follow the reviewer's format counts as blocked: show the user what is in the way and ask them to decide.
5. Tell the user, in this order and in the skill's tone with no judgement:
   - The failing checks, if any, and what they print.
   - On a later round, which earlier findings are fixed.
   - The reviewer's bugs, each with its `path:line` and how to see it fail. Give no fix. Hints on request follow the skill's Hints section.
   - At most three improvements as the reviewer phrased them, then what went well.
6. If a check fails or a bug remains, the task stays unticked. Add the log's `Review:` line with the bugs, or on a later round replace the earlier `Review:` line with the current round's bugs and which earlier findings are fixed, never adding a second one. Either way the line ends with `Snapshot: <hash>`, the hash from step 3, so the next round has a baseline. Say the user can run `/sensei:review` again when they have looked. There is no limit on rounds and no pass or fail verdict.
7. If every check passes and no bug remains:
   - Tick the task and add its log entry: `Done:`, the `Tests:` line next wrote, `Review:` if bugs were found and fixed, replacing the earlier one and ending with `Snapshot: <hash>` as in step 6, and `Next needs (task N):` or `Next needs (later tasks):` for what a later task must know.
   - Add each item of the task that no check covers to the phase's `After deploying` list, if it has one, or create the list.
   - Suggest the commit as the skill's Commits section says, and do not run it.
8. Do not start the following task. The user runs `/sensei:next` for it.
