# helm-hello-world

## Source

https://medium.com/@pablorsk/kubernetes-helm-node-hello-world-c97d20437abd

## 安装与准备 （前提条件）

1. 安装Docker （忽略）

2. 安装K8s（忽略）

3. 安装helm（忽略）

## 创建 App 及其 镜像

### 1.创建网页文件

app/index.html 内容如下

```html
<h1>Helm Hello world!</h1>
```

### 2. 创建Docker文件

Dockerfile 内容如下

```dockerfile
FROM httpd
ADD app/index.html /usr/local/apache2/htdocs/index.html
EXPOSE 80
```

### 3. 构建Docker镜像

```shell
docker build -t zknyy/helm-hello-world:latest .
```



### 4. 推送镜像

```shell
docker login
docker push zknyy/helm-hello-world:latest
```

得到

```
The push refers to repository [docker.io/zknyy/helm-hello-world]
dbd4b105d5ac: Preparing
5d727ac94391: Preparing
4cfc2b1d3e90: Preparing
484fa8d4774f: Preparing
ca9ad7f0ab91: Preparing
13cb14c2acd3: Preparing
13cb14c2acd3: Waiting
484fa8d4774f: Mounted from zknyy/httpd-docker
4cfc2b1d3e90: Mounted from zknyy/httpd-docker
5d727ac94391: Mounted from zknyy/httpd-docker
dbd4b105d5ac: Pushed
13cb14c2acd3: Mounted from zknyy/httpd-docker
ca9ad7f0ab91: Pushed
latest: digest: sha256:b3fab9a5d020b5920b48c9136eb103d1c99e7914738d1b094069b7c1074f4657 size: 1574
```

### 5. 运行镜像

```shell
docker run -d -p 80:80 --name helm-hello-world zknyy/helm-hello-world
## open your browser and check http://localhost/
```

![](https://raw.githubusercontent.com/zknyy/helm-hello-world/master/pics/helm-hello-world-docker-rum.png)

## 构建并安装Chart

### 1. 创建Chart

```shell
helm create helloworld-chart
```

### 2. 编辑

修改 **values.yaml** 文件

```yaml
image:
  repository: zknyy/helm-hello-world
  tag: "latest"
  pullPolicy: IfNotPresent
```

### 3. 打包 (非必要步骤)

```shell
helm package helloworld-chart --debug
## helloworld-chart-0.1.0.tgz file was created
```

> 如果是Helm v2 需要用下面这种方法来对 release 命名

```shell
helm install helloworld-chart-0.1.0.tgz --name helm-hello-world
```

### 4. 发布

```shell
# 如果打包可以用文件安装
helm install helm-hello-world helloworld-chart-0.1.0.tgz
# 如果没有执行打包，可以直接通过目录安装
helm install helm-hello-world ./helloworld-chart/
```

得到输出：

```
NAME: helm-hello-world
LAST DEPLOYED: Wed Jun 10 15:25:36 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=helloworld-chart,app.kubernetes.io/instance=helm-hello-world" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:80
```

### 5. 通过helm查看运行状态

```shell
helm ls
```

得到

```
NAME              NAMESPACE   REVISION  UPDATED                                 STATUS     CHART                   APP VERSION
helm-hello-world  default     1         2020-06-10 15:25:36.5310553 +0800 CST   deployed   helloworld-chart-0.1.0  1.16.0
```



### 6. 本地访问

#### 如果是在Windows下：

先设置pod的名称变量

```shell
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=helloworld-chart,app.kubernetes.io/instance=helm-hello-world" -o jsonpath="{.items[0].metadata.name}")
```

> 如果是windows系统 用  kubectl get pods --namespace default  然后找到 列表中的  helm-hello-world-helloworld-chart-7755694dc9-m6fk9

查看pod

```shell
kubectl get pod $POD_NAME
```

得到

```
NAME                                                 READY   STATUS    RESTARTS   AGE
helm-hello-world-helloworld-chart-7755694dc9-rpbf6   0/1     Running   2          2m42s
```

然后映射端口到本地

```shell
kubectl --namespace default port-forward $POD_NAME 8080:80
```

#### 如果在Linux或MacOS下

可以直接执行

```shell
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=helloworld-chart,app.kubernetes.io/instance=helm-hello-world" -o jsonpath="{.items[0].metadata.name}")
echo "Visit http://127.0.0.1:8080 to use your application"
kubectl --namespace default port-forward $POD_NAME 8080:80
```

#### 

### 补充说明

如果遇到问题用log查看详细

```shell
kubectl logs pod/$POD_NAME
kubectl get event
kubectl logs <pod_name> -c <container_name> #类似tail -f的方式查看(tail -f 实时查看日志文件 tail -f 日志文件log)
```

如果K8s集群能够通过外部IP访问可以进行如下设置

```yaml
  type: LoadBalancer
  externalPort: 80
  internalPort: 8005	
```

发布成功后可以用命令查看IP进行访问

```shell
kubectl get svc --watch # wait for a IP
```

