#### kubernetes 网络模型

**kubernetes 使用的网络方案其实类似flannel + docker实现的方式，只是将docker0网桥换成了CNI0网桥**

**CNI设计思想：Kubernetes 在启动 Infra 容器之后，就可以直接调用 CNI 网络插件，为这个 Infra 容器的 Network Namespace，配置符合预期的网络栈。**

<img src="https://static001.geekbang.org/resource/image/9f/8c/9f11d8716f6d895ff6d1c813d460488c.jpg" alt="img" style="zoom:50%;" />





#### flannel在kubernetes实现网络方案

- 实现网络本身，也就是flanneld这些逻辑
- 实现该网络对应的CNI插件，flannel已经内置了，如果没有内置，需要实现自己的CNI，并且放在 /opt/cni/bin 目录下



#### CNI插件种类（ /opt/cni/bin ）

- Main插件，是用来创建具体网络设备的二进制文件，如bridge（网桥设备）、ipvlan、loopback（lo 设备）、macvlan、ptp（Veth Pair 设备），以及 vlan。
- IPAM插件，是负责分配ip地址的二进制文件
- 由社区维护的内置CNI插件，如flannel，就是专门为 Flannel 项目提供的 CNI 插件；tuning，是一个通过 sysctl 调整网络设备参数的二进制文件；portmap，是一个通过 iptables 配置端口映射的二进制文件；bandwidth，是一个使用 Token Bucket Filter (TBF) 来进行限流的二进制文件。



#### CNI配置网络栈步骤

1. flanneld启动后会在每个宿主机上生成CNI配置文件（/etc/cni/net.d/10-flannel.conflist ），用来告诉kubernetes要使用flannel网络插件，其实是个configMap
2. Pod创建后，dockershim会先创建并启动一个Infra容器，紧接着执行SetUpPod方法，该方法的作用是为CNI插件准备参数，然后调用CNI插件为Infra容器配置网络
3. 为CNI准备插件
   1. 由dockershim设置的一组CNI参数
4. CRI开始加载CNI配置文件，如果容器使用的docker，CRI就是dockershim（k8s不支持多个网络插件，如果有多个CNI，会按照字母排序，加载第一个）

   

