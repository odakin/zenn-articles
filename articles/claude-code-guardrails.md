---
title: "「Claude Codeが全部忘れる問題」のその後 ― 20リポ運用で踏んだ3つの地雷と自動化した解決策"
emoji: "💣"
type: "tech"
topics: ["ClaudeCode", "AI", "開発環境", "プロジェクト管理", "Claude"]
published: false
---

:::message
この記事の英語版は dev.to で公開しています: [English version](https://dev.to/odakin/stop-babysitting-your-ai-how-i-made-claude-code-enforce-its-own-rules-jmm)
前回の記事: [Claude Codeが全部忘れる問題を解決する―複数プロジェクト運用の設計パターン](https://zenn.dev/odakin/articles/claude-code-multi-project)
:::

前回、Claude Code の autocompact 対策として CLAUDE.md + SESSION.md の2ファイル体制を紹介した。CLAUDE.md に「How to Resume → SESSION.md を読め」と書いておけば、コンテキストが圧縮されても復帰できるという仕組みだ。

これ自体はうまく動いた。動いたのだが、20リポを2週間回しただけで新たに3つの地雷を踏み抜いた。どれも「ルールはちゃんとある。でも現場で壊れる」というやつで、人間あるある……ではなく AI あるあるだった。

結論から言うと、3つとも Claude 自身にガードレールを持たせることで解決した。人間が頑張るのではなく、AI に自分自身を取り締まらせる。その話をする。

https://github.com/odakin/claude-config

---

# 地雷1: Claude、書く場所を盛大に間違える

Claude Code には memory（`~/.claude/` 以下に保存されるファイル）がある。SESSION.md もある。CLAUDE.md もある。DESIGN.md もある。

書き先が4つもあるとどうなるか。Claude は迷った挙句「とりあえず memory に突っ込んどくか」と判断する。気持ちはわかる。人間だってとりあえずデスクトップに保存する。

問題は、memory に書かれた情報は Git に入らないので他の端末から見えないこと。しかも Claude は memory を「関連ありそう」と判断したときしか読まない。SESSION.md なら「How to Resume」で毎回確実に読ませられるのに、memory だとそのコントロールが効かない。

ある日、別の端末でプロジェクトを再開したら「PR #42 がレビュー中」という情報が消えていた。Claude が memory に書いていたのだ。SESSION.md に書いてあれば Git 経由で同期されていたのに。

「タブではなくトグルにした理由」も memory に入っていた。これは設計判断だから DESIGN.md に書くべきもので、memory だと将来の自分が設計意図を追えなくなる。

## 口で言ってもダメなので、物理的に止める

CONVENTIONS.md に「memory にはユーザーの好みだけ書け」とルールを書いた。Claude は3日で忘れた（正確には autocompact で流れた）。

じゃあどうするか。Claude Code の [hooks](https://docs.anthropic.com/ja/docs/claude-code/hooks) で物理的にブロックすることにした。ツール実行の前にシェルスクリプトを挟んで、memory への書き込みを検出したら `exit 2` で止める。

```bash
# memory-guard.sh（簡略版）
INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

# memory ディレクトリじゃなければ素通し
[[ "$FILE_PATH" != *"/.claude/projects/"*"/memory/"* ]] && exit 0
# MEMORY.md（目次ファイル）は通す
[[ "$FILE_PATH" == */MEMORY.md ]] && exit 0

# それ以外はブロック
cat >&2 << 'EOF'
BLOCKED: メモリへの書き込み
CONVENTIONS.md §2「記録先の判別」を確認せよ。
EOF
exit 2
```

Claude が memory に何か書こうとした瞬間、「ダメ。CONVENTIONS.md の判別表を読め」と叱られる。叱られた Claude は判別表を見て、「あ、これは SESSION.md か」と書き先を選び直す。たいへんけなげ。

参照する判別表はこう:

| 情報の性質 | 書き先 |
|---|---|
| ユーザーの好み・フィードバック | memory |
| プロジェクトの作業状態 | SESSION.md |
| 永続的な仕様・構造 | CLAUDE.md |
| 設計判断の理由 | DESIGN.md |
| 全プロジェクト共通ルール | CONVENTIONS.md |
| `grep` / `git log` で分かること | 書かない |

最後の「書かない」が地味に大事。導出可能な情報をわざわざ記録するとファイルが際限なく太る。

---

# 地雷2: 規約ファイルが太りすぎてコンテキストを圧迫する

CONVENTIONS.md は最初7セクションだった。コンパクトでいい感じだった。

そこに LaTeX リポ向けの式の安全規則を足した。Google Calendar MCP のルールを足した。共有リポの Git workflow を足した。足した。足した。気づいたら13セクション。

何が起きるかというと、React でフロントエンドを書いているときにも「LaTeX の equation 環境内は原則として変更しない」というルールがコンテキストに載る。知らんがな。

しかもこれ、autocompact 対策のための CONVENTIONS.md が肥大化して autocompact を加速させるという、笑えない再帰ループになっていた。

## 使わないルールはそもそも読ませない

解決は単純で、ドメイン固有のルールを別ファイルに切り出した:

```
claude-config/
├── CONVENTIONS.md      # 共通ルールだけ（6セクションに圧縮）
└── conventions/
    ├── shared-repo.md  # 共有リポ向け
    ├── latex.md        # LaTeX 向け
    └── mcp.md          # MCP 向け
```

物理論文リポの CLAUDE.md からは `conventions/latex.md` を参照する。Web アプリからは参照しない。それだけ。これで CONVENTIONS.md は半分以下になった。

## SESSION.md にも「予算」を設ける

SESSION.md も油断すると太る。完了タスクの `[x]` が延々残り、試行錯誤の経緯が書き連ねられ、気づくと200行の大作に育っている。SESSION.md は作業日誌じゃない。autocompact 後に「今どこにいて、次に何をするか」がわかればそれでいい。

目安80行という予算を設けて、push のたびに Claude に棚卸しさせている:

- 完了タスクは消す（git log に残ってる）
- 実装の詳細は消す（コミットメッセージに書いてある）
- 恒久的な決定事項は CLAUDE.md に移して SESSION.md から消す

人間が覚えている必要はない。Claude が push のたびに自動でやる。

---

# 地雷3: ドキュメントが現実から静かに乖離する

SESSION.md に「データソースは4本」と書いてある。数えたら6本になっていた。

CLAUDE.md に「画像は `img/` に置く」と書いてある。いつの間にか `assets/` に引っ越していた。

README に「§9 安全規則」と書いてある。リファクタリングで §5 に変わっていた。

ドキュメントは書いた瞬間から腐り始める。これは自然の摂理であり、エントロピー増大の法則である（言い過ぎ）。前回紹介した「push 前チェック」はあったが、「ざっと見て問題なさそうならOK」くらいの粒度で、この手の静かなズレを見逃していた。

## 4軸レビュー: `grep` で殴る

push の前に4つの軸で機械的にチェックするルールにした:

| 軸 | 何を見るか |
|---|---|
| 整合性 | ファイル間で数値・参照先が一致しているか。`grep` で確認 |
| 無矛盾性 | 新しい記述が既存ルールと矛盾していないか |
| 効率性 | 情報の重複がないか。SESSION.md は80行以内か |
| 安全性 | 個人情報が公開リポに紛れていないか |

使い方は push の前に一言:

> 「整合性、無矛盾性、効率性をチェック。プッシュ。」

public リポでは「安全性」を足す。Claude は変更ファイルを起点に `grep` で関連箇所を洗い出し、数値の食い違いや参照の切れを見つける。

これがまあ、よく拾う。今日もこの記事のために claude-config の README を書き直したら、3件引っかかった:

1. CLAUDE.md と README で setup.sh のステップ番号が食い違っていた（片方は「1, 1b, 2-6」、もう片方は「1-7」）
2. README の表から CONVENTIONS.md にある「外部参照」が抜けていた
3. 運用Tips の例文がリファクタリング前の旧セクション番号「§9」を参照していた

全部 push 前に自動修正された。人間が目視で見つけるのは正直厳しい。

---

# おまけ: LaTeX の Unicode 地獄

共同編集者が Word で書いた文字列を `.bib` に貼ると、カーリークォートや em ダッシュが紛れ込んで BibTeX が盛大に死ぬ。よくある。手で直すのは精神衛生に悪いので、Git の pre-commit hook で自動変換するようにした。`setup.sh` が LaTeX リポを検出して勝手にインストールする。コミットした瞬間に Unicode が LaTeX コマンドに変換されるので、もう `.bib` の文字化けで夜中に目が覚めることはない。

---

# まとめ: Claude に自分を取り締まらせる

3つ並べてみると、全部同じ構造をしている:

| 地雷 | 対処 | 誰がやるか |
|------|------|-----------|
| 書き先を間違える | hooks でブロック | Claude |
| 規約が太る | 予算ルールで棚卸し | Claude |
| ドキュメントが腐る | 4軸レビューで検出 | Claude |

人間がチェックリストを毎回見るのは無理だ（3日で忘れる）。でも Claude にルールを渡して「push のたびにこれやれ」と言えば、文句も言わず何百回でもやってくれる。

前回の記事は「Claude に記憶を管理させる」という話だった。今回はその先で、**Claude に品質を管理させる**。ルールを書く → Claude が従う → hooks が逸脱を止める → 4軸レビューがズレを拾う。このループが回れば、リポが増えてもドキュメントは腐らない。

……たぶん。2週間の知見なので、次の地雷はもう埋まっているかもしれない。

---

リポジトリ: https://github.com/odakin/claude-config

前回の記事: [Claude Codeが全部忘れる問題を解決する](https://zenn.dev/odakin/articles/claude-code-multi-project)
