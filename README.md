# zenn-articles

Zenn.dev 記事の一元管理リポジトリ。zenn-cli + GitHub 連携で自動デプロイ。

## 記事一覧

| スラッグ | 記事 | 状態 |
|---------|------|------|
| `64d591c7e2be9f` | AIは《陰謀論》の烙印を恐れて証拠を歪める（前編） | 公開済 |
| `45c18ff49d5ddc` | AIは《陰謀論》の烙印を恐れて証拠を歪める（後編） | 公開済 |
| `60344cac5d34ca` | 原文ママ版（前編） | 公開済 |
| `fcb2b33a043da0` | 原文ママ版（後編） | 公開済 |
| `claude-code-multi-project` | Claude Codeが全部忘れる問題を解決する | 未公開 |

英語記事は [`odakin/zenn-articles-en`](https://github.com/odakin/zenn-articles-en) に移行済み。

## 使い方

```bash
npx zenn preview    # ローカルプレビュー → http://localhost:8000
npx zenn new:article --slug my-new-article   # 新規記事作成
```

記事を編集して `git push` すると zenn.dev に自動デプロイされる。

## セットアップ

zenn.dev ダッシュボード → デプロイ管理 → GitHub 連携で `odakin/zenn-articles` を接続。

## TODO

- [x] ~~リポを日本語用・英語用に分離~~ → 英語は `odakin/zenn-articles-en`
- [x] ~~既存の英語記事を英語リポに移行~~
- [ ] 英語用 Zenn アカウントを作成し `zenn-articles-en` を GitHub 連携
- [ ] 日本語リポから英語記事ファイルを削除（英語アカウント側で動作確認後）
- [ ] このリポの GitHub 連携が日本語アカウントに紐づいていることを確認
