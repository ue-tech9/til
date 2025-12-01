# Docker Command Cheat Sheet for Infra Ops

Created: 2025-12-02
Tags: #docker #cheatsheet #infra #linux

## English Summary
A comprehensive reference for Docker commands in infrastructure operations.
Covers image management, offline image transfer (save/load), debugging running containers, and resource cleanup.

## Japanese Summary
インフラ作業や検証で頻出するDockerコマンドの自分用メモ。
基本的なライフサイクルに加え、閉域網への持ち込みに必須な「イメージのtar化(save/load)」、稼働中コンテナへのログイン、およびファイルコピー手順を網羅する。

---

## Configuration / Code

### 1. Image Management (イメージ操作)
コンテナの元となる「イメージ」の管理。

```bash
# ローカルにあるイメージ一覧を表示
docker images

# イメージの検索 (Docker Hub)
docker search <keyword>

# イメージのダウンロード (タグ指定推奨)
docker pull ubuntu:22.04

# イメージの削除
# ※コンテナが使っている場合は削除できないので、先にコンテナを消すこと
docker rmi <image_id_or_name>

# ぶら下がりイメージ（タグなし: <none>）を一括削除
docker image prune
````

### 2\. Offline Transfer (オフライン持ち込み)

**重要:** インターネットに繋がらない環境へイメージを持ち込む手順。

```bash
# イメージをtarファイルに書き出す (Save)
# 形式: docker save -o <出力ファイル名.tar> <イメージ名>
docker save -o my-app-image.tar nginx:latest

# --- (USBメモリやSFTPで閉域網へ転送) ---

# tarファイルからイメージを読み込む (Load)
# 形式: docker load -i <入力ファイル名.tar>
docker load -i my-app-image.tar
```

### 3\. Container Lifecycle (コンテナ基本操作)

```bash
# 起動中のコンテナ一覧
docker ps

# 停止中も含めて全てのコンテナ確認
# ステータス(Up/Exited)を確認するのによく使う
docker ps -a

# コンテナの起動（バックグラウンド実行）
# -d: Detach (バックグラウンド)
# --name: 名前をつける（管理しやすくなる）
docker run -d --name my-web-server -p 8080:80 nginx

# コンテナの停止・開始・再起動
docker stop <container_name>
docker start <container_name>
docker restart <container_name>

# コンテナの削除
docker rm <container_name>
```

### 4\. Debugging & Interaction (調査・侵入)

トラブルシュート時に「中に入って確認する」ためのコマンド。

```bash
# 【頻出】稼働中のコンテナ内で新しいコマンドを実行する（シェルに入る）
# コンテナ内で ls や cat したい時はこれ。
# alpine系なら /bin/bash ではなく /bin/sh
docker exec -it <container_name> /bin/bash

# ログの確認 (リアルタイム追跡)
docker logs -f <container_name>

# ホスト ⇔ コンテナ間のファイルコピー
# 設定ファイルを修正して差し替えたい時などに便利
# ホストからコンテナへ
docker cp ./local-config.conf <container_name>:/etc/app/config.conf
# コンテナからホストへ
docker cp <container_name>:/var/log/app.log ./debug_logs/

# リソース使用状況の確認 (topコマンド的なやつ)
docker stats
```

### 5\. The "Throwaway" Pattern (使い捨て実行)

環境を汚さずにコマンドだけ実行したい場合。

```bash
# 実行完了後にコンテナを自動削除 (--rm)
# -v: カレントディレクトリをマウントして作業
docker run --rm -it \
  -v $(pwd):/work \
  -w /work \
  python:3.9 \
  python script.py
```

### 6\. Cleanup (掃除)

```bash
# 使用していない全てのオブジェクト（コンテナ、NW、イメージ）を一括削除
docker system prune
```


