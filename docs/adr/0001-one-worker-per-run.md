# One worker per run, and only the worker holds credentials

clan exposes no API outside Python, reports only an exit code and log text, and can be cancelled only by killing it, so clan-operator runs the clan CLI rather than calling a library.
Each run gets its own short-lived worker process (a Kubernetes Job in a cluster, a transient systemd unit on a machine) that does one run and exits.
The two front ends, the Kubernetes controller and the on-machine service, start workers and record outcomes, but never mount root SSH keys or the sops admin key.
The worker reports its result as a small JSON document (the pod's termination message, or a file in the unit's state directory) and has no write access to the API; the front end writes status.

## Considered Options

- **Run clan inside the front-end process.** Simpler, but the long-running process would hold every clan's credentials, and cancelling a run would mean killing the process that tracks all the others.
- **Call `clan_lib` from Python.** It is unversioned, has no stability promise, and does less than the CLI (no inventory migrations, no vars generation, no route selection), so a caller has to redo those steps.

## Consequences

- A run that loses its worker (a node drain, the service restarting during its own host's activation) has no result; the front end decides its outcome by verification, as in [ADR 0002](0002-verify-success-against-the-machine.md).
- One binary carries all three roles as subcommands: `controller`, `serve`, and `run`. `run` is documented for reproducing a run by hand.
