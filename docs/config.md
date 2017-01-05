# The `kcm` configuration directory

KCM checkpoints state in a directory structure on disk. The checkpoint
describes all configured "pools" and their options, the "CPU lists" for
those pools, and the "tasks" currently assigned to each CPU list. The
directory format is described below.

## KCM Pool filesystem configuration format

_Example:_

```
etc
└── kcm
    ├── lock
    └── pools
        ├── controlplane
        │   ├── 3,11
        │   │   └── tasks
        │   └── exclusive
        ├── dataplane
        │   ├── 4,12
        │   │   └── tasks
        │   ├── 5,13
        │   │   └── tasks
        │   ├── 6,14
        │   │   └── tasks
        │   ├── 7,15
        │   │   └── tasks
        │   └── exclusive
        └── infra
            ├── 0-2,8-10
            │   └── tasks
            └── exclusive
```

_Where:_

| Path                                    | Meaning |
| :-------------------------------------- | :------ |
| `/etc/kcm/lock`                         | A lock file to protect against conflicting concurrent modification. |
| `/etc/kcm/pools/<pool>`                 | Directories, one per pool. |
| `/etc/kcm/pools/<pool>/exclusive`       | Determines whether each CPU list in the pool can be shared (value 0) or not (value 1). Shared CPU lists can accomodate multiple tasks simultaneously, while exclusive CPU lists may only be allocated to one task at a time. |
| `/etc/kcm/pools/<pool>/<cpulist>`       | Directories whose names conform to the Linux cpuset [CPU list format](cpu-list). |
| `/etc/kcm/pools/<pool>/<cpulist>/tasks` | A file containing a comma-separated list of the root Linux process IDs of containers to which the CPUset has been allocated. |

## Creating a new configuration

KCM can set up its own initial state. See [`kcm init`][kcm-init] doc for more
information.

## Configuration changes over time

`kcm` updates the configuration as follows:

- The operator creates the initial configuration on each host, either manually
  or by using the [`kcm init`][kcm-init] helper program.
- When tasks are launched via [`kcm isolate`][kcm-isolate], an available
  CPU list for the requested pool is chosen. That CPU list's `tasks`
  file is updated to include the [`kcm isolate`][kcm-isolate] process ID.
- After a task launched via [`kcm isolate`][kcm-isolate] dies, the
  associated CPU list's `tasks` file is updated to remove the
  [`kcm isolate`][kcm-isolate] process ID.
- [`kcm reconcile`][kcm-reconcile] asks the operating system about all
  process IDs in all pools. Process IDs that are no longer valid are removed
  from the `tasks` file. [`kcm reconcile`][kcm-reconcile] should be configured to execute
  periodically on each host).

[cpu-list]: http://man7.org/linux/man-pages/man7/cpuset.7.html#FORMATS
[kcm-init]: cli.md#kcm-init
[kcm-isolate]: cli.md#kcm-isolate
[kcm-reconcile]: cli.md#kcm-reconcile