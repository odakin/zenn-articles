# zenn-articles

Zenn.dev 記事の一元管理リポジトリ。zenn-cli + GitHub 連携で自動デプロイ。

## 記事一覧

| スラッグ | 記事 | 状態 |
|---------|------|------|
| `64d591c7e2be9f` | AIは《陰謀論》の烙印を恐れて証拠を歪める（前編） | 公開済 |
| `45c18ff49d5ddc` | AIは《陰謀論》の烙印を恐れて証拠を歪める（後編） | 公開済 |
| `36225f978acbaa` | AIs Distort Evidence to Avoid "Conspiracy" (Part 1) | 公開済 |
| `90ba7a1bdeeed7` | AIs Distort Evidence to Avoid "Conspiracy" (Part 2) | 公開済 |
| `60344cac5d34ca` | 原文ママ版（前編） | 公開済 |
| `fcb2b33a043da0` | 原文ママ版（後編） | 公開済 |
| `claude-code-multi-project` | Claude Codeが全部忘れる問題を解決する | 未公開 |
| `claude-code-multi-project-en` | Solving Claude Code's Memory Loss | 未公開 |

## 使い方

```bash
npx zenn preview    # ローカルプレビュー → http://localhost:8000
npx zenn new:article --slug my-new-article   # 新規記事作成
```

記事を編集して `git push` すると zenn.dev に自動デプロイされる。

## セットアップ

zenn.dev ダッシュボード → デプロイ管理 → GitHub 連携で `odakin/zenn-articles` を接続。
