# Sleep Measurement

The `Sleep` measurement is used to introduce a pause in the execution of a ClusterLoader2 test scenario. This is useful for waiting between actions, steps, or phases.

## Usage

You configure the Sleep measurement in your test YAML using the `action: sleep` and a `duration` parameter.

### Example YAML

```yaml
measurements:
  - Identifier: SleepPhase
    Method: Sleep
    Params:
      action: sleep
      duration: 60s
```

## Parameters

| Parameter | Type     | Description                         |
|-----------|----------|-------------------------------------|
| action    | string   | Must be `"sleep"`                   |
| duration  | duration | How long to sleep (e.g., `"30s"`)   |

## Notes

- This measurement does not gather any data.
- Use it to introduce delays between phases or actions.