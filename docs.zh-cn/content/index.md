# 欢迎 { #welcome }

![体系架构(Architecture)](assets/img/traefik-architecture.png)

Traefik是[开源的](https://github.com/containous/traefik)**边缘路由器**，使发布服务变得有趣而轻松。
它代表您的系统接收请求，并找出负责处理这些请求的组件。

除了其众多功能之外，Traefik的与众不同之处还在于它会自动发现适合您服务的配置。
Traefik在检查您的基础结构时会发生神奇的事情，它找到相关信息，并发现哪个服务满足于哪个请求。

Traefik原生兼容所有主要的集群技术，诸如Kubernetes，Docker，Docker Swarm，AWS，Mesos，Marathon，
且[清单仍在增长](providers/overview.md)；并且，它可以同时处理多个集群技术。（它甚至适用于在实体机上运行的旧版软件。）

使用Traefik，无需维护和同步单独的配置文件：一切都会自动、实时地发生（无需重启，不中断连接）。
使用Traefik，你的时间花在为系统开发和部署新功能，而不是在配置和维护其工作状态上。

开发Traefik，我们的主要目标是使其易于使用，我们相信您会喜欢的。

-- Traefik 维护团队

!!! info

    加入我们用户友好且活跃的[社区论坛](https://community.containo.us)，与traefik社区进行讨论、学习和联系。

    如果你运行的是关键业务服务背后的Traefik，
    则[Containous](https://containo.us)，即赞助Traefik开发发展的公司，
    它可以提供[商业化的支持](https://info.containo.us/commercial-services)，并且其开发了一个[企业版](https://containo.us/traefikee/)的Traefik。
