## 使用minikube 

minikube.exe start   --driver=docker   --alsologtostderr    --image-mirror-country='cn'   --registry-mirror=https://registry.docker-cn.com --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers

 --registry-mirror=https://registry.docker-cn.com 

 minikube start --driver=hyperv 

1. 创建一个名为kubia的replicationController

kubectl run kubia --image=luksa/kubia --port=8080 -generator=run/v1

2. 为replicationController暴漏服务

kubectl expose rc kubia --type=LoadBalancer --name kubia-http

3. 查看服务等内容

   kubectl get/describe pod/node/service/rc

4. 暴露服务

   minikube service kubia-http
   
5. kubectl scale rc kubia --replicas=3

6. 查看pod的ip和对应的node
    kubectl get pods -o wide

7. 查看pod 的yaml文件
    kubectl get po kubia-j6pqv -o yaml

8. 查看pods的定义
    kubectl explain pods 

9. 创建pod
    kubectl create -f kubia-manual.yaml

10. 看日志
    kubectl logs kubia-manual -c 【容器】

11. 本地端口转发
    kubectl port-forward kubia-manual 8888:8080

12. 查看pod带有的标签
    kubectl get po --show-labels

13. 查看具体的某些标签
    kubectl get po -L createion_method,env

14. 修改pod的标签
    kubectl label po/node kubia-manual 【--overwrite】

15. 过滤标签得到pod
    kubectl get po -l 

16. 添加注解信息
    kubectl annotate pod kubia-manual mycompany.com/somannotation="foo bar"
    kubectl describe pod

17. 获得所有的命名空间
    kubectl get ns
    kubectl get po --namespace kube-system

19. 删除pod
    kubectl delete ns custom-namespace
    kubectl delete po [-l] kubia

20. 删除命名空间中几乎所有的资源
    kubectl delete all --all

21. 查看前一次容器的日志（一般用于容器重新创建）
    kubectl logs mypod --previous

22. 编辑yaml模板内容
    kubectl edit rc kubia

23. 删除rc但是不删除pod
    kubectl delete rc kubia --cascade=false

24. 进入pod执行命令
    kubectl exec kubia-9ml2v  -- curl -s http://10.96.138.237

25. 进入容器的内部
    kubectl exec -it kubia-jnxzb  bash

26. headless和stafulset的结合作用

    能够发现集群的pod准确ip
    
27. 获取所有的命名空间下的pod
      kubectl get pod --all-namespaces【 kubectl get pod -A】
      -n kube-system
    
27. -o yaml模式

28. 创建configmaping命令

     kubectl create configmap fortune-config --from-literal=sleep-interval=25 --from-literal=sleepl=sleep

     --from-file=（文件夹或者文件）


	kubectl create -f *.yaml 创建一个资源
	kubectl replace -f *.yaml 替换一个资源
	kubectl apply -f *.yaml 替换一个资源
	kubectl edit 
	kubectl los -f pod
	kubectl create deployment nginx --image=nginx:1.15.2
### 基础概念篇
#### Master节点
几个主要组件
1. API Server 集群控制的中枢，唯一一个和etcd交互的，各个组件通过API Server来进行交互，包含了鉴权等内容
2. Controller Manager 集群状态管理器，控制面板，控制pod的状态数量 等内容
3. Scheduler调度器，调度pod到哪个node上，根据策略选择
4. Etcd 键值存储高性能 在ssd硬盘上能体现较好的优势

#### Node节点
1. kubectl ，上报node和pod的状态信息
2. kubectl proxy 负责Pod之间的通信和负载均衡，将指定的流量分发到后端正确的机器上。
    - 查看Kube-proxy工作模式：curl 127.0.0.1:10249/proxyMode
    - Ipvs：监听Master节点增加和删除service以及endpoint的消息，调用Netlink接口创建相应的IPVS规则。通过IPVS规则，将流量转发至相应的Pod上。
    - Iptables：监听Master节点增加和删除service以及endpoint的消息，对于每一个Service，他都会场景一个iptables规则，将service的clusterIP代理到后端对应的Pod。
其他组件
 - Calico：符合CNI标准的网络插件，给每个Pod生成一个唯一的IP地址，并且把每个节点当做一个路由器。Cilium
- CoreDNS：用于Kubernetes集群内部Service的解析，可以让Pod把Service名称解析成IP地址，然后通过Service的IP地址进行连接到对应的应用上。
- Docker：容器引擎，负责对容器的管理。

