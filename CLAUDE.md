# zenn-articles

## 概要
Zenn.dev 記事の一元管理リポ。zenn-cli + GitHub 連携で自動デプロイ。

## リポジトリ情報
- パス: `~/Claude/zenn-articles/`
- ブランチ: `main`
- リモート: `odakin/zenn-articles` (public, GitHub)

## 構造
```
zenn-articles/
├── CLAUDE.md        # このファイル
├── README.md        # 記事一覧・使い方
├── articles/        # Zenn 記事（スラッグ名.md）
├── books/           # Zenn 本（未使用）
├── package.json     # zenn-cli 依存
└── .gitignore
```

## 運用
記事追加・プレビュー (`npx zenn preview`)・公開・デプロイの手順は **README の「使い方」/「セットアップ」が正本** (= 公開リポなので運用コマンドの home は README、CLAUDE.md は重複させず pointer。`~/Claude/claude-config/CONVENTIONS.md` §README の流儀)。

## 執筆規約
Zenn.dev の platform 仕様 (タイトル 70 字 / HTML サニタイズ / `:::message`・`:::details` / 文字数見積もり / frontmatter) と GFM 執筆の落とし穴 (bold × 全角句読点 等) の正本は `~/Claude/claude-config/conventions/zenn.md`。本リポは zenn-cli 運用に閉じる。

## How to Resume
1. SESSION.md を読む（未公開記事リスト・直近の作業）
2. README.md の記事一覧を参照
3. 変更後は commit + push

## 規約
- `~/Claude/CONVENTIONS.md` に従う
- **このリポは public。** 以下を絶対にコミットしない:
  - 実名（GitHub ユーザー名 `odakin` は可）
  - メールアドレス
  - 非公開リポ名
  - 金融データ・口座情報
  - 所属機関名
  - 他ユーザーのユーザー名
