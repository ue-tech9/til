# Document Standard GitHub Workflow (Issue to Merge)

Created: 2025-11-26
Tags: #git #github #workflow #beginner

## English Summary
This document outlines the standard workflow for solo development on GitHub.
It covers the cycle from creating an Issue, branching locally, committing changes, opening a Pull Request with keyword linking, merging, and finally cleaning up the local environment.

## Japanese Details
GitHubにおける標準的な作業フロー（Golden Path）の手順書です。
Issue作成から、ブランチ作成、PR、マージ、そしてローカル環境のお掃除までの一連の流れを日英併記でまとめました。

---

### Workflow Steps

#### 1. Create an Issue on GitHub (Start)
Before starting work, define the task on GitHub.
作業を始める前に、GitHub上でタスクを定義します。

* **Action:**
    * Go to the **"Issues"** tab -> Click **"New issue"**.
    * Enter a Title (e.g., `[Draft] Add new document`).
    * **Note:** Remember the **Issue Number (e.g., #10)** displayed after creation.
    * (「Issues」タブから「New issue」をクリック。タイトルを入力し、作成後に表示されるIssue番号を控える)

#### 2. Create a Local Working Branch (Branch)
Update your local environment and create a specific room (branch) for your work.
PC上のデータを最新にしてから、作業専用の部屋（ブランチ）を作ります。

* **Command:**
    ```bash
    # 1. Switch to the main branch
    # まずメインの部屋(main)に戻る
    git checkout main

    # 2. Pull the latest changes from GitHub (Crucial to avoid conflicts)
    # GitHubにある最新の状態をPCに取り込む (競合を防ぐために必須)
    git pull origin main

    # 3. Create and switch to a new branch
    # 新しい作業ブランチを作成して移動する
    # Naming: category/description (e.g., docs/add-guide)
    git checkout -b <branch-name>

    #sample
    git checkout -b docs/standard-workflow
    ```

#### 3. Edit and Commit (Commit)
Edit files and save your changes with a message.
ファイルを作成・編集し、その変更をメッセージ付きで保存（コミット）します。

* **Command:**
    ```bash
    # Stage all changes
    # 変更されたファイルをすべて「保存予定」状態にする
    git add .

    # Commit with a message (Format: "prefix: description")
    # メッセージ付きで保存する
    git commit -m "<commit-message>"

    #sample
    git commit -m "docs: add github workflow guide"
    ```

#### 4. Push to GitHub (Push)
Upload your local branch to GitHub.
PC上の作業ブランチを、そのままGitHub（インターネット上）へアップロードします。

* **Command:**
    ```bash
    # Push the current branch to origin (GitHub)
    # HEAD is a shortcut for "Current Branch".
    # HEAD は「今いるブランチ」のこと。同名でアップロードされます。
    git push -u origin HEAD
    ```

#### 5. Create a Pull Request (PR)
Send a request on GitHub to merge your work.
GitHub上で「作業が終わったので取り込んでください」という依頼（PR）を出します。

* **Action:**
    * Go to the repository page. Click the green **"Compare & pull request"** button.
    * (リポジトリページに行き、緑色の「Compare & pull request」ボタンを押す)
* **Description (Magic Word):**
    * Write the following to auto-close the issue upon merging.
    * 本文に以下の「魔法の言葉」を記述します。マージ時にIssueが自動で完了します。
    ```text
    Closes #10
    ```
    *(Replace #10 with your actual Issue number / 手順1の番号に書き換える)*

#### 6. Self-Review and Merge (Merge)
Review your own code and approve the merge.
自分で自分の作業内容を確認し、承認（マージ）します。

* **Action:**
    1.  Check the **"Files changed"** tab for any errors or secrets.
        (「Files changed」タブで、エラーや機密情報がないか確認する)
    2.  Click **"Merge pull request"** -> **"Confirm merge"**.
    3.  Click **"Delete branch"** to remove the remote branch.
        (マージ完了後、「Delete branch」ボタンでGitHub上のブランチを削除する)

#### 7. Cleanup Local Environment (Cleanup)
Clean up your local machine to get ready for the next task.
PC上の環境も綺麗にして、次の作業に備えます。

* **Command:**
    ```bash
    # 1. Switch back to main
    # メインブランチに戻る
    git checkout main

    # 2. Pull the latest merged changes
    # マージされた最新の状態(自分の作業結果)を取り込む
    git pull origin main

    # 3. Delete the old local branch
    # PC上の古い作業ブランチを削除する
    git branch -d <branch-name>

    #sample
    git branch -d docs/standard-workflow
    ```

---

## Configuration / Code (Cheat Sheet)

### Daily Command Flow

```bash
# 1. Start (Update main)
git checkout main
git pull origin main

# 2. Branch (Create & Switch)
git checkout -b <branch-name>

# 3. Work & Commit
git add .
git commit -m "<commit-message>"

# 4. Push (Upload)
git push -u origin HEAD
```

```bash
# Start new work
git checkout main && git pull origin main
git checkout -b docs/add-workflow-guide

# Work & save
git status                              # 何が変わったか確認
git add .                               # ステージング
git commit -m "docs: add standard github workflow guide"   # コミット
git push -u origin HEAD                 # 初回プッシュ

# After PR merged
git checkout main && git pull origin main
git branch -d docs/add-workflow-guide

```

### Naming Conventions (命名規則サンプル)

#### Branch Names (ブランチ名)
* `docs/add-new-guide` (New Document / 新規作成)
* `docs/update-readme` (Update / 更新)
* `fix/typo-correction` (Fix / 修正)
* `feat/add-script` (Feature / 機能追加)
* `chore/organize-folders` (Chore / 雑務)

#### Commit Messages (コミット名)
* `docs: add standard workflow guide` (ドキュメント追加)
* `docs: update README link` (ドキュメント更新)
* `fix: correct typo in okta setup` (修正)
* `feat: add setup script` (機能追加)
* `chore: clean up unused files` (雑務)
