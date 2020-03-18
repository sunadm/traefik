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

### 入口点 { #entrypoints }

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

### 优先级 { #priority }

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

### 服务 { #service }

每个请求最终都必须由一个[服务](../services/index.md)来处理，
这就是为什么每个路由器定义都应包括一个目标服务的原因，
该服务目标基本上是将请求传递到的位置。

通常，一个分配给路由器的服务，应该是已经定义过的，
但有例外，即基于标签的提供者程序。
请参阅特定的[docker](../providers/docker.md#service-definition)，
[rancher](../providers/rancher.md#service-definition)，
或[marathon](../providers/marathon.md#service-definition)文档。

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

`options`字段启用对TLS参数的细粒度控制。
它指的是[TLS选项](../../https/tls.md#tls-options)，仅在`Host`规则定义了之后才适用。

!!! info "服务器名称关联"

    即使可能会给人一种印象，即TLS选项参考是映射到路由器或路由器规则的，
    但您也应意识到，它实际上仅仅映射到主机名，该主机名在规则的`Host`部分中。
    当然，规则中也可以有多个`Host`部分，在这种情况下，TLS选项参考将映射到尽可能多的主机名。

    要记脑里的另一件事：
    TLS选项是从上述映射中选择的，并且基于TLS握手期间所提供的服务器名称，
    并且所有这些操作，都在路由实际发生之前进行。

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

!!! important "配置TLS选项"

    由于TLS选项引用映射到主机名，
    因此，如果配置引入以下情况：同一主机名（来自一条`Host`规则）匹配了两个TLS选项引用，
    则会发生冲突，如下例：

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

    如果发生那种情况，则两个映射都将被丢弃，这些路由器的主机名（此例中为`snitest.com`）将与默认的TLS选项关联。

#### `certResolver`

如果定义了`certResolver`，Traefik将尝试基于路由器的`Host`和`HostSNI`规则来生成证书。

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

!!! info "一条规则中的多个主机"
    规则```Host(`test1.traefik.io`,`test2.traefik.io`)```将以主域名`test1.traefik.io`以及SAN`test2.traefik.io`来请求证书。

#### `domains`

可以为每个主域名设置SAN（备用域）。
每个域名都必须具有指向Traefik(IP)的A/AAAA记录。
每个域名及SAN都会引发一个证书请求。

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

[ACME v2](https://community.letsencrypt.org/t/acme-v2-and-wildcard-certificate-support-is-live/55579) 支持通配证书。
如[Let's Encrypt的帖子](https://community.letsencrypt.org/t/staging-endpoint-for-acme-v2/49605)所述，通配证书只能通过[`DNS-01` challenge](../../https/acme.md#dnschallenge)来生成。
根域名很可能也应该收到证书，因此它也需要指定为SAN，然后执行了2个`DNS-01` challenges。
在这种情况下，为两个域名所生成的DNS TXT记录是相同的。
即使此行为符合[DNS RFC](https://community.letsencrypt.org/t/wildcard-issuance-two-txt-records-for-the-same-name/54528/2)，
也可能导致问题，因为所有DNS提供程序都会将DNS记录缓存一个给定时间（TTL），并且此TTL可能大于质询超时，从而使`DNS-01` challenge失败。

Traefik ACME客户端库[lego](https://github.com/go-acme/lego)支持部分，但不是全部DNS提供程序来解决此问题。
[支持的`DNS提供程序`表格](../../https/acme.md#providers)指出了它们是否允许为通配域名及其根域名生成证书。

!!! important "通配证书只能通过[`DNS-01` challenge](../../https/acme.md#dnschallenge)进行验证。"

!!! warning "双通配证书"
    不可能为域名请求双通配符证书（例如`*.*.local.com`）。

## 配置TCP路由器 { #configuring-tcp-routers }

!!! warning "字符`@`不可用于路由器名称"

### General

如果HTTP路由器和TCP路由器都监听相同的入口点，则TCP路由器将在HTTP路由器*之前*应用。
如果找不到与TCP路由器匹配的路由，则HTTP路由器将接管。

### EntryPoints

如未指定，TCP路由器将接受来自所有已定义入口点的请求。
如果要将路由器范围限制为一组入口点，请设置入口点选项。

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

??? example "监听特定的入口点"
    
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

| 规则                            | 描述                                          |
|--------------------------------|----------------------------------------------|
| ```HostSNI(`domain-1`, ...)``` | 检查服务器名称指示（SNI）是否与给定的`domains`对应。  |

!!! important "HostSNI和TLS"

    需重点注意的是，服务器名称指示（SNI，Server Name Indication）是TLS协议的扩展。
    因此，只有TLS路由器才能使用该规则来指定域名。
    不过，非TLS路由器将必须显式使用带有`*`（每个域名）的规则，来声明每个非TLS请求都将由该路由器处理。

### Services

您必须为每个TCP路由器附加一个TCP[服务](../services/index.md)。
服务是路由器的目标。

!!! important "TCP路由器仅能标向TCP服务（不能标向HTTP服务）。"

### TLS

#### General

当指定TLS部分时，它指示Traefik当前路由器仅专用于TLS请求（并且该路由器应忽略非TLS请求）。

默认下，带有TLS部分的路由器将终结TLS连接，意味着它将发送已解密的数据给服务。

??? example "用于TLS请求的路由器"

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

#### `passthrough`

如上所示，TLS路由器将默认终结TLS连接。
然而，可以指定`passthrough`选项，以设置将请求“如是”转发，保持所有数据为加密状态。

默认为`false`。

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

更多信息参见[用于HTTP路由器的`certResolver`](./index.md#certresolver)。

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

每个UDP路由器必须且仅能引用一个UDP[服务](../services/index.md)。
服务是路由器的目标。

!!! important "UDP路由器仅能标向UDP服务（不能标向HTTP或TCP服务）。"
