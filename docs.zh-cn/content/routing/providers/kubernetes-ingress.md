# Traefik和Kubernetes { #traefik-kubernetes }

Kubernetes Ingress 控制器.
{: .subtitle }

## 路由配置 { #routing-configuration }

提供者程序随即监视传入的ingresses事件，如下，并从中得出相应的动态配置，
可依次创建最终的路由器(Routers)、服务(Services)、处理程序(Handlers)等。

## 配置示例 { #configuration-example }

??? example "配置Kubernetes Ingress控制器"
    
    ```yaml tab="RBAC"
    ---
    kind: ClusterRole
    apiVersion: rbac.authorization.k8s.io/v1beta1
    metadata:
      name: traefik-ingress-controller
    rules:
      - apiGroups:
          - ""
        resources:
          - services
          - endpoints
          - secrets
        verbs:
          - get
          - list
          - watch
      - apiGroups:
          - extensions
        resources:
          - ingresses
        verbs:
          - get
          - list
          - watch
      - apiGroups:
          - extensions
        resources:
          - ingresses/status
        verbs:
          - update
    
    ---
    kind: ClusterRoleBinding
    apiVersion: rbac.authorization.k8s.io/v1beta1
    metadata:
      name: traefik-ingress-controller
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: traefik-ingress-controller
    subjects:
      - kind: ServiceAccount
        name: traefik-ingress-controller
        namespace: default
    ```
    
    ```yaml tab="Ingress"
    kind: Ingress
    apiVersion: networking.k8s.io/v1beta1
    metadata:
      name: myingress
      annotations:
        traefik.ingress.kubernetes.io/router.entrypoints: web
    
    spec:
      rules:
        - host: mydomain.com
          http:
            paths:
              - path: /bar
                backend:
                  serviceName: whoami
                  servicePort: 80
              - path: /foo
                backend:
                  serviceName: whoami
                  servicePort: 80
    ```
    
    ```yaml tab="Traefik"
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: traefik-ingress-controller
    
    ---
    kind: Deployment
    apiVersion: apps/v1
    metadata:
      name: traefik
      labels:
        app: traefik
    
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: traefik
      template:
        metadata:
          labels:
            app: traefik
        spec:
          serviceAccountName: traefik-ingress-controller
          containers:
            - name: traefik
              image: traefik:v2.2
              args:
                - --log.level=DEBUG
                - --api
                - --api.insecure
                - --entrypoints.web.address=:80
                - --providers.kubernetesingress
              ports:
                - name: web
                  containerPort: 80
                - name: admin
                  containerPort: 8080
    
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: traefik
    spec:
      type: LoadBalancer
      selector:
        app: traefik
      ports:
        - protocol: TCP
          port: 80
          name: web
          targetPort: 80
        - protocol: TCP
          port: 8080
          name: admin
          targetPort: 8080
    ```
    
    ```yaml tab="Whoami"
    kind: Deployment
    apiVersion: apps/v1
    metadata:
      name: whoami
      labels:
        app: containous
        name: whoami
    
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: containous
          task: whoami
      template:
        metadata:
          labels:
            app: containous
            task: whoami
        spec:
          containers:
            - name: containouswhoami
              image: containous/whoami
              ports:
                - containerPort: 80
    
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: whoami
    
    spec:
      ports:
        - name: http
          port: 80
      selector:
        app: containous
        task: whoami
    ```

## 注解 { #annotations }

#### On Ingress

