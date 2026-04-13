---
name: eigo-piyo-piyo
description: 英語発音ガイドスキル。英文を入力すると、ネイティブのアメリカ英語カジュアル発音を解析し、単語ごとにIPA発音記号・カタカナ読み・ストレス・連結音声現象（リンキング、リダクション、フラッピング、アシミレーション、Dark L）を付与してHTMLで表示する。「英語 発音」「カタカナ読み」「IPA」「発音記号」「リンキング」「英文を読む」「pronounce」「pronunciation」「how to say」などのキーワードで発動。英文が入力されて「発音」「読み方」「カタカナ」に言及があれば必ず使う。
---

# EigoPiyoPiyo - 英語発音ガイド

英文を受け取り、ネイティブのアメリカ英語カジュアル発音を解析して、見やすいHTMLを生成するスキル。

## ワークフロー

1. ユーザーから英文を受け取る
2. 以下の解析ルールに従って各単語/連結グループを分析
3. `references/output-format.md` を読んでHTMLを生成
4. `/mnt/user-data/outputs/` にHTMLファイルを保存して `present_files` で返す

## カードと文の区切りルール

- 入力の**1行 = 1カード**（sentence-card）
- 1行の中に複数の文（ピリオド `.` / `!` / `?` で終わる文が複数）がある場合、**文ごとに ruby-line + ipa-line を分ける**。これにより文の途中で折り返されず読みやすくなる。
- 日本語訳はカード内で全文まとめて1つ。

## 解析ルール

### グループ（group）とは

グループは、連結発音（connected speech）や韻律的まとまり（prosodic chunking）によって一緒に発音される1つ以上の単語のまとまり。

- **リンキング**で繋がる単語は1グループ（例: "eating outside" → 1グループ）
- **アシミレーション**で融合する単語は1グループ（例: "did you" → 1グループ）
- **リズムグルーピング**: 弱形の機能語（weak）が隣接し、間にストレス語がない場合は1グループにまとめる。音声現象（リンキング・アシミレーション等）がなくても、韻律的に一つの塊として発音されるためグループ化する。
  例: "for you" → どちらも弱形機能語で間にストレスがない → 1グループ「ファヤ」
  例: "to the" → どちらも弱形機能語 → 1グループ「トゥダ」
  例: "in a" → 弱形機能語同士だが /n/+/ə/ でリンキングもする → 1グループ「イナ」（リンキングとしてマーク）
  ⚠️ リズムグルーピングはあくまで**弱形同士**の結合。弱形＋ストレス語、ストレス語＋弱形は対象外（それぞれ独立グループ）。
- 単独の単語はそれ自体が1グループ（例: "How" → 1グループ）

### original配列のルール

元テキストを文字境界で分割し、各部分にタイプを付ける。**全グループのtextを結合すると元の入力文と完全一致しなければならない。**

タイプ一覧:
- `normal`: 通常の文字
- `linking`: 語末の子音と次の語頭の母音が繋がるとき。語末子音と語頭母音のみマーク。
  重要: **文字ではなく音で判定する**。
  - "y" は文字では母音に見えるが音は子音 /j/ → リンキングしない（例: "is your" は子音+子音なのでリンキングではなくアシミレーション）
  - "u" が /juː/ で始まる語も先頭音は子音 /j/ → リンキングしない（例: "a university"）
  - "h" が脱落する語は母音始まりになる → リンキングする（例: "an hour"）
  注意: 子音→子音、母音→母音、母音→子音はリンキングではない。"to", "the", "of", "a" はほとんどリンキングに関与しない。迷ったら `normal`。
