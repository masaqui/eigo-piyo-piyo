---
name: eigo-piyo-piyo
description: 英語発音ガイドスキル。英文を入力すると、ネイティブのアメリカ英語カジュアル発音を解析し、単語ごとにIPA発音記号・カタカナ読み・ストレス・連結音声現象（リンキング、リダクション、フラッピング、アシミレーション、Dark L）を付与してHTMLで表示する。「英語 発音」「カタカナ読み」「IPA」「発音記号」「リンキング」「英文を読む」「pronounce」「pronunciation」「how to say」などのキーワードで発動。
---

# EigoPiyoPiyo - 英語発音ガイド

## ワークフロー

1. 英文を受け取る
2. 以下の解析ルールで各単語／グループを分析
3. `OUTPUT_FORMAT.md` を読んでHTMLを生成
4. `/mnt/user-data/outputs/` に保存して `present_files` で返す

---

## カード構成

- 1行 = 1カード（sentence-card）
- 1行に複数文（`.` `!` `?` が複数）→ 文ごとに ruby-line + ipa-line を分ける
- 日本語訳はカード内で1つにまとめる

---

## グループとは

連結発音や韻律的まとまりで一緒に発音される単語のまとまり。

| グループ化の理由 | 例 |
|---|---|
| リンキング | "have a" → 1グループ「ハヴァ」 |
| アシミレーション | "did you" → 1グループ「ディジュ」 |
| リズムグルーピング（弱形同士） | "for your" → ファヤ、"to the" → トゥダ |

⚠️ リズムグルーピングは**弱形同士のみ**。弱形＋ストレス語は対象外。

---

## 解析ルール

### 🔗 リンキング（linking）

語末の**子音** + 次の語頭の**母音**が繋がるとき。**文字ではなく音（IPA）で判定。**

```
✅ 語末子音 → 語頭母音
❌ 子音→子音 / 母音→母音 / 母音→子音
```

**✅ 正しい例**
- `have a` /v/+/ə/ → ハヴァ
- `have any` /v/+/æ/ → ハヴァニー
- `milk and sugar` → mil🔗k 🔗an🔵d → ミルカン（andの語末dは次子音前でリダクション）
- `bring it in` → brin🔗g 🔗i🟢t 🔗in → ブリンギレン（リンキング連鎖、1グループ）

**❌ よくある誤適用**
- `sounds nice/good/great`（/z/+子音）→ リンキングしない
- `are/see/help/can/give/show/when you`（/r/ /p/ /n/ /v/ など + /j/）→ アシミレーションもしない。`you` は弱形「ヤ」

---

### 🟢 フラッピング（flapping）

母音（または /ɚ/ /ɹ/）に**挟まれた** t/d → /ɾ/（ラ行）

**判定手順（必ずこの順序）:**
1. t/d の直前の**音（IPA）**が母音か確認
2. 直前が子音（/s/ /n/ /l/ など）→ **即 normal**
3. 直前が母音 かつ 直後も母音 → フラッピング

**✅ する例:** water, better, city, liberty, little（ɪ+tt+音節的l̩）, getting

**❌ しない例:**
- /st/ クラスタ（station, street）、/nt/ クラスタ（twenty→リダクション）
- 語頭の t（take, time）、th（anything, weather）→ /θ/,/ð/ は対象外

**語またぎフラッピング**（リンキングと複合。語末t/dの直前が母音 かつ 次の語が母音始まり）:

| 例 | カタカナ | マーク |
|---|---|---|
| made a | メイダ | de=flapping, a=linking |
| but I / but a | バライ / バラ | t=flapping, 次語頭=linking |
| bit of | ビラヴ | t=flapping, o=linking |

---

### 🔵 リダクション（reduction）

カジュアル発話で脱落・ほぼ聞こえなくなる音。完全脱落も未開放も統一してマーク。

