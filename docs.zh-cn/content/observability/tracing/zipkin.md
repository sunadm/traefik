# Zipkin

To enable the Zipkin:

```toml tab="文件 (TOML)"
[tracing]
  [tracing.zipkin]
```

```yaml tab="文件 (YAML)"
tracing:
  zipkin: {}
```

```bash tab="CLI"
--tracing.zipkin=true
```

#### `httpEndpoint`

_Required, Default="http://localhost:9411/api/v2/spans"_

Zipkin HTTP endpoint used to send data.

```toml tab="文件 (TOML)"
[tracing]
  [tracing.zipkin]
    httpEndpoint = "http://localhost:9411/api/v2/spans"
```

```yaml tab="文件 (YAML)"
tracing:
  zipkin:
    httpEndpoint: http://localhost:9411/api/v2/spans
```

```bash tab="CLI"
--tracing.zipkin.httpEndpoint=http://localhost:9411/api/v2/spans
```

#### `sameSpan`

_Optional, Default=false_

Use Zipkin SameSpan RPC style traces.

```toml tab="文件 (TOML)"
[tracing]
  [tracing.zipkin]
    sameSpan = true
```

```yaml tab="文件 (YAML)"
tracing:
  zipkin:
    sameSpan: true
```

```bash tab="CLI"
--tracing.zipkin.sameSpan=true
```

#### `id128Bit`

_Optional, Default=true_

Use Zipkin 128 bit trace IDs.

```toml tab="文件 (TOML)"
[tracing]
  [tracing.zipkin]
    id128Bit = false
```

```yaml tab="文件 (YAML)"
tracing:
  zipkin:
    id128Bit: false
```

```bash tab="CLI"
--tracing.zipkin.id128Bit=false
```

#### `sampleRate`

_Required, Default=1.0_

The rate between 0.0 and 1.0 of requests to trace.

```toml tab="文件 (TOML)"
[tracing]
  [tracing.zipkin]
    sampleRate = 0.2
```

```yaml tab="文件 (YAML)"
tracing:
  zipkin:
    sampleRate: 0.2
```

```bash tab="CLI"
--tracing.zipkin.sampleRate=0.2
```
