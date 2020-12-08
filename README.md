# MIRAI BOT

**Warning: keep reading, this has almost nothing to do with the [MIRAI static analyzer](https://github.com/facebookexperimental/MIRAI)**.

MIRAI bot is essentially a repository that gets triggered when a new PR gets submitted to the [libra repository](https://www.github.com/libra/diem).

This allows us to perform github actions on libra without having admin access to the repo.

![mirai bot](mirai-bot.png)

## MIRAI

Initially the MIRAI bot was in support of running the [MIRAI static analyzer](https://github.com/facebookexperimental/MIRAI) (hence the name).
It only displays the full result to people added to the whitelist.

## Clippy

This runs [all the clippy rules](https://rust-lang.github.io/rust-clippy/master/) on a PR.

## Wannabe-Land-Blocking

This runs a set of selected clippy rule on a PR. These rules are ones we want to have as land blocking tests in CI, but cannot at the moment for various reasons.

## Move-prover

TKTK

## Move-coverage

TKTK

## Dephell

Runs [cargo-dephell](https://github.com/mimoo/cargo-dephell) on a PR and uploads it to the action