- `flapping`: 母音（または母音的な音 /ɚ/ /ɹ/）と母音に**挟まれた** t/d が /ɾ/ に変わる箇所。t/d/tt/dd の文字をマーク。
  ⚠️ **判定手順（必ずこの順序で確認）**:
  1. t/d の**直前の音（文字ではなくIPAの音）**が母音（/ɪ/, /ɛ/, /æ/, /ɑː/, /ɔː/, /ʌ/, /ɚ/, /ɹ/ など）か確認
  2. 直前が**子音**（/s/, /n/, /l/, /k/ など）なら → **フラッピングしない**（即座にnormalと判定）
  3. 直前が母音 かつ 直後も母音 → フラッピングする
  ✅ フラッピングする例: water (母音 ɔː + t + 母音 ɚ), better (ɛ + tt + ɚ), city (ɪ + t + i), liberty (ɚ + t + i), party (ɑːr + t + i), getting (ɛ + tt + ɪ)
  ❌ フラッピングしない例:
  - **destination** (de**s**+t+ination → /s/ は子音 → フラッピングしない。「デスティネイション」が正しい)
  - **station** (/s/+t → 子音クラスタ /st/)、**store**, **street**, **stand**, **stop**（/st/ 内の t はフラッピングしない）
  - **take**, **time**, **top**（語頭の t は帯気音 /tʰ/ でフラッピングしない）
  - **enta** 型: /n/+t+母音（enter, winter, center, twenty, plenty）→ 直前が /n/（子音）なのでフラッピングしない。カジュアル発話では t がほぼ脱落して /n/ が直接次の母音に繋がる（twenty→「トゥウェンニー」, center→「センナー」）。この t は**リダクション扱い**（`char-reduction`）でマークする。
  - **th は対象外**: "th" は /θ/ や /ð/（摩擦音）であり、破裂音の /t/ /d/ とは全く別の音。anything, nothing, weather, other などの "th" にフラッピングマークを付けてはいけない。フラッピングの対象は文字の "t" と "d" のみ。
- `dark_l`: 語末・音節末の L が暗い L（オ/ウのような音）で発音される箇所（例: bottle の le, people の le, all の ll）
- `reduction`: カジュアル発話で音が**脱落する、またはほとんど聞こえなくなる**箇所をマークする。完全脱落も未開放（unreleased）もすべてリダクションとして統一的に扱う。
  ✅ リダクションする例:
  - silent letter（もともと発音しない文字）:
    - silent h: "**h**onest"→アネス, "**h**our"→アワー, "**h**eir"→エア。注意: "he", "his", "have" などの h は通常発音するので reduction にしない。
    - silent t: "hones**t**"→アネス, "of**t**en"→オーフン, "lis**t**en"→リスン
    - ⚠️ **silent letter がある語では、同じ語内の発音される文字に誤ってリダクションを付けないこと**。例: "honest" の e は /ɪ/ として発音される → normal。h と t のみリダクション。
  - elision（子音クラスタ内の省略）: "san**d**wiches"→サンウィッチズ（d を発音しない /ˈsænwɪtʃɪz/）, "We**d**nesday"→ウェンズデイ, "han**d**some"→ハンサム, "gran**d**mother"→グランマザー
    - ⚠️ **カタカナでは脱落を反映しているのに、英語テキスト側にリダクションのマークを付け忘れる**というミスが起きやすい。必ず英語テキスト側の該当文字にも `char-reduction` を付けること。
  - 文末の t/d 完全脱落: "Alrigh**t**."→オーライ, "grea**t**!"→グレイ！, "goo**d**."→グッ。（文末・句末の t/d はカジュアル発話で完全に消えるのが普通）
  - 子音前での完全脱落: "No**t** so"→ナッソウ（t が次の子音の前で完全に消える）, "nex**t** to"→ネクストゥ（t が同じ t の前で消える）, "Tha**t** would"→ダウッ（t が子音 w の前で完全に消える）
  - 閉鎖音+閉鎖音の連続（前の音が吸収されて脱落）: "woul**d** be"→ウッビ（/d/+/b/ で d が b に吸収）, "tha**t** kind"→ダカインド（/t/+/k/ で t が脱落）, "goo**d** bye"→グッバイ（/d/+/b/ で d が吸収）, "kep**t** going"→ケプゴウイン（/t/+/ɡ/ で t が脱落）, "Than**k** you"→サンユー（/k/+/j/ で k が軽くなり脱落）。閉鎖音（/p/,/b/,/t/,/d/,/k/,/ɡ/）が連続するとき、前の閉鎖音は次の閉鎖音に吸収されて聞こえなくなる。
  - 未開放（unreleased）で実質聞こえない音: "I'**d** like"→アイライク（/d/ は次の /l/ の前で未開放、舌は d の位置に行くが破裂せずそのまま l に移行するため実質聞こえない）, "woul**d**"（文末）→ウッ
  - 音の省略: "probably"→probly（音節ごと省略）
  - /n/+t+母音の t 脱落: "twen**t**y"→トゥウェンニー, "cen**t**er"→センナー, "win**t**er"→ウィンナー, "plen**t**y"→プレンニー。/nt/ クラスタ内の t がカジュアル発話で脱落し、/n/ が直接次の母音に繋がる。フラッピング（ラ行の音）ではないので注意。
  - 同一調音点の子音連続（前の音が吸収されて脱落）: 同じ調音点（舌の位置）の子音が語をまたいで連続すると、前の子音が脱落して後ろの子音に吸収される。閉鎖音に限らず摩擦音でも起きる。
    - /θ/+/ð/: "wi**th** this"→ウィディス（/θ/ が /ð/ に吸収）, "wi**th** that"→ウィダッ, "wi**th** the"→ウィダ
    - /θ/+/θ/: "bo**th** things"→ボウスィングズ（前の /θ/ が脱落）
    - /s/+/s/: "thi**s** Saturday"→ディサラデイ（/s/ が1つに）
    - /z/+/z/: "chee**se** zone"→チーゾウン（/z/ が1つに）
    この場合、前の語の該当子音文字に `char-reduction` を付ける。また、前後の語を1グループにまとめてカタカナで連結発音を反映する。
  ❌ リダクションしない（normalにする）例:
  - "good morning": /d/ は未開放だが音は存在する → リダクションではない（/d/+/m/ は閉鎖音+鼻音で、吸収パターンに該当しない）
  **判定基準: カジュアル発話でネイティブがほとんど聞き取れないレベルの音はすべてリダクションとしてマークする。** 閉鎖音+閉鎖音の連続、文末の t/d、silent letter、未開放の閉鎖音は積極的にマーク。/d/+/m/ や /d/+/n/ のような閉鎖音+鼻音は音が残るのでマークしない。迷ったらリダクションにする。