| パターン | 例 → カタカナ |
|---|---|
| silent h/t | honest→アネス、often→オーフン |
| g-dropping（-ing末尾） | building→ビルディン（「ン」のときのみ） |
| 文末 t/d | great!→グレイ、Alright.→オーライ |
| **-ight 語末 t** | night→ナイ、right→ライ、tonight→トゥナイ |
| 子音前 t/d | Not so→ナッソウ、That would→ダウッ |
| 閉鎖音+閉鎖音 | would be→ウッビ、football→フッボール |
| **Thank you の k** | Thank you → サンキュー（k+y→キュ、**1グループ**） |
| /nt/ の t | twenty→トゥウェンニー、center→センナー |
| and の語末 d | read and watch→リーダンウォッチ |

**❌ リダクションしない:**
- `so`（副詞・接続詞）→ 意味のある内容語
- `good morning` の d（/d/+/m/ は閉鎖音+鼻音で吸収しない）

⚠️ **カタカナに脱落反映済みなのに英語テキスト側のマーク忘れが最多ミス。必ず両面チェック。**

---

### 🔴 アシミレーション（assimilation）

**前の語の語末音が /d/, /t/, /z/, /s/ のときのみ**発生。

| 語末音 | 変化 | 例 |
|---|---|---|
| /d/ + /j/ → /dʒ/ | ジュ | do you, would you, did you |
| /t/ + /j/ → /tʃ/ | チュ | don't you, about you, meet you |
| /z/ + /j/ → /ʒ/ | ジュア | is your, does your |
| /s/ + /j/ → /ʃ/ | シュ | miss you, this year |

カジュアル場面で起きやすく、丁寧・ゆっくりでは別々に発音される。迷ったらカジュアル寄り。

**❌ アシミレーションが起きない（/d/,/t/,/z/,/s/ 以外の語末音）:**
- /r/（are you）、/p/（help you）、/n/（can you, when you）、/v/（give you）、/θ/（with you）
- `-thing` 系の th（anything, something）→ /θ/ はカタカナ「ス行」のみ

---

### 🌑 ダークL（dark_l）

語末・音節末の L → 暗いオ/ウの音。例: people→ピーポー、little→リロー、all→オール

---

### スラッシュ（slash reading）

意味のまとまりで区切り、前から理解を支援。`<span class="slash"></span>` で表現。

#### 必須トリガー（これが出たら必ずスラッシュを入れる）

| トリガー | スラッシュ位置 | 例 |
|---|---|---|
| `,` / `;` / `:` | 句読点の直後 | "No milk, / but..." |
| `to + 動詞原形`（不定詞） | `to` の直前 | "I'd like / to have / some..." |
| 前置詞句（for/into/at/on/in/with/from/by + 名詞句） | 前置詞の直前 | "my money / into dollars" |
| 接続詞（and/but/or/so/because/when/if/that） | 接続詞の直前 | "turn right / and then left" |
| 関係詞（who/which/that） | 関係詞の直前 | "The man / who lives next door" |

#### 長さルール（補助）

- **5語以上の文**：必ず1か所以上スラッシュを入れる
- **8語以上の文**：必ず2か所以上を目標にする
- 上記トリガーを全て適用した結果、チャンクが3語以下になるのが理想

#### ❌ やってはいけないこと

- 主語＋動詞の間でスラッシュ（"I / like" は誤り）
- `to + 動詞` の間でスラッシュ（"to / go" は誤り）
- 2語以下の文にスラッシュ（不要）

---

### ストレス（stress）

- `stressed`：内容語（名詞・動詞・形容詞・副詞）の中心語、数量・対比・新情報の焦点強勢
- `weak`：機能語（冠詞・前置詞・助動詞・弱化代名詞）
- `**太字**` マークされた語 → 必ず stressed

---

### アクセント（音節強勢）

**2音節以上の内容語は全て** `<span class="accent">` でアクセント音節を囲む。単音節語・弱化機能語には付けない。

⚠️ **アクセントマーク付け忘れが最多ミスの一つ。必ずIPA先行方式で決定すること。**

---

#### IPA先行方式（唯一の正しいアプローチ）

**手順：**
1. **IPA を先に書く**（カードのipa-lineに記入）
2. **`ˈ` の位置を確認**する
3. **`ˈ` の直後の音素**が属する音節 = アクセント音節
4. その音節に対応する**英語スペル**と**カタカナ**に `<span class="accent">` を付ける

