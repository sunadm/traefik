# 路由器 { #routers }

连接请求到服务
{: .subtitle }

![routers](../../assets/img/routers.png)

路由器负责将传入请求与可以处理这些请求的服务连接起来。
在此过程中，路由器可能会使用[中间件](../../middlewares/overview.md)来更新请求，或者在将请求转发到服务之前做些事情。

## 配置示例 { #configuration-example }

??? example "请求 /foo 由 service-foo 处理 -- 使用[文件提供者](../../providers/file.md)"

    ```toml tab="TOML"
    ## 动态配置
    [http.routers]
      [http.routers.my-router]
        rule = "Path(`/foo`)"
        service = "service-foo"
    ```

    ```yaml tab="YAML"
    ## 动态配置
    http:
      routers:
        my-router:
          rule: "Path(`/foo`)"
          service: service-foo
    ```

??? example "转发端口3306上的所有非TLS请求到一个数据库服务"
    
    **动态配置**
    
    ```toml tab="文件 (TOML)"
    ## 动态配置
    [tcp]
      [tcp.routers]
        [tcp.routers.to-database]
          entryPoints = ["mysql"]
          # 捕获所有请求 (only available rule for non-tls routers. See below.)
          rule = "HostSNI(`*`)"
          service = "database"
    ```
    
    ```yaml tab="文件 (YAML)"
    ## 动态配置
    tcp:
      routers:
        to-database:
          entryPoints:
            - "mysql"
          # 捕获所有请求 (only available rule for non-tls routers. See below.)
          rule: "HostSNI(`*`)"
          service: database
    ```
    
    **静态配置**
    
    ```toml tab="文件 (TOML)"
    ## 静态配置
    [entryPoints]
      [entryPoints.web]
        address = ":80"
      [entryPoints.mysql]
        address = ":3306"   
    ```
     
    ```yaml tab="文件 (YAML)"
    ## 静态配置
    entryPoints:
      web:
        address: ":80"
      mysql:
        address: ":3306"   
    ```
    
    ```bash tab="CLI"
    ## 静态配置
    --entryPoints.web.address=:80
    --entryPoints.mysql.address=:3306
    ```

## 配置HTTP路由器 { #configuring-http-routers }

!!! warning "字符`@`不可用于路由器名称"

### EntryPoints

如果未明确指定，HTTP路由器将接受来自所有已定义的入口点的请求。
如果要将路由器范围限制为一组入口点，请设置`entryPoints`选项。

??? example "监听每个入口点"
    
    **动态配置**
    
    ```toml tab="文件 (TOML)"
    ## 动态配置
    [http.routers]
      [http.routers.Router-1]
        # 默认情况下，路由器监听每一个入口点
        rule = "Host(`traefik.io`)"
        service = "service-1"
    ```
    
    ```yaml tab="文件 (YAML)"
    ## 动态配置
    http:
      routers:
        Router-1:
          # 默认情况下，路由器监听每一个入口点
          rule: "Host(`traefik.io`)"
          service: "service-1"
    ```
    
    **静态配置**
    
    ```toml tab="文件 (TOML)"
    ## 静态配置
    [entryPoints]
      [entryPoints.web]
        address = ":80"
      [entryPoints.websecure]
        address = ":443"
      [entryPoints.other]
        address = ":9090"
    ```
    
    ```yaml tab="文件 (YAML)"
    ## 静态配置
    entryPoints:
      web:
        address: ":80"
      websecure:
        address: ":443"
      other:
        address: ":9090"
    ```
    
    ```bash tab="CLI"
    ## 静态配置
    --entrypoints.web.address=:80
    --entrypoints.websecure.address=:443
    --entrypoints.other.address=:9090
    ```

