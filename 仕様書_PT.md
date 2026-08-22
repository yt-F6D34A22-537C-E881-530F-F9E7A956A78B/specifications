# Personal Tools 仕様書

## 概要
個人専用のツール集。タブで機能を切り替えるシングルページのWebアプリ。
フロントエンドは GitHub Pages（静的ホスティング）で公開する。

## 構成
- フロントエンド: HTML / CSS / JavaScript（フレームワーク不使用）
- バックエンド: 現時点では未使用（下記「今後の方針」参照）
- 外部ライブラリ: 特定機能を実現するために不可欠なもののみ、CDN経由または自前ホスティングで個別導入を許容する
  （フレームワーク導入は引き続き禁止。現状: Tesseract.js（OCR, CDN経由）をタブ2で使用、
  ParticleNetwork（背景アニメーション, `frontend/js/vendor/`に自前ホスティング）をページ全体で使用）

## コーディング規約
- HTML: class 属性はスタイル調整専用。JavaScript からの DOM 取得は id / name / data-* を使用する。
- CSS: セレクタは class のみを用いる。id セレクタ・要素型セレクタ（body, div, input 等）・
  全称セレクタ（*）は使用しない。box-sizing 等の共通プロパティも、対象となる
  各 class に個別記述する。

## ファイル構成
```
personal-tools/
├── frontend/
│   ├── index.html
│   ├── assets/
│   │   ├── favicon.ico
│   │   ├── apple-touch-icon.png
│   │   └── apple-touch-icon-v3.png
│   ├── css/
│   │   └── style.css
│   └── js/
│       ├── common/
│       │   ├── main.js               # タブ切替の共通制御
│       │   └── particleBackground.js # 背景アニメーション（ParticleNetwork）の初期化
│       ├── vendor/
│       │   └── particleNetwork.min.js  # 背景アニメーション用ライブラリ（third-party, 自前ホスティング）
│       └── tabs/
│           ├── ohlcvMergeTab.js      # タブ1固有ロジック
│           ├── imageOcrTab.js        # タブ2固有ロジック（画像OCR）
│           └── comingSoonTab.js      # タブ3（プレースホルダ）
├── scripts/
│   └── generate_icons.py            # アイコン一式の生成スクリプト（Pillow使用）
└── 仕様書.md
```

## アイコン
- `frontend/assets/favicon.ico`（16/32/48px）, `frontend/assets/apple-touch-icon.png`（180px）,
  `frontend/assets/apple-touch-icon-v3.png`（180px、キャッシュ更新用の別名ファイル）
- デザイン: style.css の配色（クリーム #ece5d6 × 濃紺 #1d2340）に合わせ、
  yubune.co のトップページにある「水面の波紋」モチーフを同心円で簡略化したシンボルマーク。
- 生成方法: `python3 scripts/generate_icons.py`（Pillow使用。出力先は `frontend/assets/`。
  デザインを変更したい場合は `scripts/generate_icons.py` 内の
  `COLOR_BG` / `COLOR_MARK` / `ring_specs` を編集して再実行する）

## 背景アニメーション
- ページ全体（全タブ共通）に、動く粒子ネットワークの背景アニメーションを表示する。
- 使用ライブラリ: ParticleNetwork（third-party, MIT License）。
  `frontend/js/vendor/particleNetwork.min.js` に自前ホスティングし、外部CDNへは依存しない。
- 背景画像は使用せず、単色背景（style.cssの配色 --color-bg 相当）を指定する。
  パーティクル・線の色は --color-fg 相当に合わせている。
- `#particle-canvas` は `position: fixed` でビューポート全体を覆い、`z-index: -1` により
  既存UI（header/nav/main）の背面に配置する。html/body自体への変更は行わない。
  - 注意: ParticleNetworkライブラリは、渡したコンテナ自身に `position: relative` を
    インラインスタイルで強制設定する仕様のため、`#particle-canvas`に直接CSSで
    `position: fixed`を指定しても上書きされてしまう。そのため、fixed配置は
    外側のラッパー要素（`#particle-background-wrapper` / `.particle-canvas-wrapper`）側で行い、
    `#particle-canvas`はそのラッパー内で幅・高さ100%を占めるようにしている。
  - 主張が強すぎないよう、`.particle-canvas-wrapper`に`opacity: 0.35`を指定し、
    背景全体を薄く表示している（調整したい場合はこの数値のみ変更すればよい）。
