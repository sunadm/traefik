# Elastic

To enable the Elastic:

```toml tab="文件 (TOML)"
[tracing]
  [tracing.elastic]
```

```yaml tab="文件 (YAML)"
tracing:
  elastic: {}
```

```bash tab="CLI"
--tracing.elastic=true
```

#### `serverURL`

_Optional, Default="http://localhost:8200"_

APM ServerURL is the URL of the Elastic APM server.

```toml tab="文件 (TOML)"
[tracing]
  [tracing.elastic]
    serverURL = "http://apm:8200"
```

```yaml tab="文件 (YAML)"
tracing:
  elastic:
    serverURL: "http://apm:8200"
```

```bash tab="CLI"
--tracing.elastic.serverurl="http://apm:8200"
```

#### `secretToken`

_Optional, Default=""_

APM Secret Token is the token used to connect to Elastic APM Server.

```toml tab="文件 (TOML)"
[tracing]
  [tracing.elastic]
    secretToken = "mytoken"
```

```yaml tab="文件 (YAML)"
tracing:
  elastic:
    secretToken: "mytoken"
```

```bash tab="CLI"
--tracing.elastic.secrettoken="mytoken"
```

#### `serviceEnvironment`

_Optional, Default=""_

APM Service Environment is the name of the environment Traefik is deployed in, e.g. `production` or `staging`.

```toml tab="文件 (TOML)"
[tracing]
  [tracing.elastic]
    serviceEnvironment = "production"
```

```yaml tab="文件 (YAML)"
tracing:
  elastic:
    serviceEnvironment: "production"
```

```bash tab="CLI"
--tracing.elastic.serviceenvironment="production"
```

### Further

Additional configuration of Elastic APM Go agent can be done using environment variables.
See [APM Go agent reference](https://www.elastic.co/guide/en/apm/agent/go/current/configuration.html).
