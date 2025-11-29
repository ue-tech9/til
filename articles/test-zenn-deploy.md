# Zenn記事投稿フロー、手癖でミスるのでコピペ用コマンドに定型化した

## はじめに

Zennの記事を書くとき、毎回「CLIの引数ってどうだっけ？」「ブランチ名のルール決めてなかったな」といちいち手が止まるのが無駄すぎる。
脳のリソースを使わずに作業に入れるよう、変数だけ書き換えればそのままターミナルに貼り付けられる形式でメモしておく。

## 状況

**「あれ、さっきのブランチ名なんだっけ？」**

  * 適当な名前でブランチを切ると、後で探すときに苦労する。
  * `npx zenn new:article` のオプションを毎回ヘルプで確認している自分がいる。
  * マージした後、ローカルのブランチを消し忘れてゴミが溜まっていく。

思考停止でコピペ実行できる手順が欲しい。

## 解決策

今後はこの手順以外やらない。
ターミナルで変数（`SLUG`など）を定義してからコマンドを流し込むスタイルにした。

### 1\. 準備・ブランチ作成

記事のスラッグ（URLの一部になるID）さえ決めれば、あとはコピペでOK。

```bash
# ▼▼▼ ここだけ書き換える ▼▼▼
# 記事のスラッグ（例: rails-setup-guide, diary-202310）
SLUG="my-new-article-slug"
# ▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲

# mainを最新化
git checkout main
git pull origin main

# 作業用ブランチを作成（articles/スラッグ名 に統一）
git checkout -b "articles/$SLUG"
```

### 2\. 記事ファイルの生成

Zenn CLIを使ってMarkdownを生成する。
タイトルは後でファイルを開いて修正すればいいので、とりあえず仮で生成する。

```bash
# 記事ファイルを作成
npx zenn new:article --slug "$SLUG" --title "タイトル未定"

# 作成されたファイルを確認（VS Codeで開く場合）
code "articles/$SLUG.md"
```

※ VS Codeを使っていない場合は `code` コマンドの部分を無視して、生成されたファイルを開くこと。

### 3\. 作業・Push・PR作成

記事を書き終わったら、以下のコマンドでPushする。
コミットメッセージも悩みたくないので、最初は定型文でいい。

```bash
# 変更を全てステージング
git add .

# コミット（後でsquashされる前提で、とりあえずのメッセージ）
git commit -m "docs: create/update article $SLUG"

# リモートへPush
git push -u origin HEAD
```

この後、表示されたGitHubのURLからPull Requestを作成する。
**ZennのプレビューURL**がPRのコメントに付くので、そこで最終確認を行う。

### 4\. （マージ後）お掃除

GitHub上でマージが完了したら、ローカルの掃除をする。
これをサボると `git branch` した時に地獄を見る。

```bash
# mainに戻って最新化
git checkout main
git pull origin main

# 済んだ作業ブランチを一括削除（マージ済みのみ削除されるので安全）
# ※変数 $SLUG が残っている前提。ターミナル閉じた場合は直接名前指定で。
git branch -d "articles/$SLUG"
```

-----
