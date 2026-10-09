# Retiring a flag

Flags are meant to be temporary. When a rollout is finished, take the flag out
of the code first, then delete its definition. Walk the `qfg cleanup`
lifecycle instead of jumping to `qfg delete`.

```bash
qfg cleanup list                 # flags an owner marked readyForCleanup, with eval volume
qfg cleanup status my.flag       # telemetry + per-environment rules
qfg cleanup remove my.flag       # safety gate (no evals in 2 days), writes .qf/cleanup/my.flag.json
```

Then remove the call sites, using the payload at `.qf/cleanup/<key>.json`
(`type`, per-environment rules, `grepPatterns`):

1. **Find every call site.** Grep for the key together with the SDK methods in
   `grepPatterns` (`isEnabled`, `get`, `FeatureIsOn`, ...). Also look for
   wrappers the codebase built around the SDK (a `flagEnabled("...")` helper,
   a `<FlagGate flag="...">` component) and follow them to their callers.
2. **Ask the engineer once which value wins** — "keep the true branch and
   delete the false branch?", or for a variant/config "which of these values
   wins?" (list the values per environment from the payload). Do not infer it.
3. **Apply it mechanically.** The SDK call becomes the literal value, the
   unreachable branch is deleted, a helper that existed only for this flag is
   deleted. Do not refactor unrelated code.
4. **Run the repo's formatter and tests.**
5. **Open one PR** for this flag in this repo, naming the flag, the chosen
   direction, and the call sites changed. If the payload says `forced: true`,
   say so. Never auto-merge.

After the PR merges and the service redeploys:

```bash
qfg cleanup verify my.flag && qfg delete my.flag    # 7-day zero-evaluation gate, then delete
```
