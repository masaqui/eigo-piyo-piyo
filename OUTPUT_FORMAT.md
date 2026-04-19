# EigoPiyoPiyo - HTML出力フォーマット

## デザインコンセプト

英文を主役として大きく表示し、カタカナ読みを `<ruby>/<rt>` でふりがなのように振る。
IPAは英文の下に小さく1行で表示。単一の自己完結型HTMLファイル（CSS・フォント込み）を生成。

---

## ruby要素の構成

各グループを1つの `<ruby>` にする。**隣接するruby要素の間には必ず半角スペースを入れること。**

```html
<!-- ✅ 正しい -->
<ruby>my<rt>マイ</rt></ruby> <ruby>name<rt>ネイム</rt></ruby>
<!-- ❌ 誤り（単語がくっつく） -->
<ruby>my<rt>マイ</rt></ruby><ruby>name<rt>ネイム</rt></ruby>
```

### 音声現象のspan（英語テキスト内）

| 現象 | span class |
|---|---|
| リンキング | `<span class="char-linking">` |
| リダクション | `<span class="char-reduction">` |
| アシミレーション | `<span class="char-assimilation">` |
| フラッピング | `<span class="char-flapping">` |
| ダークL | `<span class="char-dark-l">` |

### rt要素のclass

| stress値 | rtのclass |
|---|---|
| stressed | `rt-stressed`（大きく濃い） |
| weak | `rt-weak`（小さく薄い） |
| normal | classなし |

### アクセント・スラッシュ

```html
<!-- アクセント（英語側・カタカナ側どちらにも付ける） -->
<ruby><span class="accent">ser</span>vice<rt class="rt-stressed"><span class="accent">サーヴ</span>ィス</rt></ruby>

<!-- スラッシュ（rubyグループの間に置く） -->
<span class="slash"></span>
```

### 出力例

入力: "To be honest, I'd like a sandwich."

```html
<div class="sentence-card">
  <div class="ruby-line">
    <ruby>To<rt class="rt-weak">トゥ</rt></ruby>
    <ruby>be<rt class="rt-weak">ビ</rt></ruby>
    <ruby><span class="char-reduction">h</span><span class="accent">on</span>es<span class="char-reduction">t</span>,<rt class="rt-stressed"><span class="accent">ア</span>ネス、</rt></ruby>
    <span class="slash"></span>
    <ruby>I'<span class="char-reduction">d</span><rt class="rt-weak">アイ</rt></ruby>
    <ruby>like<rt class="rt-stressed">ライク</rt></ruby>
    <ruby>a<rt class="rt-weak">ア</rt></ruby>
    <ruby><span class="accent">san</span><span class="char-reduction">d</span>wich.<rt class="rt-stressed"><span class="accent">サン</span>ウィッチ。</rt></ruby>
  </div>
  <div class="ipa-line">/tə bi ˈɑːnɪst aɪ ˈlaɪk ə ˈsænwɪtʃ/</div>
  <div class="translation">正直に言うと、サンドイッチが食べたいです。</div>
</div>
```

---

## HTMLテンプレート

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>EigoPiyoPiyo - レッスンタイトル</title>
  <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=M+PLUS+Rounded+1c:wght@400;700;800;900&family=Nunito:wght@700;800;900&display=swap">
  <style>/* CSSを埋め込む */</style>
</head>
<body>
  <div class="container">
    <h1>🐣 EigoPiyoPiyo</h1>
    <p class="subtitle">英語発音ガイド</p>
    <div class="toggle-bar">
      <button id="kana-toggle" class="toggle-btn" onclick="toggleKana()">🐣 カタカナ ON</button>
    </div>
    <div class="lesson-title-wrap"><span class="lesson-title">レッスンタイトル</span></div>

    <!-- 各文のカード。1行=1カード。1行に複数文がある場合は文ごとにruby-line+ipa-lineを分ける -->
    <div class="sentence-card">
      <div class="speaker">A</div>
      <div class="ruby-line">...</div>
      <div class="ipa-line">/IPA/</div>
      <div class="translation">日本語訳</div>
    </div>

    <footer class="footer">
      <div class="legend">
        <div class="legend-item"><span class="dot-linking">●</span><span class="legend-label">リンキング</span><span class="legend-desc">単語がつながる</span></div>
        <div class="legend-item"><span class="dot-assimilation">●</span><span class="legend-label">アシミレーション</span><span class="legend-desc">音が混ざって変化</span></div>
        <div class="legend-item"><span class="dot-reduction">●</span><span class="legend-label">リダクション</span><span class="legend-desc">発音しない・消える</span></div>
        <div class="legend-item"><span class="dot-flapping">●</span><span class="legend-label">フラッピング</span><span class="legend-desc">t/d がラ行に変化</span></div>
        <div class="legend-item"><span class="dot-dark-l">●</span><span class="legend-label">ダークL</span><span class="legend-desc">L がオ/ウの音に</span></div>
        <div class="legend-item"><span class="legend-slash">/</span><span class="legend-label">スラッシュ</span><span class="legend-desc">意味の区切り・息継ぎ</span></div>
      </div>
      <div class="github-link">
        <a href="https://github.com/masaqui/eigo-piyo-piyo" target="_blank" rel="noopener">🐣 EigoPiyoPiyo on GitHub</a>
      </div>
    </footer>
  </div>
  <script>
  function toggleKana() {
    const container = document.querySelector('.container');
    const btn = document.getElementById('kana-toggle');
    container.classList.toggle('hide-kana');
    btn.textContent = container.classList.contains('hide-kana') ? '🙈 カタカナ OFF' : '🐣 カタカナ ON';
    btn.classList.toggle('off', container.classList.contains('hide-kana'));
  }
  </script>
