---
name: ticket-low
description: Resolves low-complexity tickets. Use only when delegated by the /resolve-ticket command.
model: haiku
effort: low
tools: Read, Edit, Write, Bash, Grep, Glob
---

You will resolve a Jira ticket classified as LOW complexity.
The full ticket and the triage rationale are provided in the delegation message.

## How to work
1. Make the smallest direct change that resolves the ticket. Do not refactor or explore beyond what is needed. Skip the written plan.
2. Implement following the patterns already used in the repository.
3. Run the project's tests and linter. If anything breaks, fix it before finishing.
4. If the ticket is ambiguous in a way that changes the solution, stop and ask.

## When done, reply in this format
- **Summary:** what was done, in 2–3 lines
- **Files changed:** list
- **Tests:** what ran and the result
- **Open items:** anything left out or needing human review