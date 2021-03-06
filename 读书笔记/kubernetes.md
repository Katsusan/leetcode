1.概念
    - Pod(容器组)：表示一个或多个相互强相关容器的组合。Pod是kubernetes调度的基本单元，同一个pod中的容器会一起启动/重启/销毁，
        并自动部署在相同的物理节点上，共享相同的数据卷和网络命名空间。
    - Replication Controller(副本控制器): 它使用预定义模板自动创建固定数量的pod实例，并可以运行在不同的物理节点上。
    - Service(服务)：集群对外提供的具体业务抽象，表现为一个独立的访问地址和端口。实际上可能是由单个或多个pod组成的容器集合。
    - Label(标签): 用于区分Pod/Replica/Service的键值对。

2.组成
    主从分布式集群架构，由Master节点和Node节点组成。整个集群中总是有且只有一个Master节点，作为控制节点调度整个系统。
    此外所有节点都作为Node节点运行业务容器。

    Master节点：
        - kube-apiserver：整个集群管理系统的核心，部署时首先应该启动的组件。其它组件启动时都会接入这个服务，获取集群的信息和注册自己的地址。
            它主要是通过etcd记录的集群状态并对外提供restful api接口来实现的。
        - kube-scheduler：负责服务调度的子模块。可以根据集群的资源使用状况选择适合运行特定服务的节点。
        - kube-controller-manager：负责管理kubernetes系统中额外的控制器任务。
    
    Node节点：
        - kube-proxy：解决Node节点上的Pod对特定Service的访问的路由问题。
            每当kubernetes创建一个Service，每个Node节点上的kube-proxy会从etcd获取Service的服务配置信息，根据这些信息在节点上启动一个代理进程
            并监听相应的服务端口。当Pod访问相应服务时，代理进程会负责将请求分发到正确的容器中处理。
        -kubelet：负责直接根节点上的容器服务打交道。在v1.0中已经支持Docker/Rkt两种容器实现。
            此外也接收来自kube-apiserver的HTTP请求，汇报所在节点的Pod运行状态。
        
3. Pod和容器设计模式
    亲密关系 -> 运行于同一台宿主机
    超亲密关系 -> 同一个pod下(频繁发生rpc调用/通过localhost通信等)

    共享网络：
        k8s.gcr.io/pause -> 汇编写的一直处于暂停状态的100-200KB大小的镜像。
        container用-net=k8s.gcr.io/pause启动，则可以直接用localhost通信，并且该pod的IP地址与infra一样，生命周期与infra容器相同。

    共享存储：
        pod内指定volume来解决。
        例：
            ```yaml
            apiVersion: v1
            kind: Pod
            metadata: 
                name: two-containers
            spec:
                restartPolicy: Never
                volumes:
                - name: shared-data
                  host-path:
                    path: /data
                containers:
                - name: nginx-container
                  image: nginx
                  volumeMounts:
                  - name: shared-data
                    mountPath: /usr/share/nginx/html
                - name: debina-container
                  image: debian
                  volumeMounts:
                  - name: shared-data
                    mountPath: /pod/staticfiles
            ```
    
    容器设计模式：
        InitContainer：
            比spec.Containers定义的用户容器先启动，并且严格按照定义的顺序依次执行。
            ```yaml
            spec:
                initContainers:
                - image: sample:v2
                  name: warpackage
                  command: ["cp", "/sample.war", "/app"]
                  volumeMounts:
                  - mountPath: /app
                    name: app-volume
                containers:
                ...
                volumes:
                - name: app-volume
                  emptyDir: {}
        
        SideCar:
            pod里定义专门的容器来执行主业务容器需要的辅助工作。比如日志、debug、应用监控等。

            代理容器模式：
                将pod的入口统一在proxy处
            适配器容器模式：
                将业务容器暴露出的接口转换为另一种模式
            
