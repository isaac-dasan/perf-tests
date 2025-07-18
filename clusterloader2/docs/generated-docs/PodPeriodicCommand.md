# PodPeriodicCommand Measurement

`PodPeriodicCommand` is a measurement in ClusterLoader2 that periodically executes commands inside selected pods during a test run. It is typically used for polling information (e.g., profiling, resource usage, custom metrics) from containers in real time.

---

## How It Works

- On each interval (tick), the measurement selects up to `limit` pods matching the provided label selector.
- For each pod, it executes a list of configured commands inside the specified container.
- Captures stdout and stderr, exit codes, execution errors, and timeouts for each command run.
- Results are summarized and exported when the measurement is gathered.

---

## Configuration

You configure PodPeriodicCommand in your test YAML under the `measurements` section. As with other measurements, you must specify both "start" and "gather" actions.

### Example YAML

```yaml
measurements:
  - Identifier: MyPeriodicCmd
    Method: PodPeriodicCommand
    Params:
      action: start
      labelSelector: "app=my-target-pods"
      container: "main-container"
      interval: 30s
      # limit: 5
      failOnCommandError: true
      failOnExecError: true
      failOnTimeout: true
      commands:
        - name: "GetCPU"
          command: ["cat", "/proc/stat"]
          timeout: 10s
        - name: "GetMem"
          command: ["cat", "/proc/meminfo"]
          timeout: 10s

  # Sleep between start and gather to allow periodic commands to run multiple times
  - Identifier: SleepBetweenCmds
    Method: Sleep
    Params:
      action: sleep
      duration: 2m

  - Identifier: MyPeriodicCmd
    Method: PodPeriodicCommand
    Params:
      action: gather
```

---

## Supported Parameters

| Parameter             | Type           | Description                                                                 |
|-----------------------|----------------|-----------------------------------------------------------------------------|
| `labelSelector`       | string         | Selects pods to run the command in (Kubernetes label selector syntax).      |
| `container`           | string         | Name of the container in which to execute the commands.                     |
| `interval`            | duration (e.g. 30s) | Time between consecutive command executions.                           |
| `limit`               | integer        | **Optional.** Max number of pods that will have the commands executed in on every interval.<br>**If not specified or set to 0, commands will run in all matching pods.**|
| `failOnCommandError`  | bool           | If true, fail measurement on non-zero exit code from any command.           |
| `failOnExecError`     | bool           | If true, fail measurement on any execution error (e.g. API call error).     |
| `failOnTimeout`       | bool           | If true, fail measurement if any command times out.                         |
| `commands`            | array          | List of commands to execute. Each must specify:                             |
|                       |                | - `name`: unique identifier for the command                                 |
|                       |                | - `command`: array of strings (the command and args)                        |
|                       |                | - `timeout`: duration (how long to wait for command to complete)            |

### Command Example

```yaml
commands:
  - name: "GetCPU"
    command: ["cat", "/proc/stat"]
    timeout: 10s
```

---

## About `limit`

- **If `limit` is omitted or set to 0**: The measurement will run the commands in **all pods** matching the label selector and container on every interval.
- **If `limit` is set to N > 0**: Only up to N matching pods will have the commands executed per interval.

---

## About Sleep Between Start and Gather

- **You must ensure there is enough time (sleep or test actions) between the `start` and `gather` phases** for the measurement to collect meaningful data.
- If you call `gather` immediately after `start`, only one round of command execution may occur, resulting in little or no data.
- Use the `Sleep` measurement (see example above) or other test phases to allow multiple intervals to pass, and thus more data to be collected.
- The recommended sleep duration is at least several multiples of your configured `interval` (e.g., if `interval: 30s`, sleep at least 2 minutes for 4 rounds).

---

## Actions

- **start**: Begins periodic command execution.
- **gather**: Collects and summarizes the results. Exports stdout/stderr for each command run.

---

## Output

- For each tick and pod, results (stdout, stderr, exit code, errors, timeouts) are available as measurement summaries.
- Summaries are named by measurement, timestamp, pod namespace/name, command name, and output type.
- If any error conditions occur (depending on failOn* flags), the measurement may fail and stop.

---

## Typical Usage Pattern

1. **Start measurement** at the desired phase of your test, specifying which pods, container, commands, and polling interval.
2. **Sleep or perform other test actions** to allow periodic command runs to accumulate data.
3. **Gather results** at the end of the phase or test to export all collected command outputs for analysis.

---

## Advanced Notes

- Only pods matching `labelSelector` and in Running state are selected.
- If no pods match in a given tick, the tick is skipped.
- All commands run inside the specified container.
- The measurement uses Kubernetes exec API under the hood.
- If you only want to collect from a subset of pods, adjust `limit` accordingly.

---

## References

- [Source code](https://github.com/kubernetes/perf-tests/blob/master/clusterloader2/pkg/measurement/common/pod_command.go)
- See other measurement documentation for general usage patterns.
