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

### ⚠️ 1単語 = 1つのruby（絶対ルール）

**1つの単語は必ず1つの `<ruby>` 要素にまとめる。スペルの中にスペースがなければ分割しない。**

```html
<!-- ❌ 誤り：1単語を分割している -->
<ruby>no<rt>ノウ</rt></ruby> <ruby>thing<rt>スィング</rt></ruby>
<ruby>some<rt>サム</rt></ruby> <ruby>thing<rt>スィング</rt></ruby>
<ruby>every<rt>エヴリ</rt></ruby> <ruby>one<rt>ワン</rt></ruby>

<!-- ✅ 正しい：1単語1ruby -->
<ruby>nothing<rt class="rt-stressed">ナスィング</rt></ruby>
<ruby>something<rt class="rt-stressed">サムスィング</rt></ruby>
<ruby>everyone<rt class="rt-stressed">エヴリワン</rt></ruby>
```

特に紛らわしい1単語: nothing / something / anything / everything / everyone / someone / anyone / myself / yourself / because / although / without / into / onto

---

## 解析ルール

### 🔗 リンキング（linking）

語末の**子音** + 次の語頭の**母音**が繋がるとき。**文字ではなく音（IPA）で判定。**

```
✅ 語末子音 → 語頭母音
❌ 子音→子音 / 母音→母音 / 母音→子音
```

**✅ 正しい例**
- `have a` /v/+/ə/ → ハヴァ（1グループ）
- `have any` /v/+/æ/ → ハヴァニー
- `when I` /n/+/aɪ/ → ウェナイ（1グループ）
- `an apple` /n/+/æ/ → アナポー（1グループ）
- `in a` /n/+/ə/ → イナ（1グループ。「イン ア」と分けない）
- `in our` /n/+/aʊ/ → イナワー（1グループ。「イン アワー」と分けない）
- `them up` /m/+/ʌ/ → ゼマップ（1グループ。「デム アップ」と分けない）
- `you an` /j/+/æ/ → ユアン（1グループ。「ヤ アン」と分けない）
- `make it` /k/+/ɪ/ → メイキット（1グループ。「メイク イッ」と分けない）
- `milk and sugar` → mil🔗k 🔗an🔵d → ミルカン（andの語末dは次子音前でリダクション）
- `bring it in` → brin🔗g 🔗i🟢t 🔗in → ブリンギレン（リンキング連鎖、1グループ）

**❌ よくある誤適用**
- `sounds nice/good/great`（/z/+子音）→ リンキングしない
- `show up`（/oʊ/+/ʌ/）、`go out`（/oʊ/+/aʊ/）など**母音終わり→母音始まり** → リンキングしない。「つながって聞こえる」＝リンキングではない。**語末が母音のときはリンキング対象外。**
- `are/see/help/can/give/show/when you`（/r/ /p/ /n/ /v/ など + /j/）→ アシミレーションもしない。`you` は弱形「ヤ」
- `asked "Do`（/skt/+/d/）、`next day`（/kst/+/d/）、`facts change`（/kts/+/tʃ/）など**子音クラスターで終わる語 + 次が子音始まり** → リンキングしない。`-ed`・`-st`・`-xt`・`-nk`・`-nd` 等で終わる語は語末音がどの子音かをIPAで確認してから判定する。**スペル上の最後の文字が母音字（e 等）でも、発音上の語末音が子音ならリンキングしない。**

---

### 🟢 フラッピング（flapping）

母音（または /ɚ/ /ɹ/）に**挟まれた** t/d → /ɾ/（ラ行）

**判定手順（必ずこの順序）:**
1. t/d の直前の**音（IPA）**が母音か確認
2. 直前が子音（/s/ /n/ /l/ など）→ **即 normal**
3. 直前が母音 かつ 直後も母音 → フラッピング

**✅ する例:** water, better, city, liberty, little（ɪ+tt+音節的l̩）, getting

**⚠️ スペルに引っ張られて見落としやすい語内フラッピング（IPA先行で必ず確認）:**

