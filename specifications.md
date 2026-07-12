# アプリ公式仕様

> このファイルはアプリの唯一の公式仕様です。回答・実装は必ず本ファイルの内容と整合させ、矛盾がある場合はその矛盾点を明示してください。
> 元は `specifications.json`（JSON形式）で管理していましたが、Claudeによるチャットを跨いだ把握精度・編集時の事故防止（クォートのエスケープ崩れ等）を目的に、2026-07に本Markdown形式へ移行しました。

## 目次（トップレベル）

- [`frontend`](#frontend)
  - [`assets`](#assets)
  - [`libraries`](#libraries)
  - [`scripts`](#scripts)
- [`conventions`](#conventions)
  - [`domHookAttributes`](#domhookattributes)
  - [`nodeScriptModuleSystem`](#nodescriptmodulesystem)
  - [`workflowNaming`](#workflownaming)
- [`backend`](#backend)
  - [`fastapi`](#fastapi)
  - [`heuristics`](#heuristics)
  - [`heuristics_conditions`](#heuristics_conditions)
  - [`jpx_xlsx`](#jpx_xlsx)
  - [`fetch`](#fetch)
  - [`margin`](#margin)
- [`workflows`](#workflows)
  - [`download-jpx-xlsx.yml`](#download-jpx-xlsx.yml)
  - [`fetch.yml`](#fetch.yml)
  - [`margin.yml`](#margin.yml)
  - [`heuristics.yml`](#heuristics.yml)
  - [`heuristics-backfill.yml`](#heuristics-backfill.yml)
- [`dataFlow`](#dataflow)
  - [`chartData`](#chartdata)
  - [`screening`](#screening)
  - [`csvExport`](#csvexport)
  - [`compareScreening`](#comparescreening)
  - [`margin`](#margin)
  - [`jpx`](#jpx)
  - [`heuristics`](#heuristics)
  - [`heuristicsBackfill`](#heuristicsbackfill)

# `frontend`

## `assets`

### `favicon`

#### `file`

assets/favicon.ico

#### `responsibility`

ブラウザタブおよび Apple Touch Icon 用アイコン

### `stylesheet`

#### `file`

style.css

#### `responsibility`

アプリ全体のレイアウト・配色・モーダル・テーブル・チャート領域・レスポンシブ対応を定義する

#### `sections`

##### `designTokens`

- :root に定義する CSS カスタムプロパティ。
- 色（--color-*）・シャドウ（--shadow-*）・ボーダー半径（--radius-*）・サイズ（--rci-macd-height 等）・z-index 体系（--z-*）をここで一元管理する。
- ブレークポイントは CSS カスタムプロパティがメディアクエリ条件内で使用不可のため、:root のコメント（--bp-mobile: 600px）と @media の数値を一致させる運用とする

##### `layout`

body, app-container, card の基本レイアウトと配色。body には accent-color: var(--color-accent-primary) を指定する（2026-07 追加。チェックボックス・ラジオボタンのチェックマーク/ドット色は accent-color を指定しない場合ブラウザ既定の青色になり、exclude-markets-fieldset のチェックボックスや compare-source-option のラジオボタン等、他の装飾要素と色調が揃っていなかったため、全体へ継承させる形で統一した。.mode-tab / .timeframe-selector のラジオボタンは opacity: 0 で非表示のため実害はないが、こちらにも一貫して適用される）

##### `controls`

- 条件設定 UI（mode-tabs / mode-tab / mode-panels / mode-panel / mode-conditions / exclude-markets-fieldset / compare-source-fieldset / compare-source-option / compare-source-label）。
- モード切替は input[name=searchMode] ラジオボタンを .mode-tab 内に視覚的に隠して配置し（opacity: 0）、.mode-tab 全体をタップ領域とするタブ UI。
- 選択中の .mode-tab（:has(input:checked)）は、他の凹デザイン要素（.mode-conditions input・select・fieldset 等）が使う --shadow-neumo-inset とは別の専用トークン --shadow-neumo-inset-tab（inset 4px 4px 9px rgba(122, 46, 58, 0.42), inset -4px -4px 9px rgba(255, 255, 255, 0.88)。影のオフセット/ぼかしを3px/7pxから4px/9pxへ、不透明度を0.22→0.42、ハイライトを0.95→0.88とし、--shadow-neumo-inset より明確に濃い彫り込み感を持たせている）を box-shadow に用い、背景も var(--color-bg-card) ではなく var(--color-accent-primary-bg)（バーガンディの薄いティント）、文字色も var(--color-accent-primary-hover) を用いることで、入力欄等の一般的な凹デザインより明確に濃いアクセントカラートーンの凹表現とし、視覚的に差別化している（2026-07 修正。旧実装は --shadow-neumo-inset・var(--color-bg-card)・var(--color-accent-primary) を用いており、入力欄と全く同じ凹デザインだったため見分けにくいという指摘があった。当初は不透明度0.35／ハイライト0.9の中間案で導入したが、差別化をより明確にするため濃色寄りの値に調整した）。
- 選択中モードの判定と条件パネルの表示・非表示は、.mode-tabs:has(input[value="<mode>"]:checked) ~ .mode-panels .mode-panel[data-mode="<mode>"] という :has() セレクタで CSS のみで行い、JS 側の状態管理（initSearchMode によるフォーム要素の有効・無効化）とは独立している。
- exclude-markets-fieldset は ratio / heuristics 両モードで共通の DOM 構造・CSS クラスを用い、screening.js の renderExcludeMarketsFieldset() が実行時に動的生成する（詳細は scripts.screening.behavior.renderExcludeMarketsFieldsets を参照）。
- exclude-markets-fieldset は <label> で囲わず [data-exclude-markets-slot] 直下に配置し、グループ見出しは fieldset 内の <legend>除外する市場・商品区分</legend> で表現する（複数のチェックボックスを単一の <label> で包むと「フォームフィールドに紐付かない label」としてアクセシビリティ監査（axe / Lighthouse）で警告されるため、fieldset + legend を用いる）。
- exclude-markets-fieldset は border: none と box-shadow: var(--shadow-neumo-inset) を指定し、compare-source-fieldset / compare-date-fieldset / .mode-conditions input・select と同じ凹（押し込まれた）ニューモーフィズム表現に統一する（2026-07 修正。旧実装は border: 1px solid var(--color-border-input) のみのフラットな枠線で、他の入力グループと視覚表現が不揃いだった）。
- exclude-markets-fieldset はビューポート幅を問わず display: flex; flex-wrap: wrap; gap: 4px 0; min-width: 0; を指定する（2026-07 修正。`<fieldset>` 要素は中身（チェックボックス群）を1行に並べた場合の幅を自身の最小幅として持とうとする性質があり、親の flex アイテム（[data-exclude-markets-slot]）へ min-width: 0 を指定しても fieldset 自身がそれを上書きしてしまう。従来は display: flex; flex-wrap: wrap を600px以下のモバイル用メディアクエリ内でのみ指定していたため、600pxを超える横向きスマホ（iPhone 12 Proで論理幅約844px）やウィンドウ幅を狭めたデスクトップブラウザで右側にはみ出す不具合があった。基本スタイルへ移すことで発生条件を問わず解消した。モバイル用メディアクエリ内の同一指定は重複するが実害がないためそのまま残している）。
- .mode-conditions select（日付欄：#ratioDateSelect / #dateSelect / #heuristicsDateSelect / #compareFromDate / #compareToDateSelect）は appearance: none; -webkit-appearance: none; を指定し、background-image によるCSS描画のみの▼矢印（linear-gradientを2つ組み合わせた三角形2つ）に置き換える（2026-07 修正。モバイルブラウザ（特にiOS Safari）は`<select>`要素に対してOS標準の見た目を強く保持する傾向があり、appearance: noneを明示しない限りbox-shadow等のカスタムスタイルが実質的に無視され、同じ.mode-conditions input:not([type="radio"]), .mode-conditions selectルールを共有する`<input>`と異なり凹デザインが表示されない不具合があった）。あわせて color: var(--color-text-primary) を明示指定する（2026-07 追加。iOS Safariでは`<select>`のテキスト色がOS標準の青系カラーになり、PCでの見た目と異なっていたため明示的に上書きした）。
- なお .mode-conditions input（type="radio" を除く）・select には box-shadow: var(--shadow-neumo-inset) を適用するが、input[type="radio"]（compare-source-option の #compareSource 等、4モード切替タブ用ラジオボタン .mode-tab input[type="radio"] は対象外・別セレクタで非表示制御）は :not([type="radio"]) で除外し、ネイティブのラジオボタン表示のまま凹凸を付けない（2026-07 修正。汎用ルールがラジオボタンにも box-shadow・background を適用してしまい、丸い入力の周囲に不自然な凹み・ハイライトが表示される見た目上の不具合があったため）。
- exclude-markets-fieldset は width: 100% で [data-exclude-markets-slot] の残り幅を占有し、#heuristicsCodes の固定幅（200px）との横並びで余白を生じさせない。
- ratio モードでは #ratioConditions に flex-wrap: wrap、#ratioConditions [data-exclude-markets-slot] に flex: 1 1 100% を指定し、他の条件（出来高倍率・上髭実体比・日付）と同じ行に詰め込まず次行へ折り返す。
- compare-source-fieldset 配下の compare-source-option は display: grid（grid-template-columns: auto 120px minmax(0, 1fr)）で「ラジオ／ラベル文言（.compare-source-label）／対象入力欄（#compareCsvFile・#compareCodes）」を整列し、CSVアップロード・証券コード指定の2行で入力欄の開始位置を縦に揃える。
- 3列目を minmax(0, 1fr) とすることで、track 幅が利用可能幅を超えて広がらないようにしている。
- .mode-conditions label（クラス1+要素1）よりセレクタの詳細度を上げるため、compare-source-fieldset 配下にスコープした compare-source-fieldset .compare-source-option として定義している。
- compare-source-option 配下の input[type="text"] / input[type="file"] は box-sizing: border-box を指定し、width: 100% が padding・border を含めて track 幅にぴったり収まるようにしている（box-sizing 未指定の content-box のままだと padding・border 分だけ右側にはみ出すため）。
- また #compareConditions（mode-conditions の個別 ID）には align-items: flex-start を指定し、入力方法（compare-source-fieldset）と比較期間（compare-date-fieldset）の高さが異なる場合に、デフォルトの align-items: stretch によって低い方のフィールドセットが引き伸ばされ、内部要素の上部に余計な余白が生じるのを防いでいる。
- なお compare-source-option は現状 1つの <label> がラジオボタンと入力欄（#compareCsvFile / #compareCodes）の2つのフォームコントロールを内包しており、同種のアクセシビリティ改善の余地が残っている（未対応・既知の課題）

##### `buttons`

- primary / secondary / ghost / small ボタンのスタイル。
- .buttons-row の margin-top は 28px（2026-07 に16pxから拡大し、条件パネルとスクリーニング開始ボタンの間の余白を確保した）

##### `resultSectionHeader`

- .result-section-header（h2・resultCount・compareMatchRate・csvDownloadBtn を横並びにするフレックスコンテナ）と .csv-download-btn（右端配置・ホバースタイル・disabled スタイル）。
- #compareMatchRate（.compare-match-rate）は compare モード・予測あり時のみ表示し、それ以外は hidden クラスで非表示にする

##### `table`

- 固定ヘッダ方式のテーブル構造と列幅同期（table を 2 枚使用し、列幅は screening.js 側で同期）。
- 固定列の視覚スタイルは class="fixed-col" が担い、JS からの固定列セル特定には data-fixed-col（値なしのフック属性）を用いる（screening.js の syncFixedColumns() を参照）。
- 2枚のテーブルそれぞれを包む div（.table-header-sticky / .table-scroll-outer）は #resultTableStickyWrap / #resultTableScrollOuter という id を持ち、スクロール同期（screening.js）はこの id を用いて要素を取得する。
- class（.table-header-sticky / .table-scroll-outer）はスタイル定義専用でありJSからは参照しない。
- なお .table-scroll-body という類似名のルールが style.css に残存していたが、HTML/JS のどこからも参照されない孤立ルール（.table-scroll-outer へのリネーム時の残骸と推定）だったため 2026-07 の監査で削除した

##### `loadingOverlay`

最前面のローディング表示

##### `modal`

チャートモーダルのレイアウト・ヘッダー・ボディ

##### `timeframeSelector`

日足/週足/月足の切替 UI

##### `chartSettings`

チャート設定ポップアップ

##### `chartLegend`

- チャート凡例の位置・背景・影。
- スマホ（600px以下）では display: none で非表示にする

##### `heuristicsTable`

- heuristics テーブルのトレンド色分け（th-bg-trend-up / th-bg-trend-down / th-bg-trend-either）・行背景（tr-trend-up / tr-trend-down）・適用ルールハイライト（td-heuristics-applied）。
- TECH_* 判定の ○ 表示には .tech-true（screening.js が truthy 時にのみ付与。scripts.screening.behavior.techValueToText を参照）を用いる。
- 対をなす .tech-false（および design token の --color-tech-false）は style.css に定義されていたが、screening.js が × 側では span を生成せず class を一切付与しないため常に未適用のデッドコードだった。
- × に色付けする仕様は存在しないと判断し、2026-07 の監査で削除した。
- 行 hover 時（#resultTable tr:hover）は、常に hover 背景色（--color-bg-table-hover）を tr-trend-up/down の行背景（.fixed-col）・td-heuristics-applied の適用ルールハイライトより手前に表示する。
- 従来 #resultTable tr:hover のセレクタ対象が .fixed-col のみで .td-heuristics-applied を含んでいなかったため、TECH_* 列では hover 時にも td-heuristics-applied の背景色が変化せず残っていた（2026-07 修正）。
- heuristics モードの固定列群（コード/銘柄名/トレンド/スコア）の末尾＝スコア列には、screening.js が th/td に付与する data-fixed-col-last をフックに、tr-trend-up/down に応じたトレンド色の box-shadow: inset（右端アクセント線）を追加する。
- スコア列は他の固定列と同じ position: sticky のため、横スクロール中もスコア列は画面左側に固定表示され続け、アクセント線は常に視認できる（2026-07 追加。compare モードのスコア列は fixed-col ではなく data-fixed-col-last も付与されないため対象外）

##### `responsive`

- 600px 以下での UI 最適化。
- モード切替タブの横スクロール表示（.mode-tabs に overflow-x: auto。.mode-tab には flex-shrink: 0 と white-space: nowrap を指定し、タブが縮小されて内部ラベルが折り返し縦長化するのを防ぐ。2026-07 修正：flex-shrink の既定値（1）のままだとコンテナ幅に収めようとしてタブが縮小し、日本語ラベルが複数行に折り返されて縦長表示になる不具合があったため追加）・条件パネルの縦積み化・除外市場チェックボックスの折り返し・チャートモーダルのフルスクリーン化・ヘッダの2段折り返し・サブチャート高さ縮小（--rci-macd-height-mobile）・凡例非表示・結果テーブル銘柄名列の固定幅省略表示（ellipsis。固定幅は76px。2026-07 に110pxから縮小し、固定列が専有する横幅を抑えて横スクロール領域を広げた）とその他列の最小幅確保（48px。2026-07 に60pxから縮小し視認できる列数を改善。この min-width: 48px は #resultTable th:not(.fixed-col)・#resultTable td:not(.fixed-col)・.table-header-sticky th:not(.fixed-col) の3箇所に対称に適用する。2026-07 修正：当初 #resultTable td（データ行）・.table-header-sticky th（表示ヘッダ）にしか適用しておらず、#resultTable th（列幅計算専用の非表示ヘッダ #resultHeaderBody）には適用していなかった。この非対称により、JS（syncColumnWidths）が非表示ヘッダから48px未満の実測値（例：日足/週足/月足列で30px）を計測してsticky側へコピーしようとしても、sticky側は自身の min-width: 48px によって強制的に48pxへ張り付けられ、コピーされた小さい値が事実上無視される（CSSの仕様上 min-width は width より優先される）という食い違いが生じ、ヘッダ列とボディ列の境界がスクロール時にずれる不具合の直接原因になっていた。ブラウザDevToolsでの実測（sticky側とbody側で対応するセルの getBoundingClientRect().width を比較）により特定した。なお、この個別列（行1）の幅ズレは、colspanで複数列にまたがるグループヘッダ（行0。例：「移動平均線の傾き」）の幅にも連鎖して伝播していた（colspanセルの幅は配下列幅の合計に自動的に従う仕組みのため）。#resultTable th にも同じ min-width を適用し3箇所を対称にすることで、行0・行1双方の不整合を解消した）による視認列数改善・heuristicsモードのトレンド・スコア列（#resultTable th.fixed-col:nth-child(3)/(4)・#resultTable td.fixed-col:nth-child(3)/(4)）の幅縮小（2026-07 追加。単純なnth-child(3)/(4)ではなく.fixed-colと組み合わせて限定しているのは、3・4列目の意味がモードごとに異なり（ratio: 出来高倍率/上髭実体比、date: 値上がり率/終値、compare: スコア/予測等）、他モードでは3・4列目が.fixed-colを持たないため。トレンド列はmin-width/max-widthとも30px（「トレ」「ンド」で2行折り返しさせる前提。当初34px→46px→30pxと変遷しており、46pxへ広げた際の理由（2行折り返し2行目が見切れる不具合）は、#resultTable tr { height: 22px } がtbody・theadを問わず全trに適用され、heuristicsモードのヘッダ（rowspan=2、2行折り返しあり）の高さまで22pxに固定してしまっていたことが一因だったため、2026-07 に #resultTable tbody tr { height: 22px } とtbody限定に修正した。overflow: hidden の追加を試みたが効果がなかった（2026-07 検証。table-layout: auto かつテーブル自体の幅が制約されていない（.table-scroll-outer が横スクロール前提で overflow-x: auto のため、テーブルは好きなだけ広がれる）状況では、そもそも各列は「コンテンツが必要とする幅（preferred width）」で描画され、CSSのmax-widthは実質的に無視される。overflow: hidden は「箱が中身より小さくされた場合にクリップする」ためのものであり、箱自体が縮められていない今回のケースでは無効だった。この誤った対策（.result-table thead tr th への overflow: hidden 追加）は撤回した）。正しい対処として、見出しテキスト自体に <br class="mobile-line-break"> を仕込み、「トレ」「ンド」の間で改行することで「必要とする幅」自体を物理的に短くする方式に変更した（2026-07 修正。.mobile-line-break は既定 display: none（PC幅では1行表示のまま）、600px以下のメディアクエリ内で display: inline に切り替えて改行を有効化する。stickyThead.innerHTML と bodyThead.innerHTML には同一のHTML文字列が代入されるため、body側・sticky側の両テーブルに同じ<br>が反映され、両テーブルの「必要とする幅」が近い値に揃うことで、スクロール時のヘッダ列・ボディ列のズレの改善も期待できる）。min-width/max-width: 30px（#resultTable th.fixed-col:nth-child(3) / td.fixed-col:nth-child(3)）は<br>と併用する最終的な描画幅の目安・下限として残している。スコア列はデータ切れ防止のためmax-widthを指定せずmin-widthのみ26px（2026-07 修正：ヘッダ「スコア」を縦書き（writing-mode: vertical-rl; text-orientation: upright; font-size: 9px）にしたことで、列幅の必要量が文字数（3文字）ではなく1文字分の幅で済むようになったため、40pxから26pxへ再縮小した。rowspan="2"で確保されている縦方向の余裕（2段ヘッダ分の高さ）を活かしている。縦書き化はヘッダ（th）のみが対象で、データ側のtd（数値）は対象外とし、数値の読みやすさを優先して横書きのまま残している。#resultTable th.fixed-col:nth-child(4), .table-header-sticky th.fixed-col:nth-child(4) の両方に縦書きスタイルを指定する（2026-07 修正。当初 #resultTable 側のみに指定していたが、これは列幅計算専用の非表示ヘッダ（#resultHeaderBody、aria-hidden="true"）にしか適用されず、実際に画面へ表示されるsticky側ヘッダ（#resultTableSticky／.table-header-sticky）には適用されていなかった。width/left はJS（syncColumnWidths等）がcomputed値をコピーするが、writing-mode等の見た目のプロパティはコピーされないため両テーブルへの個別指定が必要であり、表示側が横書きのまま・非表示側の幅計算だけが縦書き前提で縮小されていたことで、ヘッダとボディの幅の不整合が発生していた。トレンド列の <br class="mobile-line-break"> はstickyThead.innerHTML / bodyThead.innerHTMLに同一のHTML文字列が代入されるため両テーブルに自動的に反映されるが、writing-modeはCSSプロパティのため、このようにセレクタ自体を両テーブルに対して明示する必要がある点で扱いが異なる）。さらにheuristicsモードの技術指標セル（data-rule-key属性を持つ<td>。○/×/空欄のみを表示する列）はmin-width: 30pxに縮小する（2026-07 追加。data-rule-key属性はheuristicsモードの技術指標セルにのみ付与されるため、他モードの数値列（出来高・終値・増減額等、実際に48px程度必要な列）を巻き込まずに縮小できる）・チャート設定ボタン（.chart-settings-btn）のタップ領域拡大（44×44px・bottom余白確保）・compare モード入力欄（#compareCsvFile/#compareCodes/#compareFromDate/#compareToDateSelect）の幅100%化と box-sizing: border-box によるはみ出し防止・入力方法（compare-source-fieldset .compare-source-option）は600px以下で grid-template-columns を auto 120px minmax(0, 1fr)（PC・3列）から auto 1fr（モバイル・2列）に切り替え、対象入力欄（#compareCsvFile・#compareCodes）を grid-column: 1 / -1 で2行目にフル幅展開する・比較期間（.compare-date-fieldset）の縦積み化と矢印（.compare-date-arrow）の90度回転表示（PC幅では「比較元日付 → 比較先日付」の横向き矢印、600px以下では縦積みレイアウトに合わせ transform: rotate(90deg) で縦向き矢印として表示。文字自体は → のまま、見た目のみ回転）・heuristics証券コード入力欄（#heuristicsCodes）の幅100%化を含む

##### `compareMatchRate`

- compare モードの予測合致率ラベル（.compare-match-rate）。
- border-left で #resultCount と視覚的に区切る。
- compare モードかつ予測あり行が1件以上のとき showResults() 内で calcMatchRate() を呼び出し、「合致率：N/M件（R%）　↗ n/m件　↘ n/m件」形式のテキストを設定して hidden クラスを除去する。
- それ以外のモードまたは予測なし（date / 証券コード直接入力）時は textContent を空にして hidden クラスを付与する

## `libraries`

### `xlsx`

#### `file`

libs/xlsx.full.min.js

#### `responsibility`

フロントエンドでの Excel 読み込み用ライブラリ

### `lightweightCharts`

#### `file`

libs/lightweight-charts.standalone.production.js

#### `version`

5.1.0

#### `license`

Apache-2.0

## `scripts`

### `screening`

#### `file`

screening.js

#### `responsibility`

出来高×上髭スクリーニング / 値上がり率ランキング / heuristics の実行と結果テーブル描画

#### `api`

##### `baseUrl`

https://yfinance-api-fe86988c-d3b4-f1c6-640d.onrender.com

##### `endpoints`

###### `dates`

GET /dates（降順の日付リスト）

###### `heuristics_dates`

GET /heuristics_dates（GitHub trees API + Token 認証で heuristics_YYYYMMDD.json の日付一覧を降順で返す）

###### `screening`

###### `path`

GET /screening

###### `modes`

###### `ratio`

###### `query`

- mode=ratio
- volume_ratio=<number>
- shadow_ratio=<number>
- target_date=<yyyymmdd>
- exclude_markets=<カンマ区切りの市場・商品区分>（省略可）

###### `date_ranking`

###### `query`

- mode=date_ranking
- target_date=<yyyymmdd>

###### `heuristics`

###### `query`

- mode=heuristics
- target_date=<yyyymmdd>
- exclude_markets=<カンマ区切りの市場・商品区分>（省略可）
- codes=<証券コード（4桁）をカンマ区切りで複数指定可。省略可>

###### `responsibility`

- バックエンドが GitHub Raw（RAW_HEURISTICS_PREFIX + YYYYMM/heuristics_YYYYMMDD.json）を Token 認証付きで取得し、スコア計算・除外フィルタを適用した上位20件ずつを返却する。
- codes 指定時は該当銘柄のみに絞り込み、上位20件制限を行わず全件返却する

###### `outputFormat`

###### `status`

ok

###### `target_date`

YYYYMMDD

###### `data`

###### `up`

- アップスコア降順の上位20件（codes 指定時は対象銘柄のうちトレンドが up の全件）。
- 各要素にコード・銘柄名・アップスコア・ダウンスコア・applied_up_rules・applied_down_rules・TECH_* を含む

###### `down`

- ダウンスコア降順の上位20件（codes 指定時は対象銘柄のうちトレンドが down の全件）。
- 各要素にコード・銘柄名・アップスコア・ダウンスコア・applied_up_rules・applied_down_rules・TECH_* を含む

###### `codesFilterBehavior`

###### `trendDecision`

codes 指定時、各銘柄のアップスコアとダウンスコアの大きいほうを「トレンド」としてレスポンス要素に設定する

###### `responseStructure`

data.up / data.down という既存の出力フォーマットは変更せず、トレンドに応じて振り分ける

###### `compare`

###### `query`

- mode=compare
- from_date=<yyyymmdd>（比較元日付）
- to_date=<yyyymmdd>（比較先日付）
- codes=<証券コード（4桁）をカンマ区切りで複数指定可>
- source_mode=<ratio|date|heuristics|空>（CSVアップロード時の元モード。証券コード直接入力時は空）

###### `responsibility`

指定された証券コードについて、比較元日付・比較先日付それぞれの終値を yfinance から個別取得し、増減額・増減率・上昇/下降の予測を算出して返却する

###### `predictionLogic`

###### `ratio`

予測=up（固定）

###### `date`

予測=null（予測なし）

###### `heuristics`

from_date 時点の heuristics JSON を取得し、calc_heuristics_score の up/down スコアの大きいほうを予測として設定する

###### `未指定（証券コード直接入力）`

予測=null（予測なし）

###### `dataSource`

###### `closePrice`

data.json は直近10日分のみ保持のため対象外とし、yfinance から個別取得する（fetch_close_price ユーティリティ）

###### `outputFormat`

###### `status`

ok

###### `data`

- 各要素にコード・銘柄名・比較元終値・比較先終値・増減円・増減率・予測を含む配列。
- 終値取得失敗時は該当要素に error を含む

#### `ui`

##### `modes`

###### `ratio`

###### `controls`

- #volumeRatio
- #shadowRatio
- #ratioDateSelect
- #ratioConditions fieldset（除外する市場・商品区分のチェックボックス群）

###### `tableColumns`

- コード
- 銘柄名
- 出来高倍率
- 上髭実体比
- 出来高
- 上髭
- 実体

###### `excludeMarkets`

###### `element`

data-role="exclude-market-checkbox"（JS からの選択キー。命名ルールは conventions.domHookAttributes を参照。旧く付与していた class="exclude-market-checkbox" は対応する CSS ルールが存在せず JS からも参照されない未使用属性だったため、2026-07 の監査で削除済み）

###### `fieldset`

#ratioConditions fieldset（<legend>除外する市場・商品区分</legend> を先頭に持つ。<label> でチェックボックス群を囲わないアクセシビリティ対応済みの構造）

###### `generatedBy`

screening.js の renderExcludeMarketsFieldset("ratioConditions")（heuristics と共通の EXCLUDE_MARKET_OPTIONS / EXCLUDE_MARKET_DEFAULT_CHECKED を参照。legend の生成も同関数が行う。生成するチェックボックスには data-role="exclude-market-checkbox"（JS選択用）のみを付与する。旧く付与していた class="exclude-market-checkbox" は未使用のため 2026-07 に削除済み）

###### `defaultChecked`

- ETF・ETN

###### `values`

- ETF・ETN
- PRO Market
- REIT・ベンチャーファンド・カントリーファンド・インフラファンド
- 出資証券
- グロース（内国株式）
- グロース（外国株式）
- スタンダード（内国株式）
- スタンダード（外国株式）
- プライム（内国株式）
- プライム（外国株式）

###### `disableMethod`

fieldset.disabled プロパティで fieldset 単位で有効・無効を切り替える

###### `behavior`

- ratio モード選択時のみ有効化。
- チェックされた値をカンマ区切りで exclude_markets パラメータとして送信する

###### `date`

###### `controls`

- #dateSelect

###### `tableColumns`

- コード
- 銘柄名
- 値上がり率
- 当日終値
- 前日終値

###### `heuristics`

###### `controls`

- #heuristicsDateSelect
- #heuristicsCodes（証券コード絞り込み。任意入力）
- #heuristicsConditions fieldset（除外する市場・商品区分のチェックボックス群）

###### `tableColumns`

- コード
- 銘柄名
- トレンド（up/down を ↗/↘ で表示）
- スコア
- TECH_* 判定（全項目）

###### `excludeMarkets`

###### `element`

data-role="exclude-market-checkbox"（JS からの選択キー。命名ルールは conventions.domHookAttributes を参照。旧く付与していた class="exclude-market-checkbox" は対応する CSS ルールが存在せず JS からも参照されない未使用属性だったため、2026-07 の監査で削除済み）

###### `fieldset`

#heuristicsConditions fieldset（<legend>除外する市場・商品区分</legend> を先頭に持つ。<label> でチェックボックス群を囲わないアクセシビリティ対応済みの構造）

###### `generatedBy`

screening.js の renderExcludeMarketsFieldset("heuristicsConditions")（ratio と共通の EXCLUDE_MARKET_OPTIONS / EXCLUDE_MARKET_DEFAULT_CHECKED を参照。選択肢・デフォルト値は index.html には存在せず screening.js が唯一のソース。legend の生成も同関数が行う。生成するチェックボックスには data-role="exclude-market-checkbox"（JS選択用）のみを付与する。旧く付与していた class="exclude-market-checkbox" は未使用のため 2026-07 に削除済み）

###### `defaultChecked`

- ETF・ETN

###### `values`

- ETF・ETN
- PRO Market
- REIT・ベンチャーファンド・カントリーファンド・インフラファンド
- 出資証券
- グロース（内国株式）
- グロース（外国株式）
- スタンダード（内国株式）
- スタンダード（外国株式）
- プライム（内国株式）
- プライム（外国株式）

###### `disableMethod`

fieldset.disabled プロパティで fieldset 単位で有効・無効を切り替える

###### `behavior`

- heuristics モード選択時のみ有効化。
- チェックされた値をカンマ区切りで exclude_markets パラメータとして送信する

###### `resultMerge`

data.data.up / data.data.down をそれぞれ トレンド・スコア キーを付与した上でマージして results 配列を生成する

###### `tableRowStyling`

トレンドが up の行に tr-trend-up、down の行に tr-trend-down クラスを付与する

###### `cellHighlight`

applied_up_rules（up行）または applied_down_rules（down行）に含まれる TECH_* キーに対応する td に td-heuristics-applied クラスを付与してハイライト表示する

###### `codesInput`

###### `element`

#heuristicsCodes

###### `behavior`

- 任意入力。
- 証券コード（4桁・カンマ区切りで複数可）が入力されている場合のみ /screening の codes パラメータとして送信し、対象銘柄に絞り込む。
- 未入力時は従来通り全銘柄を対象としたスクリーニングを行う

###### `layout`

- PC では親ラベルを min-width: 200px の固定幅とし、隣接する exclude-markets-fieldset のスロットは flex: 1 1 100%・min-width: 0 で、日付・証券コードとは別の専用行に折り返して表示する（2026-07 修正。従来は flex: 1 1 auto で日付・証券コードと同じ行に配置しようとしており、#heuristicsConditions に flex-wrap の指定が漏れていた（既定の nowrap のまま）ため、除外市場スロットが横方向に圧縮され、内部のラベルテキスト（white-space: nowrap）が欄外にはみ出す不具合があった。#ratioConditions で既に採用していた「除外市場は専用行」のパターンに統一した）。
- 600px 以下では幅100%にして他要素と同様に縦積み表示する

###### `compare`

###### `controls`

- #compareCsvFile（CSVアップロード。input[name=compareSource] が csv のとき有効）
- #compareCodes（証券コード直接入力。input[name=compareSource] が code のとき有効）
- #compareFromDate（比較元日付。<select>。選択肢は /trading_dates から取得した直近3か月の市場開場日。デフォルトは一覧の先頭＝直近の最新市場開場日、CSVアップロード時はそのファイル名の日付を選択状態にする〔一覧に存在しない場合は option を動的追加してから選択する〕。入力方法ラジオを切り替えても再設定はしない）
- #compareToDateSelect（比較先日付。選択肢は #compareFromDate と同じく /trading_dates から取得した直近3か月の市場開場日。デフォルトは一覧の先頭＝直近の最新市場開場日）

###### `tableColumns`

- コード
- 銘柄名
- スコア（CSVスコア列がない場合は -）
- 上昇/下降の予測
- 上昇/下降の結果
- 比較元 DATE終値（ヘッダに比較元日付を含む）
- 比較先 DATE終値（ヘッダに比較先日付を含む）
- 増減（円）
- 増減（％）

###### `sourceRadio`

###### `element`

input[name='compareSource']（csv / code）

###### `behavior`

- csv 選択時は #compareCsvFile を有効化・#compareCodes を無効化。
- code 選択時はその逆。
- スクリーニングに用いる証券コードはこの選択値で決定する

###### `layout`

- compare-source-fieldset 配下の .compare-source-option は display: grid（grid-template-columns: auto 120px minmax(0, 1fr)）でラジオ・ラベル文言・対象入力欄を3列に整列して2択を表示する。
- 600px 以下では grid-template-columns: auto 1fr の2列に切り替え、対象入力欄を2行目へフル幅展開する縦積みレイアウトにする

###### `csvUpload`

###### `fileNamePattern`

screening_<ratio|date|heuristics>_<yyyymmdd>.csv（screening.js の downloadCsv が生成するファイル名と同一形式）

###### `behavior`

- ファイル名から元モード（source_mode）と日付（比較元日付のデフォルト値）を抽出し、CSV本文の「コード」列の値一覧をスクリーニング対象として送信する。
- ファイル名形式が一致しない場合はエラーメッセージを表示してアップロードを無効化する

###### `csvParsing`

- screening.js 内に実装する簡易CSVパーサ（parseCsvLine）でダブルクォート・カンマを含むセルに対応する。
- 新規ライブラリは追加しない

###### `csvBodyParsing`

- extractCodesFromCsv() / parseCsvLine() で CSV 本文の「コード」列の値一覧を抽出し、/screening の codes パラメータとして送信する。
- ヘッダ行に「スコア」列が存在する場合は uploadedCsvScoreMap（コード→スコアのマップ）にも格納し、compare 結果テーブルのスコア列表示に使用する。
- ファイル名形式が一致しない場合は uploadedCsvScoreMap をリセットした上でアップロードを無効化する

###### `predictionDisplay`

- 予測カラムは formatDirectionMark() で ↗/↘/－ に変換して表示する。
- null の場合は空白（-）表示

###### `responsiveLayout`

#compareCsvFile / #compareCodes / #compareFromDate / #compareToDateSelect は600px以下で width: 100% にし、画面幅に応じて入力欄を伸縮させる

###### `tableRowStyling`

- 予測（r.予測）が 'up' の行に tr-trend-up、'down' の行に tr-trend-down クラスを付与する（heuristics モードと同一 CSS クラスを流用）。
- 予測が null（date / 証券コード直接入力）の行にはクラスを付与しない

###### `cellStyling`

###### `予測グループ（スコア・上昇/下降の予測）`

- 予測が 'up' の場合 background-color: var(--color-trend-up-bg)、'down' の場合 var(--color-trend-down-bg) をインラインスタイルで設定する。
- 予測が null の場合は style 属性を出力しない（白背景）

###### `結果グループ（上昇/下降の結果・比較元終値・比較先終値・増減（円）・増減（％））`

上昇/下降の結果が '↗' の場合 background-color: var(--color-trend-up-bg)、'↘' の場合 var(--color-trend-down-bg)、'横ばい' の場合 var(--color-bg-table-row) をインラインスタイルで設定する

###### `tableHeaderDateLabel`

- 「比較元終値」「比較先終値」列のヘッダには、比較元・比較先の日付を含める。
- updateTableHeader() に compareFromLabel / compareToLabel（makeDateLabel() で生成した「YYYY/MM/DD（曜）」形式）を渡し、「比較元 YYYY/MM/DD（曜）終値」「比較先 YYYY/MM/DD（曜）終値」と表示する

###### `scoreDisplay`

- スコア列には uploadedCsvScoreMap[コード] の値を表示する。
- CSVアップロード時に extractCodesFromCsv() がスコア列（ヘッダ名「スコア」）を検出した場合のみマップに格納する。
- スコア列が存在しない CSV または証券コード直接入力時は '-' を表示する

###### `matchRateDisplay`

- showResults() 完了後に calcMatchRate(results) を呼び出し、#compareMatchRate へ合致率を表示する。
- 予測あり行（r.予測 != null かつ r.error なし）が0件の場合は null を返し、#compareMatchRate を hidden にする

##### `heuristicsTypes`

###### `description`

HEURISTICS_TYPES 配列で各テクニカル指標グループを定義する

###### `fields`

###### `group_label`

グループ名（ヘッダ1行目に表示）

###### `trend_direction_css`

- グループヘッダの背景色クラス（th-bg-trend-up / th-bg-trend-down / th-bg-trend-either）。
- 省略可

###### `items`

- グループ内の各指標。
- key（TECH_* キー名）・label（ヘッダ2行目）・trend_direction_css（item個別の色クラス、省略可）を持つ

###### `headerRendering`

- itemCount > 1 の場合は colspan でグループヘッダを結合。
- item が1件かつ group_label === item.label の場合は rowspan=2 で1行に収める。
- それ以外は上段にグループ名・下段にitem.label を分けて表示する

###### `cellRendering`

###### `string（up/down/flat）`

formatDirectionMark() で ↗/↘/－ に変換して表示

###### `boolean`

- boolMark() で ○/× に変換して表示。
- boolMark は true→○、false→×、それ以外→空文字

###### `酒田五法（1 or null）`

val が truthy なら <span class="tech-true">○</span>、null なら空白

###### `TECH_RULE9_DAILY / TECH_RULE9_WEEKLY`

{ direction, count } → formatDirectionMark(direction)＋（count 本目）形式

###### `TECH_GRANVILLE`

{ direction, count } → formatDirectionMark(direction)＋（count）形式

###### `TECH_FUSHIME_UP / TECH_FUSHIME_DOWN`

{ price, tryCount } → price（tryCount回）形式

###### `TECH_CYCLE_PROGRESS`

- { direction, count, startDate, lastDate } → formatDirectionMark(direction)＋（count日）形式。
- null なら空白

##### `elements`

###### `startBtn`

#startBtn

###### `cancelBtn`

#cancelBtn

###### `elapsedTime`

#elapsedTime

###### `tbody`

#resultTable tbody

###### `dateSelect`

#dateSelect

###### `ratioDateSelect`

#ratioDateSelect

###### `heuristicsDateSelect`

#heuristicsDateSelect

###### `heuristicsCodes`

#heuristicsCodes（heuristics モードの証券コード絞り込み・任意入力）

###### `compareCsvFile`

#compareCsvFile（compare モードのCSVアップロード）

###### `compareCodes`

#compareCodes（compare モードの証券コード直接入力）

###### `compareFromDate`

#compareFromDate（compare モードの比較元日付・<select>。選択肢は /trading_dates から動的構築）

###### `compareToDateSelect`

#compareToDateSelect（compare モードの比較先日付）

###### `loadingOverlay`

#loadingOverlay

###### `resultCount`

#resultCount

###### `csvDownloadBtn`

#csvDownloadBtn（スクリーニング結果の CSV ダウンロードボタン。結果が 0 件の間は disabled。#resultSection 右上に配置）

###### `resultSection`

#resultSection（スクリーニング完了後のスクロール対象）

###### `compareMatchRate`

#compareMatchRate（compare モードの予測合致率ラベル。compare モード・予測あり時のみ表示。それ以外は hidden クラスで非表示）

###### `resultTableStickyWrap`

#resultTableStickyWrap（固定ヘッダテーブルを包む div。スクロール同期（syncColumnWidths/syncFixedColumns 実行後のスクロール連動）の取得対象。旧 .table-header-sticky class セレクタから変更）

###### `resultTableScrollOuter`

#resultTableScrollOuter（本体テーブルを包む横スクロール div。スクロール同期の取得対象。旧 .table-scroll-outer class セレクタから変更）

###### `fixedColHook`

- [data-fixed-col]（固定列の th/td を特定するための値なしフック属性。表示スタイルは引き続き class="fixed-col" が担う。旧 nth-child + class セレクタから変更。詳細は conventions.domHookAttributes を参照）。
- [data-fixed-col-last]（固定列群の末尾セルの目印。heuristics モードのスコア th/td にのみ付与し、右端のトレンド色アクセント線の対象特定に使う。2026-07 追加）

#### `behavior`

##### `init`

window.onload で renderExcludeMarketsFieldsets（initSearchMode より前に実行し、除外市場 fieldset を DOM に生成してから有効・無効の制御対象にする） → initSearchMode → loadDates → loadTradingDates → loadHeuristicsDates の順に実行

##### `modeSwitch`

- 見た目は .mode-tabs 配下のタブ UI（searchMode ラジオボタンを非表示化して .mode-tab をタップ領域にする）。
- 表示するモードパネル（.mode-panel）の切替は style.css の :has() セレクタが CSS のみで行う。
- フォーム要素の有効・無効切替は従来通り JS（initSearchMode）が ratio/date/heuristics/compare それぞれの入力欄・fieldset に対して行う（DOM 構造が変わっても対象のセレクタ・ID は変更していないため、ロジック自体は無改修）。
- heuristics モード時は #heuristicsConditions fieldset を fieldset.disabled で有効化する。
- compare モード時は compareSource ラジオ（csv/code）の選択値に応じて #compareCsvFile / #compareCodes の有効・無効をさらに切り替える

##### `startScreening`

入力検証 → テーブルヘッダ更新 → 経過時間カウンタ開始 → /screening 呼び出し → showResults → 列幅同期 → #resultSection へスムーズスクロール

##### `cancelScreening`

AbortController.abort() で fetch を中断

##### `sorting`

固定ヘッダの th[data-sort-key] クリックで currentResults を昇順/降順ソート

##### `rowClick`

行クリックで openChartModal(コード, 銘柄名, index) を呼び出し

##### `resultSharing`

window.setScreeningResults が関数なら、結果配列を渡して呼び出す

##### `fixedHeader`

- #resultTableStickyWrap と #resultTableScrollOuter（旧 .table-header-sticky / .table-scroll-outer class セレクタから id セレクタへ変更）の scrollLeft を双方向同期し、列幅は body 側 thead の th 実測幅（tHead.rows / row.cells によるライブコレクション参照。querySelectorAll は不使用）で sticky 側 thead を同期し、fixed-col は syncFixedColumns で left を再計算する。
- syncFixedColumns は本体テーブルの全行を1回だけ走査し、行ごとに row.cells[colIndex] で固定列セルへ直接アクセスして left を設定する（従来の『固定列数 × 全行』ぶん document.querySelectorAll(":nth-child") をテーブル全体に対して呼び出す実装から、パフォーマンス改善のため変更した。固定ヘッダ側は [data-fixed-col] の出現順で本体の固定列と対応付ける）。
- 同期処理（syncColumnWidths / syncFixedColumns）は showResults 完了後の afterTableRendered（setTimeout(0) 二段構え）・window の resize イベントに加えて、document.fonts.ready 解決時にも再実行する。
- 自前ホストの Webフォント（assets/fonts/fonts.css）の読み込み完了（フォントスワップ）が afterTableRendered の同期タイミングより遅れた場合、スワップ後の文字幅変化でヘッダー列とデータ列の幅がズレたまま残ることがあったため、2026-07 に追加した（フォントがキャッシュ済みの場合は document.fonts.ready が同期完了時点で既に解決済みのため実質無処理。さらに #resultTable（本体テーブル）を対象に ResizeObserver を設置し、フォントスワップ・ページの縦スクロールバー出現など原因を問わずボックスサイズが変化した場合に syncColumnWidths / syncFixedColumns を再実行する保険的な同期を追加した（2026-07 追加。初回スクリーニング時にヘッダ列とデータ列の幅がわずかにずれ、2回目以降のスクリーニングでは一致するという再現性の低い不具合に対する、原因を特定せず症状を解消する頑健な対策。既存の setTimeout 二段構え・document.fonts.ready による同期は変更せず、追加の安全網として動作する。ResizeObserver は style.width / style.left の書き込みのみを行う #resultTable 自身を監視するため、無限ループは発生しない））
- テーブルの角丸（トレイ表現）は .table-well ではなく .table-header-sticky（上2角）・.table-scroll-outer（下2角）にそれぞれ border-radius: var(--radius-input) を持たせる（2026-07 追加。.table-well（.table-header-sticky の祖先）に overflow: hidden を付けると position: sticky の基準祖先が変わり固定ヘッダの位置がずれるため、.table-well 自体はクリップできない設計になっている。そのため角丸のクリップは内側の .table-header-sticky（既存の overflow-x: auto に加え、border-radius によるクリップを効かせるため overflow-y: hidden を明示追加）と、既に overflow-y: hidden を持つ .table-scroll-outer にそれぞれ持たせている）
- syncColumnWidths（列幅コピー）・syncFixedColumns（固定列leftオフセット算出）は、getBoundingClientRect().width の実測値を小数のままコピー・累積する（2026-07 に一度 Math.round() で整数pxへ丸める対策を試みたが、後日撤回した。当初は「小数のまま body 側テーブルと sticky 側テーブルという独立した2つのテーブルにそれぞれ適用すると、ブラウザ内部の描画上の丸め処理がテーブルごとに独立して働き、わずかなズレが蓄積する」という仮説でMath.round()を追加したが、実機のDevTools実測（sticky側とbody側で対応するセルの getBoundingClientRect().width を比較する診断スクリプトを実行）により、真因は #resultTable th:not(.fixed-col) の min-width 適用漏れ（別項を参照。修正済み）であったと判明した。この状態でなお Math.round() を使うと、colspanセル（グループヘッダ）は「配下の行1サブ列（丸め済み整数）の合計」になる一方、body側は丸めていない小数のままの自然な合計であるため、丸め処理自体が新たな数px単位のズレの原因になっていた（例："酒田五法"列で2px、"パターン"列で0.99px）。Math.round() を撤去し実測値を小数のままコピーすることで、sticky側とbody側の実測値が完全に一致し、colspanセルの合計値も自然に一致するようにした。見出しテキストの必要幅自体を縮める <br class="mobile-line-break"> による強制改行（トレンド列。詳細は controls.responsive のトレンド・スコア列の項を参照）は、この修正後も引き続き有効な対策として残している）
- syncColumnWidths は、colSpan > 1 を持つセル（heuristicsモードのグループヘッダ。例：「移動平均線の傾き」が日足/週足/月足の3列にまたがる）自体には幅を設定しない（2026-07 修正。従来はグループヘッダ自身の幅も配下の個別列（日足/週足/月足等）の幅も、それぞれ独立に測定してMath.round()で丸めていたが、「グループヘッダの幅」と「配下列の幅の合計」は本来一致するはずの値であるにもかかわらず、それぞれを独立に四捨五入すると丸め誤差により合計が1px単位で食い違うことがあり、これがグループヘッダ境界での視覚的なズレの原因になっていた。colSpan > 1 のセルには幅を設定せず、HTMLテーブルの標準機能（colspanセルの幅は自動的に配下列幅の合計になる）に任せることで、この種の丸め誤差を構造的に排除した。rowspan="2"の単独列（コード・銘柄名・トレンド・スコア等、colSpanは1）は従来通り個別に幅を設定する）
- 2段ヘッダの装飾border（border-bottom: 2px double var(--color-accent-primary)）は上段（.result-table thead tr:first-child th）・下段（.result-table thead tr:nth-child(2) th）の両方に指定する（2026-07 修正。従来は下段に border-bottom の指定が無く、.result-table th, .result-table td の共通ルール（border-bottom: 1px solid var(--color-border)）にフォールバックしていたため、上段だけ装飾され下段は装飾されていないように見える不統一があった）
- #resultTable tbody tr の height: 22px（モバイル用）は tbody 内の行のみに限定する（2026-07 修正。従来は #resultTable tr として thead 側の行にも適用されており、heuristicsモードのヘッダ（rowspan="2"、2行折り返しあり）の高さまで22pxに固定されてしまい、折り返した2行目の文字が見切れる不具合の一因になっていた）
- #resultTable th（列幅計算専用の非表示ヘッダ #resultHeaderBody 内の th）にも、#resultTable td と同じモバイル用 font-size: 10px; padding: 4px 3px; を適用する（2026-07 追加。従来は #resultTable td のみにモバイル用の縮小が適用されており、同じ #resultTable 内の非表示ヘッダ行にはデスクトップサイズ（padding: 8px 6px 等）のまま適用されていなかった。table-layout: auto は同じ列のヘッダ行・データ行のうち最も広い幅を必要とするもので列幅を決めるため、この不一致により列幅計算がヘッダ側の大きめの要求に引っ張られ、JSがコピーする幅が sticky側テーブルと食い違う一因になっていた。nth-childを伴うより具体的なルール（トレンド列・スコア列等）は詳細度が高いため、本ルールでは上書きされず個別の指定が優先される）

##### `getExcludeMarkets`

- getExcludeMarkets(scopeId): 指定したコンテナ（"ratioConditions" または "heuristicsConditions"）配下の [data-role="exclude-market-checkbox"]:checked の value をカンマ区切りで返す関数（旧 .exclude-market-checkbox class セレクタから data-role セレクタへ変更。命名ルールは conventions.domHookAttributes を参照）。
- ratio / heuristics で同じ data-role を共有しているため、DOM 全体ではなくコンテナ単位でスコープする。
- それぞれのモードの fetch 前に呼び出し、1件以上の場合のみ exclude_markets パラメータに設定する

##### `renderExcludeMarketsFieldsets`

- renderExcludeMarketsFieldset(containerId): 定数 EXCLUDE_MARKET_OPTIONS（選択肢一覧）・EXCLUDE_MARKET_DEFAULT_CHECKED（デフォルトチェック値）から fieldset.exclude-markets-fieldset を動的生成し、指定コンテナ内の [data-exclude-markets-slot] へ挿入する。
- fieldset の先頭には <legend>除外する市場・商品区分</legend> を追加する（グループを <label> で囲むと「フォームフィールドに紐付かない label」としてアクセシビリティ監査で警告されるため、legend でグループ名を表現する）。
- ratio / heuristics 共通のロジックとして screening.js に一元定義し、index.html 側は空のスロット（<div data-exclude-markets-slot></div>。<label> では囲わない）のみを持つ。
- renderExcludeMarketsFieldsets() が両モード分をまとめて呼び出す。
- window.onload の先頭（initSearchMode より前）で実行する必要がある

##### `heuristicsCodesInput`

#heuristicsCodes に値が入力されている場合のみ /screening の codes パラメータとして送信し、heuristics モードの対象銘柄を絞り込む

##### `compareCsvUpload`

###### `trigger`

#compareCsvFile の change イベント

###### `fileNameParsing`

- screening_<ratio|date|heuristics>_<yyyymmdd>.csv 形式のファイル名から source_mode と日付を抽出する。
- setCompareFromDate() が #compareFromDate（<select>）の選択肢一覧に該当日付が存在するか確認し、存在しなければ option を動的追加した上で選択状態にする。
- 形式が一致しない場合はアラート表示の上アップロードを無効化する

###### `csvBodyParsing`

- extractCodesFromCsv() / parseCsvLine() で CSV 本文の「コード」列の値一覧を抽出し、/screening の codes パラメータとして送信する。
- 同時にヘッダ行の「スコア」列（heuristics CSV のみ存在）を検出して uploadedCsvScoreMap（コード→スコア）を構築する。
- ファイル名形式不一致時は uploadedCsvScoreMap = {} でリセットする

##### `compareScreening`

- compare モードの /screening 呼び出しでは mode=compare・from_date・to_date・codes・source_mode を送信する。
- source_mode は compareSource が code（証券コード直接入力）の場合は空文字を送信する。
- 証券コード直接入力時は uploadedCsvScoreMap = {} でスコアマップをリセットする。
- showResults() 完了後に calcMatchRate(results) を呼び出して #compareMatchRate へ合致率を表示する

##### `csvDownload`

###### `trigger`

#csvDownloadBtn クリック → downloadCsv() を呼び出す

###### `enableCondition`

- showResults() 完了時に results.length > 0 であれば #csvDownloadBtn の disabled を解除する。
- results.length === 0 の場合は disabled のまま

###### `dataSource`

currentResults（ソート後の状態を反映）

###### `fileName`

screening_<mode>_<YYYYMMDD>.csv（YYYYMMDD はスクリーニングに指定した対象日。モードごとの日付セレクタ値〔ratioDateSelect / dateSelect / heuristicsDateSelect〕を使用する。compare モードはサポート対象外〔CSVダウンロード機能は ratio/date/heuristics の3モードのみ〕）

###### `encoding`

BOM 付き UTF-8（Excel での文字化け防止）

###### `lineEnding`

CRLF（RFC 4180 準拠）

###### `headerFormat`

###### `1行ヘッダ（ratio / date モード）`

- 列名をそのまま出力。
- 例: コード,銘柄名,出来高倍率,...

###### `2行ヘッダ（heuristics モード）`

- 「1行目（2行目）」形式で1行に統合して出力。
- group_label === item.label の場合（rowspan=2 列）はグループ名のみ。
- 複数 item / group_label !== item.label の場合は「グループ名（item.label）」形式

###### `cellValueFormat`

techValueToText() で画面表示と同等のテキストに変換（↗/↘/－・○/×・「price（tryCount回）」等）

###### `escaping`

- RFC 4180 準拠。
- ダブルクォート・カンマ・改行を含むセルはダブルクォートで囲み、セル内のダブルクォートは "" にエスケープする

##### `loadHeuristicsDates`

全日付をそのまま option に追加する（「最新を使用」オプションなし）

##### `updateTableHeader`

- updateTableHeader(mode, label = '', compareFromLabel = '', compareToLabel = '') → void。
- compare モード時は compareFromLabel / compareToLabel を受け取り、「比較元 YYYY/MM/DD（曜）終値」「比較先 YYYY/MM/DD（曜）終値」形式でヘッダを生成する。
- startScreening() が makeDateLabel(fromDate) / makeDateLabel(toDate) を生成して渡す

#### `csvExportFunctions`

##### `description`

- CSV ダウンロード機能を構成する関数群。
- screening.js 内に実装する

##### `escapeCsvCell`

###### `signature`

escapeCsvCell(value) → string

###### `responsibility`

- RFC 4180 準拠の CSV セルエスケープ。
- ダブルクォート・カンマ・改行を含む場合はダブルクォートで囲み、内部のダブルクォートを "" に変換する

##### `techValueToText`

###### `signature`

techValueToText(key, val) → string

###### `responsibility`

- heuristics モードの TECH_* 値を画面表示と同等のプレーンテキストに変換する。
- specifications の cellRendering 定義に完全準拠

###### `note`

- showResults 内の switch 文と同一ロジックを持つ。
- 両者の乖離が生じた場合は両方を同期して修正すること

##### `buildCsvHeaders`

###### `signature`

buildCsvHeaders(mode, label) → string[]

###### `responsibility`

- 表示モードに応じた CSV ヘッダ行（1行）を生成する。
- heuristics モードは HEURISTICS_TYPES を走査して「グループ名（item.label）」形式に変換する。
- compare モードは画面表示と同一の列順（スコア・上昇/下降の予測・上昇/下降の結果・比較元終値・比較先終値・増減（円）・増減（％））でヘッダを返す。
- csvDownloadBtn は compare モードで disabled のため実際のダウンロードは発生しないが、関数としては対応済み

##### `buildCsvRow`

###### `signature`

buildCsvRow(r, mode) → string[]

###### `responsibility`

- 結果オブジェクト1件分のセル値配列を返す。
- heuristics モードは HEURISTICS_TYPES 全 key を techValueToText() 経由で変換する。
- compare モードは calcCompareResult() で上昇/下降の結果を算出し、uploadedCsvScoreMap からスコアを取得して画面表示と同一列順で出力する。
- csvDownloadBtn は compare モードで disabled のため実際のダウンロードは発生しないが、関数としては対応済み

##### `downloadCsv`

###### `signature`

downloadCsv() → void

###### `responsibility`

- currentResults を CSV 文字列に変換し、Blob + <a> 要素でダウンロードをトリガーする。
- Object URL は revokeObjectURL() で即時解放する

##### `makeDateLabel`

###### `signature`

makeDateLabel(d) → string

###### `responsibility`

- YYYYMMDD 形式の日付文字列を「YYYY/MM/DD（曜）」形式に変換するユーティリティ。
- compare モードのテーブルヘッダ日付表示（updateTableHeader）に使用する。
- makeOption() と同一ロジックを持つ独立関数（makeOption の戻り値は Option 要素のため共用しない）

##### `calcCompareResult`

###### `signature`

calcCompareResult(fromClose, toClose) → string

###### `responsibility`

- 比較元終値・比較先終値から上昇/下降の結果を算出する純粋関数。
- toClose > fromClose → '↗'、toClose === fromClose → '横ばい'、それ以外 → '↘' を返す。
- showResults()（テーブル描画）と buildCsvRow()（CSV出力）の両方から呼び出す

##### `calcMatchRate`

###### `signature`

calcMatchRate(results) → { up, down, all } | null

###### `responsibility`

- compare モードの予測合致率を算出する純粋関数。
- r.error なし・r.予測 != null・r.比較元終値 != null・r.比較先終値 != null の行のみ集計対象とする。
- 横ばいは合致に含めない（予測は ↗/↘ のみ）。
- 予測あり行が0件の場合は null を返す。
- 戻り値: { up: {matched, total}, down: {matched, total}, all: {matched, total, rate（%整数）} }

###### `usedBy`

showResults()（compare モード完了後に #compareMatchRate へ表示）

### `chartIndicators`

#### `file`

js/chart-indicators.js

#### `responsibility`

- チャート用インジケータ計算モジュール（MA / BB / RCI / MACD）。
- ES Module として calcMA / calcBB / calcRCI / calcMACD を export する（calcEMA は calcMACD 内部専用のため非公開）

#### `moduleType`

- ESM（type="module"）。
- chart-price.js / chart-rci.js / chart-macd.js から import される

#### `functions`

##### `calcMA`

###### `input`

data[{time, close}]

###### `output`

SMA（期間可変）

###### `usedBy`

chart-price.js（import）

##### `calcBB`

###### `period`

`20`

###### `sigma`

`2`

###### `output`

- mid
- upper
- lower

###### `usedBy`

chart-price.js（import）

##### `calcRCI`

###### `period`

`9`

###### `logic`

順位相関係数

###### `usedBy`

chart-rci.js（import）

##### `calcEMA`

###### `purpose`

MACD 計算の基礎

###### `usedBy`

calcMACD（同一ファイル内。export しない）

##### `calcMACD`

###### `shortPeriod`

`12`

###### `longPeriod`

`26`

###### `signalPeriod`

`9`

###### `output`

- macdData
- signalData
- histData

###### `usedBy`

chart-macd.js（import）

#### `history`

- calcIchimoku は本ファイルにも同名の実装が存在したが、chart-price.js 側の実装とスクリプト読み込み順の関係で常に上書きされ、一度も実行されないデッドコードだったため 2026-07 の監査で削除した。
- 一目均衡表の計算は chart-price.js の内部関数 calcIchimoku（非 export）に一本化されている。
- 詳細は chartPrice.functions.calcIchimoku を参照

### `chartData`

#### `file`

js/chart-data.js

#### `responsibility`

バックエンドAPIからOHLCVを取得し、Lightweight Charts用のcandleData形式に整形する

#### `api`

##### `endpoint`

/chart

##### `baseUrl`

https://yfinance-api-fe86988c-d3b4-f1c6-640d.onrender.com

##### `queryParams`

- ticker
- timeframe

#### `input`

##### `ticker`

例: '7203' → API 内部で '7203.T' に変換

##### `timeframe`

例: '1d', '1wk', '1mo'

#### `output`

##### `type`

candleData[]

##### `fields`

- time(UNIX秒)
- open
- high
- low
- close
- volume

#### `behavior`

##### `sort`

日付キー（YYYY-MM-DD）を昇順ソート

##### `convertTime`

Date → UNIX秒（ミリ秒→秒）

##### `errorHandling`

- json.Close が無い場合は指数バックオフ（初回 1500ms・以降 2倍）で最大 FETCH_MAX_RETRIES（3）回リトライし、全試行失敗時に null を返す
- fetch エラー時も同様にリトライし、全試行失敗時に null を返す

#### `usedBy`

chart-main.js（drawChart → fetchChartData。import）

#### `moduleType`

- ESM（type="module"）。
- fetchChartData を export し、chart-main.js から import される

### `chartPrice`

#### `file`

js/chart-price.js

#### `responsibility`

ローソク足 + 出来高 + MA + BB + 一目均衡表の描画

#### `moduleType`

ESM（type="module"）

#### `imports`

chart-indicators.js から calcMA / calcBB を import

#### `exports`

##### `createPriceChart`

- createPriceChart(priceChart, chartContainer, candleData)。
- chartContainer は旧実装ではファイル内から chart-main.js のグローバル変数を直接参照していたが、ES Modules 化に伴い引数として明示的に受け取る形に変更した

##### `setShowCandles / setShowMA / setShowBB / setShowIchimoku`

- 各表示トグル用セッター関数。
- 呼び出すと内部フラグ（showCandles 等）を更新した上で対応する apply*Visibility() を実行する。
- 旧実装では chart-main.js のチェックボックス変更イベント内で `showCandles = e.target.checked` のようにグローバル変数へ直接代入していたが、ES Modules 化によりモジュール外からの変数直接代入ができなくなったため、これらのセッター経由に変更した

#### `functions`

##### `calcIchimoku`

###### `scope`

本ファイル内のみで完結する非公開関数（export しない）

###### `components`

- tenkan(9)
- kijun(26)
- span1(転換線+基準線の平均を26本先)
- span2(52期間高値安値の平均を26本先)
- chikou(26本前)

###### `usedBy`

createPriceChart（同一ファイル内）

###### `note`

旧 chart-indicators.js にも同名の重複実装が存在したが、常に本ファイルの実装で上書きされ旧実装は未実行だった（2026-07 に旧実装を削除。詳細は chartIndicators.history を参照）

#### `indicatorParameters`

##### `movingAverages`

calcMA（chart-indicators.js から import）を 5 / 25 / 50 / 75 / 100 の5期間で呼び出す（ma5〜ma100Series）

##### `bollingerBands`

calcBB（chart-indicators.js から import）を period=20・k(σ)=2 で呼び出す（chartIndicators.functions.calcBB の sigma と同一パラメータ）

### `chartRci`

#### `file`

js/chart-rci.js

#### `responsibility`

RCI(9) と RCI(26) の描画

#### `moduleType`

ESM（type="module"）

#### `imports`

chart-indicators.js から calcRCI を import

#### `exports`

##### `createRciChart`

- createRciChart(rciContainer, candleData) → { chart }。
- rciContainer は引数として受け取る（旧実装は chart-main.js のグローバル変数を直接参照）。
- 内部の chart インスタンス・rciShortSeries・rciLongSeries は本関数内のローカル変数として宣言する（旧実装は宣言なしの暗黙グローバル代入だったが、ES Modules は strict mode 必須のため明示宣言に変更）

### `chartMacd`

#### `file`

js/chart-macd.js

#### `responsibility`

MACD + Signal + Histogram の描画

#### `moduleType`

ESM（type="module"）

#### `imports`

chart-indicators.js から calcMACD を import

#### `exports`

##### `createMacdChart`

- createMacdChart(macdContainer, candleData) → { chart }。
- chartRci と同様の理由で macdContainer を引数化し、chart・macdLineSeries・macdSignalSeries・macdHistSeries を関数内ローカル変数として明示宣言する

### `chartSync`

#### `file`

js/chart-sync.js

#### `responsibility`

チャート同期・リサイズ処理・初期表示範囲

#### `moduleType`

ESM（type="module"）

#### `exports`

##### `bindTimeSync`

bindTimeSync(srcChart, targetCharts)

##### `setupResize`

- setupResize(priceChart, rciChart, macdChart, chartContainer, rciContainer, macdContainer)。
- 3つの container 要素は旧実装では chart-main.js のグローバル変数を直接参照していたが、引数化した

##### `applyDefaultRange`

applyDefaultRange(priceChart, rciChart, macdChart, candleData)

#### `internalState`

- isSyncing（同期処理の再帰防止フラグ）は本ファイル内に閉じたモジュールスコープ変数とする。
- 旧実装では chart-main.js のトップレベル変数をグローバルスコープ共有で参照していたが、本ファイル内でのみ使用するためモジュール化に伴い内部化した

### `chartMain`

#### `file`

js/chart-main.js

#### `responsibility`

モーダル制御・チャート描画の司令塔

#### `moduleType`

- ESM（type="module"）。
- index.html から <script type="module" src="js/chart-main.js"> の1本のみで読み込まれるエントリーポイント。
- chart-data.js / chart-price.js / chart-rci.js / chart-macd.js / chart-sync.js を import し、依存関係の解決はモジュール側（ブラウザの ESM ローダー）に委ねる

#### `loadingOverlay`

サーバ通信（fetchChartData）を伴う描画はすべて drawChart() を経由し、drawChart() の先頭でオーバーレイを表示・finally ブロックで非表示にすることで、呼び出し元によらずローディング表示を保証する

#### `viewportHeightFix`

- updateVh(): window.innerHeight * 0.01 を CSS カスタムプロパティ --vh として documentElement に設定する（iOS Safari 等でアドレスバーの表示・非表示により 100vh の実際の高さが変動する問題への対策）。
- モジュール読み込み時に1回実行した上で window の resize イベントでも再計算する。
- style.css の .modal .modal-content（PC: height: calc(var(--vh, 1vh) * 90)／600px以下: calc(var(--vh, 1vh) * 100)）が本値を参照する

#### `modalLifecycle`

##### `openChartModal`

- openChartModal(ticker, name, index)（window.openChartModal として公開）。
- currentIndex を更新し、#chartHeaderLeft に銘柄コード・銘柄名・「（現在位置/全件数）」を描画した上でモーダルを表示（display: flex）する。
- 表示直後は #chartContainer の高さが未確定（レイアウト未反映）な場合があるため、requestAnimationFrame を二重に重ねた上で waitForHeight() が #chartContainer の getBoundingClientRect().height > 0 になるまで setTimeout(30ms) でポーリングしてから drawChart() を呼び出す

##### `closeModal`

- モーダルを非表示にし、priceChart / rciChart / macdChart の3インスタンスを remove() した上で必ず null に戻す（二重 remove 防止）。
- 各コンテナの innerHTML も空にする。
- #closeChartBtn クリックおよび #chartModalBackdrop クリックの双方から呼び出す

##### `navigation`

- 前へ/次へは screeningResults 配列を currentIndex 基準で前後にラップアラウンド（(currentIndex ± 1 + length) % length）し、window.openChartModal を再呼び出しすることで実現する。
- #prevChartBtn / #nextChartBtn クリック（window.showPrev / window.showNext を経由）に加え、モーダル表示中（modal.style.display === "flex"）は矢印キー（ArrowLeft / ArrowRight）の keydown イベントでも同じ関数を呼び出す

#### `chartSettingsModal`

- #chartSettingsBtn クリックで #chartSettingsModal の hidden クラスを toggle する。
- document 全体の click イベントで、クリック対象が設定モーダル内でも #chartSettingsBtn でもない場合に hidden クラスを付与して閉じる（モーダル外クリックで閉じる）。
- #toggleCandles / #toggleMA / #toggleBB / #toggleIchimoku の change イベントはそれぞれ chart-price.js の setShowCandles / setShowMA / setShowBB / setShowIchimoku を呼び出す

#### `timeframeSwitch`

input[name="timeframe"] の change イベントで currentTimeframe を更新し、screeningResults[currentIndex] が存在すれば同一銘柄のまま drawChart() を再実行して足種を切り替える

#### `drawChart`

##### `signature`

drawChart(ticker, name)（非公開・モジュール内関数）

##### `flow`

- #chartLoadingOverlay を表示（try の前）
- fetchChartData(ticker, currentTimeframe) を呼び出し、null またはデータ0件の場合は alert() を表示して終了する（finally でオーバーレイは必ず非表示になる）
- 既存の priceChart / rciChart / macdChart を remove()（closeModal 側で null 化済みのため二重 remove は起きない）し、各コンテナの innerHTML をクリアする
- 価格チャート本体（LightweightCharts.createChart）を chart-main.js 側で直接生成し、日本語ロケール・日付ティック整形（M/D）を適用する。この priceChart インスタンスを createPriceChart（chart-price.js）へ渡してシリーズ生成・イベント登録を行わせる（createPriceChart の戻り値 { chart: priceChart } はここでは使用せず、chart-main.js 側で改めて const price = { chart: priceChart } を組み立てて以降の同期・リサイズ処理に用いる）
- createRciChart / createMacdChart を呼び出し、戻り値の chart をそれぞれ rciChart / macdChart に代入する
- bindTimeSync を price/rci/macd の3方向すべてに対して呼び出し、いずれか1つを操作した際に他2つの表示範囲を追随させる
- setupResize（chart-sync.js）でウィンドウリサイズ時の3チャート再描画を登録する
- applyDefaultRange（chart-sync.js）で直近4ヶ月を初期表示範囲として設定した後、直近80本（論理バー番号ベース、setVisibleLogicalRange）に上書きすることで最終的な初期表示範囲とする

#### `elements`

##### `modalBackdrop`

#chartModalBackdrop（モーダル背景。クリックで closeModal を実行。旧 .modal-backdrop class セレクタから id セレクタへ変更。class は引き続き背景のスタイル定義に使用する）

#### `windowExports`

- screening.js（非モジュールの classic script のまま）との連携のため、window.setScreeningResults / window.openChartModal / window.showPrev / window.showNext を明示的に window オブジェクトへ代入する。
- ESM 化後もこの方式は変更していない（window への代入はモジュールスコープの制約を受けないため）

### `moduleArchitecture`

#### `history`

- 2026-07 にチャート関連スクリプト（chart-data.js / chart-indicators.js / chart-price.js / chart-rci.js / chart-macd.js / chart-sync.js / chart-main.js）を非モジュールの classic script（<script defer> を6本個別に読み込む方式）から ES Modules（import/export）へ移行した。
- 動機は、非モジュール構成では全スクリプトがグローバルスコープを共有するため、chart-indicators.js と chart-price.js に同名関数 calcIchimoku が存在した際にスクリプト読み込み順で暗黙に上書きされ、片方が実行されないデッドコードになっていたため。
- 移行に伴い、各ファイル内で宣言なしの暗黙グローバル変数（rciShortSeries 等）に依存していた箇所は明示的な const/let 宣言に、chart-main.js のグローバル変数（chartContainer 等の DOM 要素・showCandles 等の表示フラグ）を暗黙参照していた箇所は関数引数またはセッター関数経由の明示的な受け渡しに、それぞれ変更した

#### `loadedAs`

- index.html では chart-main.js のみを <script type="module"> で読み込み、他の chart-*.js は chart-main.js からの import チェーンで解決される。
- screening.js は本移行の対象外で、従来どおり <script defer> の classic script のまま

#### `hostingCompatibility`

- type="module" はビルド・バンドルなしの静的ファイル配信（本アプリの GitHub 経由でのホスティング方式）でもそのまま動作するため、配信方式の変更は不要。
- ただし file:// で直接開く場合はブラウザの CORS 制約によりモジュールが読み込めない点に留意する

#### `moduleExecutionOrder`

- type="module" のスクリプトは classic script の defer と同様、DOM パース後・文書内の出現順で実行される。
- screening.js（defer）→ chart-main.js（module）という記述順は移行前後で変えていないため、実行順序は従来と同じ

### `loadingOverlaySspec`

#### `rule`

フロントエンドのすべてのサーバ通信は、通信開始前にローディングオーバーレイを表示し、try/finally の finally ブロックで必ず非表示にする

#### `overlayElements`

##### `#controlsLoadingOverlay`

条件設定カード（/dates・/heuristics_dates 通信時）

##### `#loadingOverlay`

結果テーブルカード（/screening 通信時）

##### `#chartLoadingOverlay`

チャートモーダル（/chart 通信時）

#### `pattern`

- 通信関数の先頭でオーバーレイを表示し、try/finally の finally で非表示にする。
- 呼び出し元はオーバーレイを操作しない

# `conventions`

## `domHookAttributes`

### `principle`

- class 属性は style.css からのみ参照するスタイル専用の属性とする。
- JavaScript から要素を取得・特定するためのセレクタには class を使用せず、id（単一要素）・name（フォーム部品グループ）・data-*（複数要素の役割付けや値の保持）のいずれかを用いる。
- class の変更・削除（見た目のリファクタリング）が JS の動作に影響しないようにするための取り決め

### `rules`

##### id

##### `pattern`

id

##### `usage`

- ページ内に1つしか存在しない要素（モーダル本体・オーバーレイ・スクロールコンテナ等）の取得に用いる。
- 既存の #resultTable 等と同様、キャメルケースで命名する

##### `examples`

- #resultTableStickyWrap
- #resultTableScrollOuter
- #chartModalBackdrop

##### name

##### `pattern`

name

##### `usage`

ラジオボタン・チェックボックス等、ブラウザ標準のフォームグループ機構（input[name=...]:checked 等）が前提となる要素に用いる（既存踏襲。新規ルールではなく既存パターンの追認）

##### `examples`

- input[name="searchMode"]
- input[name="compareSource"]

##### data-<用途名>="<値>"（値保持型）

##### `pattern`

data-<用途名>="<値>"（値保持型）

##### `usage`

後続処理でその値自体を参照する場合に用いる（既存踏襲）

##### `examples`

- data-sort-key（ソート対象の列キー）
- data-rule-key（heuristics の TECH_* キー）

##### data-<用途名>（値なし・フラグ型）

##### `pattern`

data-<用途名>（値なし・フラグ型）

##### `usage`

値は不要で、「その要素が対象かどうか」の判定だけに用いる

##### `examples`

- data-fixed-col（固定列として扱う th/td の目印）
- data-fixed-col-last（heuristics モードの固定列群＝コード/銘柄名/トレンド/スコアの末尾セル＝スコア th/td の目印。style.css が右端のトレンド色アクセント線の対象を特定するためのCSS専用フック。2026-07 追加）

##### data-role="<役割名>"（役割グルーピング型）

##### `pattern`

data-role="<役割名>"（役割グルーピング型）

##### `usage`

- 同じ役割を持つ複数要素を、用途に応じたグループとしてまとめて選択する場合に用いる。
- class を選択キーに流用したくなる典型的なケース（同一クラスを持つ要素群からチェック状態や値を集計したい等）はここに該当する

##### `examples`

- data-role="exclude-market-checkbox"（除外市場チェックボックス群。ratio / heuristics 両モードで共有）


### `rationale`

- class は複数要素・複数箇所で見た目の一貫性を保つために再利用される属性であり、スタイル変更のたびに書き換わりうる。
- JS 側の要素取得を id/name/data-* に切り出すことで、CSS 側のクラス名変更・構造変更が screening.js / chart-main.js 等の動作に影響しないようにする

### `appliedIn`

- screening.js の getExcludeMarkets()（data-role）
- screening.js の syncFixedColumns()（data-fixed-col・#resultTableSticky）
- screening.js のソート処理 th 特定（#resultTableSticky）
- screening.js のスクロール同期（#resultTableStickyWrap / #resultTableScrollOuter）
- chart-main.js の背景クリックによるモーダル閉鎖（#chartModalBackdrop）

### `history`

- 初版策定時点（本セクション追加時）で、screening.js / chart-main.js 内に残存していた class セレクタ（.exclude-market-checkbox / .table-header-sticky / .table-scroll-outer / .fixed-col / .modal-backdrop）をすべて本ルールに沿って移行済み。
- 以後の新規実装・改修においても本ルールを適用すること。
- なお .exclude-market-checkbox については、移行時点では data-role と並行して class 属性自体は付与し続けていたが（スタイル用として残置）、対応する CSS ルールが実際には存在せず未使用のままだったため、2026-07 の監査で class 属性の付与自体を完全に削除した（screening.js の renderExcludeMarketsFieldset() を参照）

## `nodeScriptModuleSystem`

### `principle`

- package.json の "type": "module" により、リポジトリ内の scripts/*.js（Node.js から直接実行するバックエンドスクリプト）はすべて ESM として扱われる。
- 新規スクリプトを追加する場合は require() ではなく import 文を用いること

### `appliedIn`

- scripts/heuristics.js
- scripts/fetch.js
- scripts/margin.js
- scripts/download-jpx-xlsx.js
- scripts/heuristics_backfill_trigger.js

### `history`

- scripts/heuristics_backfill_trigger.js は新設時に require('fs') / require('path') という CommonJS 構文で実装されたため、CI（.github/workflows/heuristics-backfill.yml）実行時に ReferenceError: require is not defined in ES module scope で失敗する不具合があった。
- 2026-07 に import fs from 'fs'; import path from 'path'; へ修正し、他の既存スクリプトと同じ ESM 記法に統一した

## `workflowNaming`

### `principle`

- 各 .github/workflows/*.yml の name フィールドは「【<ファイル名>】<英語での処理内容> (<日本語での簡潔な説明>)」の形式に統一する（例: 【heuristics.yml】Generate heuristics_YYYYMMDD.json (経験則スクリーニング)）。
- GitHub Actions の実行履歴一覧でどのワークフローが何をするものか一目で判別できるようにするための既存の取り決め

### `appliedIn`

- heuristics.yml：【heuristics.yml】Generate heuristics_YYYYMMDD.json (経験則スクリーニング)
- heuristics-backfill.yml：【heuristics-backfill.yml】Trigger heuristics.yml range backfill (経験則バックフィルトリガー)

### `history`

heuristics-backfill.yml は新設時に name: Heuristics Backfill Trigger という本規則に沿わない表記だったため、2026-07 に既存ワークフローと同じ形式へ修正した

# `backend`

## `fastapi`

### `file`

app/main.py

### `responsibility`

フロントエンドからのスクリーニング・チャートデータ要求に応答する API サーバ

### `cors`

#### `allow_origins`

- https://yt-f6d34a22-537c-e881-530f-f9e7a956a78b.github.io

#### `allow_methods`

*

#### `allow_headers`

*

### `githubToken`

#### `envVar`

GITHUB_TOKEN

#### `responsibility`

GitHub API（trees API / Raw heuristics JSON）への Authorization: token <PAT> 付与による rate limit 拡張（60 → 5000）

#### `security`

main.py にベタ書きせず、Render の環境変数から os.getenv('GITHUB_TOKEN') で取得する

### `dataSources`

#### `data_json`

GitHub Raw の data/data.json（fetch.js が生成）

#### `data_j.xlsx`

GitHub Raw の data/data_j.xlsx（scripts/download-jpx-xlsx.js が生成）

#### `margin_json`

GitHub Raw の data/margin.json（scripts/margin.js が生成）

#### `heuristics_json`

GitHub Raw の data/heuristics/YYYYMM/heuristics_YYYYMMDD.json を RAW_HEURISTICS_PREFIX + YYYYMM/heuristics_YYYYMMDD.json で取得（Token 認証付き）

### `modules`

#### `heuristics_scoring`

##### `file`

app/heuristics_scoring.py

##### `responsibility`

1銘柄分の tech データを受け取り、ダウンスコア・アップスコアおよび加点が発生したルールキー一覧を計算して返す

##### `function`

calc_heuristics_score(tech: dict) -> dict

##### `output`

###### `down`

ダウンスコア合計（int）

###### `up`

アップスコア合計（int）

###### `applied_down_rules`

ダウンスコアが加算された TECH_* キーのリスト

###### `applied_up_rules`

アップスコアが加算された TECH_* キーのリスト

##### `dependsOn`

app/heuristics_scoring_rules.py

#### `heuristics_scoring_rules`

##### `file`

app/heuristics_scoring_rules.py

##### `responsibility`

TECH_* キーごとのスコアルール定義（HEURISTICS_SCORING_RULES）

##### `structure`

###### `types`

- str_map（"down"/"flat"/"up" の文字列値に対してスコアを定義）
- bool（true/false に対してスコアを定義）
- int_val（数値 × multiplier_up/multiplier_down でスコアを加算。酒田五法は heuristics.js で boolean → 1/null に変換されるため val=1 × multiplier=2 = 2点となる）
- int_threshold（値が threshold 以上のとき固定スコアを加算）
- dict_direction（direction キーの値に対してスコアを定義、count_threshold/count_bonus をオプションで持つ）
- dict_trycount（tryCount が threshold 以上のとき固定スコアを加算）

###### `scoreKeys`

- down（ダウントレンド用スコア）
- up（アップトレンド用スコア）

###### `policy`

- 加点が必要なキー・値のみを定義し、0点の場合は省略する。
- 定義のないキーは 0 として扱う

### `utilities`

#### `parse_exclude_markets`

##### `signature`

parse_exclude_markets(exclude_markets: str) -> set

##### `responsibility`

- カンマ区切りの除外市場文字列を set に変換する。
- 空文字・None の場合は空 set を返す

#### `fetch_close_price`

##### `signature`

fetch_close_price(code: str, date: str) -> float | None

##### `responsibility`

- 指定銘柄コードの指定日（YYYYMMDD）の終値を yfinance から個別取得する。
- data.json の保持範囲（直近10日）を超える日付の終値取得（compare モード用）に使用する。
- 取得失敗時は None を返す

##### `usedBy`

/screening?mode=compare

### `endpoints`

#### `/dates`

##### `method`

GET

##### `responsibility`

全銘柄の全日付キー（YYYYMMDD）を抽出し、降順で返す

#### `/trading_dates`

##### `method`

GET

##### `responsibility`

- compare モードの比較元日付セレクタ（#compareFromDate）用に、直近3か月の市場開場日一覧を返す。
- data.json は直近10日分のみ保持のため対象外とし、代表銘柄（日経平均株価指数 ^N225）の日足を yfinance から取得（period=3mo）し、取引のあった日付を開場日として算出する。
- リクエスト毎に yfinance を呼ぶのではなく、load_ticker_list() / load_data_json() と同様、起動時（モジュール読み込み時）に load_trading_dates() を1回だけ実行して trading_dates_cache（グローバル変数）へ格納し、本エンドポイントはそのキャッシュを返すだけにする（2026-07 修正。以前はリクエスト毎に yfinance へ問い合わせていたため、Renderのコールドスタート時に dyno 起動遅延と yfinance の外部通信時間（Yahoo Financeへの問い合わせで数秒〜十数秒かかることがある）が合算され、タイムアウトして失敗することがあった。/dates（data_json）・/heuristics_codes に相当する他の日付取得は起動時に完了済みのデータを参照するのみのため失敗しておらず、/trading_dates だけが唯一リクエスト時に重い外部通信を行っていたことが原因だった。同じ「起動時に1回だけ取得してキャッシュする」方式に統一したことで解消した。キャッシュはプロセス起動時点のものであり、プロセスが再起動されるまで自動更新されない点は data_json / ticker_list と同じ制約を持つ）。
- trading_dates_cache が空の場合、リクエスト時にその場で load_trading_dates() を再実行するフォールバックを持つ（2026-07 追加。起動時の1回きりの取得自体が外部通信の失敗で空振りした場合、リトライが一切行われず、プロセスが再起動されるまで永久に失敗し続ける不具合が発生していた（起動時キャッシュ化により、以前の「コールドスタート時のみ失敗し、その後のリロードでは正常に取得できる」という自己回復する挙動が、逆に「一度失敗すると永久に失敗し続ける」という悪化した挙動に変わってしまっていた）。キャッシュが空の場合のみリクエスト時に再取得を試みることで、キャッシュ済みの通常時は高速な応答を維持しつつ、起動時取得が失敗した場合でも次のリクエストで自己回復できるようにし、本来意図していた挙動に近い形へ修正した）。

##### `output`

###### `status`

ok

###### `dates`

YYYYMMDD の配列（降順）

#### `/heuristics_dates`

##### `method`

GET

##### `responsibility`

GitHub trees API（GIT_TREE_API + Token 認証）を利用して data/heuristics/YYYYMM/heuristics_YYYYMMDD.json を探索し、存在する日付（YYYYMMDD）の一覧を降順で返す

##### `output`

- YYYYMMDD
- ...

#### `/screening`

##### `method`

GET

##### `modes`

###### `ratio`

###### `responsibility`

出来高倍率 × 上髭実体比スクリーニング

###### `required`

- target_date

###### `parameters`

###### `exclude_markets`

- 省略可。
- カンマ区切りの市場・商品区分文字列。
- ticker_list の「市場・商品区分」列と照合し、一致する銘柄を除外する（heuristics モードと共通の parse_exclude_markets を使用）

###### `calculations`

- 出来高倍率 = today.v / prev.v
- 上髭実体比 = (high - max(open, close)) / abs(close - open)

###### `output`

コード昇順（除外市場フィルタ適用後）

###### `date_ranking`

###### `responsibility`

値上がり率ランキング

###### `required`

- target_date

###### `calculations`

- 値上がり率 = (today_close - prev_close) / prev_close * 100

###### `output`

値上がり率降順トップ100

###### `heuristics`

###### `responsibility`

- GitHub Raw から heuristics JSON を取得し、スコア計算・除外フィルタを適用して上位20件ずつ返却する。
- codes 指定時は対象銘柄に絞り込み、上位20件制限を行わず全件返却する

###### `parameters`

###### `target_date`

必須（YYYYMMDD）

###### `exclude_markets`

- 省略可。
- カンマ区切りの市場・商品区分文字列。
- ticker_list の「市場・商品区分」列と照合し、一致する銘柄を除外する

###### `codes`

- 省略可。
- 証券コード（4桁）をカンマ区切りで複数指定可。
- 指定時は対象銘柄のみに絞り込み、各銘柄のアップスコア・ダウンスコアの大きいほうを「トレンド」として設定する

###### `logic`

- yyyymm = target_date[:6]
- raw_url = RAW_HEURISTICS_PREFIX + yyyymm + '/heuristics_' + target_date + '.json'
- requests.get(raw_url, headers=github_headers()) で JSON を取得
- ticker_list から ticker_row を1回で取得し、銘柄名と市場・商品区分を参照する
- exclude_set に含まれる市場・商品区分の銘柄をスキップ
- calc_heuristics_score(tech) でスコアを計算し、アップスコア・ダウンスコア・applied_up_rules・applied_down_rules をレスポンスに含める
- codes 指定時は該当銘柄のみに絞り込み、トレンド（アップ/ダウンスコアの大きいほう）を設定して data.up / data.down に振り分け、全件返却する
- codes 未指定時は従来通りアップスコア降順で上位20件・ダウンスコア降順で上位20件を返却

###### `output`

###### `status`

ok

###### `target_date`

YYYYMMDD

###### `data`

###### `up`

- アップスコア降順の上位20件（codes 指定時はトレンドが up の対象銘柄全件）。
- 各要素にコード・銘柄名・アップスコア・ダウンスコア・applied_up_rules・applied_down_rules・TECH_* を含む

###### `down`

- ダウンスコア降順の上位20件（codes 指定時はトレンドが down の対象銘柄全件）。
- 各要素にコード・銘柄名・アップスコア・ダウンスコア・applied_up_rules・applied_down_rules・TECH_* を含む

###### `compare`

###### `responsibility`

指定された証券コードについて、比較元日付・比較先日付それぞれの終値を yfinance から個別取得（fetch_close_price）し、増減額・増減率・上昇/下降の予測を算出して返却する

###### `required`

- from_date
- to_date
- codes

###### `parameters`

###### `from_date`

- 必須（YYYYMMDD）。
- 比較元日付

###### `to_date`

- 必須（YYYYMMDD）。
- 比較先日付

###### `codes`

- 必須。
- 証券コード（4桁）をカンマ区切りで複数指定可

###### `source_mode`

- 省略可（ratio / date / heuristics / 空）。
- CSVアップロード時の元モード。
- 証券コード直接入力時は空文字を送信する

###### `calculations`

- 増減円 = to_close - from_close
- 増減率 = (to_close - from_close) / from_close * 100

###### `predictionLogic`

###### `ratio`

予測=up（固定）

###### `date`

予測=null（予測なし）

###### `heuristics`

from_date 時点の heuristics JSON を取得し、calc_heuristics_score の up/down スコアの大きいほうを予測として設定する

###### `未指定（証券コード直接入力）`

予測=null（予測なし）

###### `errorHandling`

終値取得失敗（fetch_close_price が None を返す）銘柄は、結果要素に error メッセージを含めて返却する（スクリーニング全体は失敗させない）

###### `output`

###### `status`

ok

###### `data`

- 各要素にコード・銘柄名・比較元終値・比較先終値・増減円・増減率・予測を含む配列。
- 終値取得失敗時は該当要素に error を含む

#### `/chart`

##### `method`

GET

##### `responsibility`

tickerを受け取り、内部で 'ticker.T' に変換して Yahoo Finance API から日足6000日を取得し、1d / 1wk / 1mo の OHLCV を生成して返す

##### `timeframes`

###### `1d`

日足の直近200本

###### `1wk`

週足（W-FRI）の直近200本

###### `1mo`

月足（ME）の直近200本

##### `outputFormat`

###### `Open`

日付文字列 → 値

###### `High`

同上

###### `Low`

同上

###### `Close`

同上

###### `Volume`

同上

## `heuristics`

### `file`

scripts/heuristics.js

### `responsibility`

全銘柄の日足・週足・月足を取得し、heuristics_conditions.js の SAFE 判定を適用して heuristics_YYYYMMDD.json を生成する

### `cliArgs`

#### `description`

コマンドライン引数により3種の実行モードを切り替える

#### `args`

##### `--date`

- YYYYMMDD 単一日指定。
- --from / --to と同時使用不可

##### `--from`

- 期間指定・開始日（YYYYMMDD）。
- --to とペアで指定

##### `--to`

- 期間指定・終了日（YYYYMMDD）。
- --from とペアで指定

#### `validation`

- 各引数は YYYYMMDD（8桁数字）形式であること
- --from と --to は両方指定または両方省略であること
- --from <= --to であること
- --date と --from/--to は排他

#### `runModes`

##### `single`

- --date 指定時。
- 指定日を終端として Yahoo Finance API から取得する

##### `range`

- --from と --to 指定時。
- 土日除外の営業日候補を生成し、各日付に対して runForDate を順次実行する。
- 休場日は有効データなしとして自動スキップ。
- 1日分の生成が完了するたびに commitAndPushDate() で commit・push まで行い、全期間の完了を待たない

##### `normal`

- 引数なし。
- 従来通り Yahoo Finance 最新日付を自動取得する

### `logic`

#### `safeMA`

safeCalcMA により MA 計算不足時は null を返し例外を出さない

#### `skipRules`

##### `daily_min`

`100`

##### `weekly_min`

`100`

##### `monthly_min`

`75`

##### `log_format`

Skipping SYMBOL due to insufficient candles. { daily: X > 100 required, weekly: Y > 100 required, monthly: Z < 75 required }

#### `fetchCandles`

##### `signature`

fetchCandles(code, interval, targetDate = null)

##### `normalMode`

range パラメータで取得（FETCH_RANGES: 1d→1y / 1wk→5y / 1mo→10y）

##### `targetDateMode`

period1/period2 で期間を固定し targetDate 当日を含めるよう period2 = 翌日 00:00 JST（UTC換算）を設定する

##### `period1OffsetDays`

###### `1d`

`400`

###### `1wk`

`1900`

###### `1mo`

`3700`

##### `toUnixEndOfDay`

YYYYMMDD → 翌日 00:00 JST の UNIX 秒（指定日自体を period2 に含めるため翌日起点）

#### `runForDate`

##### `signature`

runForDate(targetDate: string|null) → Promise<string|null>

##### `responsibility`

- 1日分の heuristics を生成する。
- targetDate が null の場合は通常モード。
- 全銘柄がスキップされた場合（休場日など）は null を返す

##### `nullReturn`

有効な latestDateGlobal が取得できなかった場合、null を返してスキップを呼び出し元に通知する

#### `generateBusinessDayCandidates`

##### `signature`

generateBusinessDayCandidates(fromDate, toDate) → string[]

##### `responsibility`

- fromDate〜toDate の間で土日を除外した YYYYMMDD 配列を生成する。
- 祝日は fetchCandles 側の有効データなしで自動スキップされる

#### `rangeModeDelay`

日付間で 1000ms 待機して API 負荷を軽減する

#### `commitAndPushDate`

##### `signature`

commitAndPushDate(targetDate: string) → void

##### `responsibility`

- range モード専用。
- runForDate が非 null（＝1日分の heuristics_YYYYMMDD.json 生成に成功）を返した直後に呼び出され、その日付分の成果物（data/heuristics 配下の新規 JSON、および data/backup 配下の追加・8世代超過分の削除）を git add -A → git commit → git push する。
- 全期間の完了を待たずに日付単位でリポジトリへ反映するために導入された

##### `pushRetry`

git push origin main が失敗した場合は git fetch origin → git rebase -X theirs origin/main → git push origin main でリトライする（.github/workflows/heuristics.yml.pushStrategy と同一方針をスクリプト内に実装したもの）

##### `noopCondition`

git diff --cached --quiet でステージ済み差分がないと判定された場合は commit/push をスキップする（同一内容の再実行等を想定した安全策）

##### `errorHandling`

1日分の commit/push に失敗しても例外を外へ投げず、ログ出力のみ行い後続日付の処理を継続する

##### `scopeNote`

- single / normal モードでは呼び出されない。
- これらのモードは1ファイルのみ生成するため、従来通り .github/workflows/heuristics.yml の最終ステップ（Commit and push changes）でコミットされる

##### `precondition`

- 実行前に git の user.name / user.email が設定済みであること。
- .github/workflows/heuristics.yml の Configure git identity ステップ（Run heuristics script より前に配置）で対応する

#### `runAllConditions`

##### `valueConversions`

###### `酒田五法`

- heuristics_conditions.js は boolean を返すが、runAllConditions 内で boolean ? 1 : null に変換して JSON に保存する。
- スコアルールの int_val 型（multiplier=2）と対応する

###### `TECH_MA_SPREAD`

isMaSpreadUp() の戻り値（"up"/"down"）を slope ヘルパーで変換

###### `TECH_CYCLE_PROGRESS`

computeCycleProgress() の戻り値 { direction, count, startDate, lastDate } をそのまま格納（null の場合は null）

###### `TECH_RULE9_DAILY`

computeRule9Daily() の戻り値 { direction, count } をそのまま格納

###### `TECH_RULE9_WEEKLY`

computeRule9Weekly() の戻り値 { direction, count } をそのまま格納

###### `TECH_GRANVILLE`

computeGranville() の戻り値 { direction, count } をそのまま格納

###### `TECH_FUSHIME_UP`

computeFushimeUp() の戻り値 { price, tryCount } をそのまま格納

###### `TECH_FUSHIME_DOWN`

computeFushimeDown() の戻り値 { price, tryCount } をそのまま格納

#### `output`

##### `fileFormat`

heuristics_YYYYMMDD.json

##### `directory`

data/heuristics/YYYYMM/

##### `dateSource`

Yahoo Finance API の daily データの最新日付

##### `overwrite`

同名ファイルが存在する場合は上書きする

##### `backup`

###### `directory`

data/backup

###### `fileFormat`

heuristics_YYYYMMDD.json.YYYYMMDD_hhmmss

###### `keep`

`8`

## `heuristics_conditions`

### `file`

scripts/heuristics_conditions.js

### `responsibility`

MA系・酒田五法・三尊・W底・N大・ものわかれ・Rule9 の SAFE 判定ロジック

### `safety`

calcMA の throw を完全排除し、全て safeCalcMA で null-safe に処理

### `maPeriods`

#### `daily`

##### `short`

`5`

##### `mid`

`25`

##### `long`

`75`

#### `weekly`

##### `short`

`5`

##### `mid`

`13`

##### `long`

`26`

#### `monthly`

##### `short`

`5`

##### `mid`

`12`

##### `long`

`24`

### `rules`

- MA slope
- Perfect Order / Reverse PO
- Pre-PO / Pre-RPO
- MA Congestion
- MA Spread
- MA100 Trend
- Kahanshin / Gyaku-Kahanshin
- 5MA High/Low Update
- Sakata 五法
- Head and Shoulders
- Double Bottom
- Nichi-Dai / Gyaku-Nichi-Dai
- Monowakare / Cross
- Rule9 Daily / Weekly（SAFE 実装）
- BB Zone Break Daily / Weekly / Monthly
- Box Range
- Overheat
- Granville
- Cycle Progress
- Fushime Up / Down

### `returnValueTypes`

#### `string（up/down/flat）`

- TECH_MA_SLOPE_DAILY
- TECH_MA_SLOPE_WEEKLY
- TECH_MA_SLOPE_MONTHLY
- TECH_MA_POSITION_DAILY
- TECH_MA_POSITION_WEEKLY
- TECH_MA_POSITION_MONTHLY
- TECH_MA_SPREAD
- TECH_MA100_TREND
- TECH_5MA_UPDATE
- TECH_MONOWAKARE
- TECH_MONOWAKARE_RED_BLUE_CROSS

#### `boolean`

- TECH_PERFECT_ORDER_DAILY
- TECH_PERFECT_ORDER_WEEKLY
- TECH_PERFECT_ORDER_MONTHLY
- TECH_REVERSE_PERFECT_ORDER_DAILY
- TECH_REVERSE_PERFECT_ORDER_WEEKLY
- TECH_REVERSE_PERFECT_ORDER_MONTHLY
- TECH_PRE_PERFECT_ORDER_DAILY
- TECH_PRE_PERFECT_ORDER_WEEKLY
- TECH_PRE_PERFECT_ORDER_MONTHLY
- TECH_PRE_REVERSE_PERFECT_ORDER_DAILY
- TECH_PRE_REVERSE_PERFECT_ORDER_WEEKLY
- TECH_PRE_REVERSE_PERFECT_ORDER_MONTHLY
- TECH_MA_CONGESTION
- TECH_KAHANSHIN
- TECH_GYAKU_KAHANSHIN
- TECH_HEAD_AND_SHOULDERS
- TECH_DOUBLE_BOTTOM
- TECH_NICHI_DAI
- TECH_GYAKU_NICHI_DAI
- TECH_IN_IN_HARAMI
- TECH_RED_BLUE_CROSS
- TECH_RETURN_SELL_END
- TECH_DOWN_TREND_END
- TECH_MOMIAI
- TECH_BB_ZONE_BREAK_DAILY
- TECH_BB_ZONE_BREAK_WEEKLY
- TECH_BB_ZONE_BREAK_MONTHLY
- TECH_BOX_RANGE
- TECH_OVERHEAT

#### `1 or null（heuristics.js で boolean → int_val 変換済み）`

- TECH_SAKATA_TRIPLE_TOP
- TECH_SAKATA_TRIPLE_BOTTOM
- TECH_SAKATA_SANKU_UP
- TECH_SAKATA_SANKU_DOWN
- TECH_SAKATA_SANPEI_UP
- TECH_SAKATA_SANPEI_DOWN
- TECH_SAKATA_SANPO_UP
- TECH_SAKATA_SANPO_DOWN

#### `{ direction, count }`

- TECH_RULE9_DAILY（direction: up/down/null, count: 連続本数）
- TECH_RULE9_WEEKLY（同上）
- TECH_GRANVILLE（direction: up/down/null, count: 法則番号 1〜3）

#### `{ direction, count, startDate, lastDate }`

- TECH_CYCLE_PROGRESS（direction: up/down, count: サイクル起点からの経過営業日数, startDate/lastDate: YYYYMMDD）

#### `{ price, tryCount } or { price: null, tryCount: null }`

- TECH_FUSHIME_UP（price: 節目価格, tryCount: タッチ回数）
- TECH_FUSHIME_DOWN（同上）

## `jpx_xlsx`

### `file`

scripts/download-jpx-xlsx.js

### `responsibility`

JPX data_j.xls を取得し、LibreOffice で XLSX に変換 + バックアップ管理

### `dependencies`

- node-fetch@3
- jsdom

### `systemDependencies`

- libreoffice（apt-get）

### `steps`

#### `1_getXlsUrl`

JPX 公式ページ（statistics-equities/misc/01.html）をスクレイピングして data_j.xls の URL を取得する

#### `2_downloadXls`

取得した URL から data/data_j.xls をダウンロード

#### `3_convertToXlsx`

libreoffice --headless --convert-to xlsx で data/data_j.xls → data/data_j.xlsx に変換

#### `4_backup`

data/backup/ に data_j.xls.TIMESTAMP・data_j.xlsx.TIMESTAMP を作成（タイムスタンプは JST）

#### `5_cleanup`

data/backup/ 内の data_j.xls.* / data_j.xlsx.* のうち 3日超過分を削除

### `backup`

#### `directory`

data/backup

#### `fileFormat`

data_j.xls.YYYYMMDD_hhmmss / data_j.xlsx.YYYYMMDD_hhmmss

#### `retention`

3日超過分を削除（3日以内を保持）

#### `timestampZone`

JST

### `output`

#### `primary`

data/data_j.xlsx

#### `intermediate`

data/data_j.xls（LibreOffice 変換後に残存）

## `fetch`

### `file`

scripts/fetch.js

### `responsibility`

data/data_j.xlsx から銘柄コードを読み取り、Yahoo Finance から直近10日分の OHLCV を取得して data.json を生成

### `dependencies`

- node-fetch@3
- xlsx

### `steps`

#### `1_readSymbols`

data/data_j.xlsx の Sheet1 の「コード」列から銘柄コードを読み取る（undefined・空白を除外）

#### `2_fetchYahoo`

- Yahoo Finance API（query1.finance.yahoo.com/v8/finance/chart/{code}.T?interval=1d&range=10d）から直近10日分の OHLCV を取得する。
- 各銘柄間に 500ms の遅延を挟む

#### `3_writeJson`

全銘柄のデータを data/data.json に洗い替え書き込み（JSON.stringify、indent 2）

#### `4_backup`

data/backup/ に data.json.TIMESTAMP を作成（タイムスタンプは JST）

#### `5_cleanup`

data/backup/ 内の data.json.* を古い順にソートし、最大 8 個を超えた分を削除

### `yahooApi`

#### `urlTemplate`

https://query1.finance.yahoo.com/v8/finance/chart/{code}.T?interval=1d&range=10d

#### `tickerSuffix`

.T

#### `range`

10d

#### `interval`

1d

### `outputFormat`

#### `key`

銘柄コード（4桁文字列）

#### `value`

##### `YYYYMMDD`

###### `o`

始値

###### `h`

高値

###### `l`

安値

###### `c`

終値

###### `v`

出来高

##### `error`

取得失敗時のエラーメッセージ文字列

### `backup`

#### `directory`

data/backup

#### `fileFormat`

data.json.YYYYMMDD_hhmmss

#### `keep`

`8`

#### `timestampZone`

JST

### `output`

data/data.json

## `margin`

### `file`

scripts/margin.js

### `responsibility`

JSF + 楽天規制 + JPX週次 + JPX日々公表 → margin.json 統合

### `dependencies`

- node-fetch@3
- xlsx
- jsdom
- pdfjs-dist
- iconv-lite

### `dataSources`

#### `kubunMap`

##### `url`

https://www.taisyaku.jp/data/meigara.csv

##### `encoding`

Shift_JIS

##### `format`

CSV（1行目タイトル / 2行目ヘッダ / 3行目以降データ）

##### `key`

コード列（4桁、NFKC正規化）

##### `value`

貸借銘柄区分（東証）列

##### `filter`

貸借区分が '0' または取得不可の銘柄は margin.json に出力しない

#### `rakutenRegulation`

##### `url`

https://www.rakuten-sec.co.jp/ITS/Companyfile/margin_restriction.html

##### `encoding`

EUC-JP

##### `tableLogic`

- rowspan 対応（6列論理配列に展開）。
- logical[0]=コード / logical[2]=市場 / logical[3]=規制文言

##### `marketFilter`

TOKYO_KEYWORDS=['東京'] で東証銘柄のみ抽出

##### `buyBanKeywords`

- 新規買停止
- 全取引停止

##### `sellBanKeywords`

- 新規売停止
- 全取引停止

#### `jpxWeekly`

##### `indexUrl`

https://www.jpx.co.jp/markets/statistics-equities/margin/05.html

##### `filePattern`

syumatsu を含む .pdf リンクの最新ファイル

##### `parser`

pdfjs-dist で全ページテキスト抽出 → [0-9A-Z]{4}0\s+JP\d{10} のブロック分割

##### `codeExtract`

5桁コード（例: 72030）の先頭4桁をNFKC正規化して使用

##### `fields`

- buy（買残高）
- buy_diff（前週比）
- sell（売残高）
- sell_diff（前週比）
- ratio（倍率 = buy/sell、小数点2桁）

#### `jpxDaily`

##### `indexUrl`

https://www.jpx.co.jp/markets/statistics-equities/margin/index.html

##### `filePattern`

mtdailyk を含む最新 .xls ファイル

##### `parser`

xlsx ライブラリ（range: 5 でヘッダ行スキップ）

##### `codeExtract`

[0-9A-Z]{4}0 形式の先頭4桁をNFKC正規化して使用

##### `fields`

- sell（売残高 Outstanding Sales）
- buy（買残高 Outstanding Purchases）

### `mergeLogic`

#### `applyDailyToWeekly`

- jpxDaily の buy/sell で jpxWeekly の値を上書きし、前週比（buy_diff / sell_diff）を再計算する。
- 週次に存在しない銘柄は buy_diff/sell_diff を null で追加する

### `outputSchema`

#### `key`

銘柄コード（4桁、NFKC正規化、コード昇順にソート）

#### `value`

##### `貸借区分`

kubunMap の値（'0' 除外済み）

##### `制度信用`

###### `買い建て`

buyBanKeywords に該当しない場合 true

###### `売り建て`

貸借区分='1' かつ sellBanKeywords に該当しない場合 true

##### `JPX信用買残`

null or number

##### `JPX信用買残前週比`

null or number

##### `JPX信用売残`

null or number

##### `JPX信用売残前週比`

null or number

##### `JPX信用倍率`

null or number（小数点2桁）

##### `規制`

楽天規制ページの規制文言の配列

### `backup`

#### `directory`

data/backup

#### `fileFormat`

margin.json.YYYYMMDD_hhmmss

#### `retention`

3日超過分を削除（3日以内を保持）

#### `timestampZone`

JST

### `output`

data/margin.json

# `workflows`

## `download-jpx-xlsx.yml`

### `responsibility`

JPX data_j.xls の取得・XLSX変換・バックアップ

### `schedule`

- 7 18 * * *（JST 03:07）

### `nodeVersion`

24

### `dependencies`

node-fetch@3 xlsx jsdom pdf-parse（npm install --no-save）

### `systemDependencies`

libreoffice（apt-get install -y libreoffice）

### `concurrency`

group: auto-update / cancel-in-progress: false（複数ワークフローの同時実行禁止）

### `permissions`

contents: write

### `checkoutOptions`

#### `fetch-depth`

`0`

#### `reason`

stale な origin/main を参照しないよう全履歴を取得する

### `pushStrategy`

git push 失敗時は fetch → rebase -X theirs → push でリトライ

### `outputs`

- data/data_j.xlsx
- data/backup/

## `fetch.yml`

### `responsibility`

Yahoo Finance から data.json を更新

### `schedule`

- 7 22 * * *（JST 07:07（前日UTC））
- 7 7 * * *（JST 16:07）
- 7 9 * * *（JST 18:07）
- 7 11 * * *（JST 20:07）

### `nodeVersion`

24

### `dependencies`

node-fetch@3 xlsx（npm install --no-save）

### `concurrency`

group: auto-update / cancel-in-progress: false（複数ワークフローの同時実行禁止）

### `permissions`

contents: write

### `checkoutOptions`

#### `fetch-depth`

`0`

#### `reason`

stale な origin/main を参照しないよう全履歴を取得する

### `pushStrategy`

git push 失敗時は fetch → rebase -X theirs → push でリトライ

### `outputs`

- data/data.json
- data/backup/

## `margin.yml`

### `responsibility`

margin.json の自動更新

### `schedule`

- 7 18 * * *（JST 03:07）

### `nodeVersion`

24

### `dependencies`

node-fetch@3 xlsx jsdom pdfjs-dist iconv-lite（npm install --no-save）

### `concurrency`

group: auto-update / cancel-in-progress: false（複数ワークフローの同時実行禁止）

### `permissions`

contents: write

### `checkoutOptions`

#### `fetch-depth`

`0`

#### `reason`

stale な origin/main を参照しないよう全履歴を取得する

### `pushStrategy`

git push 失敗時は fetch → rebase -X theirs → push でリトライ

### `outputs`

- data/margin.json
- data/backup/

## `heuristics.yml`

### `responsibility`

全銘柄の OHLCV を取得し heuristics_YYYYMMDD.json を生成する SAFE ワークフロー

### `schedule`

#### `active`

- 7 8 * * *（JST 17:07）
- 7 10 * * *（JST 19:07）
- 7 12 * * *（JST 21:07）

#### `commentedOut`

- 7 23 * * *（JST 08:07）

### `workflowDispatch`

#### `inputs`

##### `date`

###### `description`

- 単一日指定（YYYYMMDD）。
- from_date / to_date と同時使用不可。
- 省略時は通常処理。

###### `required`

`False`

###### `default`

##### `from_date`

###### `description`

- 期間指定・開始日（YYYYMMDD）。
- to_date と必ずペアで指定。

###### `required`

`False`

###### `default`

##### `to_date`

###### `description`

- 期間指定・終了日（YYYYMMDD）。
- from_date と必ずペアで指定。

###### `required`

`False`

###### `default`

#### `argsBuildLogic`

- --date / --from + --to を条件分岐で組み立て、heuristics.js に渡す。
- 入力がない場合は引数なし（通常モード）で実行する

### `nodeVersion`

24

### `dependencies`

node-fetch@3 xlsx（npm install --no-save）

### `concurrency`

group: auto-update / cancel-in-progress: false（複数ワークフローの同時実行禁止）

### `permissions`

contents: write

### `checkoutOptions`

#### `fetch-depth`

`0`

#### `reason`

stale な origin/main を参照しないよう全履歴を取得する

### `steps`

#### `order`

- Checkout repository（fetch-depth: 0）
- Setup Node.js（24）
- Install dependencies（node-fetch@3 xlsx）
- Ensure data directory exists（data / data/backup / data/heuristics の mkdir -p）
- Configure git identity（git config --global user.name / user.email）
- Run heuristics script（引数組み立て → node scripts/heuristics.js）
- Commit and push changes（最終コミットステップ。詳細は commitStrategy を参照）

#### `gitIdentityStepPosition`

- Configure git identity は Run heuristics script より前に配置する。
- range モードで scripts/heuristics.js 内の commitAndPushDate() が git commit を実行するため、Node 実行前に user.name / user.email が設定済みである必要がある

### `pushStrategy`

git push 失敗時は fetch → rebase -X theirs → push でリトライ（scripts/heuristics.js.logic.commitAndPushDate.pushRetry と同一方針。range モードではこのリトライは Node スクリプト側で、single/normal モードでは本ワークフローの最終ステップ側で実施される）

### `commitStrategy`

#### `single_normal`

1ファイルのみ生成するため、最終ステップ（Commit and push changes）で1回だけ commit・push する（従来通り）

#### `range`

- scripts/heuristics.js.logic.commitAndPushDate() が、1日分の heuristics_YYYYMMDD.json 生成が完了するたびに commit・push する。
- 全期間の完了を待たない。
- 最終ステップ（Commit and push changes）はこのモードでは基本的に差分なし（no-op、git commit || exit 0 で正常終了）となる安全網として残す

### `output`

#### `fileFormat`

heuristics_YYYYMMDD.json

#### `overwrite`

同名ファイルが存在する場合は上書きする

#### `backup`

data/backup に heuristics_YYYYMMDD.json.TIMESTAMP を最大 8 個保持

## `heuristics-backfill.yml`

### `responsibility`

- 2025-07-01〜2026-06-30 の heuristics バックフィルを、concurrency group「auto-update」を共有する既存4ワークフロー（fetch.yml / margin.yml / download-jpx-xlsx.yml / heuristics.yml）がいずれも稼働していない時間帯にのみ、heuristics.yml の range モード（from_date/to_date）へ委譲する新規トリガー用ワークフロー。
- 重い処理そのもの（OHLCV取得・スコア計算・営業日判定）は heuristics.yml / heuristics.js 側の既存責務のまま変更しない

### `schedule`

- 37 23 * * *（JST 08:37）

### `scheduleRationale`

- 既存4ワークフローのスケジュールを UTC 順に並べると、22:07（fetch.yml）〜翌07:07（fetch.yml）の間が唯一約9時間空く時間帯であるため、前後のジョブと十分な余裕を持たせて 23:37 UTC に設定する。
- GitHub-hosted runner のジョブ既定上限（6時間）で打ち切られても 05:37 UTC 終了となり、翌 07:07 UTC の fetch.yml までに約1.5時間の余裕がある

### `workflowDispatch`

#### `inputs`

なし（手動実行時も特別な入力パラメータは不要。resume date は data/heuristics/ の実ファイルから自動判定する）

### `nodeVersion`

24

### `dependencies`

なし（scripts/heuristics_backfill_trigger.js は Node.js 標準の fs / path のみを使用し、npm パッケージに依存しない）

### `moduleSystem`

- ESM（import fs from 'fs'; import path from 'path';）。
- package.json の "type": "module" によりリポジトリ内の .js は既定でESMとして扱われ、scripts/heuristics.js・scripts/fetch.js・scripts/margin.js・scripts/download-jpx-xlsx.js も同様にESMで書かれている。
- 本スクリプトは新設時に require('fs') / require('path') というCommonJS構文で実装されたため、CI実行時に ReferenceError: require is not defined in ES module scope で失敗していた不具合があり、2026-07 に既存スクリプト群と同じESM記法（import）へ修正した（package.json・ワークフローyml側の変更は不要）。

### `concurrency`

group: heuristics-backfill / cancel-in-progress: false（本ワークフロー自身の多重起動防止。auto-update グループには参加しない）

### `permissions`

contents: read / actions: write（同一リポジトリの heuristics.yml を workflow_dispatch で起動するために actions: write が必要。データを直接コミットしないため contents は read のみ）

### `steps`

#### `order`

- Checkout repository（data/heuristics/ 配下を resume date 判定のためローカル参照する。push しないため fetch-depth 指定は不要）
- Setup Node.js（24）
- Determine resume date（node scripts/heuristics_backfill_trigger.js）
- Check if any auto-update workflow is running or queued（fetch.yml / margin.yml / download-jpx-xlsx.yml / heuristics.yml の4ワークフローすべてを gh run list で確認）
- Dispatch heuristics.yml（range mode）（resume date が範囲内かつ4ワークフローすべてが非稼働の場合のみ gh workflow run heuristics.yml -f from_date=... -f to_date=... を実行）
- No action taken（バックフィル完了済み、または4ワークフローのいずれかが稼働中の場合はスキップし、GITHUB_STEP_SUMMARY にその旨を記録）

### `resumeDateLogic`

#### `script`

scripts/heuristics_backfill_trigger.js

#### `range`

BACKFILL_START=20250701 / BACKFILL_END=20260630（固定）

#### `logic`

- actions/checkout 済みの data/heuristics/**/heuristics_YYYYMMDD.json をローカル走査し、BACKFILL_START〜BACKFILL_END の範囲内に限定して既存日付を収集する（2026-07-01 以降の通常スケジュールで生成済みのファイルは範囲外として除外される）。
- 範囲内の最新日付の翌日を from_date とし、to_date は常に BACKFILL_END を指定する。
- 範囲内に既存ファイルが1件もなければ from_date=BACKFILL_START。
- 最新日付の翌日が BACKFILL_END を超える場合はバックフィル完了と判断し dispatch を行わない

#### `apiCalls`

0（GitHub API・GitHub trees API のいずれも呼び出さない。actions/checkout 済みのローカルファイルのみを参照するため、GitHub API のレート制限を考慮する必要がない）

#### `resumability`

- heuristics.yml の commitStrategy.range により1日分の生成が完了するたびに commit・push されるため、本ワークフローのジョブが GitHub-hosted runner の既定上限（6時間）で打ち切られても、直前までの成果は失われない。
- 翌晩のスケジュール実行時に、data/heuristics/ の最新日付の翌日から自動的に再開する

### `guardLogic`

#### `targetWorkflows`

- fetch.yml
- margin.yml
- download-jpx-xlsx.yml
- heuristics.yml

#### `reason`

- 上記4ワークフローはすべて concurrency.group: auto-update を共有するため、いずれか1つでも in_progress または queued の場合は heuristics.yml の range モード dispatch をスキップする。
- これは無駄な dispatch を避けるための予防的なガードであり、安全性そのものは heuristics.yml 自身が持つ concurrency.group: auto-update（cancel-in-progress: false）が既存仕様として担保している（ガードをすり抜けて dispatch されても自動的にキュー待ちになり、他ワークフローとの同時実行や競合は発生しない）

# `dataFlow`

## `chartData`

chart-data.js → fetchChartData → API /chart → candleData → chart-main.js → 各チャート描画

## `screening`

screening.js → API /screening → テーブル表示 → 行クリックで chart-main.js を呼び出し

## `csvExport`

screening.js（currentResults）→ buildCsvHeaders / buildCsvRow / escapeCsvCell → Blob（BOM付きUTF-8）→ <a>.click() → ブラウザダウンロード（対応モードは ratio / date / heuristics の3種。compare モードは対象外）

## `compareScreening`

window.onload の loadTradingDates() → API /trading_dates → #compareFromDate / #compareToDateSelect（共通データソース・直近3か月の市場開場日。デフォルトは先頭＝最新の市場開場日）を構築 → CSVアップロード時は setCompareFromDate() がファイル名日付を選択肢に存在しなければ動的追加した上で選択、extractCodesFromCsv() がコード列と「スコア」列を抽出して uploadedCsvScoreMap を構築 → screening.js（CSVアップロード時は parseCsvLine / extractCodesFromCsv でコード列抽出、証券コード直接入力時はテキスト入力値をそのまま使用・uploadedCsvScoreMap をリセット）→ API /screening?mode=compare（codes・from_date・to_date・source_mode）→ main.py が fetch_close_price で yfinance から個別に終値取得（source_mode=heuristics の場合のみ GitHub Raw の heuristics JSON も取得）→ 増減額・増減率・予測を算出して返却 → showResults() でテーブル描画（列順：スコア・上昇/下降の予測・上昇/下降の結果・比較元DATE終値・比較先DATE終値・増減（円）・増減（％）、行背景色=予測ベース・結果グループセル背景色=結果ベース・予測グループセル背景色=予測ベース）→ calcMatchRate() で合致率を算出して #compareMatchRate に表示

## `margin`

scripts/margin.js → margin.json（信用取引可否データ）

## `jpx`

scripts/download-jpx-xlsx.js → data_j.xlsx → scripts/fetch.js → data.json

## `heuristics`

scripts/heuristics.js → scripts/heuristics_conditions.js → heuristics_YYYYMMDD.json（single/normal モードは .github/workflows/heuristics.yml の最終ステップで、range モードは scripts/heuristics.js の commitAndPushDate() により日付ごとに GitHub へ commit・push） → main.py /screening?mode=heuristics（codes 指定時は絞り込み）→ app/heuristics_scoring_rules.py → app/heuristics_scoring.py（applied_*_rules を含むスコアを返却）→ screening.js（ tr-trend-up/down・td-heuristics-applied でハイライト表示）

## `heuristicsBackfill`

.github/workflows/heuristics-backfill.yml（毎日23:37 UTCのスケジュール実行）→ scripts/heuristics_backfill_trigger.js（data/heuristics/ をローカル走査し BACKFILL_START=20250701〜BACKFILL_END=20260630 の範囲内で resume date を決定。API呼び出しなし）→ fetch.yml / margin.yml / download-jpx-xlsx.yml / heuristics.yml の稼働状況を gh run list で確認（4ワークフローがいずれも非稼働の場合のみ次工程へ）→ gh workflow run heuristics.yml -f from_date=<resume date> -f to_date=20260630 で range モードを起動 → 以降は dataFlow.heuristics の既存フロー（commitAndPushDate() による日次 commit・push）に合流する
