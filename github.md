# 🐙 GitHub & Git Notes

---

### 1) Initialize Git

To initialize Git in any folder or directory, run the command:

```bash
git init
```

After running this, a **"U"** symbol (which means **Untracked**) will appear next to all files in your editor (e.g., VS Code).

---

### 2) Enable Tracking (Stage Files)

To enable tracking of specific files:

```bash
git add filename1 filename2 ...
```

To stage everything in the current folder:

```bash
git add .
```

---

### 3) Stop Tracking (Remove Files)

To remove a file from tracking, run:

```bash
git rm <filename>
```

> [!TIP]
> **Keep file locally, stop tracking:**
> If you want to stop tracking a file but keep the local file on your hard drive, run:
>
> ```bash
> git rm --cached <filename>
> ```

---

### 4) Commit Changes

After staging files, perform commits with a message:

```bash
git commit -m "your commit message"
```

---

### 5) View Commit History

To see all the commits in the working directory:

```bash
git log
```

> [!TIP]
> **Clean log view:**
> To see a compact, single-line version of your commit history:
>
> ```bash
> git log --oneline
> ```

---

### 6) Check Code Author (Blame)

To see which changes were made by which developer at what time, run:

```bash
git blame <filename>
```

---

### 7) Best Practices

> [!NOTE]
> It is good practice to commit each functionality individually rather than writing the entire code and pushing it all at once.

---

### 8) Undoing Changes (Moving to Previous Commit)

If our current code is buggy and we want to revert/move to a previous commit, we have two ways:

#### Way A: Hard Resetting (Destructive)

Resetting the current node (which is `HEAD`) and pointing it towards a previous commit ID:

```bash
git reset --hard <commit_id>
```

> [!WARNING]
> This method will permanently lose all commits after the target commit ID. Use with caution!

#### Way B: Reverting (Safe)

Reverting the changes:

```bash
git revert <commit_id>
```

This command reverses the changes in the desired file (removes whatever was added and adds whatever was removed) by creating a new commit.

---

---

# 🔗 Adding GitHub to VS Code (Connecting Remote)

### 1) Link Local Machine to GitHub

To link your local machine to GitHub, you need a link to a remote GitHub repository:

```bash
git remote add origin <github_repo_link>
```

_Note: You can name the remote repository whatever you like, but by convention, we generally name it `origin`._

To verify the added repository:

```bash
git remote -v
```

### 2) Push to Remote Branch

To push your local branch to GitHub and link them together:

- **Command Option 1:**
  ```bash
  git push --set-upstream origin main
  ```
- **Command Option 2:**
  ```bash
  git push -u origin main
  ```
  _Both commands are identical and establish a tracking link between local and remote._

---

---

# 💡 Handling Unrelated Histories (Important)

Suppose we have an existing repository and we create another file and want to push it to the same repository from a different machine. We might get an error like **"Repository contains some work, changes rejected"**.

To resolve this, pull and merge the unrelated histories:

```bash
git pull origin main --allow-unrelated-histories
```

This command downloads the content present in the Git repository, combines it with the local repository, and then allows you to push it back to GitHub.

---

---

# 🌿 Branching

### 1) Create a New Branch

```bash
git branch <branch_name>
```

### 2) Rename the Current Branch

```bash
git branch -M <new_branch_name>
```

### 3) List Available Branches

```bash
git branch
```

### 4) Switch Branches

```bash
git checkout <branch_name>
# Or the modern syntax:
git switch <branch_name>
```

### 5) Push a New Branch to Remote

If you push a new branch's code, it will generate an error indicating the branch exists on the local machine but not on the remote (GitHub). To solve this, push and set the upstream branch:

```bash
git push -u origin <branch_name>
```

_(Note: The original notes stated `git push -u origin main <branch name>`, but the standard way is `git push -u origin <branch_name>`.)_

---

---

# 🔀 Merging Branches

To merge a branch, we have two main methods:

### Method A: Command Line Method

Merge a remote branch directly:

```bash
git merge origin/<branch_name>
```

### Method B: Pull Requests (Recommended)

Raise a **Pull Request (PR)** on GitHub. This is the most common method in professional development.

- The pull request is merged directly on the GitHub server.
- To sync these changes back to your local machine, run:
  ```bash
  git pull
  ```

### Branch Naming Convention

Generally, write branch names in this format:

- `bug/<name_of_bug>`
- `feat/<name_of_feature>`

---

---

# 📦 Stashing (Saving Work Temporarily)

### Scenario:

Imagine you wrote some code and have uncommitted changes in your local repository. Another developer updated the remote repository, and you need to pull their changes. Running `git pull` normally will fail because of your local, uncommitted changes.

In this case, use **Stash** to temporarily put your changes on hold, pull the updates, and restore your changes.

1. **Stash your work:**
   ```bash
   git stash
   ```
