# hedge-design/lumibot: the `hedge` branch

This branch is the Lumibot build that btips (Hedge Design) installs. It is an upstream
Lumiwealth release plus a short stack of patches, rebased onto each new release. btips
pins a commit SHA from it in `requirements.txt`, and every pinned build has a tag
`hedge-<upstream version>.<n>` so the commit survives later rebases.

Upstream pull requests come from a different fork, **jbrass/lumibot**: one branch per
fix, each based on upstream `dev`. Never open PRs from this branch, and never commit to
`jbrass/dev`, which only fast-forwards from upstream.

## Patches on top of upstream

btips-only (not for upstream):

| Commit | Change |
|---|---|
| Change log level from info to debug for account summary and backup variable messages | quieter logs |
| fix: change warning to debug log for missing LUMIWEALTH_API_KEY | quieter logs |

Fixes also offered upstream (drop each one once upstream has merged it):

| Commit | jbrass branch | Upstream PR |
|---|---|---|
| fix(alpaca): parse nested bracket legs as simple orders | `fix/alpaca-bracket-legs` | pending |
| feat(alpaca): opt-in extended_hours for stock minute history in backtests | `feat/alpaca-backtest-extended-hours` | pending |
| feat(alpaca): configurable request budget for paid data plans | `feat/alpaca-request-budget` | pending |
| fix(alpaca): don't call backtesting-only process_pending_orders in _await_market_to_close | `fix/alpaca-await-market-close` | pending |

## Routine

Remotes in the local checkout: `upstream` = Lumiwealth/lumibot, `origin` = jbrass/lumibot,
`hedge` = hedge-design/lumibot.

**A new fix.** Branch off `upstream/dev`, commit, push to `origin`, open a PR to
`Lumiwealth/lumibot:dev`. Cherry-pick the commit onto `hedge`, add it to the table above,
push, tag `hedge-<upstream version>.<n+1>`, push the tag, and repin btips to the tag's
SHA. If it's urgent, commit to `hedge` first and backport to a jbrass branch afterwards.

**A btips-only change.** Commit straight to `hedge` (with a row in the btips-only table),
then tag and repin as above.

**A new upstream release.**

```bash
git fetch upstream
git rebase --onto <release merge commit on upstream/dev> <current base> hedge
```

Patches upstream has merged drop out on their own; drop any that upstream merged in an
edited or squashed form by hand. Update the tables, run the Alpaca tests below,
`git push --force-with-lease hedge hedge`, tag `hedge-<version>.1`, push the tag and
repin btips.

**Tests to run before tagging.**

```bash
pytest tests/test_alpaca_bracket_legs.py tests/test_alpaca_await_market_close.py \
       tests/test_alpaca_backtesting_multitimeframe_unit.py
```

## History

The patches used to live on jbrass/lumibot, stacked on its `dev`. That history is kept
under the tags `archive/dev-2026-11-12`, `archive/btips-f70bce9d` and
`archive/btips-0966386b` on jbrass/lumibot.
