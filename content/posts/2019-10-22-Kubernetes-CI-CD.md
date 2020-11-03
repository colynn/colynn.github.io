---
title: Kubernetes CI/CD 基于开源组件的实践
categories: ["教程"]
tags: ["kubernetes", "devops"]
date: 2019-10-22
lastmod: 2020-10-29
---

## 前言
应对敏捷开发的需求，对CI(持续集成))/CD（持续交付）的提出了更高的标准，今天来讨论下，如何基于开源组件（gitlab/jenkins/harbor/kubernetes）使用CI/CD，赋能团队的开发、运维。

## 核心组件

组件名称| 版本 | 备注 
------ | ------| ------|
kubernetes  | v1.15.3 |  10.0.0.182:6443 |
jenkins  | 2.176.2 | 集群内部署/ namespace: devops |
gitlab  | 11.8 | 主机部署 |
harbor  | v1.7.4 | docker-compose部署| 

## 基本流程

1. 在GitLab创建对应的项目。
2. 开发者将代码提交到GitLab。
3. Jenkins创建对应的任务（Job），集成该项目的Git地址和Kubernetes集群。
4. 如有配置钩子，推送（Push）代码会自动触发Jenkins构建，如没有配置钩子，需要手动构建。
5. Jenkins控制Kubernetes（使用的是Kubernetes插件）创建Jenkins Slave。
6. Jenkins Slave根据流水线（Pipeline）定义的步骤执行构建。
    1. 检出代码、打包、编译；
    2. 通过Dockerfile生成镜像。
    3. 将镜像提送（Push）到私有Harbor。
    4. Jenkins再次控制Kubernetes进行最新的镜像部署。

## 注意事项

1. 配置 __Kubernetes Pod Template__ 时注意:
  a. 如果pipeline 没有指定agent 的标签，而是使用的 `agent any`， 那么 __Usage__ 选项注意选择 __Use this node as much as possible__ . 
  b. 如果pipeline 指定的具体的agent 标签，那么 __Usage__ 选项注意选择 __ONly build jobs with label expressions matching this role__

2. 添加 jnlp-agent 类型的 __Container Template__ 时注意：
  a. __Command to run__ 和 __Arguments to pass to command__ 保持为空；
  b. 确保你拥有正确的 jenkins-jnlp-agent 镜像, 没有必要建议不要修改该镜像，直接使用默认的即可。
   ```dockerfile
   COPY jenkins-slave /usr/local/bin/jenkins-slave
   ENTRYPOINT ["jenkins-slave"]
   ```

__注__: 
    
* 上面所述为一般步骤，中间还可能会涉及自动化测试等步骤，可自行根据业务场景添加。

* 上面流水线步骤一般由应用代码库的根目录下Jenkinsfile决定，Jenkins会自动读取该文件；另外如果需要对具体的应用流水线实施强管控，可以独立管理jenkinsfile模板，然后根据jenkins API接口即时生成流水线。

* 默认使用的Dockerfile放置在代码仓库的根目录下。

