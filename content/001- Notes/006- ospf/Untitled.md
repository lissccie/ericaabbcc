

# OSPF (Open Shortest Path First) 核心架构

## 基础概念
### 链路状态协议 (IGP)，基于 Dijkstra SPF 算法计算无环拓扑
### IPv4 使用 OSPFv2 (RFC 2328)，IPv6 使用 OSPFv3 (RFC 5340)
### 层次化架构：骨干区域 (Area 0) 与非骨干区域，减少路由洪泛

## 底层通信机制
### 封装：直接封装于 IP 报文中，协议号 89
### 组播寻址：224.0.0.5 (AllSPFRouters) / 224.0.0.6 (仅用于 DR/BDR)

## 五大报文类型 (Packet Types)
### Type 1: Hello (发现与维护邻居，选举 DR)
### Type 2: DBD (交换数据库摘要)
### Type 3: LSR (请求特定链路状态)
### Type 4: LSU (发送详细 LSA 更新)
### Type 5: LSAck (可靠性确认)

## 邻居建立状态机 (Neighbor States)
### Down / Attempt：无 Hello 交互
### Init：收到单向 Hello
### 2-Way：双向通信确立 (此时进行 DR/BDR 选举)
### ExStart：主从关系确立
### Exchange：摘要信息(DBD)交换
### Loading：请求(LSR)与更新(LSU)缺失路由
### Full：数据库(LSDB)完全同步

## DR / BDR 选举机制
### 目的：在多路访问网络中减少邻接关系数量，控制洪泛
### 选举条件：按接口优先级 (默认1，0不参与) -> Router ID 比较
### 特性：非抢占式，一旦选举完成不主动让位

## OSPF 网络类型 (Network Types)
### Broadcast：以太网默认，需 DR/BDR，Hello=10s
### Point-to-Point：串行链路默认，无需 DR，Hello=10s
### Loopback：环回口默认，强制以 /32 主机路由宣告

## 核心配置与优化
### 启用方式：network 命令网段宣告 vs 接口下直接启用
### 安全控制：passive-interface 停止收发 Hello，防止恶意注入
### 默认路由注入：default-information originate
### 开销调优 (Cost)：基于 100 Mbps / 带宽，需统一修改 auto-cost reference-bandwidth 以适配高速链路