- `assimilation`: 隣接する音が融合・変化する箇所。末尾の子音と次の語頭をマーク。アシミレーションする単語同士は1グループにまとめる。
  パターン一覧:
  - /d/ + /j/ → /dʒ/: "do you"→ジュ, "would you"→ウジュ, "did you"→ディジュ, "could you"→クジュ
  - /t/ + /j/ → /tʃ/: "don't you"→ドウンチュ, "got you"→ガッチュ, "what you"→ワッチュ, "let you"→レッチュ
  - /z/ + /j/ → /ʒ/: "is your"→イジュア, "does your"→ダジュア, "as you"→アジュー, "was your"→ワジュア
  - /s/ + /j/ → /ʃ/: "miss you"→ミシュー, "bless you"→ブレシュー, "this year"→ディシアー

  **重要: アシミレーションは常に起きるわけではない。** 話すスピードやフォーマルさによって変わる:
  - カジュアル・速い会話 → アシミレーションしやすい（"Do you" → ジュ）
  - 丁寧・ゆっくりな場面 → 別々に発音される（"Do you" → ドゥ ユー）
  文脈に応じて判断する。同じ会話内でもアシミレーションする箇所としない箇所が混在してよい。
  迷ったらカジュアル寄り（アシミレーション）で処理する。

同じタイプが連続する文字はまとめる。スペースは `normal`。

### スラッシュ（slash reading）

文を意味のまとまり（チャンク）ごとにスラッシュ `/` で区切り、英語の語順のまま前から理解する「スラッシュリーディング」を視覚的に支援する。区切り位置にスラッシュマーカーを表示することで、英語の自然なリズム・ポーズの位置と、意味の切れ目が一目でわかる。

**判定ルール（優先度順）:**

