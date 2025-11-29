---
title: "ZennとGitHub連携のテスト"
emoji: "🕌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zenn", "github", "test"]
published: false
---

## これは何？
これは **GitHub連携機能** を使ってデプロイされた記事です。
ローカルのテキストエディタ（VS Code）で執筆し、`git push` だけで反映されるかテストしています。

### Zenn特有の記法のテスト
Zennには便利なメッセージ機能があります。

:::message
ここに補足情報を書きます。
これはZenn特有の記法なので、Qiita等に転送する際は変換が必要になるポイントです。
:::

:::message alert
これは警告メッセージです。
:::

### コードブロックのテスト
シンタックスハイライトが効くかのテストです。

```python
def hello_zenn():
    print("Hello, Zenn from GitHub!")

if __name__ == "__main__":
    hello_zenn()