??? info "`traefik.ingress.kubernetes.io/router.entrypoints`"

    更多信息参看[entry points](../routers/index.md#entrypoints)。

    ```yaml
    traefik.ingress.kubernetes.io/router.entrypoints: ep1,ep2
    ```

??? info "`traefik.ingress.kubernetes.io/router.middlewares`"

    更多信息参看[middlewares](../routers/index.md#middlewares)，以及[中间件概览](../../middlewares/overview.md)。

    ```yaml
    traefik.ingress.kubernetes.io/router.middlewares: auth@file,prefix@kuberntes-crd,cb@file
    ```

??? info "`traefik.ingress.kubernetes.io/router.priority`"

    更多信息参看[priority](../routers/index.md#priority)。

    ```yaml
    traefik.ingress.kubernetes.io/router.priority: "42"
    ```

??? info "`traefik.ingress.kubernetes.io/router.pathmatcher`"

    覆盖用于路径的默认路由器规则类型。
    只能指定与路径相关的匹配器名称：`Path`，`PathPrefix`。
    
    默认 `PathPrefix`

    ```yaml
    traefik.ingress.kubernetes.io/router.pathmatcher: Path
    ```

??? info "`traefik.ingress.kubernetes.io/router.tls`"

    更多信息参看[tls](../routers/index.md#tls)。

    ```yaml
    traefik.ingress.kubernetes.io/router.tls: "true"
    ```

??? info "`traefik.ingress.kubernetes.io/router.tls.certresolver`"

    更多信息参看[certResolver](../routers/index.md#certresolver)。

    ```yaml
    traefik.ingress.kubernetes.io/router.tls.certresolver: myresolver
    ```

??? info "`traefik.ingress.kubernetes.io/router.tls.domains.n.main`"

    更多信息参看[domains](../routers/index.md#domains)。

    ```yaml
    traefik.ingress.kubernetes.io/router.tls.domains.0.main: foobar.com
    ```

??? info "`traefik.ingress.kubernetes.io/router.tls.domains.n.sans`"

    更多信息参看[domains](../routers/index.md#domains)。

    ```yaml
    traefik.ingress.kubernetes.io/router.tls.domains.0.sans: test.foobar.com,dev.foobar.com
    ```

??? info "`traefik.ingress.kubernetes.io/router.tls.options`"

    更多信息参看[options](../routers/index.md#options)。

    ```yaml
    traefik.ingress.kubernetes.io/router.tls.options: foobar
    ```

#### On Service

??? info "`traefik.ingress.kubernetes.io/service.serversscheme`"

    覆盖默认的 Scheme。

    ```yaml
    traefik.ingress.kubernetes.io/service.serversscheme: h2c
    ```

??? info "`traefik.ingress.kubernetes.io/service.passhostheader`"

    更多信息参看[pass Host header](../services/index.md#pass-host-header)。

    ```yaml
    traefik.ingress.kubernetes.io/service.passhostheader: "true"
    ```

??? info "`traefik.ingress.kubernetes.io/service.sticky`"

    更多信息参看[sticky sessions](../services/index.md#sticky-sessions)。

    ```yaml
    traefik.ingress.kubernetes.io/service.sticky: "true"
    ```

??? info "`traefik.ingress.kubernetes.io/service.sticky.cookie.httponly`"

    更多信息参看[sticky sessions](../services/index.md#sticky-sessions)。

    ```yaml
    traefik.ingress.kubernetes.io/service.sticky.cookie.httponly: "true"
    ```

??? info "`traefik.ingress.kubernetes.io/service.sticky.cookie.name`"

    更多信息参看[sticky sessions](../services/index.md#sticky-sessions)。

    ```yaml
    traefik.ingress.kubernetes.io/service.sticky.cookie.name: foobar
    ```

??? info "`traefik.ingress.kubernetes.io/service.sticky.cookie.secure`"

    更多信息参看[sticky sessions](../services/index.md#sticky-sessions)。

    ```yaml
    traefik.ingress.kubernetes.io/service.sticky.cookie.secure: "true"
    ```

### TLS

#### Traefik与Pod间的通讯 { #communication-between-traefik-and-pods }

Traefik基于Ingress规范（Ingress Spec）所提供的服务，来自动请求端点信息。
尽管Traefik将直连到端点（Pods），但它仍会检查服务端口，以查看是否需要TLS通信。

有3种方法可以将Traefik配置为使用https与pod进行通信：

1. 如果Ingress规范中定义的服务端口是`443`（注意，仍然可以在Pod上用`targetPort`来使用其他端口）。
1. 如果Ingress规范中定义的服务端口是名称以https开头（例如，`https-api`，`https-web`或只是`https`）。
1. 如果Ingress规范包含注解`traefik.ingress.kubernetes.io/service.serversscheme: https`。

如果存在这些配置选项之一，则假定后端通信协议为TLS，并将自动通过TLS连接。

!!! info

    请注意，启用Traefik和Pod之间的TLS通信，则必须拥有具有正确信任链和IP主题名称(Subject Name)的受信任证书。
    如果没得选，那就可能要跳过TLS证书验证。
    更多细节，参看[insecureSkipVerify](../../routing/overview.md#insecureskipverify)设置。

#### 证书管理 { #certificates-management }

??? example "Using a secret"
    
    ```yaml tab="Ingress"
    kind: Ingress
    apiVersion: networking.k8s.io/v1beta1
    metadata:
      name: foo
      namespace: production
    
    spec:
      rules:
      - host: foo.com
        http:
          paths:
          - path: /bar
            backend:
              serviceName: service1
              servicePort: 80
    
      tls:
      - secretName: supersecret
    ```
      
    ```yaml tab="Secret"
    apiVersion: v1
    kind: Secret
    metadata:
      name: supersecret
    
    data:
      tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0=
      tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCi0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0=
    ```

TLS证书可以用Secrets对象来管理。

!!! info
    
    只有用户提供的TLS证书才能保存在Kubernetes Secrets中
    [Let's Encrypt](../../https/acme.md) 证书尚不能在Kubernetes Secrets中进行管理。

## Global Default Backend Ingresses

可以创建如下所示的Ingresses：

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
 name: cheese

spec:
 backend:
   serviceName: stilton
   servicePort: 80
```

此Ingress遵循Ingress的全局默认后端属性。
这允许用户创建匹配所有不匹配请求的“默认路由器”。

!!! info

    由于Traefik使用优先级，你可能必须将此Ingress的优先级设为低于环境中的其他Ingress，避免此全局Ingress满足可能与其他ingress匹配的请求。
    为此，请在Ingress上相应地使用`traefik.ingress.kubernetes.io/router.priority`注解（如[Ingress上的注解](#on-ingress)中所示）。
