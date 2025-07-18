# WaitForControlledPodsRunning Measurement

The `WaitForControlledPodsRunning` measurement acts as a barrier, ensuring that all pods managed by a specific controller (e.g., Deployment, ReplicaSet) are running before continuing.

## Usage

Configure this measurement in your test YAML with label or field selectors to specify the controlling objects.

### Example YAML

```yaml
measurements:
  - Identifier: WaitBarrier
    Method: WaitForControlledPodsRunning
    Params:
      action: wait
      labelSelector: "app=my-app"
      namespace: "default"
      timeout: "300s"
```

## Parameters

| Parameter      | Type    | Description                                                     |
|----------------|---------|-----------------------------------------------------------------|
| action         | string  | Usually `"wait"`                                                |
| labelSelector  | string  | Label selector for controllers (Deployment, RS, etc.)           |
| fieldSelector  | string  | Optional: Field selector for controllers                        |
| namespace      | string  | Namespace to search for controllers                             |
| timeout        | duration| Maximum time to wait for all pods to be running (e.g., `"5m"`)  |

## Notes

- The measurement waits until all pods for the specified controllers are running.
- If the timeout is exceeded, the measurement will fail.
- Use selectors to target specific controllers and their pods.