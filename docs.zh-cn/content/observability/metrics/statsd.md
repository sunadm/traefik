# StatsD

To enable the Statsd:

```toml tab="文件 (TOML)"
[metrics]
  [metrics.statsD]
```

```yaml tab="文件 (YAML)"
metrics:
  statsD: {}
```

```bash tab="CLI"
--metrics.statsd=true
```

#### `address`

_Required, Default="localhost:8125"_

Address instructs exporter to send metrics to statsd at this address.

```toml tab="文件 (TOML)"
[metrics]
  [metrics.statsD]
    address = "localhost:8125"
```

```yaml tab="文件 (YAML)"
metrics:
  statsD:
    address: localhost:8125
```

```bash tab="CLI"
--metrics.statsd.address=localhost:8125
```

#### `addEntryPointsLabels`

_Optional, Default=true_

Enable metrics on entry points.

```toml tab="文件 (TOML)"
[metrics]
  [metrics.statsD]
    addEntryPointsLabels = true
```

```yaml tab="文件 (YAML)"
metrics:
  statsD:
    addEntryPointsLabels: true
```

```bash tab="CLI"
--metrics.statsd.addEntryPointsLabels=true
```

#### `addServicesLabels`

_Optional, Default=true_

Enable metrics on services.

```toml tab="文件 (TOML)"
[metrics]
  [metrics.statsD]
    addServicesLabels = true
```

```yaml tab="文件 (YAML)"
metrics:
  statsD:
    addServicesLabels: true
```

```bash tab="CLI"
--metrics.statsd.addServicesLabels=true
```

#### `pushInterval`

_Optional, Default=10s_

The interval used by the exporter to push metrics to statsD.

```toml tab="文件 (TOML)"
[metrics]
  [metrics.statsD]
    pushInterval = 10s
```

```yaml tab="文件 (YAML)"
metrics:
  statsD:
    pushInterval: 10s
```

```bash tab="CLI"
--metrics.statsd.pushInterval=10s
```

#### `prefix`

_Optional, Default="traefik"_

The prefix to use for metrics collection.

```toml tab="文件 (TOML)"
[metrics]
  [metrics.statsD]
    prefix = "traefik"
```

```yaml tab="文件 (YAML)"
metrics:
  statsD:
    prefix: traefik
```

```bash tab="CLI"
--metrics.statsd.prefix="traefik"
```