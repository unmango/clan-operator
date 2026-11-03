# clan-operator

Runs [clan](https://clan.lol) operations on a clan's behalf, from a Kubernetes cluster or from a clan machine, and shows what ran.

## Language

### Doing the work

**Operation**:
A named kind of clan action that clan-operator knows how to perform, such as an update.
_Avoid_: command, job, task

**Run**:
One attempt to perform an **Operation** against one machine. A run is never retried in place; a second attempt is a new run.
_Avoid_: job, execution, deployment

**Worker**:
The short-lived process that carries out exactly one **Run** and then exits. It is the only part of clan-operator that holds a clan's credentials.
_Avoid_: runner, executor, agent

**Front end**:
A long-running part of clan-operator that accepts **Runs**, starts a **Worker** for each, and records the outcome. There are two: the **Controller** and the **Service**.

**Controller**:
The **Front end** that runs in a Kubernetes cluster.
_Avoid_: operator (that is the whole product)

**Service**:
The **Front end** that runs on a clan machine without Kubernetes.
_Avoid_: daemon, agent

### Outcomes

**Succeeded**:
A **Run** outcome: clan finished cleanly and the machine is running the configuration the flake evaluates to.

**Unverified**:
A **Run** outcome: clan reported success, but the machine is not running the configuration the flake evaluates to, or it could not be checked.

**Cancelled**:
A **Run** outcome: the **Worker** was stopped before the machine began switching configuration.

**Failed**:
A **Run** outcome: clan reported failure.
