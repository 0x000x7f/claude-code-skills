# claude-code-skills

📖 [English](README.md) | **日本語**

実際に使っている [Claude Code](https://claude.com/claude-code) の個人スキル集。

## `grill` / `grill-deep` — 実装前に計画を詰める

2段構えの「grilling（詰問）」インタビュー。計画を **1問ずつ**——各質問に *なぜ重要か*・*潰せるリスク*・*推奨デフォルト* を添えて——実行できる精度になるまで問い詰めます。

| コマンド | 重さ | 用途 |
|---|---|---|
| `/grill` | 軽 | 小〜中タスク — ベースのインタビュー＋唯一の必須ガードレール |
| `/grill-deep` | 重 | 高stakes（インフラ・セキュリティ・不可逆な変更・データ取扱い・残す成果物）— 成果物の検査・前提への挑戦・横断的リスク掃引・完全性パスを追加 |

どちらも **手動起動のみ**（`disable-model-invocation: true`）。勝手には発火せず、「この計画は詰める価値がある」と判断したときだけ撃ちます。

## なぜ2段か — 「検証ステップ」の欠落

これは Matt Pocock 氏の [`grilling`](https://github.com/mattpocock/skills) スキルのコピーから始まりました。鵜呑みにする前に、素のスキルと強化版を実際の計画で **A/B検証** し、技術的主張を1つずつ公式ドキュメントと突き合わせました。

素のスキルは **優秀でした——技術的主張 15/15 が正しい**。ただし *検証ステップ* が無く、機序を記憶から断定し、「裏取りした事実」と「自信のある推測」を区別できません。ある実行では **誤った機序を自信満々に断定** しました——「Looker Studio はシートを `gid` で参照する」（実際は **タブ名** で参照）。それを咎める仕組みは無し。「今回は当たった」は方法ではなく運です。

そこで強化版に5つのガードレールを追加。最重要はこれ:

> **決定を左右する主張は検証する。できなければ `[UNVERIFIED]` と明示する** — 自信のある推測を「裏取り済みの事実」に化けさせない。

`grill` はこの核だけを残し、`grill-deep` は5つすべてを実行します（他に: 見た目でなく成果物を開く／前提を疑う／privacy・冪等性・クォータ・秘密などの横断リスクを掃引／最後に「何を見落としたか」を点検）。

手法と全結果: [docs/grilling-ab-test.md](docs/grilling-ab-test.md)

## インストール

スキルのフォルダを個人スキルディレクトリにコピー:

```sh
cp -r skills/grill       ~/.claude/skills/grill
cp -r skills/grill-deep  ~/.claude/skills/grill-deep
```

コマンドが出てこない場合は Claude Code を再起動し、`/grill` または `/grill-deep` を実行。

## クレジット

`grilling` のインタビュー手法は [mattpocock/skills](https://github.com/mattpocock/skills) のもの。検証＋リスク掃引のガードレールと light/deep の分割は私によるものです。
