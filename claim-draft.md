Hi @nadaverell, I'd like to work on this one. Could you assign it to me?

I'll start by reproducing it on a local kind cluster: pause a Deployment with `kubectl rollout pause`, then confirm Radar has no way to resume it and refuses a rollback, as #2363 describes. I'll post what I find here first.

If that matches, I'll share an implementation plan here before writing code.
