- 注：master和node节点都要执行，可以在xshell里设置查看->撰写栏，所有机器一起执行

- 使用的cenos7.8

  

  1、关闭防火墙

  - systemctl stop firewalld
  - systemctl disable firewalld

  2、关闭 selinux

  - sed -i 's/enforcing/disabled/' /etc/selinux/config # 永久关闭
  - setenforce 0 # 临时关闭

  3、关闭 swap

  - swapoff -a # 临时关闭
  - vim /etc/fstab # 永久关闭   #注释掉swap这行  /dev/mapper/centos‐swap swap swap defaults 0 0
    - 没有这行就别管了，要不乱注释了会导致系统文件出问题
  - systemctl reboot #重启生效
  - free -m #查看下swap交换区是否都为0，如果都为0则swap关闭成功

  4、给三台机器分别设置主机名

  - hostnamectl set-hostname <hostname>
    - 第一台：k8s‐master
    - 第二台：k8s‐node1
    - 第三台：k8s‐node2

  5、在 k8sr机器添加hosts，执行如下命令，ip需要修改成你自己机器的ip，不行的话把内网也加上

  - cat >> /etc/hosts << EOF
    152.136.28.164 k8s-master
    152.136.228.135 k8s-node1
    120.53.224.150 k8s-node2
    EOF

  6、将桥接的IPv4流量传递到iptables

  - cat > /etc/sysctl.d/k8s.conf << EOF
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    EOF
  - sysctl --system # 生效

  7、设置时间同步

  - yum install ntpdate -y   centos8以上替换为：yum install chrony
  - ntpdate time.windows.com  centos8以上替换为：chronyc sources -v

  8、添加k8s yum源

  - cat > /etc/yum.repos.d/kubernetes.repo << EOF
    [kubernetes]
    name=Kubernetes
    baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=0
    repo_gpgcheck=0
    gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
    EOF

  9、如果之前安装过k8s，先卸载旧版本

  - yum remove -y kubelet kubeadm kubectl

  10、查看可以安装的版本 -  centos8 一直卡死不知道为啥

  - yum list kubelet --showduplicates | sort -r

  11、安装kubelet、kubeadm、kubectl 指定版本，我们使用kubeadm方式安装k8s集群

  - yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0

  12、开机启动kubelet

  - systemctl enable kubelet
  - systemctl start kubelet

  13、在master机器上执行：

  - 执行初始化操作（第一个ip是master节点ip（是内网ip，否则坑死人），版本号根据自己安装的版本填）

    kubeadm init --apiserver-advertise-address=10.0.16.15 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.18.0 --service-cidr=10.96.0.0/12 --pod-network-cidr=10.244.0.0/16

    - 可能存在的问题：[ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1   解决：sysctl -w net.ipv4.ip_forward=1

    - ```text
      Initial timeout of 40s passed.
      
              Unfortunately, an error has occurred:
                      timed out waiting for the condition
      ```

    - 上面这个报错有可能是ip用的不是内网ip

    - 如果失败了需要重新初始化：kubeadm reset 清理数据

      - **kubeadm reset 执行后不会删除$HOME/.kube文件，执行rm -rf $HOME/.kube

  - 配置使用 kubectl 命令工具(类似docker这个命令)

    mkdir -p $HOME/.kube

    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

  - 查看kubectl是否能正常使用

    kubectl get nodes

  - 安装 Pod 网络插件

    kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

  - 如果上面这个calico网络插件安装不成功可以试下下面这个

    kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kubeflannel.yml
    

  

14、在node节点上执行：

- 将node节点加入进master节点的集群里
  

kubeadm join 10.0.16.15:6443 --token k8rx9g.llzbvs29p28kuhru \
      --discovery-token-ca-cert-hash sha256:6d2330423598d9885b4911c9b1287de9931f25f6f36ad603f75bc17d40d020f2

- 遇到的问题（https://blog.csdn.net/qq_33996921/article/details/103529312）
    - Failed to request cluster-info, will try again: Get https://10.0.16.15:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers) 报错 ，先看看是不是node和master网络不通，如果外网telnet不通，如果用的云服务器，可能是端口没有对外暴露，需要云服务器管理界面添加，如果内网ping不通，执行 iptables -t nat -A OUTPUT -d 10.0.16.15 -j DNAT --to-destination 152.136.28.164（第一个ip是master内网ip，第二个是外网ip）

  

  

  

  
