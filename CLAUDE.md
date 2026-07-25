# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

このリポジトリは「ストップウォッチ」という単一ページの PWA（Progressive Web App）です。ビルドツール・パッケージマネージャ・依存関係は一切なく、すべてが `index.html` 1ファイルに完結しています。

## Files

- `index.html` — アプリ全体（HTML/CSS/JS がすべてインライン）。ビュー、スタイル、ロジックの唯一の実装箇所。
- `manifest.json` — PWA マニフェスト（アプリ名、アイコン、テーマカラー、`start_url` など）。
- `icon.png` — ホーム画面用アイコン（180x180、`apple-touch-icon` および manifest から参照）。

## Development

ビルド・lint・テストの仕組みは存在しません。`index.html` をブラウザで直接開く（またはローカルの静的サーバーで配信する）だけで動作確認できます。

```bash
python3 -m http.server 8000   # 例: ローカルで確認する場合
```

変更後は、ブラウザ（特にモバイル Safari のホーム画面追加動作）で開始/停止/ラップ/リセットの一連の操作を手動で確認してください。

## Architecture

- **状態管理**: グローバルではなく `index.html` 内の IIFE (`(function () { ... })()`) にスコープされた変数（`running`, `startTimestamp`, `elapsedBeforeStart`, `laps`, `lastLapElapsed`）で管理。フレームワークや外部状態管理ライブラリは使用していない。
- **時刻更新ループ**: `requestAnimationFrame` による `tick()` で描画を更新（`setInterval` は不使用）。経過時間は `Date.now()` の差分から都度計算し、タイマーの累積誤差を避けている。
- **ボタンの二重役割**:
  - 開始/停止ボタン (`#startStopBtn`) は状態に応じて開始・停止をトグルする。
  - ラップボタン (`#lapBtn`) は計測中は「ラップ記録」、停止中は「リセット」として機能する（テキストとイベント分岐で切り替え）。
- **ラップのベスト/ワースト判定**: `renderLaps()` 内でラップが2件以上のときのみ最速・最遅を計算し、それぞれ `.best` / `.worst` クラスを付与して色分け表示する。
- **PWA 化**: `manifest.json` と `apple-touch-icon` により、iOS/Android でホーム画面に追加してスタンドアロンアプリとして動作させることを想定している（`viewport-fit=cover` や `env(safe-area-inset-*)` でノッチ端末に対応）。