| 単語 | IPA | カタカナ | 誤りやすいカタカナ |
|---|---|---|---|
| getting | /ˈɡɛɾɪŋ/ | ゲ**リ**ング | ~~ゲティング~~ |
| putting | /ˈpʊɾɪŋ/ | プ**リ**ング | ~~プティング~~ |
| important | /ɪmˈpɔɾənt/ | インポー**ラ**ント | ~~インポータント~~ |
| interesting | /ˈɪnɾərɛstɪŋ/ | イン**ラ**レスティング | ~~インタレスティング~~ |
| totally | /ˈtoʊɾəli/ | トー**ラ**リー | ~~トータリー~~ |
| lately | /ˈleɪɾli/ | レイ**リ**ー | ~~レイトリー~~ |
| Italy | /ˈɪɾəli/ | イ**ラ**リー | ~~イタリー~~ |
| pretty | /ˈpɹɪɾi/ | プ**リ**リー | ~~プリティ~~ |
| butter | /ˈbʌɾər/ | バ**ラ**ー | ~~バター~~ |
| letter | /ˈlɛɾər/ | レ**ラ**ー | ~~レター~~ |

**判定の鉄則：スペルの `t` を見たら必ずその直前・直後の音（IPA）を確認してからカタカナを決める。**

**❌ しない例:**
- /st/ クラスタ（station, street）、/nt/ クラスタ（twenty→リダクション）
- **nt+l / nt+r**（currently, gently, winter）→ フラッピングではなくt脱落（リダクション）として扱う
- 語頭の t（take, time）、th（anything, weather）→ /θ/,/ð/ は対象外

**語またぎフラッピング**（リンキングと複合。語末t/dの直前が母音 かつ 次の語が母音始まり）:

| 例 | カタカナ | マーク |
|---|---|---|
| made a | メイダ | de=flapping, a=linking |
| but I / but a | バライ / バラ | t=flapping, 次語頭=linking |
| at a | アッラ | t=flapping（直前が母音の場合。例: work at a→ワーカッラ） |
| bit of | ビラヴ | t=flapping, o=linking |
| ate it | エイリッ | t=flapping（ate末尾t + it頭母音）|
| first ate it | ファーストエイリッ | first末t→linking、ate末t→flapping |
| but it's | バリッツ | t=flapping（but末尾t + it's頭母音）|
| that our | ダナー | that末t→フラッピング（/æt/+/aʊ/）。`that` は機能語で直前が母音 /æ/ なので常にフラッピング対象 |

⚠️ **`at` のような機能語は単語内部の母音だけでも判定できる**：`at` /æt/ は母音æ+t、次が母音始まりの語（a, a gallery等）なら、直前の単語の語末音に関わらず常にフラッピングする（"work at a" も "sit at a" も同じ）。

---

### 🔵 リダクション（reduction）

カジュアル発話で脱落・ほぼ聞こえなくなる音。完全脱落も未開放も統一してマーク。

| パターン | 例 → カタカナ |
|---|---|
| silent h/t | honest→アネス、often→オーフ~~t~~ン（`of`ではなく`t`に`char-reduction`を付ける） |
| g-dropping（-ing末尾） | building→ビルディン（「ン」のときのみ） |
| 文末 t/d | great!→グレイ、Alright.→オーライ |
| **-ight 語末 t** | night→ナイ、right→ライ、tonight→トゥナイ |
| 子音前 t/d | Not so→ナッソウ、That would→ダウッ |
| 閉鎖音+閉鎖音 | would be→ウッビ、football→フッボール |
| **Thank you の k** | Thank you → サンキュー（k+y→キュ、**1グループ**） |
| /nt/ の t | twenty→トゥウェンニー、center→センナー |
| **nt+l / nt+r クラスター t脱落** | currently→カレンリー、gently→ジェンリー、winter→ウィナー、internet→イナーネッ |
| and の語末 d | read and watch→リーダンウォッチ |
| **過去形 `-ed` が /t/ で終わる語の語末 t** | asked /æskt/→アスクッ、talked /tɔːkt/→トークッ、helped /hɛlpt/→ヘォップ など。スペルは `d` でも発音は `/t/`。子音前・文末では `char-reduction` をその `d` に付ける |

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

### 🔊 tr/dr クラスターの破擦音化（affrication）

米語カジュアル発音では、語頭・アクセント音節頭の `tr` `dr` は破擦音寄りになる。

| スペル | 変化 | 例 |
|---|---|---|
| `tr` | トゥ ではなく **チュ** 寄り | training→**チュ**レイニング、try→**チュ**ライ、truck→**チュ**ラック |
| `dr` | ドゥ ではなく **ジュ** 寄り | dragon→**ジュ**ラゴン、drive→**ジュ**ライヴ |

