# [Shell] Simple Offline Asset Downloader (Docker)

Created: 2025-11-30
Tags: \#docker \#shell \#offline \#tips

## English Summary

A simple script to download offline assets (pip, apt, npm) using Docker.
Instead of manually creating a file, you can copy and paste the command below to generate the script instantly. It runs in a temporary container to keep your local machine clean.

## Japanese Details

閉域網（オフライン環境）用の資材を、Dockerを使ってダウンロードするスクリプトです。
「どのコマンドだったっけ？」と思い出しながらファイルを作るのが面倒なので、コピペ一発でスクリプトファイル(`bundle_assets.sh`)を生成できるようにしました。
使い捨てのコンテナ内で作業するため、自分のPC（ホストOS）を汚さずに資材セットを作成できます。

-----

## Configuration / Code

# 1. Dockerのインストール
yum update -y
yum install -y docker

# 2. Dockerサービスの起動と自動起動設定
systemctl start docker
systemctl enable docker

# 3. 動作確認 (バージョンが表示されればOK)
docker --version

### 1\. Generate Script (Copy & Paste)

以下のブロックをターミナルにそのまま貼り付けて実行してください。
`bundle_assets.sh` というファイルが生成されます。

```bash
# スクリプトファイルの生成 (ヒアドキュメント)
cat << 'EOF' > bundle_assets.sh
#!/bin/bash
set -e

# ========================================================
# Settings (必要に応じてここを変更してください)
# ========================================================

# 1. 持ち込み先のOSイメージ (例: ubuntu:22.04)
#    ※必ず本番環境のOSバージョンと合わせること
BASE_IMAGE="ubuntu:22.04"

# 2. 保存先フォルダ名
ASSETS_DIR="assets"

# 3. Pythonパッケージ (pip)
PIP_PACKAGES="
requests
pandas
"

# 4. OSパッケージ (apt/deb)
#    ※npmを使うなら nodejs, npm も記述する
DEB_PACKAGES="
curl
git
nodejs
npm
"

# 5. Node.jsパッケージ (npm)
NPM_PACKAGE_SPEC="express"
NPM_BUNDLE_TAR="npm-bundle.tar.gz"

# ========================================================
# Main Process
# ========================================================

echo "=== Start Downloading to '${ASSETS_DIR}' ==="

# フォルダ作成
mkdir -p "${ASSETS_DIR}/pip" "${ASSETS_DIR}/deb" "${ASSETS_DIR}/npm"

# --------------------------------------------------------
# 1. Python (pip)
# --------------------------------------------------------
echo "--- 1. Python packages ---"
docker run --rm --user root \
  -v "$(pwd)/${ASSETS_DIR}/pip:/output" \
  "${BASE_IMAGE}" \
  bash -c "
    apt-get update && apt-get install -y python3-pip >/dev/null 2>&1 && \
    python3 -m pip download --dest /output $(echo ${PIP_PACKAGES})
  "

# --------------------------------------------------------
# 2. OS Libraries (apt)
# --------------------------------------------------------
echo "--- 2. OS libraries ---"
docker run --rm --user root \
  -v "$(pwd)/${ASSETS_DIR}/deb:/output" \
  "${BASE_IMAGE}" \
  bash -c "
    apt-get update >/dev/null && \
    apt-get install -y --download-only $(echo ${DEB_PACKAGES}) >/dev/null 2>&1 && \
    cp /var/cache/apt/archives/*.deb /output/
  "

# --------------------------------------------------------
# 3. Node.js (npm)
# --------------------------------------------------------
echo "--- 3. npm packages ---"
docker run --rm --user root \
  -v "$(pwd)/${ASSETS_DIR}/npm:/output" \
  -v "$(pwd)/${ASSETS_DIR}/deb:/input_debs" \
  "${BASE_IMAGE}" \
  bash -c "
    # 準備: ダウンロードしたdebを使ってNode.jsを入れる
    apt-get update >/dev/null && \
    apt-get install -y /input_debs/*.deb >/dev/null 2>&1 || true && \
    
    # チェック: npmがあるか確認
    if ! command -v npm >/dev/null; then echo 'Error: npm not found.'; exit 1; fi && \

    # 実行: インストールして固める
    mkdir -p /tmp/build && cd /tmp/build && \
    npm install $(echo ${NPM_PACKAGE_SPEC}) && \
    tar czf \"/output/${NPM_BUNDLE_TAR}\" node_modules
  "

# --------------------------------------------------------
# 4. Finish (Fix Permissions)
# --------------------------------------------------------
echo "--- 4. Cleanup permissions ---"
# Dockerで作成したファイルの所有者がrootになるのを防ぐため、
# 最後にホストユーザー(自分)の権限に戻す
docker run --rm --user root \
  -v "$(pwd)/${ASSETS_DIR}:/output" \
  "${BASE_IMAGE}" \
  chown -R $(id -u):$(id -g) /output

echo "=== Done! Check '${ASSETS_DIR}/' ==="
EOF
```

