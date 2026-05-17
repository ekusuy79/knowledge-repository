# Claude Desktop から GitHub を操作する（github-mcp-server セットアップ）

## 概要

GitHub はパブリックな MCP サーバ（`api.githubcopilot.com/mcp`）を提供しているが、Claude Desktop の OAuth 対応の問題で現時点では利用できない。  
代替として、ローカルに **github-mcp-server** を立てることで、Claude Desktop から GitHub リポジトリの参照・操作が可能になる。

参考記事: [Claude Desktop から GitHub を操作する ── github-mcp-server のセットアップガイド](https://zenn.dev/tjst_t/articles/260222-setup-github-mcp-server-for-claude-desktop)

---

## 環境

- OS: Windows
- Claude Desktop インストール済み
- GitHub アカウントあり

---

## 手順

### ステップ 1: MCP サーバのバイナリを取得する

以下の URL から最新版の ZIP をダウンロードして展開し、`github-mcp-server.exe` を任意の場所に配置する。

```
https://github.com/github/github-mcp-server/releases/latest/download/github-mcp-server_Windows_x86_64.zip
```

配置場所の例: `%USERPROFILE%\bin`  
※ 後の設定ファイルにフルパスを記述するので、わかりやすい場所に置く。

---

### ステップ 2: GitHub Fine-Grained Token を作成する

1. GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens
2. **Generate new token** をクリック
3. 以下を設定する

| 項目 | 設定値 |
| --- | --- |
| Token name | 任意（例: `claude-desktop-mcp`） |
| Expiration | 任意（デフォルト 30 日） |
| Repository access | 対象リポジトリを選択 |
| Permissions > Metadata | Read-only（デフォルト） |
| Permissions > Contents | Read-only または Read and write |

4. **Generate token** をクリックし、表示されたトークンをコピーして保管する

> ⚠️ トークンはこの画面でしか表示されない。必ずコピーしておくこと。

---

### ステップ 3: Claude Desktop の設定を更新する

1. Claude Desktop → Settings → Developer → Edit Config
2. `claude_desktop_config.json` に以下を追記する

```json
{
  "mcpServers": {
    "github": {
      "command": "C:\\Users\\your-username\\bin\\github-mcp-server.exe",
      "args": ["stdio"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "github_pat_XXXXXX"
      }
    }
  }
}
```

> ⚠️ `command` にはフルパスを指定する。パス区切りはバックスラッシュ2つ（`\\`）で記述する。

---

### ステップ 4: 動作確認

1. Claude Desktop を再起動する
2. 入力欄の右下アイコンから MCP サーバ一覧を確認する
3. **github** が表示されていれば接続成功

---

## まとめ

- バイナリを取得して配置する
- Fine-Grained Token で必要最小限の権限を付与する
- `claude_desktop_config.json` にサーバ情報を登録して再起動する

まずは Contents を Read-only で試し、必要に応じて Read and write に変更するのがおすすめ。