??? example "监听特定的入口点"
    
    **动态配置**
    
    ```toml tab="文件 (TOML)"
    ## 动态配置
    [http.routers]
      [http.routers.Router-1]
        # 不监听入口点web
        entryPoints = ["websecure", "other"]
        rule = "Host(`traefik.io`)"
        service = "service-1"
    ```
    
    ```yaml tab="文件 (YAML)"
    ## 动态配置
    http:
      routers:
        Router-1:
          # 不监听入口点web
          entryPoints:
            - "websecure"
            - "other"
          rule: "Host(`traefik.io`)"
          service: "service-1"
    ```

    **静态配置**
    
    ```toml tab="文件 (TOML)"
    ## 静态配置
    [entryPoints]
      [entryPoints.web]
        address = ":80"
      [entryPoints.websecure]
        address = ":443"
      [entryPoints.other]
        address = ":9090"
    ```
    
    ```yaml tab="文件 (YAML)"
    ## 静态配置
    entryPoints:
      web:
        address: ":80"
      websecure:
        address: ":443"
      other:
        address: ":9090"
    ```
    
    ```bash tab="CLI"
    ## 静态配置
    --entrypoints.web.address=:80
    --entrypoints.websecure.address=:443
    --entrypoints.other.address=:9090
    ```

### 规则 { #rule }

规则是一组配置有值的匹配器，这些值决定特定请求和特定规则是否匹配。
如果该规则得到验证，则路由器将变为活跃状态，并调用中间件，然后将请求转发到服务。

