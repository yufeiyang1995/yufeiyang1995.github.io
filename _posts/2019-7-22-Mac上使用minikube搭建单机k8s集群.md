---
layout:     post
title:      Mac上使用minikube搭建单机k8s集群
category: blog
description: minikube环境搭建的坑
---

## Mac上使用minikube搭建单机k8s集群

最近由于项目需要搭建一个测试环境，于是mentor决定用minikube搭建一个本地的测试环境，于是我也就接触到了k8s这样的技术，这里记录一下搭建过程。

#### 1. mac环境搭建

由于使用minikube需要安装virtualbox，在公司电脑的虚拟机里安装virtualbox总是失败，花了一下午也没有解决，因此一怒之下选了自己的mac搭建环境，不得不说Mac上搭建还是很简单的。

* 首先，执行指令，安装minikube；

```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.18.0/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

​	查看version，可以确定是否安装成功；

```
minikube version
```

* 之后，安装kubectl；

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl
```

* 安装VirtualBox

  这个到官网下一个安装程序就行了，安装好后基本的环境就可以了，真的好简单= =

#### 2. minikube搭建k8s集群

* 启动minikube

  * 需要翻墙下载配置，由于最开始在公司网络里没有注意到
  * 启动：`minikube start`
  * 查看版本：`kubectl version`

* 部署应用

  * 启动一个镜像，hello-minikube

    `kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.10 --port=8080`

  * 部署完毕后，可以通过命令查看：`kubectl get pods`

* 发布服务

  * 将之前部署的应用，发布为可以访问的服务。将hello-minikube应用发布成为服务，将之前容器(Pod)的8080端口，在节点（当前mac）中映射物理端口NodePort

    `kubectl expose deployment hello-minikube --type=NodePort`

  * 命令查看已经发布的内容：`kubectl get services`
  * 在浏览器中打开服务：`minikube service hello-minikube   `

* 打开dashboard

  * minikube 提供了dashboard可视化界面：`minikube dashboard`

* 删除应用

  ```
  kubectl delete services hello-minikube
  ```

* 删除服务

  ```
  kubectl delete deployment hello-minikube
  ```

* 停止minikube

  ```
  minikube stop
  ```

* 删除minikube

  ```
  minikube delete
  ```

#### 3. 公司内网搭建

本来环境搭得挺顺利，但是突然有一天晚上发现minikube报错了，查看log发现应该是有镜像pull的时候报错，也就是网络出问题了；突然发现原来是我当前网络变成公司内网了，无法访问外部的镜像源，所以我开始了改代理之旅。

* 首先，我先去改~/.bash_profile

  ```
  export http_proxy="公司代理"
  export https_proxy="公司代理"
  ```

  加上两条公司的代理地址，然后使它生效

  ```
  source .bash_profile
  ```

  生效之后，重新运行minikube，但是还是失败了，但这一次报错信息是

  ```
  Bad Gateway Timeout
  ```

* 这让我怀疑代理是配成功了，但是有一些请求不应该走代理，于是我加上no_proxy

  ```
  export no_proxy="192.168.99.0/24"
  ```

  再试着重启了minikube，然而结果还是失败

* 之后，我又参考很多资料，按照不同方法试了好多种的代理和no_proxy的组合，然后都没生效

* 最后，我找到一篇博客也是记录minikube在proxy下使用的过程，他们是将代理和no_proxy通过docker_env的参数传进minikube中，我也照着执行如下指令

  ```
  minikube start --docker-env http_proxy=http://web-proxy.oa.com:8080 --docker-env https_proxy=http://web-proxy.oa.com:8080 --docker-env no_proxy=192.168.99.0/24
  ```

  最终，minikube终于在需要proxy设置的环境下成功启动了！

  虽然之后我又试着去改docker的proxy等等方法，但是我的minikube在不用命令指定代理时总是不使用环境变量中的代理，所以一直没有成功，由于时间原因就先不折腾了，等任务做完之后再来研究怎么回事吧。

  

#### 参考

* <https://blog.zhesih.com/2018/06/24/k8s-minikube-setup/>
* <https://blog.alexellis.io/minikube-behind-proxy/>
* <https://github.com/kubernetes/minikube/blob/master/docs/http_proxy.md>