⚠️ 単語頭の `tr`/`dr` を見たら「トレ」「ドレ」ではなく「チュレ」「ジュレ」寄りのカタカナを検討する。

---

### 🧊 同一子音の衝突（geminate simplification）

語末の子音と次語の語頭子音が**同じ調音点の破裂音**（t+t, d+d, t+d, d+t など）のとき、前の音は破裂せず片方だけが聞こえる（実質1つの子音に短縮）。

| 例 | 現象 | カタカナ |
|---|---|---|
| felt tired（t + t） | 前のtは未開放でほぼ聞こえない | フォゥタイアー（feltの語末tを実質脱落として扱う） |
| that time（t + t） | 同様 | ダッタイム |
| good dog（d + d） | 同様 | グッドッグ |

**マーク方法：** 前の語の語末子音に `char-reduction` を付け、カタカナからも脱落させる。

---

### 🌀 縮約語のさらなる脱落（casual contraction collapse）

`-n't` の縮約語は、カジュアル発話で内部の子音や母音がさらに脱落し、音節数自体が減ることがある。**IPA先行で判断し、スペル通りの音節数を仮定しない。**

| 単語 | 標準的カタカナ | さらに脱落したカジュアル形 |
|---|---|---|
| didn't | ディドゥン | **ディン**（中間のdとschwaが脱落し、syllabic nがdi に直結） |

⚠️ 迷ったらまずは標準形で解析し、音声から聞き取れる場合はよりカジュアルな脱落形を優先する。

---

### 🔗 鼻音＋母音のリンキング（nasal linking）

リンキングは語末**子音**全般で起こりうるが、特に語末の鼻音（/ŋ/, /n/, /m/）は次語頭の母音と非常に強く連結し、鼻音がそのまま次の音節の頭子音であるかのように聞こえる。

| 例 | IPA | カタカナ |
|---|---|---|
| working on | /ˈwɜːrkɪŋ ɑn/ | ワー**キノン**（ing+onが1グループ） |
| turn on | /tɜːrn ɑn/ | ター**ノン** |
| come in | /kʌm ɪn/ | カ**ミン** |

マーク方法は通常のリンキングと同じ（`char-linking`、2語をまとめて1つの`<ruby>`）。

---

### ⚠️ 語末子音のリダクションは絶対ではない

`at`・`that`・`it` などの機能語末の t/d は、次が子音始まりなら基本リダクション対象だが、**発話者が意図的にはっきり発音する場合や、フレーズの区切り位置にある場合はしっかり聞こえることもある**。音源・実際の発音を優先し、機械的にリダクションを適用しない。判断に迷う場合はカジュアル形をデフォルトにしつつ、実際の聞こえ方に合わせて調整する。

---

### 🌑 ダークL（dark_l）

語末・音節末の L → 暗いオ/ウの音。例: people→ピーポー、little→リロー、all→オール

**✅ ダークLになる条件:** 語末の `l`、または子音の直前の音節末 `l`（`l` の直後が子音か語末）

**❌ ダークLにならない:** `l` の直後が母音のとき（明るいL）→ pillow /ˈpɪloʊ/、really /ˈrili/ はダークLなし

**⚠️ 見落としやすい語内ダークL（子音の前）:**

| 単語 | IPA | カタカナ | 誤りやすいカタカナ |
|---|---|---|---|
| comfortable | /ˈkʌmftərbəl/ | カンフタブ**ォ** | ~~カンフタブル~~ |
| peaceful | /ˈpisfəl/ | ピースフ**ォ** | ~~ピースフル~~ |
| healthy | /ˈhɛlθi/ | ヘ**ォ**スィー | ~~ヘルスィー~~ |
| helpful | /ˈhɛlpfəl/ | ヘ**ォ**プフォ | ~~ヘルプフル~~ |
| beautiful | /ˈbjuːɾɪfəl/ | ビュー**リ**フォ | ~~ビューティフル~~ |
| difficult | /ˈdɪfɪkəlt/ | ディフィコ**ォ**ト | ~~ディフィカルト~~ |
| already | /ɔːlˈrɛdi/ | **ォ**ーレディ | ~~オールレディ~~ |
| always | /ˈɔːlweɪz/ | **ォ**ーウェイズ | ~~オールウェイズ~~ |
| simple | /ˈsɪmpəl/ | スィンプ**ォ** | ~~スィンプル~~ |
| well | /wɛl/ | ウェ**ォ** | ~~ウェル~~ |
| call | /kɔːl/ | コー**ォ** | ~~コール~~ |
| all | /ɔːl/ | オー**ォ** | ~~オール~~ |
| people | /ˈpiːpəl/ | ピーポ**ォ** | ~~ピープル~~ |