## 组件部署
1. kubernetes【Kubernetes系列】第3篇 Kubernetes集群安装部署
2. gitlab 无忌过招:手把手教你搭建自己的GitLab库
3. harbor [安装配置指南](https://github.com/goharbor/harbor/blob/v1.7.4/docs/installation_guide.md)
4. jenkins 

_注_: 本文主要说明下jenkins的部署及配置，其他组件如果你部署有问题，欢迎留言。

## Jenkins 部署及配置
_说明_: 
    
* 以下的yaml文件均在 k8s master节点的 /home/jenkins_deploy目录下，
* 部署示例的depployment.yaml 的注解
    * `nodeName ipaddress` , ipaddress 请确认其为一个有效的ip.
    * 示例中jenkins的目录 `/var/jenkins_home` 是直接挂载到host_path, 如果你有条件，建议替换为共享存储。
    * 因使用的jenkins-master 的基础镜像来自公网，需要k8s maste 节点也要可以访问外网，或者你可以将 `jenkins/jenkins:lts-alpine` 推送至自己的内网镜像仓库。

* 部署示例的ingress.yaml 的注解
    * 需要你也需要办公网（集群外）访问，请将`jenkins.dev.hanker.net`, 改为有效的域名地址，或是你也可以通过`NodePort`的形式声明 service，就可以直接通过`ip:port`的形式访问jenkins了。

### 1. 准备部署yaml
* deployment.yaml

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: devops
  
# Deployment 
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: jenkins
  namespace: devops
spec:
  replicas: 1
  revisionHistoryLimit: 3
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      nodeName: 1.1.1.1
      serviceAccountName: jenkins-admin
      containers:
      - image: jenkins/jenkins:lts-alpine
        imagePullPolicy: IfNotPresent
        name: jenkins
        volumeMounts:
        - name: jenkins-volume
          mountPath: /var/jenkins_home
        - name: jenkins-localtime
          mountPath: /etc/localtime
        env:
          - name: JAVA_OPTS
            value: '-Xms256m -Xmx1024m -Duser.timezone=Asia/Shanghai'
          - name: TRY_UPGRADE_IF_NO_MARKER
            value: 'true'
        ports:
        - name: http
          containerPort: 8080
        - name: agent
          containerPort: 50000
        resources:
          requests:
            cpu: 1000m
            memory: 1Gi
          limits:
            cpu: 1200m
            memory: 2Gi
      volumes:
        - name: jenkins-localtime
          hostPath:
            path: /etc/localtime
        - name: jenkins-volume
          hostPath:
            path: /home/jenkins/jenkins_home
```

* 配置service, services.yaml

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins-service
  namespace: devops
spec:
  ports:
  - name: http
    protocol: TCP
    port: 8080
    targetPort: 8080
  - port: 50000
    targetPort: 50000
    name: agent
  selector:
    app: jenkins
```

* 授权jenkins对k8s的访问 rbac.yaml 

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: jenkins
  name: jenkins-admin
  namespace: devops

---

apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: jenkins-rbac
  namespace: devops
rules:
  - apiGroups: ["","extensions","app"]
    resources: ["pods","pods/exec","deployments","replicasets"]
    verbs: ["get","list","watch","create","update","patch","delete"]

---

apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: jenkins-admin
  namespace: devops
  labels:
    k8s-app: jenkins
subjects:
  - kind: ServiceAccount
    name: jenkins-admin
    namespace: devops
roleRef:
  kind: ClusterRole
  name: jenkins-rbac
  apiGroup: rbac.authorization.k8s.io
```

* 为了便于办公网（集群外）访问，ingress.yaml

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: jenkins-ingress
  namespace: devops
spec:
  rules:
  - host: jenkins.dev.hanker.net
    http:
      paths:
      - backend:
          serviceName: jenkins-service
          servicePort: 8080
        path: /
```

### 2. 应用yaml，部署jenkins

```sh
$ pwd
$ /home/jenkins_deploy
$ kubectl apply -f *.yaml
```

### 3. 确认jenkins 服务状态

```sh
[root@node0 jenkins_deploy]# kubectl -n devops get deployment jenkins
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
jenkins   1/1     1            1           51d
[root@node0 jenkins_deploy]# 
```

### 4. 访问jenkins 安装插件、设置

_注:_ 步骤1 声明的域名 `jenkins.dev.hanker.net` 已经解析至ingress，故可直接访问； 如果你也想通过自定义域名访问jenkins，麻请解析至正确的ingress服务节点，即可。

1. 确认你也已经安装了kubernetes/ kubernetes cli 插件

_操作指引_： 【Manage Jenkins】 -> 【Manage Plugins】

你应该可以通过类似的指令获取jenkins-master的密码

```sh
$ kubectl -n devops exec jenkins-pod-name cat /var/jenkins_home/secrets/initialAdminPassword
```

![image](https://user-images.githubusercontent.com/5203608/89489195-7f7e9300-d7dc-11ea-96d4-51d7868a542a.png)

2. 配置Kubernetes 插件

_操作指引_： 【Manage Jenkins】->【Configure System】 

![image](https://user-images.githubusercontent.com/5203608/89489205-84434700-d7dc-11ea-84f6-ea526dc85e1c.png)

![image](https://user-images.githubusercontent.com/5203608/89489214-886f6480-d7dc-11ea-9cfb-f4c90dd4f21c.png)

_图中标注：_

  1. 请修改为你所在环境对应的k8s master

  2. 声明jenkins-agent 的命令空间，也可以根据需要调整；
  3. jenkins-master的访问地址，本示例使用的是 service-name的形式访问。

  4. 测试与k8s分享群的连接情况。如果你获取到『
Connection test successful』，恭喜你可以继续。

3. 配置Kubernetes Pod Template

![image](https://user-images.githubusercontent.com/5203608/89489216-8c9b8200-d7dc-11ea-84b1-3fcc7e1e915f.png)

_图中标注：_

  1. 设置基础的jenkins-agent镜像, 参考下篇附录：1. 自定义jenkins-agent镜像；
  2. 指定工作目录；

      * 如果你需要下载、导出或是缓存构建的话，指定一个为共享存储的目录就很有意义了。

  3. 设置目录挂载
    
      * 如步骤2如说，你可以将宿主机的目录或是网络存储挂载至jenkins-agent.

## CI/CD - 全流程实践

## 前言
1. 本实践中已经的示例代码及jenkins-agent镜像已经推送归档至github，[-->传送门](https://github.com/Kubernetes-Best-Pratice/)
2. 注意本实践中均为内网数据，你测试时一定要改为自己的环境的有效数据。
3. 由于本实践涉及组件较多，若有操作不明确的话，你可以后台留言，我们一起完善。
4. 具体操作时若有不清楚，或是错误可以留言，大家一起解决。
5. [kubernetes jenkins plugins](https://plugins.jenkins.io/kubernetes/)


### 1. 准备基础数据
1. 配置gitlab
  * 创建项目
  * 上传示例代码

  _注_: 本次示例使用的gitlab项目地址为：`http://gitlab.hanker.com/colynn/hanker-hello.git`

2. 配置harbor
  * 创建项目, 用于存储构建的镜像
  _注_: 本次示例使用的harbor地址为 `10.0.0.185:5000/hanker/hanker-hello:v1`

3. jenkins 验证信息
  * 添加 gitlab 帐号信息

  _操作指引_：【Credentials】-> 【System】-> 【Global credentials】-> 【Add Credentials】

![image](https://user-images.githubusercontent.com/5203608/89489317-c40a2e80-d7dc-11ea-9632-4d26094bc007.png)

  * harbor 信息

  _操作指引_：【Credentials】-> 【System】-> 【Global credentials】-> 【Add Credentials】

![image](https://user-images.githubusercontent.com/5203608/89489324-c79db580-d7dc-11ea-8926-9457a77aa465.png)

  * k8s namespace验证信息

  在你的k8s master节点上执行如下操作：

1. 创建serviceaccount 

```sh
$ kubectl -n devops create serviceaccount jenkins-robot
```

命令输出：

```sh
serviceaccount/jenkins-robot created
```

2. 角色绑定

```sh
$ kubectl -n devops create rolebinding jenkins-robot-binding --clusterrole=cluster-admin --serviceaccount=devops:jenkins-robot
```

命令输出：

```sh
rolebinding.rbac.authorization.k8s.io/jenkins-robot-binding created
```

3. 获取 ServiceAccount

```sh
$ kubectl -n devops get serviceaccount jenkins-robot -o go-template --template='{{range .secrets}}{{.name}}{{"\n"}}{{end}}'
```
jenkins-robot-token-n8w6b


4. 基于base64解码 ServiceToken

```sh
$ kubectl -n devops get secrets jenkins-robot-token-n8w6b -o go-template --template '{{index .data "token"}}' | base64 --decode
```


命令输出：

```sh
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZXZvcHMiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlY3JldC5uYW1lIjoiamVua2lucy1yb2JvdC10b2tlbi1uOHc2YiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJqZW5raW5zLXJvYm90Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiOTcyZTY0OGYtMTYxZC00NmM5LWI0ZjgtYjFkNTdlOWY4NTBjIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRldm9wczpqZW5raW5zLXJvYm90In0.ArQvcaEqCaeU1ZcJ6nOC5rLaTZr_vLDrpLCt87asltMUWj2gSli_mXUTrl09hBnBDXI3A1D4rJXHKLHjIAA4nN8qRIRGbpqSNzDwmqJr-jmmmWWZFrZ3n3Al9-13KJnNOK8pcWr70rt3Rsigt4B6CIQ0-ZLK8BZhvROJSifeOfJ6xe2KBqdXBv1ccZZZfEhPLgGbaR5yWm5jLvOMr2MQiPDrZoHOEkcMt-C0xipytOp4sJCJ4bQhb-UoMu1owYydxbd6O7xO71fvqP_bMDpZXC601nA2ggK7h-vi6CJffHv5MM59q8X_DWe1NnZS6KXiMmkXqAmBn10Yu20PNj-kjg
```

5. 添加 `Secret text`验证信息
  
_操作指引_：【首页】->【Credentials】-> 【System】-> 【Global credentials】-> 【Add Credentials】-> 选择【Secret text】类型

然后将上一步 解码的结果 更新至 `Secret`, Pipeline 中

![image](https://user-images.githubusercontent.com/5203608/89489334-cec4c380-d7dc-11ea-8ef7-90ff6eb42ed6.png)


_注_: 
1. In Jenkins settings click on add cloud, select Kubernetes and fill the information, like Name, Kubernetes URL, Kubernetes server certificate key, ...

2. If Kubernetes URL is not set, the connection options will be autoconfigured from service account or kube config file.

3. When running the Jenkins master outside of Kubernetes you will need to set the credential to secret text. The value of the credential will be the token of the service account you created for Jenkins in the cluster the agents will run on.

4. If you check WebSocket then agents will connect over HTTP(S) rather than the Jenkins service TCP port. This is unnecessary when the Jenkins master runs in the same Kubernetes cluster, but can greatly simplify setup when agents are in an external cluster and the Jenkins master is not directly accessible (for example, it is behind a reverse proxy). See [JEP-222](https://jenkins.io/jep/222) for more.

### 2. 如何创建 jenkins pipeline

#### 1. 创建jenkins pipeline item 
_操作指引_：【首页】->【New Item】

![image](https://user-images.githubusercontent.com/5203608/89489341-d5533b00-d7dc-11ea-9cc1-efa4953e6ed4.png)

#### 2. pipeline script 步骤说明
_注_: pipeline主要包含三个阶段（检出代码、制作镜像、部署服务），下面跟大家解释下，如何编写pipeline， 借助Pipeline Syntax生成的只是部分代码，你可以根据语言规范将其完善。

1. 阶段1，检出代码

_操作指引_：【首页】->【hanker-hello-demo】-> 【Pipeline Syntax】

![image](https://user-images.githubusercontent.com/5203608/89489344-d84e2b80-d7dc-11ea-8584-ada6c0888e1e.png)

_注_: 本实践中选取的 `git: Git` 类型，当然你也可以选择 `checkout: Check out from version control`

获取到该步骤的脚本
```sh
git credentialsId: 'gitlab-project-auth', url: 'http://gitlab.hanker.com/colynn/hanker-hello.git'
```

2. 阶段2，构建镜像
_操作指引_：类似于 阶段1， 

![image](https://user-images.githubusercontent.com/5203608/89489671-983b7880-d7dd-11ea-9578-a3a3d2cffb60.png)

完善获取该步骤脚本

```sh
script {
    withDockerRegistry(credentialsId: 'harbor-auth', url: 'http://10.0.0.185:5000') {
        def customImage =  docker.build("10.0.0.185:5000/devops/hanker-hello:v1")
        customImage.push()
    }
}
```

_注_: 支持本阶段需要jenkins-agent镜像里包含docker命令。

3. 阶段3. 部署服务

_参考_: [jenkins kubernetes cli plugin](https://github.com/jenkinsci/kubernetes-cli-plugin/blob/master/README.md)

_注_: 支持本阶段需要jenkins-agent镜像里包含kubectl命令。

#### 3. 设置 pipeline

_注_: 
  * General/ Build Triggers/ Advanced Project Options 这三块你可以根据自己需要设置，将各阶段的脚本合并，更新至 Pipline -> Script内。


合并后的pipeline脚本内容如下：

```sh
pipeline {
    agent any
    stages {
        stage('checkout') {
            steps {
                git credentialsId: 'gitlab-project-auth', url: 'http://gitlab.hanker.com/colynn/hanker-hello.git'    
            }
        }
        
        stage('docker-publish') {
            steps{
                script {
                    withDockerRegistry(credentialsId: 'harbor-auth', url: 'http://10.0.0.185:5000') {
                        def customImage =  docker.build("10.0.0.185:5000/devops/hanker-hello:v1")
                        customImage.push()
                    }
                }
            }
        }
        
        stage('application-deploy') {
            steps {
                withKubeConfig([credentialsId: '5a5517f3-3d38-459d-bafc-12b55beeb588', serverUrl: 'https://10.0.0.182:6443']) {
                    sh '/usr/bin/kubectl apply -f k8s-setup.yml'
                }
            }
        }
    }
}

```

![image](https://user-images.githubusercontent.com/5203608/89489714-ae493900-d7dd-11ea-950a-c8a3b976bc0a.png)

### 3. 触发构建

![image](https://user-images.githubusercontent.com/5203608/89489721-b1442980-d7dd-11ea-98d2-f33ca788c96b.png)

### 4. 结果确认

1. 确认 jenkina-agent 启动状态；

```sh
$ kubectl -n devops get pods |grep jnlp
jnlp-sh8zl                                 1/1     Running   0          14s

// 查看jenkins-agent pod日志
$ kubectl -n devops logs -f [jenkins-agent-pod-name]
```

_注_: 如果长时间没有启动jenkins-agent, 可以确认下集群内是否有足够的资源。


2. 确认pipeline 执行状态；

![image](https://user-images.githubusercontent.com/5203608/89489728-b7d2a100-d7dd-11ea-8b1d-190dcdb8478c.png)


3. 确认harbor镜像仓库里是否已经有新推送的镜像

![image](https://user-images.githubusercontent.com/5203608/89489736-bdc88200-d7dd-11ea-8260-896fb3ae2915.png)

_注_: harbor里的项目是需要你先创建好的，不然推送时会报错。

4. 确认部署的服务状态

k8s master节点上执行如下操作:

```sh
$ kubectl -n devops get pod,deployment,svc,ingress |grep hanker-hello 

pod/hanker-hello-5b7586f86d-5j7kk              1/1     Running   0          173m


deployment.extensions/hanker-hello              1/1     1            1           3h8m
service/hanker-hello-svc          ClusterIP   10.233.22.19    <none>        8080/TCP             3h8m
ingress.extensions/hanker-hello-ingress              hanker-hello-demo.dev.hanker.net                   80      3h8m
```


![image](https://user-images.githubusercontent.com/5203608/89489740-c28d3600-d7dd-11ea-8133-2ff07be556ea.png)

## 附录
### 1.自定义jenkins-agent镜像

```sh
## 基于 https://github.com/Kubernetes-Best-Pratice/jenkins-jnlp-agent.git

$ git checkout  https://github.com/Kubernetes-Best-Pratice/jenkins-jnlp-agent.git

$ cd jenkins-jnlp-agent
$ docker build .
$ docker tag tag-name custom-private-repository-addr

```

_注_: 你也可以基于[基础镜像](https://github.com/jenkinsci/docker-jnlp-slave)创建自定义的镜像


## 可以做的更完善
1. 配置webhook, 自动触发jenkins job;
2. 当前我们实践时构建的镜像版本使用的是固定的, 你是否可以将其替换为依赖pipeline环境变量或是传参的形式，将其变是更有意义；
3. 上一篇文章中在设置【配置Kubernetes Pod Template】时，我们提到可以挂载主机或是网络共享存储，你是否可以通过这个将你的构建快起来；
4. 我们的示例代码使用的go, 直接是镜像内打包，如何更好的就好的其他语言的构建，你可以参考[Using Docker with Pipeline](https://jenkins.io/doc/book/pipeline/docker/)；
5. 你想过如何下载构建过程中的产物吗，等等。


参考链接 ：
1. https://github.com/jenkinsci/kubernetes-cli-plugin/blob/master/README.md

2. 下载kubectl: https://docs.docker.com/ee/ucp/user-access/kubectl/