- 初期化ロジックは `frontend/js/common/particleBackground.js` に実装する
  （タブ固有ではない共通スクリプトのため、`main.js`と同じ `js/common/` 配下に配置する）。

## タブ一覧

### タブ1: OHLCVマージ
- 移植元: デスクトップアプリ（WPF, OhlcvMergerWpf）
- 入力: 銘柄コードをキーにしたJSONファイル群（1ファイル=1日分、ファイル名の数字部分を日付として使用）
- フィルタ: カンマ区切りで指定した銘柄コードのみ抽出
- 出力: 銘柄コード -> 日付 -> {o,h,l,c,v} の入れ子構造のJSON（1ファイル）
- 処理方式: **クライアントサイドJavaScriptのみで完結**（バックエンドAPI不使用）
  - 理由: フィルタ・集計処理が軽量であり、サーバー経由にする必然性がないため。
  - 追加機能: 「既存のマージ済みJSON」を読み込んでベースにし、追記マージする機能を搭載。
    大量ファイル（想定20〜150件、1件あたり約8MB）を複数回に分けて処理する運用を想定。

### タブ2: 画像OCR
- 移植元（参考）: デスクトップアプリ（WPF, StockTools）の Tesseract エンジンを用いたOCR処理。
  ただしWPF版は株価チャート特化の前処理（二値化・膨張・色相フィルタ）を伴う専用ロジックのため、
  本タブでは汎用画像向けに「グレースケール化＋Otsu法による二値化」のみを移植する。
- 入力: 画像ファイル1枚（jpg/png等）。以下2通りの方法で指定できる
  - ファイル選択（`<input type="file">`）
  - クリップボードからの貼り付け（貼り付け欄にフォーカスしCtrl+V）
- 前処理（精度向上のため、Canvas APIのみで実装。追加ライブラリなし）
  - 幅1000px未満の画像は2倍に拡大
  - グレースケール化 → Otsu法によるしきい値算出 → 二値化
- 認識言語: 「日本語＋英語（既定）／日本語のみ／英語のみ」から選択可能
- 処理: Tesseract.js（CDN経由）でクライアントサイドOCRを実行
- 出力: 抽出テキストをテキストエリアに表示し、クリップボードへコピー可能
- 処理方式: **クライアントサイドJavaScriptのみで完結**（バックエンドAPI不使用）
  - 理由: 画像はサーバーへ送信せず、既存のバックエンド未使用方針を維持するため
  - ライブラリ本体はタブ初回利用時（抽出ボタン押下時）に遅延読込みし、
    他タブの初期表示性能へ影響させない

### タブ3
- 現時点では未実装（Coming Soon 表示のみ）

## 今後の方針・保留事項（TODO）

| # | 内容 | 対応時期 |
|---|---|---|
| 1 | Webスクレイピング機能等、今後サーバー処理が必要な機能を追加する場合、Python(FastAPI) + Render でバックエンドを新規構築する | 機能追加が具体化した時点 |
| 2 | 上記バックエンド構築時、**CORS設定 (`allow_origins`) を実際のGitHub Pages公開URLに限定すること**を必須とする（現時点ではバックエンド自体が存在しないため未設定） | バックエンド構築時 |
| 3 | GitHub Pagesの実際の公開URLが確定次第、本ドキュメントに追記する | URL確定時 |
| 4 | Tesseract.jsをCDN経由で読み込んでいる（`js/tabs/imageOcrTab.js`）。バージョン固定・SRI付与・自前ホスティング化の要否を検討する | 未定 |

## 変更履歴
- 初版: タブ1（OHLCVマージ機能、クライアントサイドJS実装）、タブ2（Coming Soon）を作成
- CSSのセレクタ規約違反（id・要素型セレクタの使用）を修正し、class セレクタのみに統一
- タブ2として画像OCR機能（Tesseract.js使用、クライアントサイド完結）を追加し、
  従来のComing Soonタブをタブ3へ移動。外部ライブラリ導入方針を「構成」節に追記
- タブ2に、精度向上のための前処理（Canvas APIによる拡大・グレースケール化・Otsu法二値化）、
  クリップボード画像貼り付け、認識言語の選択機能を追加
- ページ全体の背景として、ParticleNetwork（自前ホスティング）による粒子ネットワークの
  アニメーション背景を追加（背景画像は不使用、style.cssの配色に合わせた単色を使用）
