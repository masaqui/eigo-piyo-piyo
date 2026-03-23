# 🐣 EigoPiyoPiyo - 英語発音ガイドスキル

英文を入力すると、ネイティブのアメリカ英語カジュアル発音を解析し、単語ごとにIPA発音記号・カタカナ読み・ストレス・連結音声現象を付与してHTMLで表示する [Claude Skills](https://support.anthropic.com/en/articles/claude-ai-skills) です。

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

### Claude.ai Skills として使う

1. このリポジトリをZIPでダウンロード（または `SKILL.md` を単体でダウンロード）
2. [Claude.ai](https://claude.ai) の設定 → Skills からアップロード

### Claude Code で使う

```bash
# リポジトリをクローン
git clone https://github.com/masaqui/eigo-piyo-piyo.git

# Claude Code のスキルディレクトリに配置
# 例: プロジェクトの .claude/skills/ 配下にコピー
cp -r eigo-piyo-piyo /path/to/your-project/.claude/skills/
```

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
