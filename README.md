# MIRAI BOT

**Warning: keep reading, this has almost nothing to do with the [MIRAI static analyzer](https://github.com/facebookexperimental/MIRAI)**.

MIRAI bot is essentially a repository that gets triggered when a new PR gets submitted to the [libra repository](https://www.github.com/diem/diem).

This allows us to perform github actions on libra without having admin access to the repo.

![mirai bot](mirai-bot.png)

## List of bots:

**MIRAI**. Initially the MIRAI bot was in support of running the [MIRAI static analyzer](https://github.com/facebookexperimental/MIRAI) (hence the name).
It only displays the full result to people added to the whitelist.

**Clippy**. This runs [all the clippy rules](https://rust-lang.github.io/rust-clippy/master/) on a PR.

**Wannabe-Land-Blocking**. This runs a set of selected clippy rule on a PR. These rules are ones we want to have as land blocking tests in CI, but cannot at the moment for various reasons.

**Dephell**. Runs [cargo-dephell](https://github.com/mimoo/cargo-dephell) and create diffs for dependencies (from crates.io) on PRs from dependabots, uploads the artifacts to the github action itself.

**Move-prover**. TKTK

**Move-coverage**. TKTK

## How to add a new action

Create a new `YOUR_ACTION_NAME.yml` file in `.github/workflows` with the following template:

```yml
name: YOUR_ACTION_NAME

on:
  repository_dispatch:
    types: [new_PR]

jobs:
  run_YOUR_ACTION_NAME:
    runs-on: ubuntu-latest

    steps:
      - name: Who triggered this?
        env:
          PULL_ID: ${{ github.event.client_payload.pull_id }}
        run: |
          echo "action triggerd by https://github.com/diem/diem/pull/$PULL_ID"

```

then add the **steps** you need to make the action do the thing you want to do (the first one in the template will print out to the log the PR that triggered the action). 
You can take inspiration from other github actions in this repo, or use gadgets listed below.

## Gadgets

You can use these as **steps** in your github action to perform some useful actions:

**Checkout the PR's code in a `pull_request` directory**:

```yml
- name: Checkout the PR
  if: success()
  uses: actions/checkout@v2
  with:
    path: "pull_request"
    fetch-depth: 50 # this is to make sure we obtain the target base commit
    repository: ${{ github.event.client_payload.owner }}/${{ github.event.client_payload.repo }} # repo of the forked diem/diem
    ref: ${{ github.event.client_payload.pull_ref }} # commit that triggered the PR
```

**Only run on dependabot PR**:

```yml
- name: Only run on dependabot
  uses: actions/github-script@0.5.0
  env:
    PULL_ID: ${{ github.event.client_payload.pull_id }}
  with:
    github-token: ${{secrets.MIRAI_BOT}}
    script: |
      const fs = require('fs');
      var path = require('path');

      async function main() {
        // get info on PR
        const pullrequest = await github.pulls.get({
          owner: "diem",
          repo: "diem",
          pull_number: process.env.PULL_ID
        });

        // allowlist
        var allowlist = ["dependabot[bot]"];
        if (!allowlist.includes(pullrequest.data.user.login)) {
          throw "not a dependabot PR";
        }
      }

      main();
```

**Write a comment on a PR, deleting the previous comment that contains the same hashtag**:

```yml
- name: Write comment on PR
  if: success()
  uses: actions/github-script@0.5.0
  env:
    PULL_ID: ${{ github.event.client_payload.pull_id }}
    OWNER: ${{ github.event.client_payload.owner }}
  with:
    github-token: ${{secrets.MIRAI_BOT}}
    script: |
      const fs = require('fs');
      var path = require('path');

      async function main() {
        // create the message
        var message = `your message containing the tag #YOUR_TAG`;

        // delete previous comments from MIRAI-BOT that includes the tag "#YOUR_TAG"
        const comments = await github.issues.listComments({
          owner: "diem",
          repo: "diem",
          issue_number: process.env.PULL_ID
        });
        for (let comment of comments.data) {
          if (comment.user.login === "MIRAI-bot" && comment.body.includes("#YOUR_TAG")) {
            await github.issues.deleteComment({
              owner: "diem",
              repo: "diem",
              comment_id: comment.id
            });
          }
        }

        // comment
        await github.issues.createComment({
          owner: "diem",
          repo: "diem",
          issue_number: process.env.PULL_ID,
          body: message
        });
        console.log(message);
      }

      main();
```

**Whitelist of users that will get a comment on their PR**:

```yml
- name: Whitelist of users
  uses: actions/github-script@0.5.0
  env:
    PULL_ID: ${{ github.event.client_payload.pull_id }}
  with:
    github-token: ${{secrets.MIRAI_BOT}}
    script: |
      const fs = require('fs');
      var path = require('path');

      async function main() {
        // get info on PR
        const pullrequest = await github.pulls.get({
          owner: "libra",
          repo: "libra",
          pull_number: process.env.PULL_ID
        });

        // allowlist
        var allowlist = ["mimoo", "anomalroil", "xvschneider"];
        if (!allowlist.includes(pullrequest.data.user.login)) {
          console.log(pullrequest.data.user.login + " is not in the allowlist of this github action");
          throw "user not in whitelist!";
        }
      }

      main();
```
