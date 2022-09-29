# F5network

本组件整合了 F5 官方项目 [ f5 ipam controller ](https://github.com/F5Networks/f5-ipam-controller) 和 [ k8s bigip ctlr ](https://github.com/F5Networks/k8s-bigip-ctlr)，完成对 F5 设备的控制，实现把集群中 service 和 ingress 配置同步到 F5 硬件设备上，实现集群北向入口的负载均衡。

其中，[ k8s bigip ctlr ](https://github.com/F5Networks/k8s-bigip-ctlr) 组件负责监控 service 或 ingress 对象，实现对 F5 硬件设备的控制面规则下发；
当工作在 4 层负载均衡模式下时，[ f5 ipam controller ](https://github.com/F5Networks/f5-ipam-controller)组件主要负责 F5 硬件的入口 VIP 分配

F5 设备有两种模式来实现转发流量到集群（关于转发模式，更多信息可参考[ 官方说明 ](https://clouddocs.f5.com/containers/latest/userguide/config-options.html) ）：

1. NodePort 转发模式：F5 把流量转发到集群节点的 nodePort 上，该模式即可工作在"4层转发"和""7层转发"。

    优势：不需要在集群和F5设备之间做特殊处理，只要F5设备能够访问集群节点即可，通用性更强。

    要求：集群应用的service对象必须分配了 nodePort。

    ![ nodeport ](../../images/F5nodeport.png)

2. Cluster 转发模式：F5 把流量直接转发 pod ip上，该模式即可工作在"4层转发"和""7层转发"。

    优势：数据包转发不经过 kube proxy 的nodePort方式，直接转发到pod上，转发方式更加高效，延时更低

    要求：要求集群通过 BPG 协议把 pod 路由转发到网络中的路由器和F5设备上，或者要求集群中的节点同 F5 设备间建立的 VXLAN 隧道

   ![ cluster ](../../images/F5cluster.png)



功能说明：

1. 4 层负载均衡。
    该负载均衡模式下，配合"NodePort 转发模式"，可为 LoadBalancer service 创建 F5 负载均衡；也可配合"Cluster 转发模式"，可为 clusterIP service 创建 F5 负载均衡，
    其中，[ f5 ipam controller ](https://github.com/F5Networks/f5-ipam-controller) 组件维护一个可配置的VIP 池，为每个 service 独立分配一个独享的 EXTERNAL IP

    注：该模式下，务必安装[ f5 ipam controller ](https://github.com/F5Networks/f5-ipam-controller)，为每个service分配一个VIP

2. 7 层负载均衡。
   该负载均衡模式下，工作为 ingress controller。可配合"NodePort 转发模式"，要求 ingress 匹配的 service 是 nodePort 类型；
   可配合"Cluster 转发模式"， ingress 匹配的 service 是 clusterIP 类型即可

   注：该模式下，不需要安装[ f5 ipam controller ](https://github.com/F5Networks/f5-ipam-controller), 所有 ingress 共享一个vip


注意：本组件不能同时工作在"4 层负载均衡"和"7 层负载均衡"下，只能二选一

更多信息，可参考 [F5 官方文档](https://clouddocs.f5.com/containers/latest/userguide/)

