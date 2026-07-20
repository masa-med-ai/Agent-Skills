# Agent-Skills

Claude（claude.ai / Claude Code）向けの自作 Agent Skills 集。各 `.skill` ファイルは zip 形式で、中に `SKILL.md`（＋必要に応じて `scripts/` `references/` `agents/` `assets/`）を含む。

## インストール

- **claude.ai**: Settings → Capabilities → Skills から `.skill` ファイルをアップロード。
- **Claude Code**: `.skill` を unzip して `~/.claude/skills/<skill-name>/` に配置。
- 一部スキルは `agents/openai.yaml` を同梱しており、ChatGPT / Codex 側のエージェント設定にも対応。

## スキル一覧（概要）

| カテゴリ | スキル | 一言でいうと | 主な依存 |
|---|---|---|---|
| 文献検索 | [`pubmed-search`](#pubmed-search) | PubMed MCP を使った質の高い文献検索の作法（PICO・MeSH・引用形式） | PubMed MCP |
| 文献検索 | [`pubmed-eutils-search`](#pubmed-eutils-search) | MCP なしで NCBI E-utilities を直接叩く PubMed 検索 CLI | Python 3（標準ライブラリのみ） |
| 文献検索 | [`pubmed-systematic-review`](#pubmed-systematic-review) | 系統的検索→抄録から数値を正確に抽出して Excel/スライド化 | PubMed MCP |
| 原稿支援 | [`pubmed-reference-verifier`](#pubmed-reference-verifier) | 本文中引用と参考文献リストの整合性検証・PubMed 照合・修正提案 | PubMed MCP |
| 論文整理 | [`paper-summarize-to-notion`](#paper-summarize-to-notion) | 論文1本を日本語要約＋グラフィカルアブストラクト化して Notion DB に登録 | Notion MCP, 画像生成 |
| 論文整理 | [`paper-to-one-slide-ja`](#paper-to-one-slide-ja) | 論文 PDF を日本語1枚スライド（.pptx）に凝縮 | Python 3, Presentations/pdf スキル |
| 情報収集 | [`ai-journal-watch`](#ai-journal-watch) | 主要医学誌 RSS から AI 関連新着だけを日本語配信（既読管理つき） | Python 3 |
| 情報収集 | [`ai-news-digest`](#ai-news-digest) | 過去24時間の AI ニュース（企業・一般・医療）を出典付きで日本語まとめ | Web 検索 |
| 資料作成 | [`image-to-pptx`](#image-to-pptx) | 図解画像を「編集可能な」PowerPoint にハイブリッド再現 | pptxgenjs, LibreOffice, PIL |
| 教育・研修 | [`josler-byoreki-review`](#josler-byoreki-review) | J-OSLER 病歴要約を手引き準拠で添削し Word 出力 | Node.js（docx） |
| ユーティリティ | [`notion-upload-image`](#notion-upload-image) | 外部ホスト不要でローカル画像を Notion ページに貼る | Notion MCP, Python 3（PIL） |

---

## 文献検索系

### pubmed-search

PubMed MCP ツール群（search / count / fetch_batch / find_similar_articles / get_citation_counts / get_full_text / convert_ids など）を使った医学文献検索の「作法」を定めるスキル。

**できること**
- 臨床質問を PICO で概念分解 → MeSH ＋ フリーワード（tiab）併用の構造化クエリを構築
- `count` で検索式の妥当性を確認してから本検索する段階的ワークフロー
- 出典の捏造防止：PMID・著者・誌名・年は必ずツールの返り値からのみ引用
- 引用フォーマットの統一：`[*First Author, 誌名, Year*](https://pubmed.ncbi.nlm.nih.gov/PMID/)` のハイパーリンク付き形式を強制

**使い方**：「〜のエビデンスを調べて」「〜の RCT を探して」など文献検索の依頼で自動的に発動。明示的に「PubMed で」と言わなくてもよい。

### pubmed-eutils-search

PubMed MCP サーバーを使わず、公式 NCBI E-utilities / iCite / PMC API を直接叩く自己完結型スキル。同梱 CLI（`scripts/pubmed_eutils.py`、Python 標準ライブラリのみ）が MCP と同等の機能をカバーする。

**できること**
- `count` / `search`（日付・研究デザイン・言語・ヒト限定などの構造化フィルタ付き）
- `fetch` / `fetch-batch`（抄録・メタデータ取得）、`get-full-text`（PMC 全文をセクション指定で）
- `find-similar-articles`、`get-citation-counts`（iCite / PubMed）、`convert-ids`（PMID⇔PMCID⇔DOI）、`export-to-ris`、`batch-process`
- `NCBI_API_KEY` / `NCBI_EMAIL` 環境変数に対応（キーは出力しない設計）

**使い方**：PubMed MCP が使えない・使いたくない環境での文献検索で発動。`python3 scripts/pubmed_eutils.py --help` で全コマンド確認。エンドポイント選択や結果の解釈は `references/capabilities.md` 参照。

### pubmed-systematic-review

PubMed から複数文献を系統的に検索し、各抄録から数値データ（N 数・アウトカム・P 値など）を**正確に**抽出して Excel やスライドにまとめるスキル。

**できること**
- 5段階ワークフロー：①MeSH 検索式設計＋count 検証 → ②PMID 取得・スクリーニング → ③抄録取得・数値抽出（メインが逐次、原文照合） → ④構造化・自己検証（＋任意でサブエージェントによる独立検証） → ⑤Excel/PPTX 生成
- 最大の狙いは **fetch_batch のレスポンス切断による数値捏造の防止**。抽出フェーズをサブエージェントに分割委譲しない等の運用ルールを規定

**使い方**：「文献検索して Excel に」「RCT を調べてスライドに」「エビデンスを一覧表に」など、複数文献の比較・集計を伴う依頼で発動。1件だけの検索・要約には不要。

### pubmed-reference-verifier

医学論文原稿の本文中引用（in-text citation）と参考文献リストの整合性を検証するスキル。

**できること**
- 引用の欠落（本文にあるがリストにない）・孤立参照（リストにあるが本文にない）の検出
- Pandoc 形式（`[@key]`）・Vancouver 番号形式（`[1]`, `[1-3]`）など複数の引用スタイルに対応
- PubMed MCP（fetch_batch 等）で書誌情報の正確性を照合し、修正案を提示。MCP 不通時は E-utilities 直叩きにフォールバック

**使い方**：3モード制。

```
/pubmed-reference-verifier verify <file>   # 完全検証（相互参照 + PubMed照合）
/pubmed-reference-verifier check <file>    # 簡易検証（相互参照のみ・オフライン）
/pubmed-reference-verifier fix <file>      # 完全検証 + 自動修正の適用
```

---

## 論文整理系

### paper-summarize-to-notion

提示された論文（PDF・テキスト・書誌情報）1本を、構造化された日本語エビデンス要約＋日本語グラフィカルアブストラクトにして、Notion の論文データベースに登録するスキル。

**できること**
- 論文から研究デザイン・対象・方法・エンドポイント・数値結果・limitation を抽出し、推定は「本文からの推定:」、欠落は「記載なし」と明示した日本語要約を作成
- 画像生成前に「数値エビデンスマップ」（要約・画像で使う全数値の論文内出典位置）を作成し、**根拠を追跡できない数値を画像に入れない**。生成画像は数値・単位・小数点まで照合検証
- Notion の `DB_要チェック論文` DB を検索し（なければスキーマ付きで自動作成）、要約と画像を登録。画像は notion-upload-image と同じ SVG ラッパー方式で外部ホスト不要
- PubMed 検索・照合は行わない設計（提示された論文と付属メタデータのみ使用）

**使い方**：「この論文を要約して Notion に登録して」「グラフィカルアブストラクトを作って DB に追加して」。

### paper-to-one-slide-ja

論文 PDF を読み、「何を調べ、何が分かり、何を意味するか」が30秒で伝わる日本語1枚 PowerPoint スライドに凝縮するスキル。抄読会・学会・研究ミーティング向け。

**できること**
- `scripts/extract_paper.py` で PDF 本文・全ページ画像・埋め込み画像を一括抽出
- スライド化の前に「根拠メモ」（デザイン、N 数、主要評価項目、群別の分子/分母、95% CI、P 値など）を原文から確定し、数値の出所を追跡可能にする
- タイトルは論文の結論を表す一文にする。図表は**必ず原著から抽出**（自作グラフ・生成画像への置き換え禁止）
- フォントは日本語メイリオ＋英数字 Segoe UI。1枚要約専用のレイアウト設計。`references/quality-gates.md` の品質基準（文章量・図表選択・視覚 QA）で検証

**使い方**：「この PDF を1枚で要約」「論文の概要スライドを作成」「原著の図表を使ってまとめて」。

---

## 情報収集系

### ai-journal-watch

主要医学誌（JAMA / NEJM / NEJM AI / Lancet / Nature / Nature Medicine）の RSS を巡回し、AI 関連の新着だけを抽出して日本語配信するスキル。

**できること**
- `scripts/journal_watch.py` が RSS 取得 → AI キーワード抽出（NEJM AI は全件）→ 既読管理（`state/seen.json`）で**前回以降の新着のみ**を JSON 出力。初回は直近7日分に限定
- 各記事をタイトル和訳＋リンク＋**AI タイプ分類**（🖼画像AI / 🎥動画AI / 💬LLM・生成AI / 📈予測・信号モデル / 🧬創薬・分子AI / 🤖基盤モデル / 🛡倫理・安全性 / 📋政策・方法論）＋一言要約で整形
- 関心領域（既定：消化器・内視鏡・大腸・画像/動画解析 AI）の記事は 🔍 印で上位表示。新着0件の日は水増しせず正直に報告

**使い方**：「ジャーナルウォッチ」「医学誌 AI 新着」。毎朝の定例配信にも単発確認にも使える。テストは `--no-save`（既読を更新しない）。対象誌の追加・除外はスクリプト内 `FEEDS` を編集。

### ai-news-digest

「過去24時間の AI 最新動向」を一目で把握できる夕方ダイジェストを生成するスキル。

**できること**
- 3カテゴリを包括的に収集：①AI 企業のリリース（OpenAI / Anthropic / Google / Meta ほか）②一般 AI ニュース（規制・資金調達・研究・安全性）③医療 AI ニュース（高 IF 誌の臨床研究、FDA/PMDA 動向、創薬 AI など）
- 英語＋日本語の Web 検索を並行実行し、重複統合・重要度順ソート・各項目に出典リンク必須
- **「過去24時間」を厳守**：日付を確認し、古い話題は大ニュースでも除外。少ない日は無理に埋めない

**使い方**：「今日の AI ニュース」「夕方の AI まとめ」。定例配信にも単発依頼にも対応。

---

## 資料作成・その他

### image-to-pptx

完成済みの図解画像（ポンチ絵・概念図・フロー図・スライドのスクショ・AI 生成画像など）を、**編集可能な** PowerPoint として再現するスキル。

**できること**
- **ハイブリッド再現**：テキストはテキストボックス、枠・矢印・バッジはネイティブ図形として再構築し、アイコン・イラスト・写真だけを元画像から PIL で切り出して貼付
- `scripts/crop_icons.py` でアイコン領域を一括切り出し＋コンタクトシート生成 → 目視検証
- 出力後は LibreOffice でレンダリングして元画像と比較する QA 工程つき
- 全体を1枚画像として貼る（編集不可になる）ことも、アイコンを図形でゼロから描き直すことも禁止

**使い方**：「この画像を PPT にして」「ポンチ絵を編集可能なパワポに」。※テキスト/アウトラインからの新規スライド作成は対象外（入力が「画像」の場合に使う）。

### josler-byoreki-review

日本内科学会 J-OSLER の病歴要約を『病歴要約 作成と評価の手引き』（`reference/tebiki.md` として同梱）に準じて添削し、提出水準の Word（.docx）を出力するスキル。

**できること**
- **添削レポート**（Markdown）：①減点リスクの高い修正点（タイトル形式・個人情報・一般名・略語定義など）②「A4 2枚・紙面80%以上」を満たす加筆提案 ③カルテ確認が必要な要確認リスト
- **完成稿 Word**：手引きの様式・文体（常体、「，．」、数値と単位の間に半角スペース等）で全面改稿し、`scripts/build_docx.js`（Node.js + docx）で生成
- **実データを創作しない**のが絶対原則：不明な検査値・日付・用量は `【要確認：…】` として赤太字で空欄化

**使い方**：病歴要約の PDF/Word/テキスト/スクショを渡して「病歴要約を添削して」「二次評価に通る形にして」「文量を増やして」。

> ⚠️ 日本内科学会の手引きに準拠した参照資料を同梱するため `license: Proprietary`。個人利用の範囲で使用のこと。

### notion-upload-image

公開画像ホストなしで、ローカル/生成画像を Notion ページに表示するスキル。

**できること**
- 検証済みの転送経路：`PNG等 → JPEG Quality 80 → Base64 → SVG ラッパー → Notion attachment → ページに画像表示`。MCP ペイロードを UTF-8 テキストに保ったまま Notion 側でラスタ画像として表示される
- `scripts/prepare_notion_svg.py` が透過の白背景化・アスペクト比維持・**SVG 全体が Notion の 200 KiB 制限に収まるまで**の自動縮小を行い、`payload_safe: true` を確認してからアップロード
- GitHub 等への公開ホスティングを行わないため、非公開画像も安全に扱える

**使い方**：「この画像を Notion に貼って」「サムネを Notion に載せて」。アップロード後1時間以内に `markdown_source` をページへ挿入する制約があるため、挿入まで一気に実行される。

---

## メモ

- PubMed 系スキルのツール名プレフィックス（`mcp__pubmed__*`）は MCP サーバーの登録名に合わせて読み替える（`mcp__pubmed-remote__` 等になっている環境では frontmatter の `allowed-tools` も合わせる）。
- `ai-journal-watch` / `ai-news-digest` の「関心領域」（🔍 で優先表示するトピック）は既定で消化器・内視鏡向け。SKILL.md 冒頭の設定を書き換えれば他領域にも使える。
- `notion-upload-image` / `paper-summarize-to-notion` / `paper-to-one-slide-ja` / `pubmed-eutils-search` は `agents/openai.yaml` を同梱しており、ChatGPT / Codex 側のエージェント設定にも対応。
