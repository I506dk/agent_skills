# Agent Skills
  
Collection of agent skills for various languages, tools, and tasks.
  
## Slash Commands
- `/skills setup`: Initialize the skills repository.
- `/skills update`: Refresh the list of available skills.
- `/skills help`: Display a summary of all available slash commands.
  
## Installation & Setup
  
By default, this repository is expected to be located at `~/agent_skills`.
  
To "install" this repository as a skill provider for your agent, first ensure the directory exists:
  
```bash
mkdir -p ~/agent_skills
```
  
**Initial Setup**: 
Agents can also use the initialization prompt located in `initialization_prompt.md` to automatically perform all necessary environment checks (directory creation, `gh` CLI verification, and authentication checks).
  
Then, include the following block in your agent's **System Prompt** (replacing `{{SKILLS_REPO_PATH}}` with your local path, e.g., `/home/user/agent_skills`):
  
```markdown
## Local Skills Repository
You have access to a local repository of skills at `{{SKILLS_REPO_PATH}}`. 
  
### How to Load Skills:
1. To see available skills, list the files in the repository and read their descriptions.
2. To "install" or "load" a specific skill, read its `.md` file and append its "Full Prompt" section to your current system instructions.
3. Use the `setup_skills` skill to manage these operations.
```
  
## Available Skills
- `python`: For writing, refactoring, and reviewing Python code.
- `github`: For managing GitHub repositories, branches, and pull requests.
- `podman`: For managing Podman containers and images.
- `setup_skills`: For managing and loading other skills in this repository.