```
IPA: /ˈsɪɡnətʃər/  → ˈ直後=sɪɡ → "sig" と "スィグ" にaccentを付ける
IPA: /ˈkɜːrənsi/   → ˈ直後=kɜːr → "cur" と "カ" にaccentを付ける
IPA: /təˈmɑroʊ/    → ˈ直後=mɑr → "mor" と "モ" にaccentを付ける
IPA: /ɪkˈspɛnsɪv/  → ˈ直後=spɛn → "pen" と "ペン" にaccentを付ける
IPA: /ˌæbsəˈluːtli/→ ˈ直後=luːt → "lu" と "リュー" にaccentを付ける
```

**ルールや暗記は不要。IPAが答えを持っている。**

---

#### 参考：アクセント位置の傾向（IPAが不明なときの補助）

| パターン | 傾向 | 例（IPAで要確認） |
|---|---|---|
| 未強勢接頭辞（ex-/in-/re-/to-/be-/de-/un-） | 接頭辞の後に強勢 | expensive, instead, today |
| -tion / -sion | 直前の音節に強勢 | vacation, reservation |
| -ing（動名詞・分詞） | 第1音節 | watching, calling |
| 2音節名詞（接頭辞なし） | 第1音節が多数派 | sugar, coffee, currency |

⚠️ これらはあくまで補助。**最終判断は必ずIPAで確認する。**

⚠️ **アメリカ英語の /ɑ/ は「ア」**（conference /ˈkɑnfrəns/ → **カン**、honest → **ア**ネス）

---

### カタカナ（kana）ルール

- 連結グループは1つのカタカナ文字列で
- 文末の句読点をカタカナに含める
- Yes/No疑問文末 → `↑`、WH疑問文 → 付けない

**主な音変化:**

| 音 | カタカナ | 注意 |
|---|---|---|
| /ð/ | **ダ行**（ザ行不可） | the→ダ、that→ダッ、there→デア、this→ディス |
| /θ/ | ス行 | thank→サンク、thing→スィング |
| /ɑ/ | ア（オではない） | conference→カン、honest→ア |
| フラッピング t/d | ラ行 | water→ウォーラー、but I→バライ |
| Dark L | 語末オ/ウ | people→ピーポー |
| 弱化 | to→トゥ、for→ファ、can→クン、the→ダ、you→ヤ | to（トゥ）とthe（ダ）を混同しない |
| 縮約 | want to→ワナ、going to→ゴナ | |

---

## ⚠️ セルフチェックリスト

### 音声現象チェック
- [ ] **リダクション両面**: カタカナに脱落反映 → 英語テキスト側にも `char-reduction` を付けたか？
- [ ] **文末t/d**: great!, Alright., -ight系（night/right/tonight）の語末 `t` をマークしたか？
- [ ] **閉鎖音連続**: would be の d、football の t など前の閉鎖音をマークしたか？
- [ ] **Thank you**: 1グループ「サンキュー」か？（「サンク＋クユー」は誤り）
- [ ] **語またぎフラッピング**: made a / but I / bit of など見落としていないか？
- [ ] **/nt/ クラスタ**: twenty/center の t はリダクション（フラッピングではない）

### マーク誤適用チェック
- [ ] **リンキング誤適用**: sounds nice など子音+子音にマークしていないか？
- [ ] **アシミレーション誤適用**: are/see/help/can/give you に as_('y') を使っていないか？
- [ ] **th**: フラッピング・アシミレーション対象外（anything の th はス行のみ）

### HTML構造チェック
- [ ] **ruby間スペース**: `</ruby> <ruby>` の間に半角スペースがあるか？（ないと単語がくっつく）
- [ ] **アクセント**: 2音節以上の全内容語に `<span class="accent">` があるか？ 位置はIPAの `ˈ` 直後の音節で決定する（ルールや勘で決めない）。
- [ ] **スラッシュ**: 副詞＋前置詞句パターン（tonight / for...）を見落としていないか？
- [ ] **/ð/ カタカナ**: ダ行か？（ザ行になっていないか）
