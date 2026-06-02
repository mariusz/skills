---
name: mariusz-git-worktrees-setup
description: Mariusz's preferred bare-repo + git worktrees setup workflow for cloning any repository. Use when the user asks to clone a repo, set up a new repository locally, initialize a workspace with worktrees, set up bare clone, or mentions "worktree setup", "clone with worktrees", or wants the standard repo layout for a new project.
---

# Mariusz Git Worktrees Setup

Always use this exact workflow when cloning **any** repository for Mariusz. Never substitute a plain `git clone`.

## Quick start

Given a repo URL `git@github.com:user/repo.git` and a target folder name `repo`:

```bash
# Create and enter the workspace folder
mkdir repo && cd repo

# 1. Clone the repo into a hidden .bare folder
git clone --bare git@github.com:user/repo.git .bare

# 2. Tell the root folder where the Git history is hidden
echo "gitdir: ./.bare" > .git

# 3. Fix the fetch configuration to see all remote branches
git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"

# 4. Fetch all remote branches
git fetch origin

# 5. Create the main worktree (use the repo's actual default branch: main, master, develop, etc.)
git worktree add main

# 6. Optionally create additional worktrees for branches you'll work on
git worktree add <branch-name>
```

## Workflow checklist

When the user asks to clone or set up a repo:

1. Confirm repo URL (SSH preferred: `git@github.com:...`).
2. Confirm target folder name (default: repo name).
3. Confirm default branch name (`main`, `master`, etc.) — inspect with `git ls-remote --symref <url> HEAD` if unknown.
4. Run steps 1–5 above.
5. Ask which additional branches (if any) need worktrees and add them with `git worktree add <branch>`.
6. Report final layout to the user.

## Resulting layout

```
repo/
├── .bare/          # Bare repo — Git internals live here
├── .git            # Plain text file: "gitdir: ./.bare"
├── main/           # Worktree for main branch
└── <branch>/       # Additional worktrees as needed
```

## Rules

- **Always** use `--bare` clone into `.bare/`. Never a regular clone.
- **Always** write the `.git` pointer file in the repo root.
- **Always** fix the fetch refspec — without it, remote branches won't appear.
- **Always** `git fetch origin` after step 3 so worktrees can resolve branches.
- **Never** create a worktree inside `.bare/`.
- **Never** commit to or modify `.bare/` directly.

## Common operations after setup

```bash
# Add a worktree for an existing remote branch
git worktree add <branch-name>

# Add a worktree for a new branch off main
git worktree add -b feature/x feature-x main

# List worktrees
git worktree list

# Remove a worktree
git worktree remove <branch-name>

# Update all branches (run from any worktree)
git fetch origin
```

## Troubleshooting

- **`fatal: '<branch>' is not a commit`** → run `git fetch origin` (step 4 was skipped).
- **Remote branches missing** → re-run step 3, then `git fetch origin`.
- **`.git` not recognized** → must be a plain file containing exactly `gitdir: ./.bare`, no trailing content.
