---
description: Set up the next task of the approved phase with failing tests and a first hint
argument-hint: "[task number in the approved phase]"
---

Load the `sensei:sensei` skill with the Skill tool and follow it. It holds the file formats and the rules.

1. Find the phase with status `approved` with `grep -l '^Status: approved' docs/plan/phases/*.md`. If there is none, tell the user to run `/sensei:plan` and stop.
2. Print its outline with `grep -n -E '^(## |### Task |- \[[ x]\] [0-9]+\.)' <phase file>`. Take task number $ARGUMENTS of that phase, or its first unticked task if no number was given. The number never names a phase. If every task is ticked, suggest `/sensei:close` and stop. Read only what this command needs: with the Read tool and the outline's line numbers, read the Milestone, this task, its log entry if it has one, and Decisions. Find the `Next needs` lines addressed to it with `grep -n 'Next needs (task <number>)' <phase file>`. Read the rest only when the user asks about it.
3. If `git status` shows uncommitted changes outside `docs/plan/`, list the files and ask the user to commit them before the task starts, so the task's changes stand alone. Suggest the commit as the skill's Commits section says. Skip this step when the task's log already shows tests were written for it.
4. Read the task's `Tests:` marker. Without one, the tester is the default.
   - `Tests: tester`: hand the task to the `tester` agent as the path of the phase file and the task number, nothing more. Do not read the code yourself first. If you cannot start the `tester` agent, say so and stop. Do not write the tests yourself. When it returns, a reply that does not follow the tester's format counts as blocked.
     - Blocked: show the user what is in the way and the tester's recommendation, and ask them to decide. Record the answer under Decisions. If it changes what another task must do, rewrite that task and its "Done when" now, as rule 10 asks. Then hand the same task to a new `tester`.
     - Unverified: tell the user which tests did not run and why, and ask for the result of running them.
     - Done: add a `Tests:` line to the task's log naming the test files and `written by tester`. Tell the user which test files were added and the command that runs them, and that they fail for now.
   - `Tests: me`: skip the tester. Tell the user what the task asks for, then hint at what to test, as the skill's Hints section says: a question or a pointer to what the "Done when" can be checked with, never the test code. Add a `Tests:` line to the log once the user says they wrote them, naming the files and `written by the user`.
5. End with the task in a sentence, then one first hint, as the skill's Hints section says: a question or a pointer to a concept or doc, never the full approach and never solution code. Make it the smallest hint that gets the user started. Give no second hint until the user asks.
6. Do not start the following task. When the user says they are done, they run `/sensei:review`.
