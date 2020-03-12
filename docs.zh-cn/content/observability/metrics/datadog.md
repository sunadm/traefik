# Datadog

To enable the Datadog:

```toml tab="文件 (TOML)"
[metrics]
  [metrics.datadog]
```

```yaml tab="文件 (YAML)"
metrics:
  datadog: {}
```

```bash tab="CLI"
--metrics.datadog=true
```

#### `address`

_Required, Default="127.0.0.1:8125"_

Address instructs exporter to send metrics to datadog-agent at this address.

```toml tab="文件 (TOML)"
[metrics]
  [metrics.datadog]
    address = "127.0.0.1:8125"
```

```yaml tab="文件 (YAML)"
metrics:
  datadog:
    address: 127.0.0.1:8125
```

```bash tab="CLI"
--metrics.datadog.address=127.0.0.1:8125
```

#### `addEntryPointsLabels`

_Optional, Default=true_

Enable metrics on entry points.

```toml tab="文件 (TOML)"
[metrics]
  [metrics.datadog]
    addEntryPointsLabels = true
```

```yaml tab="文件 (YAML)"
metrics:
  datadog:
    addEntryPointsLabels: true
```

```bash tab="CLI"
--metrics.datadog.addEntryPointsLabels=true
```

#### `addServicesLabels`

_Optional, Default=true_

Enable metrics on services.

```toml tab="文件 (TOML)"
[metrics]
  [metrics.datadog]
    addServicesLabels = true
```

```yaml tab="文件 (YAML)"
metrics:
  datadog:
    addServicesLabels: true
```

```bash tab="CLI"
--metrics.datadog.addServicesLabels=true
```

#### `pushInterval`

_Optional, Default=10s_

The interval used by the exporter to push metrics to datadog-agent.

```toml tab="文件 (TOML)"
[metrics]
  [metrics.datadog]
    pushInterval = 10s
```

```yaml tab="文件 (YAML)"
metrics:
  datadog:
    pushInterval: 10s
```

```bash tab="CLI"
--metrics.datadog.pushInterval=10s
```

