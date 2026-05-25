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

1. Cloudflare Zero Trust を利用できるようにする
2. サーバー PC との Tunnel を作成する
3. デバイス登録ポリシーを設定する
4. Cloudflare のデバイス設定を変更する
5. クライアント PC に WARP をインストールし接続設定する
6. RDP 接続する

---

### 1. Cloudflare Zero Trust を利用できるようにする

1. Cloudflare にログイン → サイドバーの「Zero Trust」をクリック
2. Team name を設定する（後の手順で使うので覚えておく）
3. プランを選択する（個人利用なら Free で十分）
4. 支払方法を登録する（Free プランでも登録が必要）

---

### 2. サーバー PC との Tunnel を作成する

1. Zero Trust → ネットワーク → コネクタ → トンネルを作成
2. トンネル名を入力して「Save tunnel」
3. サーバーPCに cloudflared をインストール（Windows 64bit の場合は `.msi` インストーラを使用）
4. インストール後、コマンドプロンプトで動作確認

```bash
cloudflared --version
```

5. **管理者権限**でコマンドプロンプトを開き、Cloudflare 画面に表示されているコマンドを実行する

```bash
cloudflared.exe service install <TOKEN>
```

> ⚠️ TOKEN はコマンドに含まれる機密情報。流出させないこと。

6. Cloudflare 画面で作成したトンネル名のリンクを押下する
7. CIDRルート タブに切り替え、サーバーPC の IP を入力して「Save」
8. コネクタ の ステータス が 接続済み になれば成功

---

### 3. デバイス登録ポリシーを設定する

接続できるデバイスを制限する。設定しないと Team 名を知っている人なら誰でも接続できてしまう。

1. Zero Trust → チームとリソース → デバイス → 管理タブ → デバイスの登録の「管理」ボタンを押下。
2. 「新しいポリシーを作成」を押下。
3. ポリシールールにメールを選択し、自身のメールアドレスを設定。
4. ポリシー名を入力し、「アクション」に「許可」を選択。
5. 「テキストコントロール」に「両方向許可」を選択。
6. 「ポリシーを保存」を押下。

---

### 4. Cloudflare のデバイス設定を変更する

デフォルト設定ではプライベート IP はローカルネットワークに接続されてしまうため、Cloudflare 経由になるよう変更する。

1. Zero Trust → チームとリソース → デバイス から デバイスプロファイルタブを開く
2. 一般プロファイルの「デフォルト」を選択し、「スプリットトンネル」を「IPとドメインを含める」を選択する。
3. 警告が出るが続行する。
4. セレクタにIPを選択し、接続先PCのIPを入力し「宛先の保存」を押下する。

---

### 5. クライアント PC に WARP をインストールし接続設定する

1. Zero Trust → Settings → Downloads → 「Download the WARP client」からクライアント OS に合ったインストーラをダウンロード・インストール
2. WARP 起動後、歯車アイコン → 環境設定 → アカウント → Cloudflare Zero Trust にログイン
3. Team name を入力
4. メールアドレスを入力し、届いた認証コードを入力してログイン
5. WARP クライアントの表示が「Zero Trust」に変わったらトグルを ON にする

---

### 6. RDP 接続する

1. クライアント PC を外部ネットワークに接続し、WARP のトグルを ON にする
2. 「リモートデスクトップ接続」を起動
3. 接続先にサーバー PC のプライベート IP アドレスを入力して接続
4. 無事に接続できれば成功