1. **句読点の後**: カンマ `,` セミコロン `;` コロン `:` の後は必ず区切る
2. **接続詞の前**: and, but, or, so, because, although, when, if, while, since, after, before, that（従属接続詞）などの前で区切る
   - ただし短い列挙（"A and B" の2項目のみ）では区切らなくてもよい
   - 3項目以上の列挙（"A and B and C"）は各項目の前で区切る
3. **主語と動詞の間**（主語が長い場合）: 主語句が長い場合、動詞の前で区切る。ただし "I think" や "You can" のように主語が短い場合は区切らない
4. **動詞と目的語/補語の間**（目的語/補語が長い場合）: 動詞句の後に長い目的語や補語が続く場合、その前で区切る。"like coffee" のように短い場合は区切らない
5. **前置詞句の前**: for, with, in, at, on, to, from, by, about, of などの前置詞句の前で区切る（ただし2〜3語の短い前置詞句は前のチャンクに含めてもよい）
6. **関係代名詞・関係副詞の前**: who, which, that（関係代名詞）, where, when（関係副詞）の前で区切る
7. **準動詞（to不定詞・動名詞・分詞）の前**: to + 動詞原形、-ing形、過去分詞が新しい意味のまとまりを始める場合に区切る
8. **フィラー・呼びかけの後**: Well, Uh, You know, I mean, Let's see, Excuse me などの後で区切る

**チャンクの粒度ガイドライン:**
- 1チャンクは**2〜7語**が目安。1語だけのチャンクは避ける（フィラー・呼びかけを除く）
- 動詞句のまとまり（主語＋助動詞＋動詞）は1チャンクにまとめる（"can I get" → 1塊）
- 名詞句のまとまり（修飾語＋名詞）は1チャンクにまとめる（"two cheese burgers" → 1塊）
- 短い前置詞句（"for me", "at home"）は前後のチャンクに含めてよい

**例:**
- "Well, can I get two cheese burgers and one french fries and an iced tea, please."
  → Well, / can I get / two cheese burgers / and one french fries / and an iced tea, / please.
- "Please wait for your meal with this number card."
  → Please wait / for your meal / with this number card.
- "The man who lives next door is a teacher."
  → The man / who lives next door / is a teacher.
- "I want to go to the park to play soccer."
  → I want to go / to the park / to play soccer.

⚠️ スラッシュの位置は厳密な正解があるものではなく、読み手のレベルや文の複雑さによって変わる。意味のまとまりを意識して判定する。迷ったら区切りを入れる（区切りすぎるより、区切らなさすぎる方が読みにくい）。

**HTML表現**: スラッシュの区切り位置に `<span class="slash"></span>` を挿入する。rubyグループの**間**に置く。

### stress値

- `stressed`: 文の中で強調される語 → 太字・大きいカタカナ。以下の2つの基準で判定する:
  1. **内容語強勢**: 主要な内容語（名詞・動詞・形容詞・副詞）のうち、文の意味の中心となるもの
  2. **焦点強勢（focus stress）**: 文脈上、話者が特に伝えたい情報。文法的には機能語や数詞でも、意味的に重要なら stressed にする。
     - 数量: 注文・依頼で個数を正確に伝えたい場面（"**two** cheese burgers and **one** french fries"）
     - 対比: 選択肢を比較する場面（"not **this** one, **that** one"）
     - 訂正: 誤りを正す場面（"I said **Tuesday**, not **Thursday**"）
     - 新情報: 会話に初めて導入される重要な情報
  ⚠️ 入力テキストで `**太字**` マークされた語がある場合、その語を必ず stressed にする（ユーザーによる明示的な強調指定）。
- `weak`: 機能語（冠詞、前置詞、助動詞、弱化代名詞）→ 小さいカタカナ。ただし焦点強勢に該当する場合は stressed に昇格する。
- `normal`: その他

### 単語内アクセント（音節強勢）

各グループの英語テキストとカタカナの**第一強勢の音節**を大きく表示する。

表現方法:
- **英語 original**: アクセント音節の文字を `<span class="accent">...</span>` で囲む
- **カタカナ kana**: アクセント音節に対応するカタカナを `<span class="accent">...</span>` で囲む

