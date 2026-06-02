## ClusterIP → 负载由 kube-proxy 负责

- **机制**：kube-proxy 在每个节点上写 `iptables` / `IPVS` 规则，拦截发往 ClusterIP 的流量，随机或轮询转发到后端 Pod
- **对客户端**：完全无感知，客户端只需访问固定的 ClusterIP
- **结论**：负载均衡由 **服务端（kube-proxy）** 承担

## Headless（clusterIP: None）→ 负载由客户端负责

- **机制**：不分配虚拟 IP，DNS 查询直接返回所有 Pod IP 列表（A 记录）
- **对客户端**：客户端拿到 IP 列表后，**自己选择**访问哪一个 Pod
- **结论**：负载均衡由 **客户端** 承担，kube-proxy 不参与

## 使用 Headless 的必要条件（必须满足）

1. **客户端必须自己实现负载逻辑**
    - 轮询、最少连接、随机等算法要客户端自己做
    - 如果客户端只是简单取第一个 IP，会导致负载不均

2. **客户端需要能处理 Pod IP 变化**
    - Pod 重启 IP 可能改变
    - 客户端应该定期重新解析 DNS，或使用 Watch 机制

3. **通常配合 StatefulSet 使用**
    - Headless + StatefulSet 才能保证稳定的网络标识（`pod-name.service-name`）
    - 普通 Deployment 用 Headless 只能拿到 IP 列表，但没有稳定 hostname

4. **不需要 Service 级别的负载均衡 / 会话保持**
    - 如果业务依赖 ClusterIP 的会话亲和性（`sessionAffinity`），Headless 做不到

5. **使用场景对“拿到所有真实 Pod IP”有强需求**
    - 比如服务发现、广播消息、自定义路由、数据库集群节点感知

## 一句话总结

> **ClusterIP：服务端替你负载均衡，你只管访问一个固定 IP。**  
> **Headless：服务端只告诉你所有 Pod 在哪，你自己决定访问谁、怎么负载。**