# [Shell] Complete Offline Asset Downloader & Verifier (Docker)

Created: 2025-11-30
Tags: \#docker \#shell \#offline \#tips

## English Summary

A workflow to download and verify software assets (pip, apt, npm) for **Ubuntu offline environments**, designed to run on **Amazon Linux 2**.
By using an **Ubuntu container** on the Amazon Linux 2 host, you can fetch compatible Ubuntu assets (such as `.deb` files) correctly, overcoming host OS differences.

## Japanese Summary

**Amazon Linux 2 (Host)** 環境で Docker を使い、**Ubuntu 環境用**のライブラリ資材を収集・検証する自動化ツールセットです。
作業端末が Amazon Linux 2 であっても、**Dockerコンテナ内で Ubuntu を起動してダウンロードを行う**ため、OSの違いを気にせず、確実に Ubuntu 用の資材（debパッケージ等）を集めることができます。

-----

## 1\. Downloader Script (`bundle_assets.sh`)

まずは資材を集めるスクリプトです。
以下のブロックをターミナルに貼り付けると、`bundle_assets.sh` が生成されます。

> **Update:** `python3-pip` 自体もダウンロードリストに含めることで、pipが入っていない環境でもブートストラップ可能にしました。

```bash
cat << 'EOF' > bundle_assets.sh
#!/bin/bash
set -e

# ========================================================
# Settings (環境に合わせて変更してください)
# ========================================================
BASE_IMAGE="ubuntu:22.04"
ASSETS_DIR="assets"

# 1. Python Packages (pip)
PIP_PACKAGES="
requests
pandas
"

# 2. OS Packages (apt/deb)
#    ※ python3-pip は必須（インストール用コマンドのため）
DEB_PACKAGES="
curl
git
nodejs
npm
python3-pip
"

# 3. Node.js Packages (npm)
NPM_PACKAGE_SPEC="express"
NPM_BUNDLE_TAR="npm-bundle.tar.gz"

# ========================================================
# Main Process
# ========================================================
echo "=== Start Downloading to '${ASSETS_DIR}' ==="

# Clean up (古い資材は削除)
rm -rf "${ASSETS_DIR}"
mkdir -p "${ASSETS_DIR}/pip" "${ASSETS_DIR}/deb" "${ASSETS_DIR}/npm"

# --------------------------------------------------------
# Task 1: Python (pip)
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
# Task 2: OS Libraries (apt)
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
# Task 3: Node.js (npm)
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
    
    # チェック
    if ! command -v npm >/dev/null; then echo 'Error: npm not found.'; exit 1; fi && \

    # 実行
    mkdir -p /tmp/build && cd /tmp/build && \
    npm install $(echo ${NPM_PACKAGE_SPEC}) && \
    tar czf \"/output/${NPM_BUNDLE_TAR}\" node_modules
  "

# --------------------------------------------------------
# Cleanup permissions
# --------------------------------------------------------
echo "--- 4. Cleanup permissions ---"
# rootで作成されたファイルの権限を自分(ホストユーザー)に戻す
docker run --rm --user root \
  -v "$(pwd)/${ASSETS_DIR}:/output" \
  "${BASE_IMAGE}" \
  chown -R $(id -u):$(id -g) /output

echo "=== Done! Assets ready in '${ASSETS_DIR}/' ==="
EOF
```

-----

## 2\. Verifier Script (`verify_install.sh`)

「集めた資材で本当にインストールできるか？」を検証するスクリプトです。
`--network none` を付与した完全オフラインコンテナでテストを実行します。

```bash
cat << 'EOF' > verify_install.sh
#!/bin/bash
set -e

# ========================================================
# Settings
# ========================================================
BASE_IMAGE="ubuntu:22.04"
ASSETS_DIR="$(pwd)/assets"
PIP_TARGETS="requests pandas" # テストしたいパッケージ

# ========================================================
# Main Process
# ========================================================
echo "=== 1. Checking Assets Directory ==="
if [ ! -d "$ASSETS_DIR" ]; then
  echo "Error: 'assets' directory not found at $ASSETS_DIR"
  exit 1
fi

echo "=== 2. Starting Offline Container (Network: None) ==="
# 解説:
#   --network none : インターネット接続を完全に遮断
#   -v ...:/input  : ホストの assets フォルダをコンテナ内の /input にマウント

docker run --rm \
  --network none \
  -v "${ASSETS_DIR}:/input" \
  "${BASE_IMAGE}" \
  bash -c "
    set -e
    echo '--- Inside Container (Offline) ---'

    # 1. OS Packages (apt)
    echo '[APT] Installing local .deb files...'
    # 依存関係解決のため *.deb をまとめて指定する
    apt-get install -y /input/deb/*.deb >/dev/null

    echo 'Checking commands:'
    which curl git npm python3 pip3
    echo 'OK.'

    # 2. Python Packages (pip)
    echo '[PIP] Installing python libraries...'
    # --no-index: ネットを見に行かない
    # --find-links: ローカルフォルダを探す
    python3 -m pip install --no-index --find-links=/input/pip ${PIP_TARGETS} >/dev/null
    
    echo 'Checking python imports:'
    python3 -c 'import requests; import pandas; print(\"Success: libs imported\")'

    # 3. Node.js Packages (npm)
    echo '[NPM] Restoring node_modules...'
    mkdir -p /app && cd /app
    if [ -f /input/npm/npm-bundle.tar.gz ]; then
      tar xzf /input/npm/npm-bundle.tar.gz
      if [ -d node_modules ]; then echo 'Success: node_modules exists.'; fi
    fi

    echo '--- All Tests Passed! ---'
"

echo "=== 3. Verification Finished Successfully ==="
EOF
```

-----

## Usage (使い方)

### Step 1. Generate & Run Downloader

上記の `bundle_assets.sh` 生成コードブロックをターミナルに貼り付けます。
その後、スクリプトを実行して資材をダウンロードします。

```bash
chmod +x bundle_assets.sh
./bundle_assets.sh
# -> 'assets' フォルダが作成されます
```

### Step 2. Generate & Run Verifier

上記の `verify_install.sh` 生成コードブロックをターミナルに貼り付けます。
その後、検証を実行します。

```bash
chmod +x verify_install.sh
./verify_install.sh
# -> '=== 3. Verification Finished Successfully ===' と出れば成功
```

### Step 3. Production Install (現場での手順)

現場のサーバーに `assets` フォルダを持ち込み、以下のコマンドでインストールします。

```bash
# 1. OS Packages
apt-get install -y ./assets/deb/*.deb

# 2. Python Packages
# (aptで python3-pip が入った後に実行)
python3 -m pip install --no-index --find-links=./assets/pip requests pandas

# 3. Node.js
tar xzf ./assets/npm/npm-bundle.tar.gz
```

-----

## Notes

  * **OS Version:** `BASE_IMAGE` は持ち込み先と完全に一致させてください（例: `ubuntu:20.04` or `22.04`）。
  * **Pip Command:** 初期状態のOSには `pip` コマンドがないことが多いため、`bundle_assets.sh` 内で `python3-pip` をダウンロードリストに含めています。
  * **Permission:** 生成された `assets` フォルダが削除できない場合は、`sudo rm -rf assets` を使ってください。

