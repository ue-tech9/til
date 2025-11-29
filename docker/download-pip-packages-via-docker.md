
# [Shell] Simple Offline Asset Downloader (Docker)

Created: 2025-11-29
Tags: #docker #shell #offline #tips

## English Summary
A simple script to download offline assets (pip, apt, npm) using Docker.
It runs in a temporary container, keeping your local machine clean and organized.

## Japanese Details
閉域網（オフライン環境）用の資材を、Dockerを使ってダウンロードするスクリプトです。
使い捨てのコンテナ内で作業するため、自分のPCを汚さずに資材セットを作成できます。



---

## The Script (`bundle_assets.sh`)

以下のコードを `bundle_assets.sh` として保存してください。
「設定 (Settings)」を書き換えるだけで動きます。

```bash
#!/bin/bash
set -e

# ========================================================
# Settings (ここを変更してください)
# ========================================================

# 1. 持ち込み先のOSイメージ (例: ubuntu:22.04)
BASE_IMAGE="ubuntu:22.04"

# 2. 保存先フォルダ名
ASSETS_DIR="assets"

# 3. Pythonパッケージ (pip)
PIP_PACKAGES="
requests
pandas
"

# 4. OSパッケージ (apt/deb)
#    ※npmを使うなら nodejs, npm も書いてください
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
# ファイルの持ち主を自分(ホストユーザー)に戻す
docker run --rm --user root \
  -v "$(pwd)/${ASSETS_DIR}:/output" \
  "${BASE_IMAGE}" \
  chown -R $(id -u):$(id -g) /output

echo "=== Done! Check '${ASSETS_DIR}/' ==="
````

## 使い方 (Usage)

1.  **保存**: 上記コードを `bundle_assets.sh` という名前で保存します。
2.  **権限付与**: 実行できるようにします（初回のみ）。
    ```bash
    chmod +x bundle_assets.sh
    ```
3.  **実行**:
    ```bash
    ./bundle_assets.sh
    ```
4.  **確認**: `assets` フォルダができているので、これを閉域網へコピーして使います。

-----

## 注意点 (Note)

  * **OSを合わせる**: `BASE_IMAGE` は、持ち込み先のサーバーと同じOS（Ubuntuのバージョン等）を指定してください。これが違うと動かないことがあります。
  * **権限エラーが出たら**: もし `assets` フォルダが消せなくなった場合は、もう一度スクリプトを実行するか、`sudo rm -rf assets` で削除してください。
