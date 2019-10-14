---
title: kubernetes Pod资源清单注解
categories: kubernetes
tags:
  - kubernetes
---


> 创建资源的方法：  
> 定义yaml格式提供配置清单,将资源清单提交给apiServer  
  apiServer可自动将其转换为json格式，而后提交给Scheduler(集群中的调度器)  
  由Scheduler完成调度，调度目标节点完成创建，并启动相关服务  


## Pod核心资源配置清单：
> 资源清单定义帮助 ``kubectl explain`` ``kubectl explain pods`` 等查看可嵌套字段  

### apiVersion
> 格式为group/version，所属群组版本  
支持的版本 ``kubectl api-versions`` 可查看  

### kind
> 定义资源类别，Pod,Service,Deployment,Event,Secret等(注意大小写)  

<!--more-->

### metadata 
> 元数据 ``kubectl explain pods.metadata`` 查看帮助  

```yaml
metadata:
    name: pod_name
    namespace: 名称空间  
    labels: -- 标签  
       key: value
    annotations: 资源注解
```
 

### spec   
> disired state 期望状态, ``kubectl explain pods.spec``查看帮助  

```yaml
spec:
    containers:
    - name: <string>
      image: <string>
      imagePullPolicy: <string> [IfNotPresent,Always,Never]
        # Always：不管镜像是否存在都会进行一次拉取。  
        # Never：不管镜像是否存在都不会进行拉取  
        # IfNotPresent：只有镜像不存在时，才会进行镜像拉取,默认为IfNotPresent，但:latest标签的镜像默认为Always
       
      ports: <[]Object>
        - name: <string>
          containerPort: <integer>
      command: <[]string>
      args: <[]string> # args将参数传给command
      
      livenessProbe: <Object> 
          # 存活探针,确定何时重启容器当应用程序处于运行状态但无法做进一步操作，
          # liveness探针将捕获到deadlock，重启处于该状态下的容器 
          # Kubernetes支持3种类型的应用健康检查动作，分别为HTTP Get、Container Exec和TCP Socket  
      readinessProbe: <Object>
          # 就绪探针,确定容器是否已经就绪可以接受流量,
          # 只有当Pod中的容器都处于就绪状态时kubelet才会认定该Pod处于就绪状态
          # 就绪状态, pod才会按照标签加入service  
      lifecycle:
          postStart: <Object> 
              # 容器创建成功后，运行前的任务，用于资源部署、环境准备等
          preStop: <Object> 
              # 在容器被终止前的任务，用于优雅关闭应用程序、通知其他系统等等
          
          # livenessProbe,readinessProbe,lifecycle[postStart,preStop] 中接受属性
          exec: <Object>
              command: <[]string>  
          httpGet: <Object>
              host: <string> # 连接的主机名，默认连接到pod的IP。你可能想在http header中设置”Host”而不是使用IP
              httpHeaders: <[]Object>  # 自定义请求的header。HTTP运行重复的header
              path: <string> # 访问的HTTP server的path
              port: <string> # 访问的容器的端口名字或者端口号。端口号必须介于1和65525之间
              scheme: <string> # 连接使用的schema，默认HTTP
          tcpSocket: <Object>
              host: <string>
              port: <string>
          # livenessProbe,readinessProbe 中接受属性
          initialDelaySeconds: <integer> # 容器启动后，等待多少秒之后进行第一次探测
          periodSeconds: <integer> # 执行探测的频率。默认是10秒，最小1秒
          successThreshold: <integer> # 探测失败后，最少连续探测成功多少次才被认定为成功。默认是1。对于liveness必须是1。最小值是1
          failureThreshold: <integer> # 探测成功后，最少连续探测失败多少次才被认定为失败。默认是3。最小值是1
          timeoutSeconds: <integer> # 探测超时时间。默认1秒，最小1秒
      
    nodeName: nodename 指定node节点
    nodeSelector: <map[string]string> 指定node的label标签
    restartPolicy: <string> Always, OnFailure, Never. Default to Always.
```




### status 
> 当前状态 current state(只读), 由kubernetes集群维护,不需要用户自己定义



## Demo Code

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: flask-app
  namespace: default
  labels:
    language: python
    frame: flask
spec:
  containers:
    - name: flaskapp
      image: sakuragaara/flaskapp:v1
      imagePullPolicy: Never
      ports:
        - name: http
          containerPort: 5000
      livenessProbe:
          tcpSocket:
              port: 5000
          timeoutSeconds: 3
  restartPolicy: OnFailure
```

***tcpSocket没有加host，主要是host默认为containers的IP，而flask启动是0.0.0.0，使用127.0.0.1监听会报错（不知道为什么）***
