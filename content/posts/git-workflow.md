+++
title = 'Git Workflow'
date = 2024-06-28T13:51:58-07:00
+++

### MY SOLO WORKFLOW
1. Make changes to files
2. `git add .` (or `git add <files>` if I only want to add specific files)
3. `git commit -m "a message describing the changes"`
4. `git push origin main`

### MY TEAM WORKFLOW
- Update my local `main` branch with `git pull origin main`
- Checkout a new branch for the changes I want to make with `git switch -c <branchname>` 
- Make changes to files
- `git add .`
- `git commit -m "a message describing the changes"`
- `git push origin <branchname>` (I push to the *new* branch name, not `main`)
- Open a pull request on GitHub/GitLab/Bitbucket to merge my changes into `main`
- Ask a team member to review my pull request
- Once approved, click the "Merge" button on GitHub to merge my changes into `main`
- Delete my feature branch, and repeat with a new branch for the next set of changes

### WHAT TO IGNORE (.gitignore)
Some rules for coding projects:

1. Ignore things that can be *generated* (e.g. compiled code, minified files, etc.)
2. Ignore dependencies (or don't have them in the first place) (e.g. `node_modules`, `venv`, `packages`, etc.)
3. Ignore things that are personal or specific to how you like to work (e.g. editor settings)
4. Ignore things that are sensitive or dangerous (e.g. `.env` files, passwords, API keys, etc.)
