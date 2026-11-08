---
layout: page
title: Running e2e 0.23 agent steps through Claude Code over ACP
---

> 日本語版はこちらをご覧ください: [e2e 0.23でエージェントステップをACP経由でClaude Codeに実行させる](https://qiita.com/two_pack/items/71b27f8d66c78bebe56d)
>
> (A Japanese version of this article is available on Qiita at the link above.)

> I wrote this article together with Claude Code. Everything here was checked as of November 8, 2026.

[e2e](https://github.com/tester-army/e2e) is an E2E testing tool that lets an agent operate and check your screen. `agent.act` performs an action written in natural language, and `agent.assert` checks the state of the screen. I call these two "agent steps" below.

e2e 0.23 (released on November 5, 2026) can now run agent steps through Claude Code, using your own Claude login. e2e talks to Claude Code over ACP. I gave it a try.

> **Note on terms of use**
>
> This article follows the procedure described on [the e2e site](https://e2e.com/docs/subscriptions#use-your-coding-agent) and reports what happened. Anthropic's [Legal and compliance](https://code.claude.com/docs/en/legal-and-compliance) page says that third-party developers may not build claude.ai login into their own apps, or send requests on a user's behalf with their subscription credentials. It also says that this does not stop users from signing in to an unmodified Claude Code with their own subscription. Please read the terms yourself and use the feature in compliance with them.

The repository I used for the trial is public:

<https://github.com/two-pack/e2e-0.23-trial>

## What is ACP?

ACP stands for Agent Client Protocol, a common protocol for communication between a client such as an editor and a coding agent. The specification is published at [agentclientprotocol.com](https://agentclientprotocol.com). Here, e2e acts as the ACP client and drives Claude Code through an adapter.

The adapter, `@agentclientprotocol/claude-agent-acp`, is published by Zed Industries. It runs on Anthropic's official Claude Agent SDK, and e2e's documentation also introduces it as [Zed's adapter](https://e2e.com/docs/subscriptions#use-your-coding-agent).

According to the [release post](https://e2e.com/blog/e2e-0-23), e2e can also drive Codex with `codex()` and any other ACP agent with `acp({ command, args })`. In this article I only try Claude Code.

## Environment

- Windows 11
- WSL2 (Ubuntu 24.04 LTS)
- Node.js 24.22.2
- e2e 0.23.0
- @e2e-dev/acp 0.1.1
- @agentclientprotocol/claude-agent-acp 0.101.0

The Claude Code that the adapter ran was version 2.1.318, which is bundled with the Agent SDK the adapter uses.

## Setup

First, install two packages. The first handles ACP on the e2e side, and the second is the adapter for Claude Code.

```
npm install -D @e2e-dev/acp @agentclientprotocol/claude-agent-acp
```

In `e2e.config.ts`, add the `claudeCode` import and one `agents` line.

```ts
import type { E2EConfig } from 'e2e';
import { claudeCode } from '@e2e-dev/acp';

export default {
  // targets omitted
  agents: { default: { executor: claudeCode({ model: 'sonnet' }) } },
} satisfies E2EConfig;
```

The app under test is the todo app in the `app/` directory of the repository. I configured `targets` so that e2e starts it with `node app/server.js`.

For authentication, either sign in with `claude` or set the environment variable `ANTHROPIC_API_KEY`, as [the e2e documentation](https://e2e.com/docs/subscriptions#use-your-coding-agent) says. I only signed in with `claude` and did not set `ANTHROPIC_API_KEY`. If you are not signed in, e2e stops with "the agent is not signed in".

### Tools Claude Code can use

e2e does not let Claude Code use its own tools. Tools such as Bash and file read/write are disabled, and your local user settings, project settings, and other MCP servers are not loaded. The only tools available are the step tools that e2e provides. They include tools for inspecting and operating the screen, and `complete_step`, which returns the result of the step.

The reason is that if Claude Code operated with its own tools, e2e could not record those operations, and so could not replay them later. The explanation is in [What the agent can use](https://e2e.com/docs/subscriptions#what-the-agent-can-use) in the documentation. If Claude Code runs one of its own tools without permission, that step fails with `POLICY_DENIED`.

## Checking that it works

`npx e2e doctor --check-model` is a command that diagnoses the e2e environment. It checked Node.js, the config file, and the browser, but the model line was shown as `?`.

```
? model: [default] executor "claude-code" brings its own agent, which doctor does not check
```

Claude Code calls the model with its own login and does not use e2e's model settings, so `doctor` does not check this line. To find out whether the login works, I ran a single agent step: a test with only `agent.assert`.

```ts
import { test } from '@e2e-dev/web';

test('agent sees the todo form', async ({ app, agent }) => {
  await app.open('/');
  await agent.assert('a todo form with a "New todo" input and an Add button is shown');
});
```

Here is an excerpt of the output.

```
✓ agent sees the todo form > agent.assert "a todo form with a "New todo" input and an Add button is shown" 5.35s · 1 model call
AI  24.7k tokens · 84% cached · $0.0200 · 1 model calls · @agentclientprotocol/claude-agent-acp/sonnet
```

The last line shows the adapter and the model. The model was called once, and the step passed.

## Replaying act

e2e records the actions of `agent.act` and [replays them from the recording](https://e2e.com/docs/cache) on the next run. I checked whether this works the same way over ACP, with a test that puts an `agent.assert` after an `agent.act`. When the `agent.assert` passes, the recording of the `act` is saved.

```ts
test('adds a todo', async ({ app, agent }) => {
  await app.open('/');
  await agent.act('add a todo named {title}', { params: { title: 'Buy milk' } });
  await agent.assert('"Buy milk" is listed and the count shows "1 todo"');
});
```

These are the results of running `npm run test:e2e` twice in a row. Each time is a single measurement and varies from run to run.

| Run | `act` | `assert` |
| --- | --- | --- |
| 1st | 7.81 s, 1 model call | 8.23 s, 1 model call |
| 2nd | 0.252 s, no model call | 5.04 s, 1 model call |

On the second run, `act` was replayed from the recording without calling the model. `assert` is executed by the model every time. Note that ACP does not report requests to the model, so e2e counts one round trip with the agent as one model call. This is stated under Limits in [the documentation](https://e2e.com/docs/subscriptions#use-your-coding-agent).

## Impressions

I confirmed that two lines in the config file are enough to have Claude Code run the agent steps.