#### POD
#####  定义一个POD
#####  POD的探针
- StartupProbe：k8s1.16版本后新加的探测方式，用于判断容器内应用程序是否已经启动。如果配置了startupProbe，就会先禁止其他的探测，直到它成功为止，成功后将不在进行探测。
- LivenessProbe：用于探测容器是否运行，如果探测失败，kubelet会根据配置的重启策略进行相应的处理。若没有配置该探针，默认就是success。
- ReadinessProbe：一般用于探测容器内的程序是否健康，它的返回值如果为success，那么久代表这个容器已经完成启动，并且程序已经是可以接受流量的状态。
##### 探针的检测方式
- ExecAction：在容器内执行一个命令，如果返回值为0，则认为容器健康。
- TCPSocketAction：通过TCP连接检查容器内的端口是否是通的，如果是通的就认为容器健康。
- HTTPGetAction：通过应用程序暴露的API地址来检查程序是否是正常的，如果状态码为200~400之间，则认为容器健康。

### 资源管理
#### Deployment
1. 基础概念
2. 更新操作
	改动到spec.template才会生成一个新的rs
	- 更新新的镜像操作
		kubectl set image deploy nginx-dep nginx=nginx:1.15.4 --record
		如果新的镜像不行 则不会进行更新 要看配置的程度
	- 查看更新的状态
		kubectl rollout status deploy nginx-dep 或者 descirbe 
	- 查看所有的历史jilu
		kubectl rollout history deploy nginx-dep
3. 回滚操作
	- 回滚操作
		[root@k8s-master01 ~]# # 回滚到上一个版本
		[root@k8s-master01 ~]# kubectl rollout undo deploy nginx 
    - 查看版本详细信息
       [root@k8s-master01 ~]# # 查看指定版本的详细信息
       [root@k8s-master01 ~]# kubectl rollout history deploy nginx --revision=5
    - 回滚到指定版本 
       kubectl rollout undo deploy nginx --to-revision=5
4. 扩容镜像
    kubectl scale deploy nginx-dep --replicas=3 
5. 暂停和恢复
	- 暂停功能 针对set指令 用replace 和edit 
		[root@k8s-master01 ~]# # Deployment 暂停功能
		[root@k8s-master01 ~]# kubectl rollout pause deployment nginx 
		kubectl set resources deploy nginx  -c nginx --limits=cpu=100m,memory=100Mi --requests=cpu=50m,memory=56Mi
	- 回复功能
	kubectl rollout resume deploy nginx 
6. 注意的参数
- spec.revisionHistoryLimit：设置保留RS旧的revision的个数，设置为0的话，不保留历史数据
- spec.minReadySeconds：可选参数，指定新创建的Pod在没有任何容器崩溃的情况下视为Ready最小的秒数，默认为0，即一旦被创建就视为可用。
- 滚动更新的策略：
	- spec.strategy.type：更新deployment的方式，默认是RollingUpdate
	- RollingUpdate：默认滚动更新，可以指定maxSurge和maxUnavailable
		- maxUnavailable：指定在回滚或更新时最大不可用的Pod的数量，可选字段，默认25%，可以设置成数字或百分比，如果该值为0，那么maxSurge就不能0
		- maxSurge：可以超过期望值的最大Pod数，可选字段，默认为25%，可以设置成数字或百分比，如果该值为0，那么maxUnavailable不能为0
	- Recreate：重建，先删除旧的Pod，在创建新的Pod
#### StatefulSet
1. 常用于部署有状态的且需要有序启动的应用程序，启动是一个个启动，前面还没启动好不会往后处理
2. 删除一个StatefulSet时，不保证对Pod的终止，要在StatefulSet中实现Pod的有序和正常终止，可以在删除之前将StatefulSet的副本缩减为0。
3. 定义一个sts
4. 扩容一个sts kubectl scale sts web --replicas=3
5. 执行容器内部内容 kubectl exec -it busybox -- sh
6. 查看某个pod/service的地址 nslookup web-1.nginx 
	service会返回所有的pod内容
	/ # nslookup web-1.nginx
    Server:    10.96.0.10
    Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

    Name:      web-1.nginx
    Address 1: 172.18.195.9 web-1.nginx.default.svc.cluster.local
    / # nslookup nginx
    Server:    10.96.0.10
    Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

    Name:      nginx
    Address 1: 172.25.244.202 web-2.nginx.default.svc.cluster.local
    Address 2: 172.18.195.9 web-1.nginx.default.svc.cluster.local
    Address 3: 172.25.244.201 web-0.nginx.default.svc.cluster.local
7 -w是一个watch状态
8. 更新的策略，
	- RollingUpdate	，从数量大的往上更新，更新过程中如果0挂掉 要先启动0在进行更新
		大于partition数字的才更新
	- Ondelete  删除时候才更新 ，可以用于灰度更新
#### DaemonSet

### 常见问题
#### 网络方面
1. 网络具体如何通信，如何查看
查看ipvs的转发规则（linux 命令）
	nestat -lntp
	ipvsadmin -ln  配置了服务到pod 的转发内容
	route -n    配置服务到具体物理机的路由关系
	vim set paste的作用