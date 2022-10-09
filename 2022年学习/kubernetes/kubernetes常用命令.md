1、创建一次deployment部署

kubectl create deployment nginx --image=nginx

kubectl expose deployment nginx --port=80 --type=NodePort

2 # 查看Nginx的pod和service信息

 kubectl get pod,svc -o wide

docker ps | grep -v NAME | awk '{print $1}' | while read cid; do echo $cid; docker inspect -f {{.State.Pid}} $cid; done  定位异常容器

```
strace -o output.txt -T -tt -e trace=all -p 28979 追踪进程系统调用

```

3、创建service

kubectl apply -f a.yaml

4、查看路由表

route -n       

ip route 

5、查看标签

kubectl get pod --show-lebels

6、修改标签

kubectl label pods pod app=nginx

7、查看service代理的几个pod

kubectl get endpoints ws 

journalctl -u kubelet -n 1000 查看kubelet日志
