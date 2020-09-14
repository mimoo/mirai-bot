# MIRAI BOT

MIRAI bot is essentially a repository that gets triggered when a new PR gets submitted to the [libra repository](https://www.github.com/libra/libra).

This allows us to perform github actions on libra without having admin access to the repo.

![mirai bot](mirai-bot.png)

## MIRAI

Initially the MIRAI bot was in support of running the [MIRAI static analyzer](https://github.com/facebookexperimental/MIRAI) (hence the name).

## Clippy

This runs [all the clippy rules](https://rust-lang.github.io/rust-clippy/master/) on a PR (last updated to reflect all rules of clippy v0.0.212, it is missing panic_in_fn_result).

## Move-prover

TKTK

## Move-coverage

TKTK
