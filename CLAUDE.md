# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 概要

Claude Codeの操作練習用ワークスペース。WEBアプリを作成してGitHub Pagesで公開することを目的としています。

## 方針

- **返答は日本語で簡潔に**
- GitHub Pages での公開を前提に、静的サイト（HTML/CSS/JS）またはフレームワーク（Vite等）で構成する
- `claude_code_reference.md` はClaude Code / Anthropic APIのリファレンスとして参照可能

## GitHub Pages 公開手順

```bash
# リポジトリのリモートを確認
git remote -v

# main ブランチにプッシュ（GitHub Pages の設定に応じて）
git push origin main

# または gh-pages ブランチを使う場合
git checkout -b gh-pages
git push origin gh-pages
```

GitHub リポジトリの Settings → Pages でソースブランチを選択して公開。

## よく使うコマンド

WEBアプリの構成が決まったら、このセクションにビルド・起動コマンドを追記する。

```bash
# Vite を使う場合の例
npm install
npm run dev      # 開発サーバー起動
npm run build    # dist/ に静的ファイル生成
npm run preview  # ビルド結果をプレビュー
```