</body>
</html>
```

---

## CSS

```css
* { margin: 0; padding: 0; box-sizing: border-box; }
body { font-family: 'M PLUS Rounded 1c','Nunito',-apple-system,BlinkMacSystemFont,'Hiragino Sans',sans-serif; background: #ffde59; color: #333; min-height: 100vh; }
.container { width: 100%; padding: 32px clamp(16px,4vw,80px); }
h1 { font-family: 'Nunito','M PLUS Rounded 1c',sans-serif; font-size: 2.2rem; font-weight: 900; text-align: center; margin-bottom: 6px; color: #333; }
.subtitle { text-align: center; color: #555; margin-bottom: 24px; font-size: 0.9rem; font-weight: 700; }
.lesson-title { font-family: 'M PLUS Rounded 1c',sans-serif; font-size: 1.3rem; font-weight: 900; background: #fff; border-radius: 16px; padding: 12px 24px; border: 3px solid #333; display: inline-block; }
.lesson-title-wrap { text-align: center; margin-bottom: 24px; }
.sentence-card { background: #fff; border-radius: 20px; padding: 28px; margin-bottom: 16px; border: 4px solid #333; border-bottom-width: 6px; }
.speaker { font-size: 0.75rem; font-weight: 900; color: #aaa; margin-bottom: 4px; letter-spacing: 0.08em; text-transform: uppercase; }
.ruby-line { font-family: 'Nunito','M PLUS Rounded 1c',sans-serif; font-size: clamp(1.8rem,3.2vw,2.8rem); font-weight: 900; color: #333; line-height: 3.2; }
ruby { ruby-align: center; }
rt { font-family: 'M PLUS Rounded 1c',sans-serif; font-size: 0.42em; font-weight: 800; color: #666; ruby-align: center; }
rt.rt-stressed { font-size: 0.48em; font-weight: 900; color: #333; }
rt.rt-weak { font-size: 0.34em; font-weight: 700; color: #aaa; }
rt .accent { font-size: 1.5em; font-weight: 900; color: #222; }
.char-linking { color: #ff922b; }
.char-reduction { color: #339af0; text-decoration: line-through; text-decoration-thickness: 2.5px; }
.char-assimilation { color: #ff6b6b; }
.char-flapping { color: #51cf66; }
.char-dark-l { color: #2b8a3e; }
.slash { display: inline-block; width: 1em; text-align: center; vertical-align: middle; margin: 0 0.1em; color: #ccc; font-size: 0.8em; font-weight: 900; }
.slash::before { content: "/"; }
.ipa-line { font-family: 'Lucida Grande','Arial Unicode MS','Gentium Plus',serif; font-size: 0.85rem; color: #bbb; line-height: 1.4; margin-top: 2px; }
.translation { font-size: 0.95rem; font-weight: 700; color: #555; line-height: 1.5; margin-top: 10px; padding-top: 10px; border-top: 3px solid #eee; }
.footer { text-align: center; margin-top: 28px; padding: 16px 0; }
.legend { display: grid; grid-template-columns: repeat(auto-fill,minmax(220px,1fr)); gap: 6px; max-width: 720px; margin: 0 auto; font-size: 0.78rem; font-weight: 800; color: #555; }
.legend-item { display: flex; align-items: center; gap: 6px; background: #fff; padding: 6px 12px; border-radius: 12px; border: 2px solid #333; }
.legend-label { white-space: nowrap; }
.legend-desc { font-weight: 600; color: #999; font-size: 0.7rem; white-space: nowrap; }
.dot-linking{color:#ff922b} .dot-assimilation{color:#ff6b6b} .dot-reduction{color:#339af0} .dot-flapping{color:#51cf66} .dot-dark-l{color:#2b8a3e} .legend-slash{color:#ccc;font-weight:900;font-size:1.1em}
.github-link { margin-top: 12px; font-size: 0.75rem; font-weight: 700; }
.github-link a { color: #888; text-decoration: none; }
.github-link a:hover { color: #333; text-decoration: underline; }
.toggle-bar { text-align: center; margin-bottom: 20px; position: sticky; top: 12px; z-index: 100; }
.toggle-btn { font-family: 'M PLUS Rounded 1c',sans-serif; font-size: 0.95rem; font-weight: 900; color: #333; background: #fff; border: 3px solid #333; border-bottom-width: 5px; border-radius: 14px; padding: 8px 24px; cursor: pointer; transition: all 0.15s; box-shadow: 0 2px 8px rgba(0,0,0,.08); }
.toggle-btn:hover { background: #fffbe6; }
.toggle-btn:active { border-bottom-width: 3px; transform: translateY(2px); }
.toggle-btn.off { background: #f1f3f5; color: #888; border-color: #aaa; }
.hide-kana rt { opacity: 0; pointer-events: none; }
@media (max-width:600px) { .container{padding:20px 12px} h1{font-size:1.8rem} .sentence-card{padding:20px 16px} .ruby-line{font-size:1.6rem;line-height:3.0} .ipa-line{font-size:0.75rem} }
```