**判定の鉄則：スペルに `l` を見たら3択で判定してからカタカナを書く。**

| 判定 | 条件 | 例 |
|---|---|---|
| **ライトL** | `l` の直後が母音 | sleep/really/pillow/listen |
| **ダークL** | `l` の直後が子音 or 語末 | simple/well/call/comfortable |
| **サイレントL** | `-alm`/`-alk`/`-ould` 等の特定パターン | calm/talk/would |

---

### 🔇 サイレントL（silent_l）

特定のスペルパターンで `l` が完全に発音されない。**`char-reduction`（青い取り消し線）で表示する。**

| スペルパターン | 例 | IPA | カタカナ |
|---|---|---|---|
| `-alm` | calm, palm, psalm | /kɑːm/, /pɑːm/ | カーム、パーム |
| `-alk` | talk, walk, chalk | /tɔːk/, /wɔːk/ | トーク、ウォーク |
| `-alf` | half, calf | /hæf/, /kæf/ | ハーフ、キャーフ |
| `-ould` | would, could, should | /wʊd/, /kʊd/, /ʃʊd/ | ウッド、クッド、シュッド |
| `-alve` | calm+ing系 | /ˈkɑːmɪŋ/ | カーミング |

**HTML表記例:**
```html
ca<span class="char-reduction">l</span>ming → カーミング
ta<span class="char-reduction">l</span>k → トーク
wou<span class="char-reduction">l</span>d → ウッド
```

**⚠️ チェックリスト追加：** スペルに `l` を見たら3択（ライトL/ダークL/サイレントL）を必ず判定してからカタカナを書く。

---

### スラッシュ（slash reading）

意味のまとまりで区切り、前から理解を支援。`<span class="slash"></span>` で表現。

#### 必須トリガー（これが出たら必ずスラッシュを入れる）

| トリガー | スラッシュ位置 | 例 |
|---|---|---|
| `,` / `;` / `:` | 句読点の直後（例外なし） | "Hello, / I'm Rory, / and today..." |
| `to + 動詞原形`（不定詞、定型チャンク外） | `to` の直前（ただし定型チャンク内は分割しない） | "I'm going / to the store" |
| 前置詞句（for/into/at/on/in/with/from/by + 名詞句） | 前置詞の直前 | "my money / into dollars" |
| 接続詞（and/but/or/so/because/when/if/that） | 接続詞の直前 | "turn right / and then left" |
| 関係詞（who/which/that） | 関係詞の直前 | "The man / who lives next door" |

#### 長さルール（補助）

- **5語以上の文**：必ず1か所以上スラッシュを入れる
- **8語以上の文**：必ず2か所以上を目標にする
- 上記トリガーを全て適用した結果、チャンクが3語以下になるのが理想

#### チャンク（分割してはいけない塊）

以下は意味・リズム的に一塊。内部でスラッシュを入れない。

| チャンク | 例 | 備考 |
|---|---|---|
| 定型チャンク（modal+to） | I'd like to / want to / going to / used to / have to / need to | 動詞原形まで含めてひとまとまり |
| 句動詞（動詞＋前置詞） | talk about / think about / look for / go to / listen to / belong to | セットで一つの動詞として扱う。前置詞が動詞の意味に不可欠なペア（belong to =「〜に属する」など）も同様に分割しない |
| **句動詞＋不定詞の連鎖** | try to go to bed / try to get up / try to sleep | 「try to ＋ 句動詞＋目的語」は全体で一つの意味のまとまり。途中にスラッシュを入れない |
| be動詞＋形容詞 | is amazing / are ready | 分離しない |
| 名詞句（名詞＋修飾語） | something tasty / a big bowl / ground meat / two garlic cloves | 形容詞・数量詞が名詞を修飾する塊。動詞の直後でスラッシュを入れ、目的語の内部では切らない |

**✅ 正しい例:**
- I'd like to / talk about two / of my favorite things.
- I want to / go to / the store.
- First, / try to go to bed / at the same time / every night.（`try to go to bed` は途中で切らない）

