---
title: "ZennとGitHub連携のテスト"
emoji: "🕌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zenn", "github", "test"]
published: false
---

# 1. 念のため最新の状態にする
git checkout main
git pull origin main

git add .
git commit -m "docs: add new article"

# git push origin main


# 1. 今の状態（コミット済み）のまま、ブランチを作成して移動する
git checkout -b feature/new-article-name

# 2. 改めてプッシュする
git push origin feature/new-article-name

# 1. メインに戻って、マージされた最新状態を取り込む
git checkout main
git pull origin main

# 2. ローカルの作業ブランチを削除
git branch -d feature/new-article-name

# 3. リモートの作業ブランチも削除（もしGitHub上で消し忘れていたら）
git push origin --delete feature/new-article-name


...