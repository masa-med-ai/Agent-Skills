# Agent-Skills

Claude（claude.ai / Claude Code）向けの自作 Agent Skills 集。各 `.skill` ファイルは zip 形式で、中に `SKILL.md`（＋必要に応じて `scripts/` `references/`）を含む。

## インストール

- **claude.ai**: Settings → Capabilities → Skills から `.skill` ファイルをアップロード。
- **Claude Code**: `.skill` を unzip して `~/.claude/skills/<skill-name>/` に配置。

## スキル一覧

| スキル | 概要 | 主な依存 |
|---|---|---|
| `pubmed-systematic-review` | PubMed系統検索→抄録から数値データを正確に抽出しExcel/スライド化。数値捏造を防ぐ5段階ワークフロー | PubMed MCP |
| `pubmed-reference-verifier` | 論文原稿の本文中引用と参考文献リストの整合性検証・PubMed照合・修正提案 | PubMed MCP |
| `josler-byoreki-review` | J-OSLER 病歴要約を手引き準拠で添削し Word（.docx）出力 | Node.js（docx） |
| `image-to-pptx` | 図解画像を編集可能な PowerPoint にハイブリッド再現 | pptxgenjs, LibreOffice |
| `ai-journal-watch` | 主要医学誌RSSからAI関連新着だけを抽出して日本語配信（既読管理つき） | Python 3 |
| `ai-news-digest` | 過去24時間のAIニュース（企業・一般・医療）を出典リンク付きで日本語まとめ | Web検索 |
| `notion-upload-image` | ローカル画像をJPEG圧縮→Base64埋め込みSVGとしてNotionへアップロード | Notion MCP, ImageMagick |

## メモ

- PubMed 系スキルのツール名プレフィックス（`mcp__pubmed__*`）は MCP サーバーの登録名に合わせて読み替える。
- `ai-journal-watch` / `ai-news-digest` の「関心領域」（🔍 で優先表示するトピック）は既定で消化器・内視鏡向け。SKILL.md 冒頭の設定を書き換えれば他領域にも使える。
- `josler-byoreki-review` は日本内科学会『病歴要約 作成と評価の手引き』に準拠した参照資料を同梱するため `license: Proprietary`（個人利用の範囲で使用のこと）。
- `notion-upload-image` は `agents/openai.yaml` を同梱しており、ChatGPT / Codex 側のエージェント設定にも対応。