**❌ 誤った例:**
- I'd like / to talk / about two… （like to を分割、about を句動詞から切り離し）
- try to go / to bed（`go to bed` は句動詞連鎖。途中で切るのは誤り）
- I belong / to a family…（`belong to` は句動詞。動詞と前置詞の間で切るのは誤り。正しくは "I belong to / a family..."）

#### ❌ やってはいけないこと

- 主語＋動詞の間でスラッシュ（"I / like" は誤り）
- `to + 動詞` の間でスラッシュ（"to / go" は誤り）
- 定型チャンク（I'd like to / want to など）の内部でスラッシュ
- 句動詞（talk about / look for / go to bed など）の内部でスラッシュ
- **句動詞＋不定詞連鎖**（try to go to bed など）の途中でスラッシュ
- **リンキンググループの内部でスラッシュ**（"have / a passion" は誤り → "have a" は1グループ「ハヴァ」）
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
- [ ] **語内フラッピング見落とし**: `getting`→ゲ**リ**ング、`important`→インポー**ラ**ントのように、スペルに `t` があっても直前・直後が母音なら必ずフラッピング。**スペルで判断せずIPAで確認。**
- [ ] **リダクション両面**: カタカナに脱落反映 → 英語テキスト側にも `char-reduction` を付けたか？
- [ ] **過去形 `-ed` のリダクション見落とし**: `asked`・`talked`・`helped` など、スペルが `d` で終わっていても発音が `/t/` になる語は、子音前・文末で `char-reduction` を `d` に付ける。「スペルがdだからリダクション不要」と判断しない。
- [ ] **リダクションspan位置**: `often` の `t`、`honest` の `h` など、**消える文字そのものに** `char-reduction` を付けたか？（隣接する文字に誤適用していないか）特に **`-ight` 系（night/right/tonight）は `t` だけ**にマーク。`nigh` まで囲むのは誤り（`<span class="char-reduction">nigh<span>` は❌ → `nigh<span class="char-reduction">t</span>` が✅）。
- [ ] **文末t/d**: great!, Alright., -ight系（night/right/tonight）の語末 `t` をマークしたか？
- [ ] **閉鎖音連続**: would be の d、football の t など前の閉鎖音をマークしたか？
- [ ] **Thank you**: 1グループ「サンキュー」か？（「サンク＋クユー」は誤り）
- [ ] **語またぎフラッピング**: made a / but I / bit of / at a / **that our**（ダナー） など見落としていないか？特に `that`・`what`・`it`・`but` など機能語末の t は直前が母音なので、次語が母音始まりなら**常にフラッピング**する。
- [ ] **動詞＋about it / into it などの連鎖**: `talk about it`（トーカバウリッ）のように、**動詞末子音→about/into等の語頭母音でリンキング**、さらに **about/into末尾t→it等の語頭母音で語またぎフラッピング** が連鎖する。リンキングだけ・フラッピングだけで止めず、連鎖全体をたどること。
- [ ] **/nt/ クラスタ**: twenty/center の t はリダクション（フラッピングではない）
- [ ] **tr/dr 破擦音化**: 語頭・アクセント音節頭の tr/dr を「トレ/ドレ」ではなく「チュレ/ジュレ」寄りで表記したか？
- [ ] **同一子音衝突**: felt tired（t+t）のように前後で同じ調音点の破裂音が連続する場合、前の子音を脱落として扱ったか？
- [ ] **縮約語の追加脱落**: didn't→ディンのように、カジュアル発話でスペル通りの音節数より短く聞こえる縮約語がないか、実際の聞こえ方を優先したか？
- [ ] **鼻音＋母音リンキング**: working on→ワーキノン、turn on→ターノンのように、語末の ŋ/n/m が次語頭の母音と強くリンキングしていないか見落としていないか？

