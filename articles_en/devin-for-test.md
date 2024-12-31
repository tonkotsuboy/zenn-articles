---
title: How to Let Devin AI Write Tests and Work Until CI Passes
published: true
description: the practical usage of Devin and tips for effectively utilizing it
tags: ai,devin,testing
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/u3oo1ka5ec94k7zujs6e.png
# Use a ratio of 100:42 for best results.
published_at: 2024-12-31 08:14+0000
---

Devin is an AI platform designed to automate and streamline tasks in software development. It was officially released in December 2024 and was recently introduced to Ubie, the company I work for. Among its many features, I was blown away by how seamlessly it wrote missing tests for a repository—so much so that I almost fell off my chair.

https://devin.ai/

This article introduces the practical usage of Devin and tips for effectively utilizing it.

## 1. Requesting Test Creation via Slack

You can ask Devin to "write tests for this and that" in Slack, and it will generate test code and create a new pull request (PR) on GitHub.

Here’s an example of a request:

```markdown
Hello, @Devin, please do the following tasks:
- Access the `ubie-inc/<repository-name>` repo.
- Write tests for `<test-path>`.
- Use the following test examples as a reference:
  - `foo/index.test.tsx`
  - `bar/index.test.tsx`
- Create a PR with the implemented changes.
- Write the PR title and summary in Japanese.
- Start the branch name with `test`.
- Include `Co-Authored-By: GitHubUsername <email>` in the commit message.
- Ensure no warnings appear in the "Lint and Test" step of CI.
- Do not include `import '@testing-library/jest-native/extend-expect';`.
- Do not include `import React from 'react';`.
- Create the PR as a draft.
```

### Example Request Made in Slack

![Example Request in Slack](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1mie9dvd5pdaz9ek3x94.png)

### Tips for Requests

It’s smoother if you provide test examples that follow the conventions of your project. If there are no examples, Devin can start from scratch, but the initial tests might take longer to develop. Writing the first one or two tests yourself can speed things up.

A recommended approach is to use tools like [GitHub Copilot](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot), [Cursor](https://www.cursor.com/), or [JetBrains AI Assistant](https://plugins.jetbrains.com/plugin/22282-jetbrains-ai-assistant) to analyze your project while writing initial tests.

## 2. Wait for Updates Once Work Starts

Devin will start working on your request. Wait for updates as it proceeds.

![Devin Creating Tests](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8bw7okjoerfsk37n6df8.png)

## 3. CI Failures Are Automatically Fixed

One of Devin's most exciting features is its ability to automatically fix CI failures until the tests pass. This eliminates the need for additional requests like "Please fix the CI issues."

![Fixing CI Failures Automatically](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3rhy4ov3c2s5tddsax6m.png)
*Devin detects CI failures and automatically fixes them.*

If you want to ignore specific CI failures, you can comment on Slack or Devin with instructions like "Ignore failures in <specific CI>" to proceed.

## 4. Requesting Additional Work

After Devin completes its task and creates a PR, you can request additional work either during or after the process. Here are some methods:

### Method 1: Request via Slack Comments

The easiest way is to comment on Slack for additional fixes. For example, you can ask Devin to remove unnecessary `import` statements, as shown below:

![Requesting Fixes in Slack](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jd8kx12tr17rdrjtitjo.png)


This convenience allows for multitasking—I was able to request corrections from my iPhone while watching TV in my living room.

### Method 2: Request via GitHub PR

You can also request work directly on GitHub by commenting on the diff sections of the PR. Devin will pick up the comments and start working.

![Requesting Fixes on GitHub PR](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xdh9nbbza3regu243ykk.png)



### Method 3: Request via Devin's UI

You can request work through Devin's UI. Once Devin starts working, a session is created at `https://app.devin.ai/sessions/<session-id>`. Open the editor in the Editor tab to make references or additional requests for the changes you want. Here’s an example of requesting a change in mock test methods:

![Requesting Fixes in Devin UI](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/x1h14mmia443xoppk12w.png)

## Feedback Knowledge Accumulation

One powerful feature of Devin is its ability to learn and retain feedback. For example, if you instruct it to "Use `jest.spyOn()` for mocking," Devin will save this as "Knowledge" and apply it in future tasks. You can choose whether to use the saved instructions as they are, modify them, or reject them.

![Saving Knowledge Instructions](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/u2lixdpgz37ehgb18yll.png)
*Devin suggesting to save instructions as Knowledge.*

The saved Knowledge is tied to specific repositories and will be reused in subsequent tasks.

![Knowledge Overview](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4boax5q8tip9vj4or46x.png)

You can freely add, modify, or delete Knowledge, eliminating the need to write lengthy prompts every time and ensuring consistency in repository coding policies.

[Learn More About Knowledge](https://docs.devin.ai/onboard-devin/knowledge)

## Areas for Improvement

While Devin is incredibly helpful, engineers still need to verify test content for domain knowledge and final checks. Initially, we considered having non-engineers add and generate tests entirely, but some of the outputs required revisions. However, it’s far easier to tweak outputs than to write everything from scratch.

I also felt that work speed could be improved. As mentioned earlier, providing examples and clear instructions upfront generally results in quicker and higher-quality outputs.

## Testing Is Just the Start—The Possibilities Are Endless

While this article focused on automatic test generation, Devin can handle a wide range of tasks. For example, our team has used it for:

- Refactoring tasks we've been postponing
- Adding error handling
- Migrating settings from local storage to a database
- Removing unused `node_modules` from `package.json`
- Fixing UI bugs under specific conditions
- Writing documentation
- Setting up CI/CD pipelines we've been meaning to create
- And much more

## Pricing

Devin offers usage-based pricing starting at $500 per month. Whether this is expensive or reasonable depends on your team’s circumstances, but having a full-stack engineer working 24/7 for $500/month seems like a bargain to me. For comparison, hiring an engineer at $50/hour for 20 days costs $8,000, or $16,000 at $100/hour. Discuss with your team to see if it’s a good fit.

[Pricing Details](https://devin.ai/pricing)

## Let AI Handle Tedious Tests

An AI that analyzes your codebase on GitHub, writes test code autonomously, works until CI passes, and handles additional requests has been a long-time dream of mine.

I’ll continue tweeting about new AI updates, so be sure to follow me!

https://x.com/tonkotsuboy_en/
