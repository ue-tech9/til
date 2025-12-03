# [Tips] Rename Jupyter Kernel Display Name via CLI

Created: 2025-12-03
Tags: #jupyter #bash #linux #config

## English Summary
This note explains how to programmatically change the display name of a Jupyter Kernel.
The display name is defined in the `kernel.json` file. By modifying the `"display_name"` field using `sed`, you can customize how the kernel appears in the JupyterLab launcher.

## Japanese Summary
JupyterLabのランチャー画面に表示されるカーネル名（Display Name）を、画面操作ではなくコマンドライン（スクリプト）から変更する方法のメモ。
実体は `kernel.json` という設定ファイルにあるため、ここを `sed` コマンド等で書き換えることで変更可能です。

---

## Configuration / Code

### 1. Target File (編集対象)

Jupyterのカーネル設定は、通常以下のディレクトリ配下にフォルダごとに格納されています。
その中の `kernel.json` が編集対象です。

* **Path:** `/opt/conda/share/jupyter/kernels/<kernel-name>/kernel.json`
    * *※環境により `/usr/local/share/jupyter/...` の場合もあります。*

**Before (Original `kernel.json`):**
```json
{
 "argv": [
  "python",
  "-m",
  "ipykernel_launcher",
  "-f",
  "{connection_file}"
 ],
 "display_name": "Python 3",
 "language": "python"
}
````

### 2\. Command to Rename (変更コマンド)

`sed` コマンドを使用して、`"display_name"` の行を直接置換します。
JSON形式を崩さないよう、ダブルクォーテーションのエスケープ処理に注意が必要です。

```bash
#!/bin/bash

# ターゲットとなるカーネル設定ファイルのパス
KERNEL_JSON="/opt/conda/share/jupyter/kernels/python3/kernel.json"

# display_name を "xxxxxxxxxxxxx" に書き換える
if [ -f "$KERNEL_JSON" ]; then
    sed -i 's/"display_name":.*/"display_name": "xxxxxxxxxxxxx",/' "$KERNEL_JSON"
    echo "Updated kernel display name in $KERNEL_JSON"
fi
```

### 3\. Result (変更結果)

コマンド実行後、ファイルの中身は以下のように書き換わります。
JupyterLabをリロード（または再起動）すると、ランチャー上の表示が `xxxxxxxxxxxxx` に変化します。

**After:**

```json
{
 "argv": [
  ...
 ],
 "display_name": "xxxxxxxxxxxxx",
 "language": "python"
}
```

