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

18. 

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
   kubectl get pod --all-namespaces
   -n kube-system
28. -o yaml模式