




![[Pasted image 20260315165521.png]]



1.  Dynamic entry: 
2. clear dynamic entry: clear ip nat translation *
           
3. If a new packet arrives from yet another inside host, and it needs a NAT entry, but all the pooled IP addresses are in use, the router simply discards the packet.




![[Pasted image 20260315230027.png]]



![[Pasted image 20260315232334.png]]













Dynamic NAT中 pool的netmask是干什么用的？

在 `ip nat pool` 命令中，`netmask`（子网掩码）的主要作用是**告诉路由器这个公网地址池属于哪个网络范围**，并辅助路由器确定这些地址的**网络部分和主机部分**。

具体来说，它的功能体现在以下几个方面：

### 1. 定义地址池的边界

虽然你已经指定了 `first-address`（起始地址）和 `last-address`（结束地址），但 `netmask` 明确了这些地址在逻辑上所属的子网。这有助于 IOS 软件验证你输入的起始和结束地址是否合法（即它们是否处于同一个主网或子网内）。

### 2. 配合路由通告

这是最核心的实际用途。当路由器收到去往地址池中某个 IP 的数据包时，它需要知道如何处理。

- 如果地址池中的地址与外部接口（Outside Interface）处于**同一个子网**，路由器通常会通过代理 ARP（Proxy ARP）来响应对这些地址的请求。
    
- `netmask` 帮助路由器判断这些地址是本地直连子网的一部分，还是一个需要特殊路由指示的独立网段。
    

### 3. 语法要求

在 Cisco IOS 的 CLI 逻辑中，定义一段 IP 空间时，掩码是标准参数。它确保了地址池的定义符合标准的 IP 编址规范。

---

### 举例说明

假设你有一段从 ISP 申请到的公网地址：`202.100.1.1` 到 `202.100.1.10`，子网掩码是 `255.255.255.0`。

Bash

```
ip nat pool MY_POOL 202.100.1.1 202.100.1.10 netmask 255.255.255.0
```

- **如果不写 netmask：** 路由器将不知道这 10 个地址是属于一个大的 `/24` 网络，还是属于一个更小的子网。
    
- **替代方案：** 你也可以使用 `prefix-length` 来代替 `netmask`，效果是一样的： `ip nat pool MY_POOL 202.100.1.1 202.100.1.10 prefix-length 24`
    

### 常见疑问：它会限制转换后的地址吗？

**不会。** NAT 转换本身只看你定义的 `first-address` 到 `last-address` 这个范围。哪怕你的 `netmask` 定义的是 `/24`（254个可用地址），但如果你只写了 1.1 到 1.10，NAT 进程也只会使用这 10 个地址进行转换。










![[Pasted image 20260316132423.png]]



在这张截图中，`sh ip nat statistics` 输出中的 **Hits** 是一个非常关键的排错指标。

### 简单来说：

**Hits（命中）** 表示数据包通过路由器时，**成功匹配**到了现有的 NAT 转换表项（Translation Table）。

---

### 详细解释

当一个数据包进入路由器并触发 NAT 逻辑时，会发生以下两种情况：

- **Hits (命中):**
    
    - 这意味着路由器在内存的 NAT 表中**已经找到了**对应的映射关系。
        
    - 数据包不需要创建新的转换条目，直接按照现有的规则进行地址转换并转发。
        
    - **高 Hits 值**通常表示网络连接已经建立，并且数据流正在稳定传输。
        
- **Misses (未命中):**
    
    - 这意味着数据包在 NAT 表中**找不到**对应的映射。
        
    - 这时路由器必须去查找配置（比如你的 ACL 1 和地址池 TEST01），尝试创建一个**新的**转换条目。
        
    - 如果成功创建了新条目，这就叫一次 "Miss"；如果创建失败（比如地址池满了），Miss 也会增加，但转换不会成功。
        

---

### 从你的截图分析

从截图看，你的数据比较有意思：

- **Total translations: 0**: 当前 NAT 表里是空的。
    
- **Hits: 4210 / Misses: 4413**: 这说明之前有过大量的流量尝试通过。
    
- **Expired translations: 3320**: 说明有很多之前的转换连接因为超时已经被清理掉了。
    

**目前的状况：**

虽然你有 4210 次命中，但因为 `Total translations` 是 0，说明目前**没有任何活跃的连接**。之前的流量要么已经结束，要么因为某种原因（比如配置变动或清理命令）被清空了。

> **小贴士：** 如果你发现 Hits 完全不增加，而 Misses 一直在跳，通常说明数据包触发了 NAT 匹配，但路由器无法成功建立转换（例如：ACL 没匹配上，或者地址池 pool 里的地址被用光了）。





R1 telnet 4.4.4.4

![[Pasted image 20260316132602.png]]




