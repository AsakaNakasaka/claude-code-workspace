# Claude Code リファレンス

## 目次
1. [利用可能なモデル](#モデル)
2. [CLIコマンド一覧](#cliコマンド)
3. [スラッシュコマンド](#スラッシュコマンド)
4. [主要API機能](#主要api機能)
5. [ツール使用（Tool Use）](#ツール使用)
6. [コード例](#コード例)
7. [Managed Agents](#managed-agents)
8. [プラットフォーム対応](#プラットフォーム対応)

---

## モデル

| モデル名 | モデルID | 用途 |
|---------|---------|------|
| Claude Fable 5 | `claude-fable-5` | 最新・最高性能 |
| Claude Opus 4.8 | `claude-opus-4-8` | 高度な推論・複雑タスク |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | バランス型（デフォルト） |
| Claude Haiku 4.5 | `claude-haiku-4-5-20251001` | 高速・軽量 |

---

## CLIコマンド

```bash
# 基本起動
claude                          # インタラクティブモード起動
claude "質問やタスク"            # 直接プロンプト実行
claude --print "質問"           # 出力のみ表示（-p）
claude --output-format json     # JSON形式で出力

# ファイル操作
claude --add-dir /path/to/dir   # ディレクトリを追加
claude --resume                 # 前のセッションを再開

# モデル指定
claude --model claude-opus-4-8

# デバッグ・詳細表示
claude --verbose                # 詳細ログ表示

# バージョン・ヘルプ
claude --version
claude --help
```

---

## スラッシュコマンド

### 会話・セッション管理

| コマンド | 説明 |
|---------|------|
| `/clear` | 会話履歴をクリア |
| `/compact` | コンテキストを圧縮してトークンを節約 |
| `/resume` | 前のセッションを再開 |
| `/exit` / `/quit` | Claude Code を終了 |

### コード・ファイル操作

| コマンド | 説明 |
|---------|------|
| `/add-dir <path>` | 作業ディレクトリを追加 |
| `/diff` | 変更差分を表示 |

### 設定・モード

| コマンド | 説明 |
|---------|------|
| `/model <name>` | 使用モデルを切り替え |
| `/fast` | Fast モード切替（Opusで高速出力） |
| `/settings` | 設定を表示・変更 |
| `/memory` | メモリ（記憶）を管理 |
| `/help` | ヘルプを表示 |

### 高度な機能

| コマンド | 説明 |
|---------|------|
| `/code-review ultra` | マルチエージェントによるコードレビュー（ブランチ全体） |
| `/code-review ultra <PR#>` | GitHub PRのコードレビュー |
| `/ultrareview` | `/code-review ultra` の旧エイリアス |

---

## 主要API機能

### Messages API 基本構造

```python
import anthropic

client = anthropic.Anthropic()

message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "こんにちは！"}
    ]
)
print(message.content[0].text)
```

### システムプロンプト

```python
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system="あなたは親切な日本語アシスタントです。",
    messages=[
        {"role": "user", "content": "Pythonの基本を教えて"}
    ]
)
```

### ストリーミング

```python
with client.messages.stream(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "長い文章を書いて"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

### マルチターン会話

```python
messages = []

# ターン1
messages.append({"role": "user", "content": "Pythonとは何ですか？"})
response = client.messages.create(model="claude-sonnet-4-6", max_tokens=512, messages=messages)
messages.append({"role": "assistant", "content": response.content[0].text})

# ターン2
messages.append({"role": "user", "content": "それの主な用途を教えて"})
response = client.messages.create(model="claude-sonnet-4-6", max_tokens=512, messages=messages)
```

### 構造化出力（Structured Outputs）

```python
import json

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": "商品名: リンゴ、価格: 150円、カテゴリ: 果物 をJSON形式で返して"
    }],
    system="必ずJSON形式で回答してください。"
)
data = json.loads(response.content[0].text)
```

### Extended Thinking（拡張思考）

```python
response = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=16000,
    thinking={
        "type": "enabled",
        "budget_tokens": 10000  # 思考に使うトークン上限
    },
    messages=[{"role": "user", "content": "この数学の問題を解いて: ..."}]
)
# thinking ブロックと text ブロックが返る
for block in response.content:
    if block.type == "thinking":
        print("思考プロセス:", block.thinking)
    elif block.type == "text":
        print("回答:", block.text)
```

### プロンプトキャッシング

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system=[{
        "type": "text",
        "text": "長いシステムプロンプト...",
        "cache_control": {"type": "ephemeral"}  # キャッシュ有効化
    }],
    messages=[{"role": "user", "content": "質問"}]
)
# cache_creation_input_tokens と cache_read_input_tokens で確認
print(response.usage)
```

---

## ツール使用

### ユーザー定義ツール

```python
tools = [
    {
        "name": "get_weather",
        "description": "指定した都市の現在の天気を取得する",
        "input_schema": {
            "type": "object",
            "properties": {
                "city": {
                    "type": "string",
                    "description": "都市名（例: Tokyo）"
                }
            },
            "required": ["city"]
        }
    }
]

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "東京の天気は？"}]
)

# tool_use ブロックを処理
if response.stop_reason == "tool_use":
    for block in response.content:
        if block.type == "tool_use":
            tool_name = block.name
            tool_input = block.input
            # 実際のツール実行 → 結果をClaudeに返す
```

### サーバーサイドツール（API提供）

| ツール名 | 説明 |
|---------|------|
| `web_search` | Webを検索 |
| `code_execution` | コードを実行 |
| `advisor` | アドバイスを提供 |

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    tools=[{"type": "web_search_20250305", "name": "web_search"}],
    messages=[{"role": "user", "content": "最新のPythonバージョンは？"}]
)
```

### クライアントサイドツール（Claude Code内）

| ツール名 | 説明 |
|---------|------|
| `bash` | シェルコマンドを実行 |
| `text_editor` | ファイルを読み書き・編集 |
| `memory` | 永続メモリの読み書き |

---

## Managed Agents

### 主要概念

| 概念 | 説明 |
|-----|------|
| **Agent** | 自律的にタスクを実行するAIエージェント |
| **Session** | エージェントとの会話セッション |
| **Environment** | エージェントが動作する実行環境 |
| **Vault** | 安全なシークレット管理 |
| **Memory Store** | エージェントの長期記憶 |
| **Deployment** | エージェントのデプロイ設定 |

### エージェント作成例（Python SDK）

```python
from anthropic import Anthropic

client = Anthropic()

# エージェントとの会話（セッション）
session = client.beta.agents.sessions.create(
    agent_id="your-agent-id"
)

response = client.beta.agents.sessions.message(
    session_id=session.id,
    messages=[{"role": "user", "content": "タスクを実行して"}]
)
```

---

## コード例

### TypeScript / Node.js

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const message = await client.messages.create({
  model: "claude-sonnet-4-6",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Hello, Claude!" }],
});

console.log(message.content[0].text);
```

### 画像入力（Vision）

```python
import base64

with open("image.jpg", "rb") as f:
    image_data = base64.b64encode(f.read()).decode("utf-8")

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "base64",
                    "media_type": "image/jpeg",
                    "data": image_data,
                },
            },
            {"type": "text", "text": "この画像を説明してください"}
        ],
    }]
)
```

### PDFファイル入力

```python
with open("document.pdf", "rb") as f:
    pdf_data = base64.b64encode(f.read()).decode("utf-8")

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=2048,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "document",
                "source": {
                    "type": "base64",
                    "media_type": "application/pdf",
                    "data": pdf_data,
                },
            },
            {"type": "text", "text": "このPDFの要点をまとめてください"}
        ],
    }]
)
```

### バッチ処理（Message Batches API）

```python
batch = client.messages.batches.create(
    requests=[
        {
            "custom_id": "request-1",
            "params": {
                "model": "claude-sonnet-4-6",
                "max_tokens": 1024,
                "messages": [{"role": "user", "content": "質問1"}]
            }
        },
        {
            "custom_id": "request-2",
            "params": {
                "model": "claude-sonnet-4-6",
                "max_tokens": 1024,
                "messages": [{"role": "user", "content": "質問2"}]
            }
        }
    ]
)
print(f"バッチID: {batch.id}")
```

---

## SDK対応言語

| 言語 | インストール |
|-----|------------|
| Python | `pip install anthropic` |
| TypeScript/Node.js | `npm install @anthropic-ai/sdk` |
| Go | `go get github.com/anthropics/anthropic-sdk-go` |
| Ruby | `gem install anthropic` |
| Java | Maven/Gradle で `com.anthropic:anthropic-java` |
| C# | NuGet で `Anthropic` |
| PHP | `composer require anthropic-php/client` |

---

## プラットフォーム対応

| プラットフォーム | 説明 |
|---------------|------|
| **Anthropic API** | 直接APIアクセス（api.anthropic.com） |
| **Amazon Bedrock** | AWS経由でClaudeを利用 |
| **Google Vertex AI** | GCP経由でClaudeを利用 |
| **Microsoft Azure AI Foundry** | Azure経由でClaudeを利用 |

### Bedrock での利用例

```python
import anthropic

client = anthropic.AnthropicBedrock(
    aws_region="us-east-1"
)

response = client.messages.create(
    model="anthropic.claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "こんにちは"}]
)
```

---

## Claude Code 設定ファイル

### CLAUDE.md
プロジェクトルートに置くとClaude Codeが自動的に読み込む設定・指示ファイル。

```markdown
# プロジェクト概要
このプロジェクトは...

# コーディング規約
- コメントは日本語で記述
- 関数名はキャメルケース

# よく使うコマンド
- テスト実行: `npm test`
- ビルド: `npm run build`
```

### 設定ファイルの場所
- グローバル設定: `~/.claude/settings.json`
- プロジェクト設定: `.claude/settings.json`（プロジェクトルート）

---

## 便利なパターン

### エラーハンドリング

```python
from anthropic import APIError, RateLimitError, APIConnectionError

try:
    response = client.messages.create(...)
except RateLimitError:
    print("レート制限に達しました。しばらく待ってから再試行してください。")
except APIConnectionError:
    print("接続エラー。ネットワークを確認してください。")
except APIError as e:
    print(f"APIエラー: {e.status_code} - {e.message}")
```

### 環境変数での認証

```bash
# .env ファイルまたは環境変数
export ANTHROPIC_API_KEY="sk-ant-..."
```

```python
import os
from anthropic import Anthropic

# ANTHROPIC_API_KEY 環境変数から自動読み込み
client = Anthropic()
```

---

*最終更新: 2026-06-28*
*参考: https://docs.anthropic.com*