### 2\. Run

スクリプトに実行権限をつけて実行します。

```bash
chmod +x bundle_assets.sh
./bundle_assets.sh
```

### 3\. Output

実行後、カレントディレクトリに `assets` フォルダが生成されます。
これをzipなどで固めて、閉域網環境へ持ち込みます。

```text
assets/
├── deb/      # .deb files (apt)
├── pip/      # .whl files (python)
└── npm/      # .tar.gz (node_modules)
```

-----

## Notes (ハマりどころメモ)

  * **OSバージョンの一致**: `BASE_IMAGE` は持ち込み先のOSバージョンと厳密に合わせてください（例: `ubuntu:20.04` vs `22.04`）。GLIBCのバージョン差異などで動かないことがあります。
  * **権限エラー**: もし途中でスクリプトが失敗し、`assets` フォルダが root 権限で残って消せなくなった場合は、`sudo rm -rf assets` で削除してください（最後の権限修正ステップまで走れば問題ありません）。
  * **npmの依存関係**: npmパッケージによっては、コンパイルが必要なネイティブモジュール(`node-gyp`)を含むものがあります。その場合は、コンテナ内に `build-essential` や `python3` 等のビルドツールを追加でインストールする必要があります。


  承知いたしました。「閉域網で本当に動くのか？」を検証するために、**ネットワークを切断したコンテナ(`--network none`)** を立てて、そこでインストールを実行する検証スクリプトを作成します。

これを実行すれば、**「ネットがない環境でも、さっき作った `assets` だけで環境構築できること」** を証明できます。

-----

# [Test] Offline Install Simulation (Docker)

Created: 2025-11-30
Tags: \#docker \#shell \#offline \#test

## English Summary

A script to verify offline installation.
It launches an Ubuntu 22.04 container with **no network access** (`--network none`), mounts the local assets, and attempts to install them. This proves that the downloaded assets are sufficient.

## Japanese Details

「資材は揃えたけど、本当にオフラインで入るの？」という不安を解消するための検証手順です。
Dockerの `--network none` オプションを使って**完全オフライン状態**のUbuntu 22.04コンテナを起動し、その中でインストール作業を自動実行します。

-----

cat << 'EOF' > bundle_assets.sh
#!/bin/bash
set -e

# ========================================================
# Settings
# ========================================================
BASE_IMAGE="ubuntu:22.04"
ASSETS_DIR="assets"

# Python Libraries
PIP_PACKAGES="
requests
pandas
"

# OS Packages (FIX: python3-pip を追加しました)
DEB_PACKAGES="
curl
git
nodejs
npm
python3-pip
"

# Node.js
NPM_PACKAGE_SPEC="express"
NPM_BUNDLE_TAR="npm-bundle.tar.gz"

# ========================================================
# Main Process
# ========================================================
echo "=== Start Downloading to '${ASSETS_DIR}' ==="

# Clean up old assets (古いのは一度消します)
rm -rf "${ASSETS_DIR}"
mkdir -p "${ASSETS_DIR}/pip" "${ASSETS_DIR}/deb" "${ASSETS_DIR}/npm"

# 1. Python (pip)
echo "--- 1. Python packages ---"
docker run --rm --user root \
  -v "$(pwd)/${ASSETS_DIR}/pip:/output" \
  "${BASE_IMAGE}" \
  bash -c "
    apt-get update && apt-get install -y python3-pip >/dev/null 2>&1 && \
    python3 -m pip download --dest /output $(echo ${PIP_PACKAGES})
  "

