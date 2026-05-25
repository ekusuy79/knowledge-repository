# Cloudflare Tunnel を利用して外部から Windows Pro マシンへ RDP する

## 概要

Cloudflare Zero Trust の Cloudflare Tunnel を使い、\
VPSのwindows serverに対してセキュアに接続する方法

参考記事: [Cloudflare Tunnelを利用して外部からWindows ProマシンへRDPする](https://zenn.dev/nakurei/articles/rdp-from-outside-to-windows-using-cloudflare)

---

## 環境

- 接続元（クライアント側）: Windows 11
- 接続先（サーバー側）: Windows Server 2022

---

## 前提条件

- 接続先 PC が RDP を受け付けられる状態になっている
- Cloudflare アカウントを作成済み

---

## 全体の流れ

1. サーバー PC のプライベート IP アドレスを固定する
2. Cloudflare Zero Trust を利用できるようにする
3. サーバー PC との Tunnel を作成する
4. デバイス登録ポリシーを設定する
5. Cloudflare のデバイス設定を変更する
6. クライアント PC に WARP をインストールし接続設定する
7. RDP 接続する

---

## 手順

### 1. サーバー PC のプライベート IP アドレスを固定する

再起動後も同じ設定で RDP できるよう、接続先 PC の IP を固定する。

1. コマンドプロンプトで `ipconfig` を実行し、以下を控える
   - IPv4 アドレス（プライベート IP）
   - デフォルトゲートウェイ

2. Windows 設定 → ネットワークとインターネット → 該当アダプター → IP 割り当て → 編集

3. 「手動」に切り替えて IPv4 をオンにし、以下を入力して保存

| 項目 | 入力値 |
| --- | --- |
| IP アドレス | `ipconfig` で確認したプライベート IP |
| サブネットマスク | `255.255.255.0` |
| デフォルトゲートウェイ | `ipconfig` で確認したゲートウェイ |
| 優先 DNS | `ipconfig` で確認したゲートウェイ |

---

### 2. Cloudflare Zero Trust を利用できるようにする

1. Cloudflare にログイン → サイドバーの「Zero Trust」をクリック
2. Team name を設定する（後の手順で使うので覚えておく）
3. プランを選択する（個人利用なら Free で十分）
4. 支払方法を登録する（Free プランでも登録が必要）

---

### 3. サーバー PC との Tunnel を作成する

1. Zero Trust → Access → Tunnels → 「Add a tunnel」
2. トンネル名を入力して「Save tunnel」
3. 環境に合わせて cloudflared をインストール（Windows 64bit の場合は `.msi` インストーラを使用）
4. インストール後、コマンドプロンプトで動作確認

```bash
cloudflared --version
```

5. **管理者権限**でコマンドプロンプトを開き、Cloudflare 画面に表示されているコマンドを実行する

```bash
cloudflared.exe service install <TOKEN>
```

> ⚠️ TOKEN はコマンドに含まれる機密情報。流出させないこと。

6. Cloudflare 画面の Connectors に「Connected」なデバイスが表示されたら Next
7. Route tunnel の画面で Private Networks タブに切り替え、サーバー PC のプライベート IP を入力して「Save tunnel」
8. Tunnel の Status が `HEALTHY` になれば成功

---

### 4. デバイス登録ポリシーを設定する

接続できるデバイスを制限する。設定しないと Team 名を知っている人なら誰でも接続できてしまう。

1. Zero Trust → Settings → WARP Client → Device enrollment permissions → Manage
2. Rules タブ → 「Add a rule」
3. Selector を「Emails」にし、Value に自分のメールアドレスを入力
4. Rule action が「Allow」であることを確認して「Save」

---

### 5. Cloudflare のデバイス設定を変更する

デフォルト設定ではプライベート IP はローカルネットワークに接続されてしまうため、Cloudflare 経由になるよう変更する。

1. Zero Trust → Settings → WARP Client → Device settings → Profile settings → 「Create profile」
2. 名前を付けて「Build an expression」でメールアドレス等のルールを設定
3. 「Create profile」をクリック
4. 作成したプロファイルを選択 → 「Configure」
5. 「Split Tunnels」を「Include IPs and domains」に変更（WARNING が出るが「Continue and delete」）
6. サーバー PC のプライベート IP を入力して「Save destination」
7. 「Back to Profile」→「Save profile」で保存

---

### 6. クライアント PC に WARP をインストールし接続設定する

1. Zero Trust → Settings → Downloads → 「Download the WARP client」からクライアント OS に合ったインストーラをダウンロード・インストール
2. WARP 起動後、歯車アイコン → 環境設定 → アカウント → Cloudflare Zero Trust にログイン
3. Team name を入力
4. メールアドレスを入力し、届いた認証コードを入力してログイン
5. WARP クライアントの表示が「Zero Trust」に変わったらトグルを ON にする

---

### 7. RDP 接続する

1. クライアント PC を外部ネットワークに接続し、WARP のトグルを ON にする
2. 「リモートデスクトップ接続」を起動
3. 接続先にサーバー PC のプライベート IP アドレスを入力して接続
4. 無事に接続できれば成功
