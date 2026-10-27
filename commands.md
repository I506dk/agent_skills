# Slash Commands

These slash commands provide a shortcut for managing our agent skills repository.

## /skills setup
Use this command to initialize the skills repository.
- This will prompt you to provide or confirm the `SKILLS_REPO_PATH`.
- It will verify that the directory exists and is accessible.
- It will check for and install the `gh` CLI if it's missing.
- It will check for GitHub authentication and prompt for `gh auth login` if necessary.
- It will update the metadata and configuration if necessary.

## /skills update
Use this command to refresh the list of available skills.
- This will scan the repository for all `.md` files.
- It will update the "Available Skills" list in `README.md` and `metadata.json`.
- It will list the newly discovered skills and their descriptions.
- **Branch Safety**: Before performing any updates/pulls, check that the current branch is `main`. If it is not `main`, notifications will be sent to the user and no updates will be fetched/pulled.
- **Note**: After updating, you may need to start a new session for the latest skill definitions to be fully integrated into the agent's primary context.
