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

* docker daemon的切换

  * 切换到minikube的docker daemon

    ```
    eval $(minikube docker-env)
    ```

  * 切换回用户docker daemon

    ```
    eval $(minikube docker-env -u)
    ```

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

* 更新：后面发现这样设置代理内网docker镜像无法访问，于是修改启动指令：

  ```
  minikube start  --vm-driver virtualbox --docker-env http_proxy="http://web-proxy.tencent.com:8080" --docker-env https_proxy="http://web-proxy.tencent.com:8080" --docker-env no_proxy="localhost,.oa.com,.local,.1,.2,.3,.4,.5,.6,.7,.8,.9,.10,.11,.12,.13,.14,.15,.16,.17,.18,.19,.20,.21,.22,.23,.24,.25,.26,.27,.28,.29,.30,.31,.32,.33,.34,.35,.36,.37,.38,.39,.40,.41,.42,.43,.44,.45,.46,.47,.48,.49,.50,.51,.52,.53,.54,.55,.56,.57,.58,.59,.60,.61,.62,.63,.64,.65,.66,.67,.68,.69,.70,.71,.72,.73,.74,.75,.76,.77,.78,.79,.80,.81,.82,.83,.84,.85,.86,.87,.88,.89,.90,.91,.92,.93,.94,.95,.96,.97,.98,.99,.100,.101,.102,.103,.104,.105,.106,.107,.108,.109,.110,.111,.112,.113,.114,.115,.116,.117,.118,.119,.120,.121,.122,.123,.124,.125,.126,.127,.128,.129,.130,.131,.132,.133,.134,.135,.136,.137,.138,.139,.140,.141,.142,.143,.144,.145,.146,.147,.148,.149,.150,.151,.152,.153,.154,.155,.156,.157,.158,.159,.160,.161,.162,.163,.164,.165,.166,.167,.168,.169,.170,.171,.172,.173,.174,.175,.176,.177,.178,.179,.180,.181,.182,.183,.184,.185,.186,.187,.188,.189,.190,.191,.192,.193,.194,.195,.196,.197,.198,.199,.200,.201,.202,.203,.204,.205,.206,.207,.208,.209,.210,.211,.212,.213,.214,.215,.216,.217,.218,.219,.220,.221,.222,.223,.224,.225,.226,.227,.228,.229,.230,.231,.232,.233,.234,.235,.236,.237,.238,.239,.240,.241,.242,.243,.244,.245,.246,.247,.248,.249,.250,.251,.252,.253,.254,.255"
  ```

#### 4. 用yaml文件创建服务

* 首先，创建镜像到minikube的docker daemon中去，在构建好dockerfile的目录中运行指令（这里暂时不关心dockerfile的书写）

  ```
  eval $(minikube docker-env)
  docker build -t docker.oa.com/unlikezhang/yunsou_web:t ./
  ```

* 然后，再利用yaml文件构建k8s服务（这里同样先不关心yaml文件的书写）

  ```
  kubectl create -f xxx.yaml
  ```

---

**突然，发生了一连串的问题问题**

* 1. 先是报错如下，搜索后发现由于kubernate版本过旧。

```
Error from server (BadRequest): error when creating "yunsou_web_service.yaml": Deployment in version "v1" cannot be handled as a Deployment: no kind "Deployment" is registered for version "apps/v1"
```

于是卸载kubectl，即删除`/usr/local/bin`里的kubectl文件，安装最新版

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl \
	&& chmod +x kubectl \
	&& mv kubectl /usr/local/bin/
```

* 2. 然后报错如下，发现kubernate客户端与服务端版本相差太多

```
Error from server (NotFound): the server could not find the requested resource
```

于是卸载minikube，即删除`/usr/local/bin`里的minikube文件

```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 \
  && chmod +x minikube \
  && mv minikube /usr/local/bin/
```

* 3. 又报错，发现由于kubernate中docker没有获取拉镜像权限

```
Error response from daemon: Get https://docker.oa.com/v2/: Gateway Time-out
```

登陆之前, 需要设置证书

```
minikube ssh
sudo mkdir -p /etc/docker/certs.d/docker.oa.com; 
sudo mkdir -p /etc/docker/certs.d/registry.oa.com; 
sudo wget docker.oa.com/cert/gaia.crt -O /etc/docker/certs.d/docker.oa.com/ca.crt ;  
sudo cp /etc/docker/certs.d/docker.oa.com/ca.crt /etc/docker/certs.d/registry.oa.com/ ;
```

然后登录:

```
docker login docker.xxx.com
Username:
Password:
```

* 最后，重新用yaml创建服务成功



#### 参考

* <https://blog.zhesih.com/2018/06/24/k8s-minikube-setup/>
* <https://blog.alexellis.io/minikube-behind-proxy/>
* <https://github.com/kubernetes/minikube/blob/master/docs/http_proxy.md>
* <https://blog.csdn.net/wucong60/article/details/81586272>

