# Haystack

To enable the Haystack:

```toml tab="文件 (TOML)"
[tracing]
  [tracing.haystack]
```

```yaml tab="文件 (YAML)"
tracing:
  haystack: {}
```

```bash tab="CLI"
--tracing.haystack=true
```

#### `localAgentHost`

_Require, Default="127.0.0.1"_

Local Agent Host instructs reporter to send spans to haystack-agent at this address.

```toml tab="文件 (TOML)"
[tracing]
  [tracing.haystack]
    localAgentHost = "127.0.0.1"
```

```yaml tab="文件 (YAML)"
tracing:
  haystack:
    localAgentHost: 127.0.0.1
```

```bash tab="CLI"
--tracing.haystack.localAgentHost=127.0.0.1
```

#### `localAgentPort`

_Require, Default=35000_

Local Agent port instructs reporter to send spans to the haystack-agent at this port.

```toml tab="文件 (TOML)"
[tracing]
  [tracing.haystack]
    localAgentPort = 35000
```

```yaml tab="文件 (YAML)"
tracing:
  haystack:
    localAgentPort: 35000
```

```bash tab="CLI"
--tracing.haystack.localAgentPort=35000
```

#### `globalTag`

_Optional, Default=empty_

Apply shared tag in a form of Key:Value to all the traces.

```toml tab="文件 (TOML)"
[tracing]
  [tracing.haystack]
    globalTag = "sample:test"
```

```yaml tab="文件 (YAML)"
tracing:
  haystack:
    globalTag: sample:test
```

```bash tab="CLI"
--tracing.haystack.globalTag=sample:test
```

#### `traceIDHeaderName`

_Optional, Default=empty_

Specifies the header name that will be used to store the trace ID.

```toml tab="文件 (TOML)"
[tracing]
  [tracing.haystack]
    traceIDHeaderName = "Trace-ID"
```

```yaml tab="文件 (YAML)"
tracing:
  haystack:
    traceIDHeaderName: Trace-ID
```

```bash tab="CLI"
--tracing.haystack.traceIDHeaderName=Trace-ID
```

#### `parentIDHeaderName`

_Optional, Default=empty_

Specifies the header name that will be used to store the parent ID.

```toml tab="文件 (TOML)"
[tracing]
  [tracing.haystack]
    parentIDHeaderName = "Parent-Message-ID"
```

```yaml tab="文件 (YAML)"
tracing:
  haystack:
    parentIDHeaderName: Parent-Message-ID
```

```bash tab="CLI"
--tracing.haystack.parentIDHeaderName=Parent-Message-ID
```

#### `spanIDHeaderName`

_Optional, Default=empty_

Specifies the header name that will be used to store the span ID.

```toml tab="文件 (TOML)"
[tracing]
  [tracing.haystack]
    spanIDHeaderName = "Message-ID"
```

```yaml tab="文件 (YAML)"
tracing:
  haystack:
    spanIDHeaderName: Message-ID
```

```bash tab="CLI"
--tracing.haystack.spanIDHeaderName=Message-ID
```

#### `baggagePrefixHeaderName`

_Optional, Default=empty_

Specifies the header name prefix that will be used to store baggage items in a map.

```toml tab="文件 (TOML)"
[tracing]
  [tracing.haystack]
    baggagePrefixHeaderName = "sample"
```

```yaml tab="文件 (YAML)"
tracing:
  haystack:
    baggagePrefixHeaderName: "sample"
```


```bash tab="CLI"
--tracing.haystack.baggagePrefixHeaderName=sample
```
