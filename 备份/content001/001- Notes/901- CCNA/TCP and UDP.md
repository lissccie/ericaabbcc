


1. TCP:
	. error recovery
	. flow control


2. TCP segment 又叫 L4PDU

3. TCP Header
![[Pasted image 20260317112058.png]]


4. Multiplexing Using TCP Port Numbers

![[Pasted image 20260317113313.png]]


5. TCP/UDP 端口号（Port Numbers）

    (1) 熟知端口 / 系统端口 (Well Known / System Ports)
      0 -1023

    (2) 注册端口 / 用户端口 (User / Registered Ports)
      1024 - 49151

	(3) 动态端口 / 临时端口 (Ephemeral / Dynamic / Private Ports)
	  49152 - 65535


6. 为什么user这一端的web brower port 不是80？


![[Pasted image 20260317114413.png]]

这是一个非常经典的问题，也是理解 **网络多路复用（Multiplexing）** 的关键！

简单来说：**80 端口是“服务窗口号”，而 User 端的端口是“排队票号”。**

### 1. 角色不同：监听 vs 发起

- **Server（服务端）使用 80 端口**：因为它需要在一个**固定、已知**的地方等待别人来访问。就像 119 是火警电话，全世界都知道打这个号能找到消防队。如果 Web 服务器的端口经常变，别人就找不到网页了。
    
- **User（用户端）使用 49152 端口**：用户端不需要别人主动来找它，它是**主动发起请求**的一方。它从“动态端口（Ephemeral Ports）”范围里随便抓一个没被占用的号（比如 49152）来用。
    

### 2. 为了支持“同时开多个网页”

想象一下，如果你（User）这一端的端口也固定死必须是 80：

- 当你打开第一个网页（百度）时，占用了 80 端口。
    
- 当你同时想打开第二个网页（谷歌）时，你的 80 端口已经被占用了，第二个网页就打不开了。
    

**正是因为 User 端使用的是随机的动态端口，你才能在浏览器里同时打开几十个标签页而互不干扰。**

---

### 3. 如何区分不同的通信（Socket 五元组）

计算机靠五个元素来唯一确定一个连接，只要其中一个不同，就是不同的通信：

$$源 IP + 源端口 + 目的 IP + 目的端口 + 协议$$

看你图片里的例子（最右边那条黄线）：

- **User 发出的包：** $(10.1.1.1, TCP, 49152) \rightarrow (10.1.1.2, TCP, 80)$
    
- **Server 回来的包：** $(10.1.1.2, TCP, 80) \rightarrow (10.1.1.1, TCP, 49152)$
    

因为 Server 记住了你的“临时票号”是 **49152**，所以它把数据送回来时，你的电脑一看端口号，就知道：“哦！这是给 Web 浏览器的数据，不是给 Skype 的。”

### 总结

- **Server 端端口 (80)**：是固定的，为了让别人**能找到我**。
    
- **User 端端口 (49152)**：是随机的，为了让自己的不同程序**能同时上网**且互不干扰。
    



7. 常用端口号

![[Pasted image 20260317114900.png]]

![[Pasted image 20260317114913.png]]



8. TCP 三次握手  three-way handshake

![[Pasted image 20260317115341.png]]

![[Pasted image 20260317115351.png]]




9.  UDP Header
![[Pasted image 20260317130700.png]]
























