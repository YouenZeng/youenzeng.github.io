# Minikube 代理 Windows 10下环境下安装

最近在学习k8s，按照官方文档安装，遇到一些网络相关的问题，在这里记录下。

安装步骤参考[这个链接](https://kubernetes.io/docs/setup/minikube/)， 坑在网络这一块，在国内网络环境下安装比较困难。

## 方案1

使用阿里云编译的支持Docker Mirror版本的，链接在[这里](https://yq.aliyun.com/articles/221687)

## 方案2

使用自己的梯子，以hyperv为例，SS开启了允许来自LAN的连接，Proxy的bypass关掉，可能需要打开防火墙端口。

* 以管理员身份打开CMD
* 启动minikube，这里虚拟机里面的docker要pull镜像，给`docker-env`设置代理 `minikube start  --vm-driver hyperv --hyperv-virtual-switch="minikube" --docker-env HTTP_PROXY=http://192.168.2.105:1080 --docker-env HTTPS_PROXY=http://192.168.2.105:1080`,这个命令就可以。
* 部署好之后，找到minikube的IP，把`minikube ip`，`%userprofile%\.minikube\machines\minikube\config.json`里面`Env`一段，加上`"NO_PROXY=localhost,192.0.0.0/24"`

顺利的话，输出是下面这样的。

```text
PS C:\WINDOWS\system32> minikube start  --vm-driver hyperv --hyperv-virtual-switch="minikube" --docker-env HTTP_PROXY=http://192.168.2.105:1080 --docker-env HTTPS_PROXY=http://192.168.2.105:1080
o   minikube v0.34.1 on windows (amd64)
>   Creating hyperv VM (CPUs=2, Memory=2048MB, Disk=20000MB) ...
-   "minikube" IP address is 192.168.2.116
-   Configuring Docker as the container runtime ...
    - env HTTP_PROXY=http://192.168.2.105:1080
    - env HTTPS_PROXY=http://192.168.2.105:1080
-   Preparing Kubernetes environment ...
-   Pulling images required by Kubernetes v1.13.3 ...
-   Launching Kubernetes v1.13.3 using kubeadm ...
-   Configuring cluster permissions ...
-   Verifying component health .....
+   kubectl is now configured to use "minikube"
=   Done! Thank you for using minikube!
```

### 遇到的问题

* 安装成功了，但是访问网络有503错误

`minikube ssh`进去看到一个docker镜像没有启动起来，`docker logs containerID`查看错误（最终没有解决），删掉`%userprofile%\.minikube`整个目录，`minikube delete`然后重新用上面的start 脚本启用

* 安装过程遇到问题

启用过程加`--v=7`把verbose级别调到debug，看输出的log

* 虚拟机内部pull mirror报错

这个是proxy设置错了，可以先`minikube stop`，修改`%userprofile%\.minikube\machines\minikube\config.json`里面的配置，然后重新start

* stop卡住，从hyperv manager进去，账号root，不需要密码,然后`shutdown now`

* 提示`TLS handshake timeout`， 把NOPROXY配置加上

* 其他

多（重）喝（启）热（试）水（试）