例:
- destination → `desti<span class="accent">na</span>tion` / `デスティ<span class="accent">ネイ</span>ション`
  - 第一強勢は "-na-" /ˈneɪ/ にある（IPAのˈの直後の音節）
- Statue → `<span class="accent">Sta</span>tue` / `<span class="accent">スタ</span>チュー`
  - 第一強勢は "Sta-" /ˈstæ/ にある
- Liberty → `<span class="accent">Li</span>berty` / `<span class="accent">リ</span>バーリー`
  - 第一強勢は "Li-" /ˈlɪ/ にある

判定ルール:
1. IPAの `ˈ` マークの直後の音節が第一強勢
2. 英語テキスト側: その音節に対応するアルファベット文字を `accent` で囲む（音声現象のspanと組み合わせ可能）
3. カタカナ側: その音節に対応するカタカナ部分を `accent` で囲む
4. 単音節語（I, go, like, the など）にはアクセントマークを付けない（語全体が1音節なのでマーク不要）
5. 弱化した機能語（to, the, of, a, your など）にはアクセントマークを付けない

### カタカナ（kana）ルール

教科書的でなく、**実際のカジュアルなネイティブアメリカ英語発音**を反映:

- 複数語のリンクグループは連結発音を1つのカタカナ文字列で表現
- 文末の句読点はカタカナに含める（例: "サニー。" "ゴウ？"）
- Yes/No疑問文は末尾に "↑" を付ける。WH疑問文には付けない。
- 主な音変化:
  - /θ/→ス, /ð/→ダ行, /v/→ヴ, /f/→フ
    ⚠️ **/ð/ は必ずダ行で統一する**（ザ行にしない）: there→デア, that→ダッ, this→ディス, they→デイ, the→ダ, them→ダム, then→デン, though→ドウ。「ゼア」「ザ」「ゾウ」などザ行を使わないこと。
  - フラッピング: 母音間 t/d → ラ行（water→ウォーラー, better→ベラー, city→スィリー, party→パーリー）。**⚠️ /st/ クラスタ内は絶対にフラッピングしない**（destination→デス**ティ**ネイション ※「ラ」にならない, station→ス**テイ**ション）。判定基準: t の直前の音が子音なら通常発音。
  - Dark L: 語末 l → オ/ウ（people→ピーポー, little→リロー）
  - グロッタルストップ: n前の t → ッ（button→バッン）
  - 弱化: to→トゥ（※「タ」「ダ」にはしない。「ダ」は the の発音）, for→ファ, can→クン, and→アン/ン, of→ア/アヴ, you→ヤ/ユ, the→ダ
  - **"to" と "the" を混同しない**: to /tə/ =トゥ（/t/ の音）, the /ðə/ =ダ（/ð/ 舌を噛む音）。この2語は音が全く違う。
  - 縮約: want to→ワナ, going to→ゴナ, got to→ガラ, kind of→カインダ

### IPA

アメリカ英語のIPA表記。リンキングは ‿ で表す。ストレスマーク ˈ を含める。

### 日本語訳

各文に自然な会話調の日本語訳を付ける。

## ⚠️ セルフチェックリスト（HTML生成前に必ず確認）

HTML生成前に、以下の項目を1つずつ確認する。過去の実際のミスに基づくリストなので、特に注意すること。

1. **silent letter の両面チェック**: silent h/t/d などがある語（honest, hour, sandwich, Wednesday など）で:
   - ✅ 英語テキスト側に `char-reduction` が付いているか？
   - ✅ カタカナ側でその音が省略されているか？
   - ✅ 同じ語内の**発音される文字**に誤ってリダクションを付けていないか？（例: honest の e は発音する）
