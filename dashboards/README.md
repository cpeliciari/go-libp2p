# Grafana Dashboards

This directory contains prebuilt dashboards (provided as JSON files) for various components.
For steps on how to import and use them [please read the official Grafana documentation.](https://grafana.com/docs/grafana/latest/dashboards/export-import/#import-dashboard)

## Public dashboards

You can check the following prebuilt dashboards in action:

1. [AutoNAT](https://protocollabs.grafana.net/public-dashboards/fce8fdeb629742c89bd70f0ce38dfd97)
2. [Auto Relay](https://protocollabs.grafana.net/public-dashboards/380d52aded12404e9cf6ceccb824b7f9)
3. [Eventbus](https://protocollabs.grafana.net/public-dashboards/048029ac2d7e4a71b281ffea3535026e)
4. [Identify](https://protocollabs.grafana.net/public-dashboards/96b70328253d47c0b352dfae06f12a1b)
5. [Relay Service](https://protocollabs.grafana.net/public-dashboards/4a8cb5d245294893874ed65279b049be)
6. [Swarm](https://protocollabs.grafana.net/public-dashboards/2bd3f1bee9964d40b6786fbe3eafd9fc)

These metrics come from one of the public IPFS DHT [bootstrap nodes](https://docs.ipfs.tech/concepts/nodes/#bootstrap) run by Protocol Labs.
At the time of writing (2023-08), these nodes handle many connections across various libp2p implementations, versions, and configurations (they don't handle large file transfers).

## Using locally

For local development and debugging, it can be useful to spin up a local Prometheus and Grafana instance.

To expose metrics, we first need to expose a metrics collection endpoint. Add this to your code:

```go
import (
    "github.com/prometheus/client_golang/prometheus/promhttp"
    "github.com/grafana/pyroscope-go"
)

go func() {
http.Handle("/debug/metrics/prometheus", promhttp.Handler())
log.Fatal(http.ListenAndServe(":5001", nil))
}()

pyroscope.Start(pyroscope.Config{
    ApplicationName: "simple.golang.app",
    
    // replace this with the address of pyroscope server
    ServerAddress:   "http://127.0.0.1:4040",
    
    // you can disable logging by setting this to nil
    Logger:          pyroscope.StandardLogger,
    
    // you can provide static tags via a map:
    Tags:            map[string]string{"hostname": os.Getenv("HOSTNAME")},
    
    ProfileTypes: []pyroscope.ProfileType{
        // these profile types are enabled by default:
        pyroscope.ProfileCPU,
        pyroscope.ProfileAllocObjects,
        pyroscope.ProfileAllocSpace,
        pyroscope.ProfileInuseObjects,
        pyroscope.ProfileInuseSpace,
        
        // these profile types are optional:
        pyroscope.ProfileGoroutines,
        pyroscope.ProfileMutexCount,
        pyroscope.ProfileMutexDuration,
        pyroscope.ProfileBlockCount,
        pyroscope.ProfileBlockDuration,
    },
})
```

This exposes a metrics collection endpoint at http://localhost:5001/debug/metrics/prometheus. Note that this is the same endpoint that [Kubo](https://github.com/ipfs/kubo) uses, so if you want to gather metrics from Kubo, you can skip this step.

On macOS:
```bash
docker compose -f docker-compose.base.yml up
```
On Linux, dashboards can be inspected locally by running:
```bash
docker compose -f docker-compose.base.yml -f docker-compose-linux.yml up
```

and opening Grafana at http://localhost:3000.


### Details on Pyroscope (Developer Profiling)

Pyroscope is a continuous profiling platform that allows you to monitor the performance of your applications in real-time. In the provided code, it is used alongside Prometheus for profiling and metrics collection.

#### Features:

- Application Performance Monitoring: Tracks various profile types, including CPU usage, memory allocation.
- Real-time Insights: Enables detailed analysis of bottlenecks and issues in your code.

### Making Dashboards usable with Provisioning

The following section is only relevant for creators of dashboards.

Due to a bug in Grafana, it's not possible to provision dashboards shared for external use directly. We need to apply the workaround described in https://github.com/grafana/grafana/issues/10786#issuecomment-568788499 (adding a few lines in the dashboard JSON file).
