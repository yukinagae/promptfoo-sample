[Promptfoo](https://www.promptfoo.dev/) は生成 AI の出力を評価するテストフレームワークです。

https://www.promptfoo.dev/

https://github.com/promptfoo/promptfoo

## 背景

生成 AI の機能・サービスを開発するプロセスの中で、以下のような様々な変更作業があります。

- 生成 AI モデルの変更（例: GPT-4 から Gemini 1.5 Pro への変更 etc.）
- プロンプトの変更・改善
- RAG 用のデータの登録・更新・追加
- etc.

これらの変更作業は生成 AI の機能改善のために適宜行われますが、小さな変更でも大きく出力が変わる可能性があります。

例えば、以下様々なデグレが起こりえます。

- 例: 新しくリリースされたモデルに変更したら過去のモデルに比べて出力の品質が低下した
- 例: プロンプトを 1 行変更したら余計な出力が出るようになった
- 例: RAG 用のデータを変更したら、今まで回答できていた質問に答えなくなった
- etc.

変更のたびに人間がデグレをチェックするのは大変ですし、時間やリソースが大きくかかってしまいます。

調べたところ、以下様々な生成 AI 評価フレームワークが存在します（全てチェックできてるわけではないです）

- [LangChain](https://github.com/langchain-ai/langchain)
- [Langfuse](https://github.com/langfuse/langfuse)
- [OpenAI Evals](https://github.com/openai/evals)
- [DeepEval](https://github.com/confident-ai/deepeval)
- [PromptLayer](https://github.com/MagnivOrg/prompt-layer-library)

## Promptfoo

何個か触ってみたところ、今回の記事の主題である [Promptfoo](https://www.promptfoo.dev/) というツールが使いやすそうなので紹介します。

特徴としては、以下いろいろ書いてますが、とりあえず良さそうな印象(_´∀ ｀_)

- ローカルで完結できる
- 学習コストが低い（頑張れば非エンジニアでも利用できるかもしれません）
  - CLI をスタンドアローンでインストールするだけなので、環境構築が比較的簡単
  - yaml ベースで記述できる
  - 基本的な assert 処理がビルトインで用意されている
- prompt x モデル x テストケース のようにマトリクスでテストできる
  - prompt やモデル比較が容易
- 意外とカスタマイズできる
  - assert 処理は javascript/python で独自処理を書ける
  - provider という仕組みでテスト対象をフレキシブルに変更できる
  - 基本は CLI 使用を想定しているが、npm パッケージからも呼べる

## 試しに動かしてみる

今回作成した sample は以下 github にアップしているので、これを git clone して試すこともできます。

https://github.com/yukinagae/promptfoo-sample

### インストール

[Promptfoo - Installation](https://www.promptfoo.dev/docs/installation/) を参考にしてください。

- ディレクトリを作成

```bash
$ mkdir promptfoo-sample
```

- promptfoo の CLI をインストール

```bash
# おすすめ
$ npm install -g promptfoo
# or
$ brew install promptfoo
```

- インストール確認

※npm で最新バージョンをインストールすることを推奨しますが、今回は brew でインストールします。

```bash
$ which promptfoo
/opt/homebrew/bin/promptfoo
$ promptfoo --version
0.83.2
```

- 初期化（※とりあえず一番 minimum として以下を選択）
  - What would you like to do?: `1`
  - Which model providers would you like to use?: `Choose later`

```bash
$ promptfoo init

? What would you like to do? 1
  1) Not sure yet
  2) Improve prompt and model performance
  3) Improve RAG performance
  4) Improve agent/chain of thought performance
  5) Run a red team evaluation

? Which model providers would you like to use? (press enter to skip)
❯◉ Choose later
 ◯ [OpenAI] GPT 4o, GPT 4o-mini, GPT-3.5, ...
 ◯ [Anthropic] Claude Opus, Sonnet, Haiku, ...
 ◯ [HuggingFace] Llama, Phi, Gemma, ...
 ◯ Local Python script
 ◯ Local Javascript script
 ◯ Local executable

✅ Wrote promptfooconfig.yaml. Run `promptfoo eval` to get started!
```

以下 2 つのファイルがまず作成されているはずです。

- README.md
- promptfooconfig.yaml

### テストしてみる

新規作成されている `README.md` に記載されている通り、以下 3 ステップでサンプルのテストが実行できます。

1. OpenAI の API キーを環境変数に設定する

```bash
$ export OPENAI_API_KEY=<your-openi-api-key>
```

2. promptfoo で評価を実行する

```bash
$ promptfoo eval
```

3. Web GUI でテスト結果を確認します。以下のコマンドで自動的に `http://localhost:15500/eval/` でブラウザが開きます。

```bash
$ promptfoo view --yes
```

![Promptfooの評価結果サンプル - グラフ](https://raw.githubusercontent.com/yukinagae/promptfoo-sample/main/docs/1.png)

![Promptfooの評価結果サンプル - 表](https://raw.githubusercontent.com/yukinagae/promptfoo-sample/main/docs/2.png)

※補足: 12 ケース実行されている内訳は [prompt（2 パターン）] x [providers（2 パターン）] x [テストケース（3 パターン）] でマトリクスでテスト実行されるためです。

### テスト内容を見てみる

`promptfooconfig.yaml` に全てのテストが記述されています。

```yaml
description: "My eval"

prompts:
  - "Write a tweet about {{topic}}"
  - "Write a concise, funny tweet about {{topic}}"

providers:
  - "openai:gpt-4o-mini"
  - "openai:gpt-4o"

tests:
  - vars:
      topic: bananas

  - vars:
      topic: avocado toast
    assert:
      - type: icontains
        value: avocado

      - type: javascript
        value: 1 / (output.length + 1)

  - vars:
      topic: new york city
    assert:
      - type: llm-rubric
        value: ensure that the output is funny
```

内容を順々に見ていきます。

- description

テスト全体の説明文です。Web GUI で見た時に左上に表示されます。

```yaml
description: "My eval"
```

![Promptfooの評価結果サンプル - descriptionの表示場所](https://raw.githubusercontent.com/yukinagae/promptfoo-sample/main/docs/3.png)

- prompts

生成 AI に渡す prompt を指定します。ここでは 2 パターンの prompt を記載しています。

```yaml
prompts:
  - "Write a tweet about {{topic}}"
  - "Write a concise, funny tweet about {{topic}}"
```

- providers

実行する生成 AI モデルを指定します。ここでは 2 パターンのモデルを記載しています。

指定できる provider は [Promptfoo - Providers](https://www.promptfoo.dev/docs/providers/) を参照。

OpenAI の場合には [Promptfoo - Providers > OpenAI](https://www.promptfoo.dev/docs/providers/openai/) に全て記載されています。

```yaml
providers:
  - "openai:gpt-4o-mini"
  - "openai:gpt-4o"
```

- tests

`vars` で変数、`assert` でテスト項目を記述できます。

特に指定できるテスト項目は多いので [Promptfoo - Assertions & metrics](https://www.promptfoo.dev/docs/configuration/expected-outputs/) を参照してください。

```yaml
tests:
  - vars: # 変数 `topic` に `bananas` を指定して実行（※assert無しで出力するだけ）
      topic: bananas

  - vars: # 変数 `topic` に `avocado toast` を指定して実行
      topic: avocado toast
    assert:
      - type: icontains # テスト結果に `avocado` の文字列が含まれていることをチェック
        value: avocado

      - type: javascript # `javascript` や `python` でカスタムチェックできる
        value: 1 / (output.length + 1)

  - vars: # 変数 `topic` に `new york city` を指定して実行
      topic: new york city
    assert:
      - type: llm-rubric # 別の生成AIモデルで出力結果をチェックさせる（デフォルト: gpt-4o）
        value: ensure that the output is funny
```

## 応用

promptfoo の github repo に様々な活用例があるので参考にしてください。

https://github.com/promptfoo/promptfoo/tree/main/examples

また、公式ドキュメントが豊富なため、特に以下見るとやりたいことができるかなと思います。

https://www.promptfoo.dev/docs/configuration/parameters/

## 注意点

**Promptfoo は基本的にローカル完結なのですが、以下の点に注意してください。**

**Web GUI の share ボタンや `promptfoo share` を実行すると、Promptfoo が提供している Cloudflare KV 経由でテスト結果がパブリックに共有されてしまいます（2 週間のみ保存）。**

基本的にはこの機能は使わない方がよいと思います。

> No, Promptfoo operates locally, and all data remains on your machine. The only exception is when you explicitly use the share command, which stores inputs and outputs in Cloudflare KV for two weeks.

https://www.promptfoo.dev/docs/faq/#does-promptfoo-store-llm-inputs-and-outputs
