Please analyze and fix the Jira ticket: $ARGUMENTS.

Follow these steps:

1. Understand the problem described in the ticket
2. Search the codebase for relevant files
3. **Create a new feature branch** from the repository base branch (`main` / `master` / `develop`). Name it after the ticket or OpenSpec change (e.g. `feature/SCRUM-123-payroll-config`). **Do not** implement on the base branch directly (see `ai-specs/specs/git-workflow.mdc`).
4. Implement the necessary changes to solve the ticket, following the order of the different tasks and making sure you accomplish all of them in order, like writing and running tests to verify the solution, updating documentation, etc.
5. Ensure code passes linting and type checking
6. Stage only the files affected by the ticket, and leave any other file changed out of the commit. Create a descriptive commit message (English)
7. Push the feature branch and **open a Pull Request** with `gh pr create`. **Do not merge** the PR: the **maintainer must review and approve** before merge. Use the ticket ID in the PR title/body (e.g. SCRUM-123) for Jira linking when applicable

Remember to use the GitHub CLI (`gh`) for all GitHub-related tasks.