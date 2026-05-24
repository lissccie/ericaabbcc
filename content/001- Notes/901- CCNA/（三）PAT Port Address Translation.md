
NAT Overload




![[Pasted image 20260315165650.png]]




![[Pasted image 20260317095328.png]]



![[Pasted image 20260317095528.png]]


![[Pasted image 20260317101523.png]]

![[Pasted image 20260317101500.png]]







下面这个配置，R1 ping 4.4.4.4  成功的时候，R2 就不通。 反之亦然。
因为是Dynamic NAT, 而且只有一个公网地址。

![[Pasted image 20260317103600.png]]



下面这个配置，R1 ping 4.4.4.4  成功的时候，R2 也可以通。
因为有关键词 overload，是PAT。


![[Pasted image 20260317103853.png]]