??? tip "反引号还是引号？"
    要设置规则的值，请使用[反引号](https://en.wiktionary.org/wiki/backtick)``` ` ```，或转义的双引号`\"`。

    不接受单引号`'`，因为其值是[Golang的字符串文字](https://golang.org/ref/spec#String_literals)。

!!! example "主机名为traefik.io"

    ```toml
    rule = "Host(`traefik.io`)"
    ```

!!! example "主机名为traefik.io或主机名为containo.us且路径为/traefik"

    ```toml
    rule = "Host(`traefik.io`) || (Host(`containo.us`) && Path(`/traefik`))"
    ```

下表列出了所有可用匹配器：

| 规则                                                                  | 描述                                                                   |
|----------------------------------------------------------------------|-----------------------------------------------------------------------|
| ```Headers(`key`, `value`)```                                        | 检查标头中是否有名为`key`的键，且其值为`value`。                              |
| ```HeadersRegexp(`key`, `regexp`)```                                 | 检查标头中是否有名为`key`的键，且其值匹配正则表达式`regexp`                     |
| ```Host(`domain-1`, ...)```                                          | 检查请求的主机名/域名目标，是否是给定的`domains`之一。                          |
| ```HostRegexp(`traefik.io`, `{subdomain:[a-z]+}.traefik.io`, ...)``` | 检查请求的主机名/域名，是否匹配给定的正则表达式`regexp`。                        |
| ```Method(`GET`, ...)```                                             | 检查请求方法是否为给定的`methods`（`GET`, `POST`, `PUT`, `DELETE`, `PATCH`） |
| ```Path(`/path`, `/articles/{category}/{id:[0-9]+}`, ...)```         | 精确匹配请求路径。接受文字及正则表达式路径的序列。                               |
| ```PathPrefix(`/products/`, `/articles/{category}/{id:[0-9]+}`)```   | 匹配请求路径前缀。接受文字及正则表达式路径前缀的序列。                            |
| ```Query(`foo=bar`, `bar=baz`)```                                    | 匹配查询字符串参数。接受键=值对的序列。                                       |

!!! important "正则表达式语法"

    为将正则表达式与`Host`和`Path`表达式一起使用，
    必须声明一个任意命名的变量，后面跟着用冒号分隔的正则表达式，所有这些都用花括号括起来。
    可以使用[Go的regexp软件包](https://golang.org/pkg/regexp/)支持的任何模式（如: `/posts/{id:[0-9]+}`）。

!!! info "使用运算符和括号组合匹配器"

    可以使用AND (`&&`) 和 OR (`||`)运算符来组合匹配器。也可以使用括号。

!!! important "规则，中间件，以及服务"

    规则将会被评估，评估发生在任何中间件有机会工作"之前"，以及将请求转发到服务"之前"。

!!! info "路径 Vs 路径前缀"

    使用`路径`，如果服务仅监听在确切的路径上。例如，`Path: /products`将匹配`/products`，但不匹配`/products/shoes`。

    使用`*路径前缀*`匹配器，如果服务在特定的基本路径上侦听，且也服务子路径上的请求。
    例如，`PathPrefix: /products`将匹配`/products`，且也将匹配`/products/shoes`和`/products/shirts`。
    Since the path is forwarded as-is, your service is expected to listen on `/products`.

### Priority

为避免路径重叠，路由器是排序过的，默认情况下，使用规则长度降序排序。
优先级直接等于规则的长度，最长的长度具有最高优先级。

优先级值`0`将被忽略：`priority = 0`表示使用默认的长度排序规则。

??? info "默认优先级是怎样计算出来的"

    ```toml tab="文件 (TOML)"
    ## 动态配置
    [http.routers]
      [http.routers.Router-1]
        rule = "HostRegexp(`.*\.traefik\.com`)"
        # ...
      [http.routers.Router-2]
        rule = "Host(`foobar.traefik.com`)"
        # ...
    ```
    
    ```yaml tab="文件 (YAML)"
    ## 动态配置
    http:
      routers:
        Router-1:
          rule: "HostRegexp(`.*\.traefik\.com`)"
          # ...
        Router-2:
          rule: "Host(`foobar.traefik.com`)"
          # ...
    ```
    
    此例中，所有主机`foobar.traefik.com`的请求，都将通过`Router-1`而不是`Router-2`进行路由。
    
    | Name     | Rule                                 | Priority |
    |----------|--------------------------------------|----------|
    | Router-1 | ```HostRegexp(`.*\.traefik\.com`)``` | 30       |
    | Router-2 | ```Host(`foobar.traefik.com`)```     | 26       |
    
    前表显示`Router-1`具有比`Router-2`高的优先级。
    
    要解决此问题，必须(显式)设置优先级。

??? example "设置优先级 -- 使用[文件提供者](../../providers/file.md)"
    
    ```toml tab="文件 (TOML)"
    ## 动态配置
    [http.routers]
      [http.routers.Router-1]
        rule = "HostRegexp(`.*\.traefik\.com`)"
        entryPoints = ["web"]
        service = "service-1"
        priority = 1
      [http.routers.Router-2]
        rule = "Host(`foobar.traefik.com`)"
        entryPoints = ["web"]
        priority = 2
        service = "service-2"
    ```
    
    ```yaml tab="文件 (YAML)"
    ## 动态配置
    http:
      routers:
        Router-1:
          rule: "HostRegexp(`.*\.traefik\.com`)"
          entryPoints:
          - "web"
          service: service-1
          priority: 1
        Router-2:
          rule: "Host(`foobar.traefik.com`)"
          entryPoints:
          - "web"
          priority: 2
          service: service-2
    ```

    此配置中，优先级配置为允许`Router-2`来处理带`foobar.traefik.com`主机名称的请求。

### 中间件 { #middlewares }

可以附加一个[中间件](../../middlewares/overview.md)列表到每个HTTP路由器。
仅当规则匹配，且在将请求转发到服务之前，中间件才会生效。

!!! warning "字符`@`不可用于中间件名称。"

!!! tip "中间件排序"
    
    中间件的应用顺序，与其在**路由器**中声明的排序一致。

??? example "带有一[中间件](../../middlewares/overview.md) -- 使用[文件提供者](../../providers/file.md)"

    ```toml tab="TOML"
    ## 动态配置
    [http.routers]
      [http.routers.my-router]
        rule = "Path(`/foo`)"
        # 在其他地方声明
        middlewares = ["authentication"]
        service = "service-foo"
    ```

    ```yaml tab="YAML"
    ## 动态配置
    http:
      routers:
        my-router:
          rule: "Path(`/foo`)"
          # 在其他地方声明
          middlewares:
          - authentication
          service: service-foo
    ```

### Service

Each request must eventually be handled by a [service](../services/index.md),
which is why each router definition should include a service target,
which is basically where the request will be passed along to.

In general, a service assigned to a router should have been defined,
but there are exceptions for label-based providers.
See the specific [docker](../providers/docker.md#service-definition), [rancher](../providers/rancher.md#service-definition),
or [marathon](../providers/marathon.md#service-definition) documentation.

!!! warning "字符`@`不可用于中间件名称。"

!!! important "HTTP路由器只能标向HTTP服务（不能标向TCP服务）。"

### TLS

#### General

当指定TLS部分时，它指示Traefik当前路由器仅专用于HTTPS请求（且该路由器应忽略HTTP（非TLS）请求）。
Traefik将终结SSL连接（意味着它将发送解密后的数据给服务）。

??? example "配置路由器以仅接受HTTPS请求"

    ```toml tab="文件 (TOML)"
    ## 动态配置
    [http.routers]
      [http.routers.Router-1]
        rule = "Host(`foo-domain`) && Path(`/foo-path/`)"
        service = "service-id"
        # 将终结TLS请求
        [http.routers.Router-1.tls]
    ```
    
    ```yaml tab="文件 (YAML)"
    ## 动态配置
    http:
      routers:
        Router-1:
          rule: "Host(`foo-domain`) && Path(`/foo-path/`)"
          service: service-id
          # 将终结TLS请求
          tls: {}
    ```

!!! important "HTTP和HTTPS的路由器"

    如果需要定义同一路由，同时用于HTTP和HTTPS请求，则需要定义两个不同的路由器：
    一个带tls部分，一个不带。

    ??? example "HTTP和HTTPS路由"

        ```toml tab="文件 (TOML)"
        ## 动态配置
        [http.routers]
          [http.routers.my-https-router]
            rule = "Host(`foo-domain`) && Path(`/foo-path/`)"
            service = "service-id"
            # 将终结TLS请求
            [http.routers.my-https-router.tls]

          [http.routers.my-http-router]
            rule = "Host(`foo-domain`) && Path(`/foo-path/`)"
            service = "service-id"
        ```

        ```yaml tab="文件 (YAML)"
        ## 动态配置
        http:
          routers:
            my-https-router:
              rule: "Host(`foo-domain`) && Path(`/foo-path/`)"
              service: service-id
              # 将终结TLS请求
              tls: {}

            my-http-router:
              rule: "Host(`foo-domain`) && Path(`/foo-path/`)"
              service: service-id
        ```

#### `options`

The `options` field enables fine-grained control of the TLS parameters.
It refers to a [TLS Options](../../https/tls.md#tls-options) and will be applied only if a `Host` rule is defined.

!!! info "Server Name Association"

    Even though one might get the impression that a TLS options reference is mapped to a router, or a router rule,
    one should realize that it is actually mapped only to the host name found in the `Host` part of the rule.
    Of course, there could also be several `Host` parts in a rule, in which case the TLS options reference would be mapped to as many host names.

    Another thing to keep in mind is:
    the TLS option is picked from the mapping mentioned above and based on the server name provided during the TLS handshake,
    and it all happens before routing actually occurs.

??? example "配置TLS选项"

    ```toml tab="文件 (TOML)"
    ## 动态配置
    [http.routers]
      [http.routers.Router-1]
        rule = "Host(`foo-domain`) && Path(`/foo-path/`)"
        service = "service-id"
        # 将终结TLS请求
        [http.routers.Router-1.tls]
          options = "foo"
    
    [tls.options]
      [tls.options.foo]
        minVersion = "VersionTLS12"
        cipherSuites = [
          "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384",
          "TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256",
          "TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256",
          "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256",
          "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256",
        ]
    ```
    
    ```yaml tab="文件 (YAML)"
    ## 动态配置
    http:
      routers:
        Router-1:
          rule: "Host(`foo-domain`) && Path(`/foo-path/`)"
          service: service-id
          # 将终结TLS请求
          tls:
            options: foo
    
    tls:
      options:
        foo:
          minVersion: VersionTLS12
          cipherSuites:
            - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
            - TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256
            - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
            - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
            - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
    ```

!!! important "Conflicting TLS Options"

    Since a TLS options reference is mapped to a host name,
    if a configuration introduces a situation where the same host name (from a `Host` rule) gets matched with two TLS options references,
    a conflict occurs, such as in the example below:

    ```toml tab="文件 (TOML)"
    ## 动态配置
    [http.routers]
      [http.routers.routerfoo]
        rule = "Host(`snitest.com`) && Path(`/foo`)"
        [http.routers.routerfoo.tls]
          options = "foo"

    [http.routers]
      [http.routers.routerbar]
        rule = "Host(`snitest.com`) && Path(`/bar`)"
        [http.routers.routerbar.tls]
          options = "bar"
    ```

    ```yaml tab="文件 (YAML)"
    ## 动态配置
    http:
      routers:
        routerfoo:
          rule: "Host(`snitest.com`) && Path(`/foo`)"
          tls:
            options: foo

        routerbar:
          rule: "Host(`snitest.com`) && Path(`/bar`)"
          tls:
            options: bar
    ```

    If that happens, both mappings are discarded, and the host name (`snitest.com` in this case) for these routers gets associated with the default TLS options instead.

#### `certResolver`

If `certResolver` is defined, Traefik will try to generate certificates based on routers `Host` & `HostSNI` rules.

```toml tab="文件 (TOML)"
## 动态配置
[http.routers]
  [http.routers.routerfoo]
    rule = "Host(`snitest.com`) && Path(`/foo`)"
    [http.routers.routerfoo.tls]
      certResolver = "foo"
```

```yaml tab="文件 (YAML)"
## 动态配置
http:
  routers:
    routerfoo:
      rule: "Host(`snitest.com`) && Path(`/foo`)"
      tls:
        certResolver: foo
```

!!! info "Multiple Hosts in a Rule"
    The rule ```Host(`test1.traefik.io`,`test2.traefik.io`)``` will request a certificate with the main domain `test1.traefik.io` and SAN `test2.traefik.io`.

#### `domains`

You can set SANs (alternative domains) for each main domain.
Every domain must have A/AAAA records pointing to Traefik.
Each domain & SAN will lead to a certificate request.

```toml tab="文件 (TOML)"
## 动态配置
[http.routers]
  [http.routers.routerbar]
    rule = "Host(`snitest.com`) && Path(`/bar`)"
    [http.routers.routerbar.tls]
      certResolver = "bar"
      [[http.routers.routerbar.tls.domains]]
        main = "snitest.com"
        sans = ["*.snitest.com"]
```

```yaml tab="文件 (YAML)"
## 动态配置
http:
  routers:
    routerbar:
      rule: "Host(`snitest.com`) && Path(`/bar`)"
      tls:
        certResolver: "bar"
        domains:
          - main: "snitest.com"
            sans:
              - "*.snitest.com"
```

[ACME v2](https://community.letsencrypt.org/t/acme-v2-and-wildcard-certificate-support-is-live/55579) supports wildcard certificates.
As described in [Let's Encrypt's post](https://community.letsencrypt.org/t/staging-endpoint-for-acme-v2/49605) wildcard certificates can only be generated through a [`DNS-01` challenge](../../https/acme.md#dnschallenge).

Most likely the root domain should receive a certificate too, so it needs to be specified as SAN and 2 `DNS-01` challenges are executed.
In this case the generated DNS TXT record for both domains is the same.
Even though this behavior is [DNS RFC](https://community.letsencrypt.org/t/wildcard-issuance-two-txt-records-for-the-same-name/54528/2) compliant,
it can lead to problems as all DNS providers keep DNS records cached for a given time (TTL) and this TTL can be greater than the challenge timeout making the `DNS-01` challenge fail.

The Traefik ACME client library [lego](https://github.com/go-acme/lego) supports some but not all DNS providers to work around this issue.
The [supported `provider` table](../../https/acme.md#providers) indicates if they allow generating certificates for a wildcard domain and its root domain.

!!! important "Wildcard certificates can only be verified through a [`DNS-01` challenge](../../https/acme.md#dnschallenge)."

!!! warning "Double Wildcard Certificates"
    It is not possible to request a double wildcard certificate for a domain (for example `*.*.local.com`).

## 配置TCP路由器 { #configuring-tcp-routers }

!!! warning "字符`@`不可用于路由器名称"

### General

If both HTTP routers and TCP routers listen to the same entry points, the TCP routers will apply *before* the HTTP routers.
If no matching route is found for the TCP routers, then the HTTP routers will take over.

### EntryPoints

If not specified, TCP routers will accept requests from all defined entry points.
If you want to limit the router scope to a set of entry points, set the entry points option.

??? example "监听每一个入口点"
    
    **动态配置**

    ```toml tab="文件 (TOML)"
    ## 动态配置
    
    [tcp.routers]
      [tcp.routers.Router-1]
        # By default, routers listen to every entrypoints
        rule = "HostSNI(`traefik.io`)"
        service = "service-1"
        # will route TLS requests (and ignore non tls requests)
        [tcp.routers.Router-1.tls]
    ```
    
    ```yaml tab="文件 (YAML)"
    ## 动态配置
    
    tcp:
      routers:
        Router-1:
          # By default, routers listen to every entrypoints
          rule: "HostSNI(`traefik.io`)"
          service: "service-1"
          # will route TLS requests (and ignore non tls requests)
          tls: {}
    ```

    **静态配置**
    
    ```toml tab="文件 (TOML)"
    ## 静态配置
    
    [entryPoints]
      [entryPoints.web]
        address = ":80"
      [entryPoints.websecure]
        address = ":443"
      [entryPoints.other]
        address = ":9090"
    ```
    
    ```yaml tab="文件 (YAML)"
    ## 静态配置
    
    entryPoints:
      web:
        address: ":80"
      websecure:
        address: ":443"
      other:
        address: ":9090"
    ```
    
    ```bash tab="CLI"
    ## 静态配置
    --entrypoints.web.address=:80
    --entrypoints.websecure.address=:443
    --entrypoints.other.address=:9090
    ```

??? example "Listens to Specific Entry Points"
    
    **动态配置**
    
    ```toml tab="文件 (TOML)"
    ## 动态配置
    [tcp.routers]
      [tcp.routers.Router-1]
        # won't listen to entry point web
        entryPoints = ["websecure", "other"]
        rule = "HostSNI(`traefik.io`)"
        service = "service-1"
        # will route TLS requests (and ignore non tls requests)
        [tcp.routers.Router-1.tls]
    ```
    
    ```yaml tab="文件 (YAML)"
    ## 动态配置
    tcp:
      routers:
        Router-1:
          # won't listen to entry point web
          entryPoints:
            - "websecure"
            - "other"
          rule: "HostSNI(`traefik.io`)"
          service: "service-1"
          # will route TLS requests (and ignore non tls requests)
          tls: {}
    ```

    **静态配置**
    
    ```toml tab="文件 (TOML)"
    ## 静态配置
    
    [entryPoints]
      [entryPoints.web]
        address = ":80"
      [entryPoints.websecure]
        address = ":443"
      [entryPoints.other]
        address = ":9090"
    ```
    
    ```yaml tab="文件 (YAML)"
    ## 静态配置
    
    entryPoints:
      web:
        address: ":80"
      websecure:
        address: ":443"
      other:
        address: ":9090"
    ```
    
    ```bash tab="CLI"
    ## 静态配置
    --entrypoints.web.address=:80
    --entrypoints.websecure.address=:443
    --entrypoints.other.address=:9090
    ```

### Rule

| Rule                           | Description                                                             |
|--------------------------------|-------------------------------------------------------------------------|
| ```HostSNI(`domain-1`, ...)``` | Check if the Server Name Indication corresponds to the given `domains`. |

!!! important "HostSNI & TLS"

    It is important to note that the Server Name Indication is an extension of the TLS protocol.
    Hence, only TLS routers will be able to specify a domain name with that rule.
    However, non-TLS routers will have to explicitly use that rule with `*` (every domain) to state that every non-TLS request will be handled by the router.

### Services

You must attach a TCP [service](../services/index.md) per TCP router.
Services are the target for the router.

!!! important "TCP routers can only target TCP services (not HTTP services)."

### TLS

#### General

当指定TLS部分时，它指示Traefik当前路由器仅专用于TLS请求（并且该路由器应忽略非TLS请求）。

默认情况下，Traefik将终结SSL连接（这意味着它将发送已解密的数据给服务），但Traefik可以配置为使请求直通（数据保持加密），然后“如是”转发给服务。

??? example "配置TLS终结"

    ```toml tab="文件 (TOML)"
    ## 动态配置
    [tcp.routers]
      [tcp.routers.Router-1]
        rule = "HostSNI(`foo-domain`)"
        service = "service-id"
        # 默认将终结TLS请求
        [tcp.routers.Router-1.tls]
    ```

    ```yaml tab="文件 (YAML)"
    ## 动态配置
    tcp:
      routers:
        Router-1:
          rule: "HostSNI(`foo-domain`)"
          service: service-id
          # 默认将终结TLS请求
          tls: {}
    ```

??? example "配置直通"

    ```toml tab="文件 (TOML)"
    ## 动态配置
    [tcp.routers]
      [tcp.routers.Router-1]
        rule = "HostSNI(`foo-domain`)"
        service = "service-id"
        [tcp.routers.Router-1.tls]
          passthrough = true
    ```

    ```yaml tab="文件 (YAML)"
    ## 动态配置
    tcp:
      routers:
        Router-1:
          rule: "HostSNI(`foo-domain`)"
          service: service-id
          tls:
            passthrough: true
    ```

#### `options`

`options`字段启用对TLS参数的细粒度控制。
它指的是[TLS选项](../../https/tls.md#tls-options)，仅在定义了`HostSNI`规则后才适用。

!!! example "配置TLS选项"

    ```toml tab="文件 (TOML)"
    ## 动态配置
    [tcp.routers]
      [tcp.routers.Router-1]
        rule = "HostSNI(`foo-domain`)"
        service = "service-id"
        # 将终结TLS请求
        [tcp.routers.Router-1.tls]
          options = "foo"
    
    [tls.options]
      [tls.options.foo]
        minVersion = "VersionTLS12"
        cipherSuites = [
          "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384",
          "TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256",
          "TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256",
          "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256",
          "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256",
        ]
    ```

    ```yaml tab="文件 (YAML)"
    ## 动态配置
    tcp:
      routers:
        Router-1:
          rule: "HostSNI(`foo-domain`)"
          service: service-id
          # 将终结TLS请求
          tls:
            options: foo
    
    tls:
      options:
        foo:
          minVersion: VersionTLS12
          cipherSuites:
            - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
            - TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256
            - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
            - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
            - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
    ```

#### `certResolver`

更多信息参见[`certResolver` for HTTP router](./index.md#certresolver)。

```toml tab="文件 (TOML)"
## 动态配置
[tcp.routers]
  [tcp.routers.routerfoo]
    rule = "HostSNI(`snitest.com`)"
    [tcp.routers.routerfoo.tls]
      certResolver = "foo"
```

```yaml tab="文件 (YAML)"
## 动态配置
tcp:
  routers:
    routerfoo:
      rule: "HostSNI(`snitest.com`)"
      tls:
        certResolver: foo
```

#### `domains`

更多信息参见[`domains` for HTTP router](./index.md#domains)。

```toml tab="文件 (TOML)"
## 动态配置
[tcp.routers]
  [tcp.routers.routerbar]
    rule = "HostSNI(`snitest.com`)"
    [tcp.routers.routerbar.tls]
      certResolver = "bar"
      [[tcp.routers.routerbar.tls.domains]]
        main = "snitest.com"
        sans = ["*.snitest.com"]
```

```yaml tab="文件 (YAML)"
## 动态配置
tcp:
  routers:
    routerbar:
      rule: "HostSNI(`snitest.com`)"
      tls:
        certResolver: "bar"
        domains:
          - main: "snitest.com"
            sans: 
              - "*.snitest.com"
```

## 配置UDP路由器 { #configuring-udp-routers }

!!! warning "字符`@`不可用于路由器名称"

### General

与TCP类似，因为UDP是传输层，所以没有请求的概念，因此也没有URL路径前缀与传入的UDP数据包匹配的概念。 此外，针对多个主机名，目前还没有良好的TLS支持，因此没有主机SNI概念来做匹配。因此，没有规范可用作传入数据包的匹配规则，以对其进行路由。
因此，此时的UDP“路由器”，几乎只是一种或另一种形式的负载均衡器。

!!! important "会话和超时"

	即使UDP是无连接的（也因此），在Traefik中的UDP路由器实现，也依赖于我们（以及其他一些实现）称之为`session`的东西。
  基本上，这意味着保持某种状态，此状态有关于客户端与后端服务器之间正在进行的通信，
  特别是，由此代理可以知道，从后端转发响应数据包的位置。
  正如预期的那样，一个`timeout`与其中的每一个会话关联，因此，如果它们的闲置时间长于给定的持续时间（目前硬编码为3秒），则会被清除。
  如果此问题获得更多使用反馈，则稍后将考虑使此超时可配置。

### EntryPoints

如果未指定，则UDP路由器将接受来自所有已定义（UDP）入口点的数据包。
如果要将路由器范围限制为一组入口点，则应设置入口点选项。

??? example "监听每一个入口点"

    **动态配置**

    ```toml tab="文件 (TOML)"
    ## 动态配置

    [udp.routers]
      [udp.routers.Router-1]
        # 默认情况下，路由器侦听所有UDP入口点，
        # i.e. "other", and "streaming".
        service = "service-1"
    ```

    ```yaml tab="文件 (YAML)"
    ## 动态配置

    udp:
      routers:
        Router-1:
          # 默认情况下，路由器侦听所有UDP入口点，
          # i.e. "other", and "streaming".
          service: "service-1"
    ```

    **静态配置**

    ```toml tab="文件 (TOML)"
    ## 静态配置

    [entryPoints]
      # 不用于UDP路由器
      [entryPoints.web]
        address = ":80"
      # 用于UDP路由器
      [entryPoints.other]
        address = ":9090/udp"
      [entryPoints.streaming]
        address = ":9191/udp"
    ```

    ```yaml tab="文件 (YAML)"
    ## 静态配置

    entryPoints:
      # 不用于UDP路由器
      web:
        address: ":80"
      # 用于UDP路由器
      other:
        address: ":9090/udp"
      streaming:
        address: ":9191/udp"
    ```

    ```bash tab="CLI"
    ## 静态配置
    --entrypoints.web.address=":80"
    --entrypoints.other.address=":9090/udp"
    --entrypoints.streaming.address=":9191/udp"
    ```

??? example "监听特定入口点"

    **动态配置**

    ```toml tab="文件 (TOML)"
    ## 动态配置
    [udp.routers]
      [udp.routers.Router-1]
        # 不监听入口点"other"
        entryPoints = ["streaming"]
        service = "service-1"
    ```

    ```yaml tab="文件 (YAML)"
    ## 动态配置
    udp:
      routers:
        Router-1:
          # 不监听入口点"other"
          entryPoints:
            - "streaming"
          service: "service-1"
    ```

    **静态配置**

    ```toml tab="文件 (TOML)"
    ## 静态配置

    [entryPoints]
      [entryPoints.web]
        address = ":80"
      [entryPoints.other]
        address = ":9090/udp"
      [entryPoints.streaming]
        address = ":9191/udp"
    ```

    ```yaml tab="文件 (YAML)"
    ## 静态配置

    entryPoints:
      web:
        address: ":80"
      other:
        address: ":9090/udp"
      streaming:
        address: ":9191/udp"
    ```

    ```bash tab="CLI"
    ## 静态配置
    --entrypoints.web.address=":80"
    --entrypoints.other.address=":9090/udp"
    --entrypoints.streaming.address=":9191/udp"
    ```

### Services

There must be one (and only one) UDP [service](../services/index.md) referenced per UDP router.
Services are the target for the router.

!!! important "UDP routers can only target UDP services (and not HTTP or TCP services)."
