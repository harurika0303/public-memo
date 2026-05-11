# Llama 3.1 8B を VSCode + WSL エージェントとして使う手順

## 全体の流れ

```
[Step 1] Ollama を Windows にインストール
    ↓
[Step 2] Llama 3.1 8B をダウンロード・動作確認
    ↓
[Step 3] WSL からアクセスできるか確認
    ↓
[Step 4] VS Code に拡張機能をインストール
    ↓
[Step 5] 拡張機能を Ollama に接続して使う
```

---

## Step 1: Ollama を Windows にインストール

PowerShell で実行:

```powershell
irm https://ollama.com/install.ps1 | iex
```

または [https://ollama.com/download/OllamaSetup.exe](https://ollama.com/download/OllamaSetup.exe) から手動ダウンロード。

インストール後、Ollama はバックグラウンドサービスとして自動起動します。

---

## Step 2: Llama 3.1 8B をダウンロード・動作確認

### ダウンロード

```powershell
ollama pull llama3.1:8b
```

ダウンロードサイズ: 約 4.7 GB（Q4_K_M 量子化）

### 動作確認（チャット）

```powershell
ollama run llama3.1:8b
```

`>>> ` プロンプトが表示されたら成功。日本語で話しかけてみる。  
終了は `/bye` または `Ctrl+D`。

### API の動作確認

bash（Git Bash / MINGW64 含む）はシングルクォートで囲む:

```bash
curl http://localhost:11434/api/chat -d '{"model": "llama3.1:8b", "messages": [{"role": "user", "content": "Hello"}, ], "stream": false}'
```

> **注意**: bash の `"..."` 内に `!` を含めると `bash: !\: event not found` エラーになります。  
> JSON はシングルクォート `'...'` で囲む、またはメッセージ文末の `!` を除くことで回避できます。

PowerShell の場合:

```powershell
$body = '{"model": "llama3.1:8b", "messages": [{"role": "user", "content": "Hello"}], "stream": false}'
Invoke-RestMethod -Method Post -Uri "http://localhost:11434/api/chat" -ContentType "application/json" -Body $body
```

成功すると以下のような JSON が返ります（初回は約 10 秒かかる場合あり）:

```json
{
  "model": "llama3.1:8b",
  "message": {"role": "assistant", "content": "Is there something I can help you with?"},
  "done": true
}
```

---

## Step 3: WSL からアクセスできるか確認

WSL のターミナルで確認:

```bash
curl http://localhost:11434/api/tags
```

`{"models":[...]}` が返れば OK。

### アクセスできない場合 → ミラーネットワークを有効にする

WSL 2.0.0 以降（`wsl --version` で確認）では、**ミラーネットワーク**を有効にすると WSL 内でも `localhost` でそのままアクセスできます。

#### 手順

**1. `.wslconfig` を作成**（`C:\Users\<ユーザー名>\.wslconfig`）

```ini
[wsl2]
networkingMode=mirrored
```

**2. WSL を再起動**

PowerShell で実行:

```powershell
wsl --shutdown
```

**3. WSL から再度確認**

```bash
curl http://localhost:11434/api/tags
```

#### 補足

- `wsl --version` で WSL バージョンを確認できます（2.0.0 以上が対象）
- `.wslconfig` が存在しない場合は新規作成でOK
- `wsl --shutdown` 後は WSL ターミナルを開き直すと設定が反映されます

#### セキュリティについて

ミラーネットワークは **セキュリティ上安全**です。

Ollama のデフォルトは `127.0.0.1:11434`（ローカルホスト）で待ち受けるため、LAN の他の PC からはアクセスできません。ミラーネットワークは「WSL と Windows のループバックを共有する」だけで、外部への露出は増えません。

> ⚠️ **注意**: 代替手段の `OLLAMA_HOST=0.0.0.0` を設定すると Ollama が全インターフェースで待ち受けるようになり、LAN 上の誰でも API にアクセスできるようになります（Ollama の API に認証はないため）。カフェや会社のネットワークでは特に危険です。

#### ミラーネットワークが使えない環境（WSL 1.x など）の場合

Windows ホスト IP を使う方法もあります:

```bash
# WSL 内で Windows ホスト IP を取得
WINDOWS_IP=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}')
curl http://$WINDOWS_IP:11434/api/tags
```

この場合、Windows 側の Ollama に `OLLAMA_HOST=0.0.0.0` の設定が必要です:

```powershell
# PowerShell（管理者不要）
[System.Environment]::SetEnvironmentVariable("OLLAMA_HOST", "0.0.0.0", "User")
# → Ollama サービスをタスクトレイから再起動
```

---

## Step 4: VS Code に拡張機能をインストール

エージェントとして使うには **Continue** または **Cline** が有力です。

### 選択肢の比較

| 拡張機能                                                                            | 特徴                                                   | 向いている用途             |
| ----------------------------------------------------------------------------------- | ------------------------------------------------------ | -------------------------- |
| [Continue](https://marketplace.visualstudio.com/items?itemName=Continue.continue)   | チャット・インライン補完・エージェント全部入り         | 普段使いのコード補助       |
| [Cline](https://marketplace.visualstudio.com/items?itemName=saoudrizwan.claude-dev) | 完全自律型エージェント（ファイル操作・ターミナル実行） | 大きなタスクを丸ごと任せる |

まずは **Continue** から始めるのがおすすめ（設定が簡単で日本語にも対応）。

### Continue のインストール

1. VS Code の左サイドバーの拡張機能アイコン（`Ctrl+Shift+X`）をクリック
2. 検索ボックスに `Continue` と入力
3. **Continue - open-source AI code agent**（発行元: Continue）を選択してインストール
4. インストール後、左サイドバーに Continue のアイコン（矢印マーク）が追加される

---

## Step 5: Continue を Ollama に接続する

### 設定ファイルを開く

設定ファイルは以下のパスに固定されています:

```
C:\Users\<ユーザー名>\.continue\config.yaml
```

開く方法は 2 つあります:

- **エクスプローラーから**: `C:\Users\mm33s\.continue\config.yaml` を直接開く
- **コマンドパレット**: `Ctrl+Shift+P` → `Continue: Open Config File`（`yaml` ではなく `File` の場合あり）

> **注意**: コマンドパレットで `Continue: Open config.yaml` が出てこない場合はエクスプローラーから直接開いてください。

### config.yaml を編集する

デフォルトでは `models: []` となっています。以下の内容で**丸ごと置き換え**てください:

```yaml
name: Local Config
version: 1.0.0
schema: v1
models:
  - name: Llama 3.1 8B (Local)
    provider: ollama
    model: llama3.1:8b
    apiBase: http://localhost:11434

tabAutocompleteModel:
  name: Llama 3.1 8B (Local)
  provider: ollama
  model: llama3.1:8b
  apiBase: http://localhost:11434
```

> **ポイント**: `models:` の値が `[]`（空）のままだとモデルが表示されません。必ず上の内容に書き換えてください。

### 接続確認

1. ファイルを保存（`Ctrl+S`）
2. VS Code をリロード: `Ctrl+Shift+P` → `Developer: Reload Window`
3. 左サイドバーの Continue アイコンをクリック
4. チャットパネル上部のドロップダウンに **「Llama 3.1 8B (Local)」** が表示されれば成功
5. 「Hello」と送信して応答が返れば完了

---

## 使い方

### チャット（`Ctrl+L`）

コードを選択してから `Ctrl+L` でチャットパネルに送れます。

例:
- 「このコードを説明して」
- 「バグを修正して」
- 「テストコードを書いて」

### インライン編集（`Ctrl+I`）

コードを選択 → `Ctrl+I` → 指示を入力 → その場でコードが変換されます。

### エージェントモード

Continue のチャットで `@` を入力するとファイルやフォルダをコンテキストとして渡せます。

```
@README.md このプロジェクトの構成を理解して、XXXの機能を追加して
```

---

## WSL 環境での注意点

- Ollama は **Windows 側**で起動しておく（WSL からでも `localhost:11434` でアクセス可能）
- VS Code の WSL ウィンドウ（`code .` in WSL）でも Continue は動作する
- WSL 側で Ollama をインストールする方法もあるが、Windows 側の Ollama を共用するほうがシンプル

### WSL 側に Ollama をインストールする場合（任意）

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama serve &
ollama pull llama3.1:8b
```

この場合、Continue の `apiBase` は `http://localhost:11434` のまま。

---

## トラブルシューティング

| 症状                            | 対処                                                                                                            |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `ollama run` が遅い             | GPU オフロードが効いていない可能性。`ollama ps` で確認                                                          |
| WSL から接続できない            | `OLLAMA_HOST=0.0.0.0` を設定して Ollama を再起動                                                                |
| Continue でモデルが表示されない | `models: []` のままになっていないか確認。`ollama list` でモデル名を確認し、config.yaml の `model:` と一致させる |
| 応答が途切れる                  | コンテキスト長を短くする（config.yaml に `contextLength: 4096` を追加）                                         |

### Ollama の環境変数設定（Windows サービス）

> ⚠️ `OLLAMA_HOST=0.0.0.0` は LAN 全体に公開されます。**ミラーネットワーク（Step 3）で解決した場合はこの設定は不要です。**

Windows の場合、Ollama のサービス設定は以下の環境変数で変更できます:

```powershell
# 全インターフェースからのアクセスを許可（ミラーネットワークが使えない場合のみ）
[System.Environment]::SetEnvironmentVariable("OLLAMA_HOST", "0.0.0.0", "User")

# 設定後はタスクトレイの Ollama アイコンから再起動
```

---

## 参考リンク

- [Ollama ドキュメント](Ollamaについて.md)
- [Continue 公式ドキュメント](https://docs.continue.dev/)
- [Cline GitHub](https://github.com/cline/cline)
- [CanIRun.ai - Llama 3.1 8B](https://www.canirun.ai/model/llama3.1-8b)
