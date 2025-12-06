### 全体の流れ
1.  **AWS:** 必要なURL（ACS URLなど）をコピーする
2.  **Okta:** アプリを追加し、AWSのURLを貼り付ける
3.  **Okta:** メタデータ（設定ファイル）をダウンロードする
4.  **AWS:** Oktaのファイルをアップロードして完了


https://dev.classmethod.jp/articles/okta_identity_center_sso/

---

### 手順 1: AWS側の準備 (情報のコピー)

まず、AWSマネジメントコンソールでの操作です。

1.  AWSで **IAM Identity Center** のコンソールを開きます。
2.  左メニューの **[設定]** (Settings) をクリックします。
3.  「アイデンティティソース」というタブにある **[アクション]** ボタンを押し、**[アイデンティティソースを変更]** を選択します。
4.  以下の項目を選びます。
    *   **[外部 ID プロバイダー]** (External identity provider) を選択。
    *   **[次へ]** をクリック。
5.  「サービスプロバイダーのメタデータ」というセクションが表示されます。ここに表示されている以下の2つの情報を、メモ帳などにコピーするか、画面を開いたままにしてください。
    *   **IAM Identity Center アサーションコンシューマーサービス (ACS) URL**
    *   **IAM Identity Center 発行者 URL** (Issuer URL)

---

### 手順 2: Okta側の設定 (アプリ作成)

次に、Oktaの管理画面（Admin Console）での操作です。

1.  Okta管理画面の左メニューから **[Applications]** > **[Applications]** をクリック。
2.  **[Browse App Catalog]** (アプリカタログを参照) ボタンをクリック。
3.  検索窓に `AWS IAM Identity Center` と入力して検索。
4.  検索結果に出てきた **AWS IAM Identity Center** をクリックし、**[Add Integration]** (追加) をクリック。
5.  **General Settings (基本設定):**
    *   **Application Label**: 表示名です。そのままでOK。
    *   **[Done]** または **[Next]** をクリック。
6.  **Sign-On Options (サインオン設定):**
    *   **SAML 2.0** が選ばれていることを確認。
    *   下にスクロールし、**[Advanced Sign-on Settings]** というセクションを探します。
    *   ここに、**手順1でAWSからコピーしたURL**を貼り付けます。
        *   **AWS SSO ACS URL**: 手順1の「ACS URL」を貼り付け
        *   **AWS SSO Issuer URL**: 手順1の「発行者 URL」を貼り付け
7.  **メタデータのダウンロード:**
    *   同じ画面の少し下に「SAML Signing Certificates」などの項目付近、または右側の黄色いボックス内などに **Identity Provider metadata** という青いリンクがあるはずです。
    *   これをクリック（または右クリック保存）して、XMLファイル（例: `metadata.xml`）としてPCに保存してください。
8.  **[Save]** (保存) をクリックしてOkta側の設定を保存します。

---

### 手順 3: AWS側の設定 (ファイルのアップロード)

AWSの画面（手順1の続き）に戻ります。

1.  「ID プロバイダーのメタデータ」セクションにある **[ファイルを選択]** ボタンを押します。
2.  手順2でOktaからダウンロードした **XMLファイル** をアップロードします。
3.  **[次へ]** を押し、確認画面で `ACCEPT` と入力して変更を完了させます。

---

### 手順 4: ユーザーの割り当て (重要！)

連携はできましたが、まだ誰も使えません。「誰にこのアプリを使わせるか」をOkta側で指定する必要があります。

1.  **Okta管理画面**に戻ります。
2.  **[Applications]** > **[Applications]** から、先ほど作ったAWSアプリをクリック。
3.  **[Assignments]** (割り当て) タブをクリック。
4.  **[Assign]** > **[Assign to People]** をクリック。
5.  自分のユーザー（自分自身）の横にある **[Assign]** を押し、**[Save and Go Back]** > **[Done]** をクリック。

---

### 仕上げの注意点

これで連携設定は完了ですが、ログインを成功させるには**「Oktaのメールアドレス」と「AWS Identity Center上のユーザー名（メールアドレス）」が完全に一致している**必要があります。

*   **AWS側:** IAM Identity Centerの「ユーザー」メニューで、Oktaと同じメールアドレスのユーザーを作成し、AWSアカウントへのアクセス権（Permission Set）を割り当てておいてください。

