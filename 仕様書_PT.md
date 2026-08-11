# Personal Tools 仕様書

## 概要
個人専用のツール集。タブで機能を切り替えるシングルページのWebアプリ。
フロントエンドは GitHub Pages（静的ホスティング）で公開する。

## 構成
- フロントエンド: HTML / CSS / JavaScript（フレームワーク不使用）
- バックエンド: 現時点では未使用（下記「今後の方針」参照）

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
│       ├── main.js                  # タブ切替の共通制御
│       └── tabs/
│           ├── ohlcvMergeTab.js      # タブ1固有ロジック
│           └── comingSoonTab.js      # タブ2（プレースホルダ）
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

### タブ2
- 現時点では未実装（Coming Soon 表示のみ）

## 今後の方針・保留事項（TODO）

| # | 内容 | 対応時期 |
|---|---|---|
| 1 | Webスクレイピング機能等、今後サーバー処理が必要な機能を追加する場合、Python(FastAPI) + Render でバックエンドを新規構築する | 機能追加が具体化した時点 |
| 2 | 上記バックエンド構築時、**CORS設定 (`allow_origins`) を実際のGitHub Pages公開URLに限定すること**を必須とする（現時点ではバックエンド自体が存在しないため未設定） | バックエンド構築時 |
| 3 | GitHub Pagesの実際の公開URLが確定次第、本ドキュメントに追記する | URL確定時 |

## 変更履歴
- 初版: タブ1（OHLCVマージ機能、クライアントサイドJS実装）、タブ2（Coming Soon）を作成
- CSSのセレクタ規約違反（id・要素型セレクタの使用）を修正し、class セレクタのみに統一
