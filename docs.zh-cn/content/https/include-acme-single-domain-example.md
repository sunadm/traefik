
```yaml tab="Docker"
## 动态配置
labels:
  - traefik.http.routers.blog.rule=Host(`company.com`) && Path(`/blog`)
  - traefik.http.routers.blog.tls=true
  - traefik.http.routers.blog.tls.certresolver=myresolver
```

```yaml tab="Docker (Swarm)"
## 动态配置
deploy:
  labels:
    - traefik.http.routers.blog.rule=Host(`company.com`) && Path(`/blog`)
    - traefik.http.routers.blog.tls=true
    - traefik.http.routers.blog.tls.certresolver=myresolver
    - traefik.http.services.blog-svc.loadbalancer.server.port=8080"
```

```yaml tab="Kubernetes"
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: blogtls
spec:
  entryPoints:
    - websecure
  routes:
  - match: Host(`company.com`) && Path(`/blog`)
    kind: Rule
    services:
    - name: blog
      port: 8080
  tls:
    certresolver: myresolver
```

```json tab="Marathon"
labels: {
  "traefik.http.routers.blog.rule": "Host(`company.com`) && Path(`/blog`)",
  "traefik.http.routers.blog.tls": "true",
  "traefik.http.routers.blog.tls.certresolver": "myresolver",
  "traefik.http.services.blog-svc.loadbalancer.server.port": "8080"
}
```

```yaml tab="Rancher"
## 动态配置
labels:
  - traefik.http.routers.blog.rule=Host(`company.com`) && Path(`/blog`)
  - traefik.http.routers.blog.tls=true
  - traefik.http.routers.blog.tls.certresolver=myresolver
```

```toml tab="Single Domain"
## 动态配置
[http.routers]
  [http.routers.blog]
  rule = "Host(`company.com`) && Path(`/blog`)"
  [http.routers.blog.tls]
    certResolver = "myresolver"
```

```yaml tab="文件 (YAML)"
## 动态配置
http:
  routers:
    blog:
      rule: "Host(`company.com`) && Path(`/blog`)"
      tls:
        certResolver: myresolver
```
