# 🐣 EigoPiyoPiyo - 英語発音ガイドスキル

英文を入力すると、ネイティブのアメリカ英語カジュアル発音を解析し、単語ごとにIPA発音記号・カタカナ読み・ストレス・連結音声現象を付与してHTMLで表示する [Claude Skills](https://support.anthropic.com/en/articles/claude-ai-skills) です。

<img width="956" height="984" alt="Image" src="https://github.com/user-attachments/assets/e1cd73ce-a303-4b2a-9f96-205e4005a3a3" />

## Claude Skills とは？

[Claude](https://claude.ai) は Anthropic が開発した AI アシスタントです。**Claude Skills** は、Claudeに特定のタスクのやり方を教える拡張機能のようなもの。`SKILL.md` というファイルに手順やルールを書いておくと、Claudeが会話の中で自動的にそのスキルを使ってくれます。

たとえば「この英文の発音を教えて」と聞くだけで、このEigoPiyoPiyoスキルが自動的に発動し、音声現象の解析からHTML生成まで一貫して行います。プラグインのような感覚で、誰でも自分のスキルを作って追加できます。

詳しくは [Claude Skills 公式ドキュメント](https://support.claude.com/en/articles/claude-ai-skills) をご覧ください。

## 特徴

- **ふりがな方式**: 英文の上にカタカナ読みを `<ruby>` タグで表示
- **音声現象の可視化**: リンキング・リダクション・フラッピング・アシミレーション・Dark L を色分け表示
- **IPA発音記号**: アメリカ英語のIPA表記を併記
- **アクセント表示**: 第一強勢の音節を強調表示
- **自然な日本語訳**: 会話調の日本語訳を付与

## 対応する音声現象

| 現象 | 色 | 例 |
|------|-----|-----|
| リンキング | 🟠 オレンジ | eating outside → イーリンガウサイ |
| アシミレーション | 🔴 レッド | did you → ディジュ |
| リダクション | 🔵 ブルー (打消し線) | honest → ~~h~~ones~~t~~ |
| フラッピング | 🟢 グリーン | water → ウォーラー |
| Dark L | 🟢 ティール | people → ピーポー |

## インストール

`SKILL.md` のフォーマットはオープンスタンダードで、Claude のどの製品でも共通です。

### Claude.ai / Claudeアプリ（Mac・iOS・Android）

Web版（claude.ai）とデスクトップ・モバイルアプリは同じアカウント・同じSkillsを共有しています。

1. このリポジトリをZIPでダウンロード（または `SKILL.md` を単体でダウンロード）
2. 設定 → カスタマイズ → Skills → 「+」ボタン → 「Upload a skill」からアップロード

スキルは自動的に認識され、関連する質問をすると自動で発動します。

### Claude Code（ターミナルCLI）

```bash
# プロジェクトのスキルディレクトリに配置（プロジェクト固有）
git clone https://github.com/masaqui/eigo-piyo-piyo.git .claude/skills/eigo-piyo-piyo

# または個人のグローバルスキルとして配置（全プロジェクト共通）
git clone https://github.com/masaqui/eigo-piyo-piyo.git ~/.claude/skills/eigo-piyo-piyo
```

Claude Code ではスキルが自動検出されるほか、`/eigo-piyo-piyo` のようにスラッシュコマンドとしても呼び出せます。

### Claude API

Skills APIを使ってプログラムからアップロード・利用できます。詳しくは [Skills APIドキュメント](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) を参照してください。

## 使い方

Claude に英文を送り、「発音」「読み方」「カタカナ」などのキーワードを添えるだけです。

**例:**
```
"I'd like to grab a cup of coffee." の発音を教えて
```

出力はHTMLファイルとして生成され、ブラウザで閲覧できます。

## 出力イメージ

英文の上にふりがなのようにカタカナが振られ、音声現象ごとに色分けされた見やすいHTMLが生成されます。

## ファイル構成

```
eigo-piyo-piyo/
├── README.md       # このファイル
├── SKILL.md        # スキル本体（解析ルール・HTMLテンプレート込み）
└── LICENSE         # ライセンス
```

## トリガーキーワード

「英語 発音」「カタカナ読み」「IPA」「発音記号」「リンキング」「英文を読む」「pronounce」「pronunciation」「how to say」

## ライセンス

MIT License