2. **Pull remote changes:**
   ```bash
   git pull
   ```
3. **Bring your changes back:**
   ```bash
   git stash apply
   # Or to apply and remove it from stash history:
   git stash pop
   ```

---

---

# 🤝 How to Contribute (Open Source Workflow)

### Step 1: Fork the Repository

First, fork the repository in which you want to contribute.
_Forking creates a copy of the repository under your own GitHub account._

### Step 2: Set Up Local Environment & Make Changes

1. Clone your forked repository:
   ```bash
   git clone <https_address_of_your_forked_repo>
   ```
2. Create a new branch:
   ```bash
   git branch <branch_name>
   ```
3. Switch to your branch:
   ```bash
   git checkout <branch_name>
   ```
4. **Make your changes** in the code editor.
5. Stage the files:
   ```bash
   git add .
   ```
6. Commit your changes:
   ```bash
   git commit -m "detailed commit message"
   ```
7. Push the changes to your forked repository:
   ```bash
   git push origin <branch_name>
   ```
   _(If remote is not added, run `git remote add origin <https_link_of_your_forked_repo>` first)_

### Step 3: Raise a Pull Request (PR)

In your remote forked repository page on GitHub, click **Compare & pull request** to submit your changes to the original project.

### Step 4: Branch Naming Conventions

Follow standardized branch names:

- `feat-<name_of_feat>` or `feat/<name_of_feat>`
- `bug-<name_of_bug>` or `bug/<name_of_bug>`
- `wip-<name_of_work>` (WIP stands for **Work In Progress**)
  - _Example:_ `feat/dark-mode`

### Step 5: Syncing with the Original Repository (Upstream)

Because your fork is a copy, it does not automatically update when the original repository gets new changes. To sync your fork with the original project:

1. Add the original repository as an `upstream` remote:
   ```bash
   git remote add upstream <https_link_of_original_repo>
   ```
2. Pull latest changes from the upstream main/master branch:
   ```bash
   git pull upstream main
   ```
3. Push those updates to your forked repository on GitHub:
   ```bash
   git push origin main
   ```
   Your repository is now successfully synced!

---

---

# ➕ Essential Extra Git Topics

### A. Checking Repository Status

To see which files are staged, unstaged, or untracked:

```bash
git status
```

---

### B. Discarding Local Changes (Restore)

If you made changes to a file but want to discard them and revert back to the last committed version:

```bash
git restore <filename>
```

---

### C. Viewing Differences (Diff)

To see exact line changes that have not yet been staged:

```bash
git diff
```

To see differences in staged files:

```bash
git diff --staged
```

---

### D. Setup Git Configuration

Configure your git username and email globally (needed for commit author information):

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

---

### E. Handling Merge Conflicts

When two branches modify the same line of the same file, Git cannot decide which version to keep. This causes a **Merge Conflict**.

#### How to Resolve:

1. Open the conflicted file in VS Code.
2. You will see markers:
   ```text
   <<<<<<< HEAD
   Your local changes
   =======
   Changes from the merged branch
   >>>>>>> branch-name
   ```
3. Choose to **Accept Current Change**, **Accept Incoming Change**, or **Keep Both**.
4. Save the file, stage it (`git add <filename>`), and commit it (`git commit -m "Resolved merge conflict"`).

### Important Notes

let say you are logged in github account A and want to push code on github B . It will generate an error saying that you don't have permissions to push in this repo

To solve this we logout from our old account and login using new account and after some code verification , you are all set

Commands to do so =>

gh auth status (for checking logged in user)

gh auth logout (for logging out )

gh auth login (for logging in )

=> copy the code for vscode and paste it in the respected browser
=> after it do email verification and you are all set

.....

### Important Notes

=> Let's say you staged(means just use git add command and not commit the changes) an unwanted file/folder (like .env or node modules) in the stagging area.
=> To restore / revert the changes , use command =>

git restore --stagged <file/folder name> or

git reset <file/folder name>

=> this command restore changes

> if you commit the changes , then above command is not going to work , you should use following commands

(i) git rm --cached <file/folder name> or

(ii) git revert <commit id> ( it generate another commit )

> if you want that your new push should merged with previous commit and don't create a new commit , we use => git ammend command

git commit --ammend -m "<new message>"

....

> > > > > > > > > > > > > > > > > > > > [IMPORTANT] <<<<<<<<<<<<<<<<<

#git pull vs git fetch

> always try to use git fetch command because it download the changes from the remote but , don't merge with your existing repo , and after using git status , you can see the changes , and decide wether to merge that changes or not .
> on the other hand git pull overwrite your code in the local machine

...

# Skipping the Staging area

> To skip the staging area and directly push the code to remote , we use the command =>

git commit <filename> -m "<message>" >single file

git commit -a -m "<message>" > all files
