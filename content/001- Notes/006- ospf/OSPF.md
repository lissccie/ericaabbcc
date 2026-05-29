

# OSPF (Open Shortest Path First) 核心架构

## 基础概念
- **链路状态协议 (IGP)**：基于 Dijkstra SPF 算法计算无环拓扑。
- **版本支持**：IPv4 使用 OSPFv2 (RFC 2328)，IPv6 使用 OSPFv3 (RFC 5340)。
- **层次化架构**：骨干区域 (Area 0) 与非骨干区域，减少路由洪泛。

## 底层通信机制
- **封装**：直接封装于 IP 报文中。
> [!tip] 核心考点
> OSPF 拥有专属的**协议号 89**。在配置 ACL（访问控制列表）抓取 OSPF 报文时，必须匹配协议号 89，绝不能混淆成传输层的 TCP 或 UDP 端口。
- **组播寻址**：
  - `224.0.0.5` (AllSPFRouters)：所有 OSPF 路由器监听。
  - `224.0.0.6` (AllDRouters)：仅 DR/BDR 监听。

## 五大报文类型 (Packet Types)
1. **Type 1 (Hello)**：发现与维护邻居，选举 DR/BDR。
2. **Type 2 (DBD)**：Database Description，交换数据库摘要。
3. **Type 3 (LSR)**：Link-State Request，请求特定链路状态。
4. **Type 4 (LSU)**：Link-State Update，发送详细 LSA 更新。
5. **Type 5 (LSAck)**：Link-State Acknowledgment，可靠性确认。

## 邻居建立状态机 (Neighbor States)
- **Down / Attempt**：无 Hello 交互。
- **Init**：收到单向 Hello。
- **2-Way**：双向通信确立（**注意：此时进行 DR/BDR 选举**）。
- **ExStart**：主从关系确立。
- **Exchange**：摘要信息 (DBD) 交换。
- **Loading**：请求 (LSR) 与更新 (LSU) 缺失路由。
- **Full**：数据库 (LSDB) 完全同步。

## DR / BDR 选举机制
- **目的**：在多路访问网络中减少邻接关系数量，避免由于 $n(n-1)/2$ 带来的庞大洪泛。
- **选举条件**：按接口优先级 (默认 1，设置为 0 则不参与) -> Router ID 比较。
> [!warning] 避坑：非抢占特性
> DR 选举是**非抢占式**的。一旦选举完成，即使接入优先级更高（如 255）的新设备也不会主动让位，除非原 DR 宕机或手动清理 OSPF 进程。

## OSPF 网络类型 (Network Types)
- **Broadcast**：以太网默认，需选举 DR/BDR，Hello 间隔 10s。
- **Point-to-Point**：串行链路默认，无需 DR，Hello 间隔 10s。
- **Loopback**：环回口默认，强制以 `/32` 主机路由宣告（可通过配置 P2P 网络类型还原真实掩码）。

## 核心配置与优化
- **启用方式**：`network` 命令网段宣告 vs 接口直下 `ip ospf` 命令。
- **安全控制**：`passive-interface` 停止收发 Hello 报文，防止非法抓包与恶意路由注入。
- **默认路由下发**：`default-information originate`。
- **开销调优 (Cost)**：默认基于 `100 Mbps / 接口带宽`。对于千兆/万兆网络，需全局统一修改 `auto-cost reference-bandwidth`，否则高速链路会被误判为同等开销。