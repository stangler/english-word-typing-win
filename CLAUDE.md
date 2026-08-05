# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 概要

NEW CROWN Lesson 1〜4 + 小学校の単語を対象とした、ブラウザで直接動作するタイピング練習アプリです。
フレームワーク不使用の単一HTMLファイルアプリで、ExcelデータをJSONに変換するビルドスクリプトを含みます。

## 開発コマンド

```bash
# 依存パッケージのインストール
pnpm install

# Excel → JSON ビルド（単語データの生成）
pnpm run build

# テスト — 未設定（このプロジェクトにはテストは存在しない）
```

## 主なファイル

| ファイル | 説明 |
|---|---|
| `english_typing.html` | メインアプリ単一ファイル。HTML/CSS/JSが全て含まれる。ブラウザで直接開くだけで動作 |
| `build.mjs` | SheetJSを使って `xlsx/` のExcelファイルから `json/` にJSONデータを生成するビルドスクリプト |
| `json/words-data.js` | ビルド成果物。`window.WORDS` として全単語データを公開（HTMLから直接読み込み用） |
| `xlsx/*.xlsx` | ソースの単語データ。編集したら `pnpm run build` で再生成 |

## アーキテクチャ

- **単一HTMLファイル構成**: `english_typing.html` にCSS・JSが全て埋め込まれている。SPA的な画面遷移（start → quiz → result/history）を `<section>` の hidden属性で制御
- **データフロー**: Excel (`xlsx/`) → `build.mjs` → JSON (`json/`) → `json/words-data.js` → HTML内JSで `window.WORDS` として読み込み
- **状態管理**: グローバルの `state` オブジェクトに現在の問題キュー・スコア・間違えた問題などを保持
- **永続化**: `localStorage` にテスト履歴 (`typingHistory`) と出題済み問題の進捗 (`typingAttempted`) を保存
- **チャート**: Canvas APIによる手描きの折れ線グラフ（外部ライブラリ不使用）

## 単語データ構造

各単語オブジェクト:
```js
{ lesson: "1-1", part: "1", en: "hello", answer: "hello", ipa: "heˈlóu", pos: "名", ja: "こんにちは", ex_en: "...", ex_ja: "...", memo: "..." }
```

- `lesson`: サブレッスンID（`1-1` のような `番号-パート` 形式、`elementary`、または数値のみ）
- `answer`: `〜` 以下の表現を除いた答え用英語（ビルド時に自動生成）
- `pos`: 品詞タグ（名/動/形/副/句など）

## 制約と注意

- サーバー不要で動作するため `file://` プロトコルでも動くが、`json/words-data.js` は外部スクリプトとして `<script src>` で読み込まれる
- パッケージマネージャーは pnpm (v11.10.0)。`.npmrc` で `shamefully-hoist=true` が設定されている
- `json/` と `xlsx/` はGit管理対象。クローン後は `pnpm install && pnpm run build` でデータを再生成可能
- 依存は `xlsx` (SheetJS) だけのミニマム構成
- Dockerコンテナ環境 (`devcontainer/`) が定義されているが、開発には必須ではない