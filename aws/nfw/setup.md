# [AWS] Network Firewall Setup (Rule Group & Policy)

Created: 2025-12-14
Tags: #aws #network #firewall #security

## English Summary
This document outlines the procedure for creating **Rule Groups** and **Firewall Policies** in AWS Network Firewall.
It focuses on a **Stateful Domain List** configuration with **No TLS Inspection** (SNI-based control) to implement a whitelist strategy allowing only Identity Center and Okta traffic.

## Japanese Summary
本手順書は、**AWS Network Firewall において「ルールグループ」と「ファイアウォールポリシー」を作成するまで**の作業を、設計意図と判断理由つきでまとめたものです。
ステートフルドメインリスト方式（SNIベース制御・TLS復号なし）を採用し、Identity Center / Okta のみを許可するホワイトリスト運用の構成を対象としています。

対象構成は以下です。

* ステートフル ドメインリスト方式
* TLS 復号なし（SNI ベース制御）
* ホワイトリスト運用（Identity Center / Okta のみ許可）

---

## 0. 前提条件・設計方針

### 前提

* AWS マネジメントコンソールにログイン可能
* VPC は既に存在（Firewall 本体・ルーティングは本手順の対象外）

### 設計方針

* **許可したドメイン以外はすべて遮断**
* DNS ベースではなく **SNI ベース制御** を利用
* 運用コストと複雑性を最小化

---

## 1. ルールグループの作成

### 1.1 ルールグループ作成画面へ

VPC → Network Firewall → ルールグループ → ルールグループを作成

---

### 1.2 ルールグループタイプの選択

* ルールグループタイプ：**ステートフル**
* ルールグループ形式：**ドメインリスト**

**理由**：

* HTTPS の ClientHello に含まれる SNI を評価できる
* TLS 復号が不要

---

### 1.3 ルールグループの基本情報

* 名前例：`allow-idc-okta-egress`
* 説明：任意（例：Allow Identity Center and Okta egress only）
* キャパシティ：`30000`

**補足**：

* ドメインリスト方式では 30000 を指定して問題なし

---

### 1.4 ドメインリストルールの設定

#### ドメイン名（1行1ドメイン）

```
amazon.com
amazonaws.com
aws.amazon.com
awsapps.com
cloudfront.net
```

**注意事項**

* 末尾のドット（`.`）は不要
* ワイルドカード `*.` は使用不可（自動的にサブドメインを含む）

#### プロトコル

* HTTP
* HTTPS

#### アクション

* **許可（Allow）**

---

### 1.5 暗号化（KMS）

* カスタマーマネージドキー：**未設定（AWS 管理キー）**

**理由**：

* 通信制御に影響なし
* PoC / 通常運用では不要

---

### 1.6 作成

設定を確認し、ルールグループを作成する。

---

## 2. ファイアウォールポリシーの作成

### 2.1 作成画面へ

VPC → Network Firewall → ファイアウォールポリシー → 作成

---

### 2.2 ファイアウォールポリシーの基本情報

* 名前例：`idc-okta-egress-policy`
* 説明：任意

---

### 2.3 ストリーム例外ポリシー

* 設定：**拒否（Reject）**

**理由**：

* TCP Reset を返すためクライアントが即時失敗を検知できる
* タイムアウト待ちが発生しない
* トラブルシューティングが容易

---

### 2.4 ルールグループの追加

#### ステートレスルールグループ

* **追加しない**

#### ステートフルルールグループ

* 追加：`allow-idc-okta-egress`
* 優先度：`1`

---

### 2.5 ステートレスデフォルトアクション

* 完全なパケット：**ステートフルルールグループに転送**
* フラグメントされたパケット：**ステートフルルールグループに転送**

---

### 2.6 ステートフルルールのデフォルトアクション

* アプリケーションアラート：有効
* アプリケーションドロップ：有効

**意味**：

* ドメインに一致しない通信は記録され、拒否される

---

### 2.7 詳細オプション

以下は **すべてデフォルトのまま** とする。

* カスタマーマネージドキー：未設定
* ポリシー変数（HOME_NET）：未変更
* TCP アイドルタイムアウト：未変更

---

### 2.8 TLS 検査設定

* **設定しない**

**理由**：

* SNI ベース制御が目的
* TLS 復号は不要かつ運用負荷が高いため

---

### 2.9 タグ

* 任意（PoC・検証では省略可）

---

### 2.10 作成

設定内容を確認し、ファイアウォールポリシーを作成する。

---

## 3. 作成後の状態（期待値）

* 許可されたドメインのみ HTTP / HTTPS 通信が成立
* IP 直打ち通信は拒否
* DoH サーバーへの通信も拒否
* ログおよびアラートで未許可通信を可視化可能

---

## 4. 次のステップ（本手順外）

* Network Firewall 本体の作成
* Firewall Subnet への配置
* ルートテーブルの変更（NAT Gateway / IGW 経由）
* 実通信テスト（IdC / Okta ログイン確認）

---


