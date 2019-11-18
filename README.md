### 一、k8s核心组件

![图片.png](https://ask.qcloudimg.com/draft/6211241/mpqtjx4n4q.png)

**Kubernetes 主要由以下几个核心组件组成：**

> * etcd 保存了整个集群的状态；
> * apiserver 提供了资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现等机制；
> * controller manager 负责维护集群的状态，比如故障检测、自动扩展、滚动更新等；
> * scheduler 负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上；
> * kubelet 负责维护容器的生命周期，同时也负责Volume（CVI）和网络（CNI）的管理；
> * Container runtime 负责镜像管理以及Pod和容器的真正运行（CRI）；
> * kube-proxy 负责为Service提供cluster内部的服务发现和负载均衡

### 二、kubectl简介

kubectl 是 Kubernetes 的命令行工具（CLI），是 Kubernetes 用户和管理员必备的管理
工具。该kubectl工具控制Kubernetes集群管理器。它可以让您检查集群资源，创建、删除和更新组
件以及更多功能。kubectl 提供了大量的子命令，方便管理 Kubernetes 集群中的各种功能。

#### **1.kubectl用法**

> - kubectl -h 查看子命令列表
> - kubectl options 查看全局选项
> - kubectl <command> --help 查看子命令的帮助
> - kubectl [command] [PARAMS] -o=<format> 设置输出格式（如 json、yaml、jsonpath 等）
> - kubectl explain [RESOURCE] 查看资源的定义

#### **2.kubectl 插件krew**

`krew` 是一个用来管理 kubectl 插件的工具，类似于 apt 或 yum，支持搜索、安装和管理kubectl 插件。

### 三、krew安装

#### **1.git安装**

```bash
[root@master ~]# yum -y install git
```

#### **2.安装krew**

```bash
  set -x; cd "$(mktemp -d)" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/download/v0.3.2/krew.{tar.gz,yaml}" &&
  tar zxvf krew.tar.gz &&
  ./krew-"$(uname | tr '[:upper:]' '[:lower:]')_amd64" install \
    --manifest=krew.yaml --archive=krew.tar.gz
```

![图片.png](https://ask.qcloudimg.com/draft/6211241/6tznu88sez.png)

可能由于网络原因介质无法下载，已上传github：https://github.com/loong576/krew-for-kubectl.git

#### **3.加载环境变量**

```bash
[root@master ~]# export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
```

可以永久写的用户的环境变量文件，避免登出后失效。

#### **4.安装确认**

```bash
[root@master ~]#  kubectl plugin list 
The following compatible plugins are available:

/root/.krew/bin/kubectl-krew
```

安装完成

### 四、krew使用

#### **1.插件索引更新**

```bash
[root@master ~]# kubectl krew update
Updated the local copy of plugin index.
```

#### **2.插件搜索**

```
[root@master ~]# kubectl krew search
[root@master ~]# kubectl krew search crt
```

搜索全部插件和模糊搜索

![图片.png](https://ask.qcloudimg.com/draft/6211241/rrjh1ov3vy.png)

#### **3.安装插件**

```bash
[root@master ~]# kubectl krew install get-all
[root@master ~]# kubectl krew install ns tail
```

![图片.png](https://ask.qcloudimg.com/draft/6211241/9pncfd3vkf.png)

#### **4.查看已装插件**

```
[root@master ~]# kubectl krew list
PLUGIN   VERSION
get-all  v1.2.1
krew     v0.3.2
ns       v0.7.1
tail     v0.10.1
```

#### **5.查看插件详情**

```bash
[root@master ~]# kubectl krew info ns
NAME: ns
URI: https://github.com/ahmetb/kubectx/archive/v0.7.1.tar.gz
SHA256: 6df4def2caf5a9c291310124098ad6c4c3123936ddd4080b382b9f7930a233ec
VERSION: v0.7.1
HOMEPAGE: https://github.com/ahmetb/kubectx
DESCRIPTION: 
Also known as "kubens", a utility to set your current namespace and switch
between them.

CAVEATS:
\
 |  If fzf is installed on your machine, you can interactively choose
 |  between the entries using the arrow keys, or by fuzzy searching
 |  as you type.
 |  
 |  See https://github.com/ahmetb/kubectx for customization and details.
/
```

#### **6.插件更新**

```bash
[root@master ~]# kubectl krew upgrade ns
Updated the local copy of plugin index.
F1118 17:21:47.271927   81116 root.go:58] failed to upgrade plugin "ns": can't upgrade, the newest version is already installed
```

更新插件ns，由于是最新版所以更新失败，可通过命令'kubectl krew upgrade'更新全部插件

#### **7.使用插件--ns**

```bash
[root@master ~]# kubectl
kubectl          kubectl-get_all  kubectl-krew     kubectl-ns       kubectl-tail  
[root@master ~]# kubectl ns weave
[root@master ~]# kubectl-ns default
```

![图片.png](https://ask.qcloudimg.com/draft/6211241/7lct62ijl9.png)

装完插件后可通过命令kubectl    <plugin-name> 或者kubectl-<plugin-name> 使用插件，比如'kubectl ns weave'和'kubectl-ns default'都可以切换默认表空间

#### **8.使用插件--get-all**

```bash
[root@master ~]# kubectl-get_all
```

![图片.png](https://ask.qcloudimg.com/draft/6211241/2wr12sbh3a.png)

该命令类似'kubectl get all --all-namespaces'，但更全。

#### **9.使用插件--tail**

```bash
[root@master ~]# kubectl-tail
[root@master ~]# kubectl-tail --ns default 
[root@master ~]# kubectl-tail --rs kubeapps-8fd98f6f5
[root@master ~]# kubectl-tail --rs kubeapps/kubeapps-8fd98f6f5 
```

tail为输出pod日志，以上命令分别为：输出全部pod日志、输出所有命名空间default的pod日志、输出全部命名空间中所有replicaset为kubeapps-8fd98f6f5的pod日志、输出命名空间为kubeapps且replicaset为kubeapps-8fd98f6f5的pod日志。

![图片.png](https://ask.qcloudimg.com/draft/6211241/62cyep0x9t.png)

#### **10.卸载插件**

```
[root@master ~]# kubectl krew uninstall tail
Uninstalled plugin tail
```

卸载插件tail

### 五、krew卸载

#### **1.查看安装目录**

```rm -rf ~/.krew
[root@master ~]# kubectl krew version
OPTION        VALUE
GitTag        v0.3.2
GitCommit     bd754e1
IndexURI      https://github.com/kubernetes-sigs/krew-index.git
BasePath      /root/.krew
IndexPath     /root/.krew/index
InstallPath   /root/.krew/store
DownloadPath  /tmp/krew-downloads
BinPath       /root/.krew/bin
```

#### **2.卸载**

```
[root@master ~]# rm -rf  /root/.krew 
```



![](https://ask.qcloudimg.com/draft/6211241/475ldqsxa2.png)