2. **文末 t/d の脱落**: 文末・句末の t/d にリダクションマークが付いているか？（great!, Alright., that's it. など）
3. **閉鎖音+閉鎖音の連続**: Thank you の k, would be の d, kept going の t など、前の閉鎖音にリダクションマークが付いているか？
4. **未開放の閉鎖音**: I'd like の d, would の文末 ld など、カジュアル発話で実質聞こえない音にリダクションマークが付いているか？
5. **英語テキストとカタカナの整合性**: 英語テキストでリダクションにした文字が、カタカナ側でも適切に反映されているか？（マークしたのにカタカナではその音を含めている、またはその逆がないか）
6. **アクセントの位置**: 多音節語の第一強勢が正しい音節に付いているか？（IPAの ˈ 直後の音節を確認）
7. **/ð/ のカタカナがダ行か**: there, that, this, the, they, them, then, though などの /ð/ を「ゼア」「ザ」などザ行にしていないか？ 必ずダ行（デア、ダッ、ディス、ダ、デイ、ダム、デン、ドウ）で統一。
8. **/nt/ クラスタの t**: twenty, center, winter, plenty などの /n/+t+母音 の t にリダクションマーク（`char-reduction`）が付いているか？ フラッピングマークではないことを確認。
9. **th にフラッピングを付けていないか**: anything, nothing, weather, other などの th は /θ/ /ð/（摩擦音）であり、フラッピングの対象ではない。
10. **同一調音点の子音連続**: with this, with the, this Saturday など、同じ調音点の子音が語をまたいで連続する場合、前の子音にリダクションマークが付いているか？ また、それらの語を1グループにまとめてカタカナで連結発音を反映しているか？

## HTML出力フォーマット

### デザインコンセプト

**ふりがな方式**: 英文を主役として大きく表示し、カタカナ読みを `<ruby>` / `<rt>` タグで英単語の上にふりがなのように振る。IPAは脇役として英文の下に小さく1行で表示。単一の自己完結型HTMLファイル（CSS・フォント込み）を生成し、`/mnt/user-data/outputs/` に保存して `present_files` で返す。

### HTMLテンプレート

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>EigoPiyoPiyo - 英語発音ガイド</title>
  <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=M+PLUS+Rounded+1c:wght@400;700;800;900&family=Nunito:wght@700;800;900&display=swap">
  <style>/* 後述のCSSを埋め込む */</style>
</head>
<body>
  <div class="container">
    <h1>🐣 EigoPiyoPiyo</h1>
    <p class="subtitle">英語発音ガイド</p>
    <div class="toggle-bar">
      <button id="kana-toggle" class="toggle-btn" onclick="toggleKana()">🐣 カタカナ ON</button>
    </div>
    <!-- 各文のカード。入力の1行=1カード。1行に複数文がある場合は文ごとにruby-line+ipa-lineを分ける -->
    <div class="sentence-card">
      <div class="ruby-line">
        <ruby>英単語<rt class="rt-stressed">カタカナ</rt></ruby> ...
      </div>
      <div class="ipa-line">/IPA/</div>
      <div class="translation">日本語訳</div>
    </div>
    <footer class="footer">
      <div class="legend">
        <div class="legend-item"><span class="dot-linking">●</span><span class="legend-label">リンキング</span><span class="legend-desc">単語がつながる</span></div>
        <div class="legend-item"><span class="dot-assimilation">●</span><span class="legend-label">アシミレーション</span><span class="legend-desc">音が混ざって変化</span></div>
        <div class="legend-item"><span class="dot-reduction">●</span><span class="legend-label">リダクション・Hドロッピング</span><span class="legend-desc">発音しない・消える</span></div>
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
    if (container.classList.contains('hide-kana')) {
      btn.textContent = '🙈 カタカナ OFF';
      btn.classList.add('off');
    } else {
      btn.textContent = '🐣 カタカナ ON';
      btn.classList.remove('off');
    }
  }
  </script>
