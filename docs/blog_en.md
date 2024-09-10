In this blog post, we'll explore how to use [Promptfoo](https://www.promptfoo.dev/), a test framework designed to evaluate the output of generative AI models.

https://www.promptfoo.dev/

https://github.com/promptfoo/promptfoo

## Background

In the process of developing generative AI features and services, there are various changes such as:

- Changing the generative AI model (e.g., from GPT-4 to Gemini 1.5 Pro, etc.)
- Modifying and improving prompts
- Registering, updating, and adding data for RAG

These changes are made as needed to improve the functionality of generative AI, but even small changes can significantly alter the output.

For example, various regressions can occur:

- Example: Changing to a newly released model resulted in lower output quality compared to the previous model
- Example: Modifying a single line in the prompt caused unnecessary output
- Example: Changing the data for RAG resulted in the inability to answer questions that could previously be answered

Manually checking for regressions with each change is labor-intensive and requires significant time and resources.

Upon investigation, various generative AI evaluation frameworks exist (not all have been checked).

- [LangChain](https://github.com/langchain-ai/langchain)
- [Langfuse](https://github.com/langfuse/langfuse)
- [OpenAI Evals](https://github.com/openai/evals)
- [DeepEval](https://github.com/confident-ai/deepeval)
- [PromptLayer](https://github.com/MagnivOrg/prompt-layer-library)

## Promptfoo

After trying out a few tools, I found that [Promptfoo](https://www.promptfoo.dev/), the main subject of this article, seems to be user-friendly, so I will introduce it.

Here are some features that give a good impression:

- Can be completed locally
- Low learning cost (even non-engineers might be able to use it with some effort)
  - It's relatively easy to set up the environment since you only need to install the CLI standalone
  - Can be described in yaml
  - Basic assert processing is built-in
- Can test in a matrix like prompt x model x test case
  - Easy to compare prompts and models
- Surprisingly customizable
  - Assert processing can be written in javascript/python
  - The target of the test can be flexibly changed with the provider mechanism
  - Although it is mainly intended for CLI use, it can also be called from an npm package

## Try Running It

The sample created this time is uploaded to the following GitHub repository, so you can try it by cloning it.

https://github.com/yukinagae/promptfoo-sample

### Installation

Refer to [Promptfoo - Installation](https://www.promptfoo.dev/docs/installation/).

- Create a directory

```bash
$ mkdir promptfoo-sample
$ cd promptfoo-sample
```

- Install the promptfoo CLI

```bash
# Recommended
$ npm install -g promptfoo
# or
$ brew install promptfoo
```

- Verify the installation

It is recommended to install the latest version via npm, but for this example, we will use brew.

```bash
$ which promptfoo
/opt/homebrew/bin/promptfoo
$ promptfoo --version
0.83.2
```

- Initialization (For now, select the most minimal option below)
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

The following two files should be created first:

- README.md
- promptfooconfig.yaml

### Try Testing

As described in the newly created `README.md`, you can run the sample test in the following three steps:

1. Set the OpenAI API key as an environment variable

```bash
$ export OPENAI_API_KEY=<your-openai-api-key>
```

2. Run the evaluation with the following command:

```bash
$ promptfoo eval
```

4. Check the test results in the Web GUI. The following command will automatically open the browser at `http://localhost:15500/eval/`:

```bash
$ promptfoo view --yes
```

![Sample evaluation results of Promptfoo - Graph](https://raw.githubusercontent.com/yukinagae/promptfoo-sample/main/docs/1.png)

![Sample evaluation results of Promptfoo - Table](https://raw.githubusercontent.com/yukinagae/promptfoo-sample/main/docs/2.png)

_Note: The 12 cases executed are due to matrix testing of [prompts (2 patterns)] x [providers (2 patterns)] x [test cases (3 patterns)]._

### Check the Test Contents

All tests are described in `promptfooconfig.yaml`.

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

Let's go through the contents step by step.

- description

This is the overall description of the test. It will be displayed in the top left corner when viewed in the Web GUI.

```yaml
description: "My eval"
```

![Sample evaluation results of Promptfoo - Location of description](https://raw.githubusercontent.com/yukinagae/promptfoo-sample/main/docs/3.png)

- prompts

Specify the prompts to be passed to the generative AI. Here, two patterns of prompts are listed.

```yaml
prompts:
  - "Write a tweet about {{topic}}"
  - "Write a concise, funny tweet about {{topic}}"
```

- providers

Specify the generative AI models to be used. Here, two patterns of models are listed.

Refer to [Promptfoo - Providers](https://www.promptfoo.dev/docs/providers/) for the providers that can be specified.

For OpenAI, all details are listed in [Promptfoo - Providers > OpenAI](https://www.promptfoo.dev/docs/providers/openai/).

```yaml
providers:
  - "openai:gpt-4o-mini"
  - "openai:gpt-4o"
```

- tests

You can specify variables with `vars` and test items with `assert`.

There are many test items that can be specified, so please refer to [Promptfoo - Assertions & metrics](https://www.promptfoo.dev/docs/configuration/expected-outputs/).

```yaml
tests:
  - vars: # Execute with the variable `topic` set to `bananas` (output only without assert)
      topic: bananas

  - vars: # Execute with the variable `topic` set to `avocado toast`
      topic: avocado toast
    assert:
      - type: icontains # Check that the test result contains the string `avocado`
        value: avocado

      - type: javascript # Custom checks can be done with `javascript` or `python`
        value: 1 / (output.length + 1)

  - vars: # Execute with the variable `topic` set to `new york city`
      topic: new york city
    assert:
      - type: llm-rubric # Check the output result with another generative AI model (default: gpt-4o)
        value: ensure that the output is funny
```

## Advanced Usage

There are various examples of usage in the promptfoo GitHub repository, so please refer to them.

https://github.com/promptfoo/promptfoo/tree/main/examples

Additionally, the official documentation is extensive, so you might find what you want to do by looking at the following:

https://www.promptfoo.dev/docs/configuration/parameters/

## Caution

**Promptfoo is primarily designed to operate locally, but please be aware of the following points.**

**If you use the share button in the Web GUI or execute `promptfoo share`, the test results will be publicly shared via Cloudflare KV provided by Promptfoo (stored for two weeks).**

It is generally recommended to avoid using this feature.

> No, Promptfoo operates locally, and all data remains on your machine. The only exception is when you explicitly use the share command, which stores inputs and outputs in Cloudflare KV for two weeks.

https://www.promptfoo.dev/docs/faq/#does-promptfoo-store-llm-inputs-and-outputs

## Conclusion

In conclusion, Promptfoo is a user-friendly framework for evaluating generative AI outputs. Its local operation, low learning curve, and customization options make it valuable for both developers and non-engineers. Matrix testing and various assertions ensure thorough evaluation of changes.

For more detailed examples and advanced configurations, refer to the [Promptfoo GitHub repository](https://github.com/promptfoo/promptfoo) and the [official documentation](https://www.promptfoo.dev/docs/configuration/parameters/).

Happy testing!
