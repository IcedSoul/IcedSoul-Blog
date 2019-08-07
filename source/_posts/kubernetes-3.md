title: 在k8s集群上运行你的应用

tags:

- Kubernetes

- Pod

- Deployment

- Service

  categories: Kubernetes
---
## 前言

在成功搭建好k8s集群后，我们接下来先运行一个简单的应用。在这个过程中了解一下把一个项目部署至k8s集群的过程。

## 系统准备

首先，我们所准备的系统也很简单，是我之前用spring boot写的一个很简单的留言板系统。你也可以用自己开发的系统来代替，不过前提是可以进行容器化改造并且你对这个过程比较熟悉，那么可以参考下面的说明来把自己的系统打包成镜像。

当然，你也可以直接使用我的系统来部署，下面给出的配置文件在项目中都有，可以对照作为参考。

项目地址：[LeaveWord](https://github.com/IcedSoul/LeaveWord/tree/k8s)

这个项目之前只是一个非常简单的Spring Boot项目，需要使用MySQL数据库。它根本算不上一个微服务系统，但是这不重要，我们就以这个项目为例来将其部署至k8s集群。

首先，先把leaveword的数据库链接地址修改为：

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://leaveword-mysql-service:3306/leaveword?characterEncoding=utf8&useSSL=false
    username: root
    password: root
```

其中，leaveword-mysql-service为我们等会儿要为leaveword新建的mysql的service名字，用户名和密码都为默认值。

接下来，就需要把项目打包为镜像然后上传至DockerHub或者私有镜像仓库了，此处我选择将其上传至DockerHub。

首先，添加Dockerfile以构建镜像

```dockerfile
FROM java:8-jre

ADD ./target/leaveword-0.0.1-SNAPSHOT.jar /app/
CMD ["java", "-Xmx200m", "-jar", "/app/leaveword-0.0.1-SNAPSHOT.jar"]

EXPOSE 8081
```

注意，上面jar包名字和路径都应以自己的项目为准。

接下来，执行maven打包命令生成jar包。

```shell
mvn clean package -DskipTests
```

接下来就是使用docker打包并上传镜像：

```shell
# 注意-t后面为[DockerHub UserName/Repository Name:Version],注意修改为自己的，且不要忘记最后的 . 
docker build -t icedsoul/leaveword:latest .
# 登录自己的DockerHub账号，没有需要先去注册
docker login

# 把镜像上传至DockerHub仓库
docker push icedsoul/leaveword:latest
```

## 项目部署运行

在准备好镜像之后，我们可以开始准备在集群上部署此项目。在第一篇博客中我们已经了解到Pod和各种常见的Controller的概念，首先我们来尝试一下使用最常见的Deployment来创建Pod。

### 创建Mysql的Pod和Service

根据上面的需求，我们需要创建的服务一共有两个，一个是Mysql，一个是LeaveWord，我们先来通过Deplyment创建Mysql，在Master节点上创建一个文件。

```shell
mkdir leaveword
vi leaveword/leaveword-mysql-deployment.yaml
```

复制粘贴以下内容：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: leaveword-mysql
spec:
    replicas: 1
    strategy:
        type: Recreate
    template:
        metadata:
            labels:
                database: leaveword-mysql
        spec:
            containers:
            - args:
              - --character-set-server=utf8mb4
              - --collation-server=utf8mb4_unicode_ci
              env:
              - name: MYSQL_DATABASE
                value: leaveword
              - name: MYSQL_ROOT_PASSWORD
                value: root
              image: mysql:5.6
              name: leaveword-mysql
              ports:
              - containerPort: 3306
```

以上就是deployment文件的格式，其具体含义在下面我们会一一解释，我们先根据这个文件创建对应的Pod。其中指定了Pod的副本数量，容器，镜像，启动参数等内容。

```shell
kubectl create -f leaveword/leaveword-mysql-deployment.yaml
```

稍等片刻，使用

```shell
kubectl get pods
```

查看Pod状态，如果正常会显示以下内容：

```tex
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
leaveword-mysql   1/1     1            1           4m10s
```

如果没有显示READY说明正在拉取镜像，不用着急，稍等几分钟即可。

但是只有Pod并不能对外提供服务，我们还需要定义一下Service才能让其它服务能够访问Pod。

```shell
vi leaveword/leaveword-mysql-service.yaml
```

复制粘贴一下内容：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: group-mysql
spec:
  ports:
    - name: "3306"
      port: 3306
      targetPort: 3306
  selector:
    database: leaveword-mysql
```

标签同样之后再解释，这个文件定义了service的name，关联的pod，开放的端口等等。接下来创建对应的Service：

```
kubectl create -f leaveword/leaveword-mysql-service.yaml
```

然后执行：

```shell
kubectl get services
```

查看启动的service的状态。

### 创建Leaveword的Pod和Service

mysql的Pod和Service启动之后，我们也把leaveword的Pod给启动起来：

```shell
vi leaveword/leaveword-deployment.yaml
```

复制粘贴一下内容：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: leaveword
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: leaveword
    spec:
      containers:
        - image: icedsoul/leaveword
          name: leaveword
          ports:
            - containerPort: 8081
      restartPolicy: Always
```

接下来创建Pod：

```shell
kubectl create -f leaveword/leaveword-deployment.yaml
```

稍等几分钟后，查看pod即可看到所有Pod正常运行：

```shell
[root@master ~]# kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
leaveword-6d686d9889-f55kq         1/1     Running   0          22s
leaveword-6d686d9889-khg95         1/1     Running   0          22s
leaveword-6d686d9889-ptbgp         1/1     Running   0          22s
leaveword-mysql-57f7c65b77-kth9n   1/1     Running   0          63m
```

之后，继续创建leaveword对应的Service：

```shell
vi leaveword/leaveword-service.yaml
```

然后粘贴以下内容：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: leaveword-service
spec:
  type: NodePort
  ports:
    - name: "8081"
      port: 8081
      targetPort: 8081
      nodePort: 30001
  selector:
    app: leaveword
```

接下来，执行创建命令：

```shell
kubectl create -f leaveword/leaveword-service.yaml 
```

查看所有Service：

```shell
[root@master ~]# kubectl get service
NAME                      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kubernetes                ClusterIP   10.96.0.1      <none>        443/TCP          25h
leaveword-mysql-service   ClusterIP   10.99.108.17   <none>        3306/TCP         59m
leaveword-service         NodePort    10.107.6.254   <none>        8081:30001/TCP   7s
```

其中kubernetes是集群的服务，leaveword-mysql-service和leave-service是我们所创建的的服务，并且我们可以看到leaveword-service端口暴露在了30001端口。我们可以通过集群Masterde IP+30001来访问主页：

![01](http://img.icedsoul.cn/img/blog/kubernetes03/1.png)

到此为止，我们成功把一个单体项目和MySQL数据库部署到了k8s集群，并且成功访问到了项目。

但是Deployment的参数代表着什么意思，可以做怎样的调整，Service的文件的参数又是什么意思，又是怎么和Deployment创建的Pod关联起来的？

下面我们先来仔细看一下Deployment，慢慢理解上面我们的配置文件为什么要那样子写～

## Deployment

我们已经知道，Deployment是k8s集群中用来管理Pod生命周期的一种Controller，也是我们最经常使用的一种Controller，我们上面所创建的leaveword-mysql-deployment.yaml以及leaveword-deployment.yaml就是两个Deployment，用于创建对应的Pod。

Deployment可以直接在命令行创建，如直接运行：

```shell
kubectl run nginx-deployment --image=nginx1.7.9 --replicas=2
```

就可以直接创建一个名称为nginx-deployment，镜像为nginx，副本数量为2的Pod。

但是当参数比较多时，命令行写参数可读性很差，且不易保存。所以我们一般会采用配置文件的方式来定义和创建Deployment，也就是我们上面所使用的方式。接下来，我们以上面的两个Deployment文件为例来说明一下各项配置的作用，建议大家配合官方文档[3]理解：

leaveword-mysql-deployment.yaml

```yaml
# 见下方说明1
apiVersion: apps/v1
# 见下方说明2
kind: Deployment
# 里面可以有名字，注释等内容，也可以定义label
metadata:
    name: leaveword-mysql
# Deployment资源的规格说明
spec:
	# 见下方说明3
    replicas: 1
    # 见下方说明4
    strategy:
        type: Recreate
    # 定义Pod的模板
    template:
    	# 见下方说明5
        metadata:
            labels:
                database: leaveword-mysql
        # Pod的规格说明
        spec:
            containers:
            # 容器启动时的附加参数，此处设置了mysql的字符集
            - args:
              - --character-set-server=utf8mb4
              - --collation-server=utf8mb4_unicode_ci
              # 容器启动时的环境变量，此处设置了新建数据库的名字和连接数据库的密码。
              env:
              - name: MYSQL_DATABASE
                value: leaveword
              - name: MYSQL_ROOT_PASSWORD
                value: root
              # 见下方说明6
              image: mysql:5.6
              name: leaveword-mysql
              # 见下方说明7
              ports:
              - containerPort: 3306
```

说明：

1. apiVersion是当前资源所在的api版本。alpha表示可能会包含错误，可能会随时丢弃；beta表示经过测试，功能正常。细节可能会改变，但是功能不会被删除。;stable表示稳定版本。

   大概就是一些资源可能刚开始在beta，但是后来慢慢会被加入到stable。比如Deployment就经历了：

   1.6之前：extensions/v1beta1

   1.6~1.9：apps/v1beta1

   1.9后：apps/v1

   所以我们现在使用Deployment一般使用apps/v1，偶尔看到extensions/v1beta1或者apps/v1beta1的写法也不要太疑惑，因为这三种写法目前都还能用，但是还是建议大家用稳定版本的即apps/v1，这也是官方推荐写法。

2.  kind后面跟随的是资源的类型，我们在上面新建了deployment和service，实际上还有其它类型的资源，就考kind后面跟随的类型名称来区分。

3. replicas是控制副本数量的，比如上面的leaveword-mysql replica为1，那么只会开启一个Pod副本，而leaveword的replicas为3，则开启了三个Pod副本，而且集群会自动监控副本数量，如果小于3则会自动创建新的副本，保证数量始终为文件定义的数量。

4. 这个规定的是该deployment的升级策略，Recreate这种策略是一种非常简单的策略，旧的Pod直接被杀死，新的取代旧的。但是这个参数可以被设置为其它类型以满足各种情况下的需要，具体可见参考资料[4]。

5. 这里的metadata仍然是元数据，经常被用来定义labels，label的键值都可以自己设置。用途是为了完成Service和Pod的绑定，根据label就可以筛选出对应的Pod，比如这里的label设置为leaveword-mysql，那么leaveword-mysql-service中selector就可以设置为同样的key:value，这样就能够完成Pod和Service的绑定，而且即使是一对一，一对多，多对多都可以绑定。

6. image是镜像的名字，Pod启动时，会根据image去拉取相关镜像，比较重要。

7. 设置容器内暴露的端口。注意，这里的端口并不是外界可以访问的端口，即便是集群内部也是无法直接访问的，这个端口仅仅在容器内暴露，原则上来说Pod副本IP+这个端口号可以访问，但是因为Pod IP是动态生成的，而且每个副本的IP也不一样，所以我们一般借助Service定义的端口来进行访问。

下面我们再看一下leaveword的deployment定义文件：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: leaveword
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: leaveword
    spec:
      containers:
        - image: icedsoul/leaveword
          name: leaveword
          ports:
            - containerPort: 8081
      restartPolicy: Always
```

基本和上面类似，最后多了一个restartPolicy是指重启策略，字面意思，应该很容易理解。

除了上面的内容之外，deployment还可以有很多其它的标签，具体请以[3]为准。

## Service

接下来我们来看一下Service的定义文件：

leaveword-mysql-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: group-mysql
spec:
  ports:
    - name: "3306"
      port: 3306
      targetPort: 3306
  selector:
    database: leaveword-mysql
```

leaveword-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: leaveword-service
spec:
  type: NodePort
  ports:
    - name: "8081"
      port: 8081
      targetPort: 8081
      nodePort: 30001
  selector:
    app: leaveword
```



apiVersion、kind、metadata、spec意义都同上，主力需要注意的一点是端口，我们可以注意到的是ports这个标签的不同，leaveword-mysql-service有两个端口号，leaveword-service有三个端口号，接下来我们详细看一下ports这个标签。

ports一共可以定义三种端口：

- nodePort：暴露给集群外部的端口，比如leaveword-service我们需要在集群外访问这个服务，那么就在这里设置nodePort来暴露端口，而且上面需要添加type: NodePort。但是，外部流量访问集群不只有这一种方式，这也是一种比较有风险的方式，因为是实验系统简单起见所以如此配置。上面的leaveword-mysql-service因为无需对集群外暴露端口，所以不需要设置nodePort。
- port：暴露给集群内部其它服务的端口号，设置了这个端口那么集群内其它服务就可以通过http://[service-name]:[port]的方式来访问这个服务对应的Pod。上面的两个服务都设置了这个端口。

- targetPort：对应于Deployment中容器的端口号，它的含义为kube-proxy会把来自于nodePort和port的流量转发到容器的targetPort，这个应该与定义Pod时设置的容器内应用暴露的端口号一致，否则会转发失败，导致报错。

最后值得一提的一个标签是selector，这个就是用于service和pod的绑定，比如leaveword-mysql-service中的spec.selector与leaveword-mysql-deployment的spec.template.metadata.labels保持一致，leaveword-service的spec.selector与leaveword-deployment的spec.template.metadata.labels保持一致。并且，selector和labels都可以设置多个，可以很方便的进行多对多的映射。

## 总结

这篇博客主要内容是把一个简单的应用部署到了k8s集群上运行，并且对使用到的deployment和service的配置文件进行了简单的介绍。下一篇博客我们会专门深入Controller，来分别尝试一下Deployment、DaemonSet、NodeSelector、NodeAffinity、PodAffinity、Taints、Tolerations、Job、自定义调度器等，来看一下他们分别有什么特点，在什么场景下比较适合。

## 参考资料

1. 《Kubernetes 权威指南 第四版 从Docker到Kubernetes实践全接触》 龚正 等 中国工信出版社
2. 《每天五分钟 玩转Kubernetes》 CloudMan 著 清华大学出版社
3. Kubernetes Deployment：https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
4. Kubernetes Deployment Strategies：https://www.weave.works/blog/kubernetes-deployment-strategies