</body>
</html>
```

### ruby要素の構成

各グループを1つの `<ruby>` にする: `<ruby>英語テキスト<rt class="rt-XXX">カタカナ</rt></ruby>`

英語テキスト内の音声現象span:
- `normal`: そのまま, `linking`: `<span class="char-linking">`, `reduction`: `<span class="char-reduction">`, `assimilation`: `<span class="char-assimilation">`, `flapping`: `<span class="char-flapping">`, `dark_l`: `<span class="char-dark-l">`

rt要素のclass: `stressed`→`rt-stressed`(大きく濃い), `weak`→`rt-weak`(小さく薄い), `normal`→classなし

アクセント: rt内で `<span class="accent">` で囲む。単音節語や弱化機能語には付けない。

IPA行: `<div class="ipa-line">/...IPA.../</div>`

### CSS

```css
* { margin: 0; padding: 0; box-sizing: border-box; }
body {
  font-family: 'M PLUS Rounded 1c', 'Nunito', -apple-system, BlinkMacSystemFont, 'Hiragino Sans', sans-serif;
  background: #ffde59; color: #333; min-height: 100vh;
}
.container { width: 100%; padding: 32px clamp(16px, 4vw, 80px); }
h1 { font-family: 'Nunito', 'M PLUS Rounded 1c', sans-serif; font-size: 2.2rem; font-weight: 900; text-align: center; margin-bottom: 6px; color: #333; }
.subtitle { text-align: center; color: #555; margin-bottom: 24px; font-size: 0.9rem; font-weight: 700; }
.sentence-card { background: #fff; border-radius: 20px; padding: 28px; margin-bottom: 16px; border: 4px solid #333; border-bottom-width: 6px; }
.ruby-line { font-family: 'Nunito', 'M PLUS Rounded 1c', sans-serif; font-size: clamp(1.8rem, 3.2vw, 2.8rem); font-weight: 900; color: #333; line-height: 3.2; }
ruby { ruby-align: center; }
rt { font-family: 'M PLUS Rounded 1c', sans-serif; font-size: 0.42em; font-weight: 800; color: #666; ruby-align: center; }
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
.ipa-line { font-family: 'Lucida Grande', 'Arial Unicode MS', 'Gentium Plus', serif; font-size: 0.85rem; color: #bbb; line-height: 1.4; margin-top: 2px; }
.translation { font-size: 0.95rem; font-weight: 700; color: #555; line-height: 1.5; margin-top: 10px; padding-top: 10px; border-top: 3px solid #eee; }
.footer { text-align: center; margin-top: 28px; padding: 16px 0; }
.legend { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 6px; max-width: 720px; margin: 0 auto; font-size: 0.78rem; font-weight: 800; color: #555; }
.legend-item { display: flex; align-items: center; gap: 6px; background: #fff; padding: 6px 12px; border-radius: 12px; border: 2px solid #333; }
.legend-label { white-space: nowrap; }
.legend-desc { font-weight: 600; color: #999; font-size: 0.7rem; white-space: nowrap; }
.dot-linking { color: #ff922b; } .dot-assimilation { color: #ff6b6b; } .dot-reduction { color: #339af0; } .dot-flapping { color: #51cf66; } .dot-dark-l { color: #2b8a3e; } .legend-slash { color: #ccc; font-weight: 900; font-size: 1.1em; }
.github-link { margin-top: 12px; font-size: 0.75rem; font-weight: 700; }
.github-link a { color: #888; text-decoration: none; }
.github-link a:hover { color: #333; text-decoration: underline; }
.toggle-bar { text-align: center; margin-bottom: 20px; position: sticky; top: 12px; z-index: 100; }
.toggle-btn { font-family: 'M PLUS Rounded 1c', sans-serif; font-size: 0.95rem; font-weight: 900; color: #333; background: #fff; border: 3px solid #333; border-bottom-width: 5px; border-radius: 14px; padding: 8px 24px; cursor: pointer; transition: all 0.15s; box-shadow: 0 2px 8px rgba(0,0,0,0.08); }
.toggle-btn:hover { background: #fffbe6; }
.toggle-btn:active { border-bottom-width: 3px; transform: translateY(2px); }
.toggle-btn.off { background: #f1f3f5; color: #888; border-color: #aaa; }
.hide-kana rt { opacity: 0; pointer-events: none; }
@media (max-width: 600px) { .container { padding: 20px 12px; } h1 { font-size: 1.8rem; } .sentence-card { padding: 20px 16px; } .ruby-line { font-size: 1.6rem; line-height: 3.0; } .ipa-line { font-size: 0.75rem; } }
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
