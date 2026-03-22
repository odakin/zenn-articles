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
- 記事の追加: `npx zenn new:article --slug my-slug` → 編集 → push
- プレビュー: `npx zenn preview` → http://localhost:8000
- 公開: frontmatter の `published: false` → `true` に変えて push
- デプロイ: GitHub 連携により `articles/` を含む push で自動デプロイ

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
