---
title: "ZennとGitHub連携のテスト"
emoji: "🕌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zenn", "github", "test"]
published: false
---

# ZennとGitHub連携のテストで手順が怪しかったので、正解をメモした

## はじめに

Zennの記事をGitHubで管理する設定をしたはいいが、久しぶりすぎて\*\*記事冒頭のフォーマット（Front Matter）\*\*や、**安全なプッシュの手順**を忘れていた。
適当にやってデプロイエラーになるのも、誤ってmainに直接反映させるのも面倒なので、自分用のテンプレートと手順をここに残しておく。

## 状況

**「あれ、ヘッダーの形式どう書くんだっけ？ あとブランチ切る前にコミットしちゃった」**

記事ファイルを新規作成したものの、Zenn特有のメタデータ記述を空で覚えていなかった。
さらに、勢いで作業して `main` ブランチのままコミットしてしまい、「ここからどうやって別ブランチとしてプッシュするか」で一瞬手が止まった。

## 解決策

毎回ドキュメントを探すのが手間なので、以下をコピペして使う。

### 1\. 記事ファイルの必須フォーマット

ファイルの先頭（Front Matter）にはこれを書く必要がある。
テスト中は `published: false` にしておかないと、うっかり公開されるので注意。

```yaml
---
title: "ZennとGitHub連携のテスト"
emoji: "🕌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zenn", "github", "test"]
published: false
---
```

### 2\. 作業〜プッシュまでの手順

「mainで作業してコミットまでしちゃった」という、よくやるパターンのリカバリー手順も含めておく。

```bash
# 1. 念のためmainを最新にする
git checkout main
git pull origin main

# 2. 作業ファイルの保存（うっかりmainでコミットまでしてしまった場合）
git add .
git commit -m "docs: add new article"

# --- ここからリカバリー ---

# 3. 今の状態（コミット済み）を持って、新しいブランチを作成・移動
# これでmainは汚れず、新ブランチにコミットが移動する
git checkout -b feature/new-article-name

# 4. 新しいブランチをリモートへプッシュ
git push origin feature/new-article-name
```

### 3\. マージ後の後始末

プルリクを出してマージされたら、ローカルとリモートのゴミ掃除をしておく。

```bash
# 1. メインに戻って、マージされた最新状態を取り込む
git checkout main
git pull origin main

# 2. ローカルの作業ブランチを削除
git branch -d feature/new-article-name

# 3. リモートの作業ブランチも削除（GitHub上で消し忘れていたら）
git push origin --delete feature/new-article-name
```

## さいごに

Zennのプレビュー機能を使うためにも、まずはこの形式で書いて `git push` する癖をつける。意外と `published: false` を忘れて焦る未来が見える。