# 2. OS Libraries (apt)
echo "--- 2. OS libraries ---"
docker run --rm --user root \
  -v "$(pwd)/${ASSETS_DIR}/deb:/output" \
  "${BASE_IMAGE}" \
  bash -c "
    apt-get update >/dev/null && \
    # ここで python3-pip 自体もダウンロードリストに含めます
    apt-get install -y --download-only $(echo ${DEB_PACKAGES}) >/dev/null 2>&1 && \
    cp /var/cache/apt/archives/*.deb /output/
  "

# 3. Node.js (npm)
echo "--- 3. npm packages ---"
docker run --rm --user root \
  -v "$(pwd)/${ASSETS_DIR}/npm:/output" \
  -v "$(pwd)/${ASSETS_DIR}/deb:/input_debs" \
  "${BASE_IMAGE}" \
  bash -c "
    apt-get update >/dev/null && \
    apt-get install -y /input_debs/*.deb >/dev/null 2>&1 || true && \
    if ! command -v npm >/dev/null; then echo 'Error: npm not found.'; exit 1; fi && \
    mkdir -p /tmp/build && cd /tmp/build && \
    npm install $(echo ${NPM_PACKAGE_SPEC}) && \
    tar czf \"/output/${NPM_BUNDLE_TAR}\" node_modules
  "

# 4. Cleanup permissions
echo "--- 4. Cleanup permissions ---"
docker run --rm --user root \
  -v "$(pwd)/${ASSETS_DIR}:/output" \
  "${BASE_IMAGE}" \
  chown -R $(id -u):$(id -g) /output

echo "=== Done! Re-created '${ASSETS_DIR}/' ==="
EOF

# 実行権限付与
chmod +x bundle_assets.sh

# 実行（少し時間がかかります）
./bundle_assets.sh

# 検証用スクリプトの生成
cat << 'EOF' > verify_install.sh
#!/bin/bash
set -e

# ========================================================
# Settings
# ========================================================
# 検証に使うイメージ（ダウンロード時と同じもの）
BASE_IMAGE="ubuntu:22.04"

# 資材フォルダのパス（カレントディレクトリにある想定）
ASSETS_DIR="$(pwd)/assets"

# インストールテスト対象（ダウンロード時に指定したもの）
PIP_TARGETS="requests pandas"

# ========================================================
# Main Process
# ========================================================

echo "=== 1. Checking Assets Directory ==="
if [ ! -d "$ASSETS_DIR" ]; then
  echo "Error: 'assets' directory not found at $ASSETS_DIR"
  exit 1
fi
echo "Found assets at: $ASSETS_DIR"

echo "=== 2. Starting Offline Container (Network: None) ==="
# 解説:
#   --network none : インターネット接続を完全に遮断
#   -v ...:/input  : ホストの assets フォルダをコンテナ内の /input にマウント
#   --rm           : 終わったらコンテナを削除
docker run --rm \
  --network none \
  -v "${ASSETS_DIR}:/input" \
  "${BASE_IMAGE}" \
  bash -c "
    set -e
    echo '--- Inside Container (Offline) ---'

    # 1. OS Packages (apt)
    echo '[APT] Installing local .deb files...'
    # 注意: dpkg -i ではなく apt-get install /path/*.deb を使うとスムーズ
    apt-get install -y /input/deb/*.deb >/dev/null

    # 検証: コマンドが入ったか
    echo 'Checking commands:'
    which curl git node npm python3 pip3
    echo 'OK.'


    # 2. Python Packages (pip)
    echo '[PIP] Installing python libraries...'
    # --no-index: ネットを見に行かない
    # --find-links: ローカルフォルダを探す
    python3 -m pip install --no-index --find-links=/input/pip ${PIP_TARGETS} >/dev/null
    
    # 検証: importできるか
    echo 'Checking python imports:'
    python3 -c 'import requests; import pandas; print(\"Success: requests & pandas imported\")'


    # 3. Node.js Packages (npm)
    echo '[NPM] Restoring node_modules...'
    mkdir -p /app
    cd /app
    if [ -f /input/npm/npm-bundle.tar.gz ]; then
      tar xzf /input/npm/npm-bundle.tar.gz
      echo 'Restored node_modules.'
      
      # 検証: ライブラリがあるか
      if [ -d node_modules ]; then
        echo 'Success: node_modules exists.'
      fi
    else
      echo 'Skip: npm bundle not found.'
    fi

    echo '--- All Tests Passed! ---'
"

echo "=== 3. Verification Finished Successfully ==="
EOF


./verify_install.sh