### マーク誤適用チェック
- [ ] **リンキング誤適用**: sounds nice など子音+子音にマークしていないか？
- [ ] **子音クラスター語末のリンキング誤適用**: `asked`（/æskt/）、`next`（/nɛkst/）、`helped`（/hɛlpt/）など、語末が子音クラスターで終わる語は、**必ずIPAで語末の実際の音を確認**してからリンキング判定する。スペルが `-ed` や `-e` で終わっていても語末音が子音なら次語頭母音とはリンキングしない（次語頭が子音なら当然しない）。
- [ ] **母音終わり→母音始まりの誤適用**: `show up`（/oʊ/+/ʌ/）、`go out`（/oʊ/+/aʊ/）など、**語末が母音のときはリンキング対象外**。「つながって聞こえる」＝リンキングではない。リンキングは必ず「語末**子音**→語頭母音」のみ。
- [ ] **母音終わり→子音始まりの誤適用**: `By doing`（/aɪ/+/d/）、`you can`（/uː/+/k/）、`for health`（/r/+/h/）、`in the`（/n/+/ð/）など、**前の語が母音終わりでも次の語が子音始まりならリンキングしない**。「母音で終わっているから」だけで判断しない。
- [ ] **単独語へのchar-linking誤適用**: `things,` や `Having` のように、リンキング相手のない語の一部にchar-linkingを付けていないか。
- [ ] **z/v/k/g + 母音リンキング見落とし**: `because it`→ビコー**ズィッ**、`make it`→メ**イキッ**、`have a`→**ハヴァ** など、閉鎖音・摩擦音終わりの語も必ずIPAで確認する。「スペルがkやveで終わっているから大丈夫」と決めつけない。
- [ ] **n/m + 母音リンキング**: `in a`→イナ、`them up`→ゼマップ など、鼻音（n/m）終わりの語も見落とさないか？
- [ ] **アシミレーション見落とし**: `did you`（ディジュ）、`would you`（ウッジュ）、`don't you`（ドウンチュ）、`about you`（アバウチュ）など /d/,/t/+/j/ のペアを見落としていないか？これらは1グループ（`char-assimilation`）にまとめること。
- [ ] **アシミレーション誤適用**: are/see/help/can/give you に as_('y') を使っていないか？
- [ ] **th**: フラッピング・アシミレーション対象外（anything の th はス行のみ）
- [ ] **ダークL見落とし**: `comfortable`→カンフタブ**ォ**、`peaceful`→ピースフ**ォ**、`healthy`→ヘ**ォ**スィーのように、語末・子音前の `l` はダークL。`l` の直後が母音なら明るいL（pillow/reallyはダークLなし）。スペルに `l` を見たら直後の音を必ず確認。
- [ ] **サイレントL**: `-alm`（calm/palm）、`-alk`（talk/walk）、`-ould`（would/could）などのパターンは `l` が発音されない。`char-reduction`（青い取り消し線）でマークし、カタカナにも `l` の音を入れない。

### HTML構造チェック
- [ ] **句読点のruby混入**: `and dogs!".` のように句読点（`,.!?"` など）を英語テキストと一緒にrubyに含めない。句読点は ruby の外に出すか、英語テキスト側にのみ含める。カタカナ（rt）に句読点が混入するバグの原因になる。
- [ ] **リンキングの範囲**: `cats and dogs` のケースでは `cats→and` がリンキング（/s/+/æ/）、`and→dogs` は子音+子音でリンキングなし。リンキングは**境界の2語のみ**を1グループにし、それ以降の語は別rubyにする。: `<span class="char-reduction">t</span>` などのspanが ruby要素をまたいで漏れていないか？特に `<ruby>` の**閉じ直前**で `</span>` が閉じられているか目視確認する。閉じ忘れると次の `<ruby>` の英語テキストに文字が混入するバグが発生する。
- [ ] **1単語1ruby**: nothing/something/everyone/because など1単語を複数rubyに分割していないか？
- [ ] **ruby間スペース**: `</ruby> <ruby>` の間に半角スペースがあるか？（ないと単語がくっつく）
- [ ] **リンキンググループのruby統合**: `youngest of`、`at a` など2語以上のリンキング・アシミレーションは1つの`<ruby>`にまとめたか？（2つのrubyに分けるとカタカナに余分なスペースが入る）
- [ ] **アクセント**: 2音節以上の全内容語に `<span class="accent">` があるか？ 位置はIPAの `ˈ` 直後の音節で決定する（ルールや勘で決めない）。
- [ ] **複合マーク語のアクセント漏れ**: `often`（リダクション）のように、リダクション/フラッピング等の他のspanが付く単語は、アクセントspanを付け忘れやすい。複数spanが重なる単語は両方チェックする。
- [ ] **スラッシュ**: 副詞＋前置詞句パターン（tonight / for...）を見落としていないか？
- [ ] **/ð/ カタカナ**: ダ行か？（ザ行になっていないか）
