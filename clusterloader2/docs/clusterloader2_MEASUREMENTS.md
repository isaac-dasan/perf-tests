# ClusterLoader2 Measurements Documentation

ClusterLoader2 provides a flexible framework for gathering various metrics and validating SLOs in Kubernetes clusters through *measurements*. This document thoroughly explains the measurement system, available measurement types, configuration, and usage.

---

## Measurement Overview

Measurements are core components in ClusterLoader2 used to collect, validate, and summarize metrics during performance tests. Each measurement implements the following Go interface:

```go
type Measurement interface {
    Execute(config *MeasurementConfig) error
    Dispose()
    String() string
}
```
- **Execute**: Runs the measurement logic. Typically, measurements are started and later gathered (results collected) by calling this method with different parameters.
- **Dispose**: Cleans up resources after measurement is done.
- **String**: Returns the measurement's name.

---

## Configuration

Measurements are defined in the test config YAML as follows:

```yaml
- Identifier: PodStartupLatency
  Method: PodStartupLatency
  Params:
    action: start
    labelSelector: group = test-pod
    threshold: 20s
- Identifier: PodStartupLatency
  Method: PodStartupLatency
  Params:
    action: gather
```

Each measurement instance is described by:

| Field        | Description                                                                                          |
|--------------|------------------------------------------------------------------------------------------------------|
| Method       | Name of the measurement type (registered in ClusterLoader2 factory).                                 |
| Identifier   | Uniquely identifies this instance (useful for multiple instances of the same measurement type).       |
| Params       | Map of parameters passed to the measurement. Common param: `action` (`start`/`gather`).              |

See also the struct from code:

```go
type Measurement struct {
    Method     string                 `json:"method"`
    Params     map[string]interface{} `json:"params"`
    Identifier string                 `json:"identifier"` // or Instances for wrapper measurements
}
```
---

## How Measurements Work

- **Start**: Begin collecting metrics, usually with `action: start`.
- **Gather**: Collect and summarize results, usually with `action: gather`.
- **Dispose**: Clean up.

Some measurements support additional parameters to fine-tune their behavior.

---

## Built-in Measurement Types

### Control Plane & API

- **APIAvailabilityMeasurement**  
  Measures control plane availability by polling `/readyz` endpoints.

- **APIResponsivenessPrometheusSimple**  
  Calculates API call latencies using Prometheus data. Validates API call latency SLOs.

- **APIResponsivenessPrometheus**  
  Provides a summary of API call latency and counts from Prometheus.

### Resource Usage & Profiles

- **ResourceUsageSummary**  
  Collects resource usage (CPU, memory) for cluster components. Can validate against constraints.

- **CPUProfile** / **MemoryProfile**  
  Gathers CPU/memory profiles via pprof for a component.

- **MetricsForE2E**  
  Collects metrics from core components, optionally all kubelets.

- **EtcdMetrics**  
  Gathers etcd metrics, including DB size.

### Scheduler & Pod Lifecycle

- **SchedulingMetrics**  
  Collects scheduler metrics.

- **SchedulingThroughput**  
  Measures scheduling throughput.

- **PodStartupLatency**  
  Measures and validates pod startup latency SLOs.

- **WaitForControlledPodsRunning**  
  Waits until all pods for given controllers (RC, RS, Deployment, DaemonSet, Job) are running.

### Other Measurements

- **Timer**  
  Time actions within the test, supports multiple independent timers.

- **PodPeriodicCommand**  
  Periodically runs a command in selected pods, collects output over time.

- **NetworkPerformanceMeasurement**  
  (Implemented under `common/network/`) Measures network performance. Supports start/gather actions and various parameters.

- **ClusterOOMsTracker**  
  Tracks OOM (out-of-memory) events in the cluster.

- **TestMetricsMeasurement, MaxRunningPods, ...**  
  Many additional measurements exist under `pkg/measurement/common/bundle` and other subdirectories.

---

## Adding Custom Measurements

To add a new measurement:
1. Implement the `Measurement` interface.
2. Register the measurement in an `init()` function using:
    ```go
    measurement.Register("MyMeasurement", createMyMeasurement)
    ```
3. Configure it in your test YAML.

See [`docs/DEVELOPING_MEASUREMENT.md`](docs/DEVELOPING_MEASUREMENT.md) for a step-by-step guide and example.

---

## Example Measurement Configuration

```yaml
measurements:
  - Identifier: APIResponsivenessPrometheus
    Method: APIResponsivenessPrometheus
    Params:
      action: start
      prometheusAddress: http://prometheus-server
  - Identifier: APIResponsivenessPrometheus
    Method: APIResponsivenessPrometheus
    Params:
      action: gather
```

---

## Where to Find Measurement Implementations

- All measurement code is under `clusterloader2/pkg/measurement/common/` and its subdirectories.
- For Prometheus-based measurements, see `common/prometheus_measurement.go`.
- For network, see `common/network/`.
- See the [Measurement interface](https://github.com/kubernetes/perf-tests/blob/master/clusterloader2/pkg/measurement/interface.go) for latest definition.

---

## Advanced Usage & Testing

- You can unit test Prometheus-based measurements using the internal framework with sample time series, without running a live cluster.
  - Example: [container_restarts_test.go](pkg/measurement/common/container_restarts_test.go)
  - Input data: [testdata/container_restarts](pkg/measurement/common/testdata/container_restarts)

## References

- [Getting Started](docs/GETTING_STARTED.md)
- [Design Docs](docs/design.md)
- [Developing Measurements](docs/DEVELOPING_MEASUREMENT.md)
- [Measurement Source Code](pkg/measurement/common/)

---

## Summary Table: Key Measurements

| Measurement Name              | Purpose/What it Measures                   | Key Params                |
|-------------------------------|--------------------------------------------|---------------------------|
| APIAvailabilityMeasurement    | API `/readyz` checks (cluster/host)        | action, pollInterval      |
| APIResponsivenessPrometheus   | API call latencies (Prometheus)            | action, prometheusAddress |
| ResourceUsageSummary          | CPU/memory usage, constraints              | constraintsFile           |
| PodStartupLatency             | Pod startup latency SLO                    | labelSelector, threshold  |
| SchedulingMetrics             | Scheduler metrics                          | action                    |
| Timer                         | Arbitrary latency measurements             | timerName                 |
| WaitForControlledPodsRunning  | All pods running for controllers           | labelSelector, namespace  |
| NetworkPerformanceMeasurement | Network throughput/latency                 | action, config params     |

---

## Tips

- Always use `action: start` to begin measurement and `action: gather` to collect results.
- Use `Identifier` to differentiate multiple instances of the same measurement.
- Refer to the source for each measurement for specific supported parameters.
