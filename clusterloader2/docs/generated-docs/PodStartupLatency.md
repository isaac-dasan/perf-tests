# PodStartupLatency Measurement

The `PodStartupLatency` measurement tracks and validates the time it takes for pods to start in a Kubernetes cluster, often used to ensure pod startup SLOs are met.

## Usage

You must specify both a "start" and a "gather" action for this measurement. The "start" action begins tracking, and "gather" collects and summarizes results.

### Example YAML

```yaml
measurements:
  - Identifier: PSL
    Method: PodStartupLatency
    Params:
      action: start
      labelSelector: "app=test-pods"
      threshold: 20s

  - Identifier: PSL
    Method: PodStartupLatency
    Params:
      action: gather
```

## Parameters

| Parameter     | Type    | Description                                            |
|---------------|---------|--------------------------------------------------------|
| action        | string  | `"start"` or `"gather"`                                |
| labelSelector | string  | Selects pods to track (Kubernetes label selector)      |
| threshold     | duration| SLO threshold for startup time (e.g., `"20s"`)        |

## Notes

- The measurement will fail if the startup latency for any pod exceeds the threshold.
- Use labelSelector to target specific pods for latency tracking.