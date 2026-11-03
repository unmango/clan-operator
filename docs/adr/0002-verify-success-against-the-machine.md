# A run succeeds only when the machine is running the evaluated configuration

`clan machines update` exiting 0 does not prove the machine switched: a cancel after the build or copy step returns normally, and activation runs under `systemd-run` on the target and outlives the client.
So a run is Succeeded only when clan exits 0 and the target's `/run/current-system` matches the toplevel the flake evaluates to for that machine.
Exit 0 without a match is Unverified, a cancel before activation is Cancelled, and a nonzero exit is Failed.
Cancel is best effort, and the API says so: a cancelled run may still leave the machine updated.
