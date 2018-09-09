# 如何在Mac中创建MiniKube

这篇文章介绍了如何在Mac系统中创建MiniKube。

## 什么事MiniKube?

Minikube是一个工具，可以在本地轻松运行Kubernetes。 Minikube在笔记本电脑的VM中运行单节点Kubernetes集群，供希望尝试Kubernetes或日常开发的用户使用。

项目地址：https://github.com/kubernetes/minikube

## 搭建

在官方项目中，在搭建MiniKube的过程中，需要使用到谷歌官方的镜像，由于某些原因，镜像下载不下来。如果使用VPN，可以根据官方的项目搭建，本文基于阿里社区开源的Minikube进行搭建。

### 安装Docker

Mac电脑安装Docker，下载地址https://download.docker.com/mac/stable/Docker.dmg ，下载完成安装即可。

安装成功，执行docker -version命令，确认。

### 安装virtualbox

下载地址：https://download.virtualbox.org/virtualbox/5.2.18/VirtualBox-5.2.18-124319-OSX.dmg

下载成功安装即可。

### 安装 kubeCtl组件


 在mac终端运行以下命令：
 
 ```
 brew install kubernetes-cli
 
 ```
  
  安装成功，执行
  
  ```
  kubectl version
  ```
###安装 minikuke

 在终端执行以下命令：
 
 ```
 curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v0.28.1/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
 
 ```
 下载完成后，执行以下命令：
 
 ```
 
 minikube start --registry-mirror=https://registry.docker-cn.com
 
 ```
 
 如果执行失败，多半是镜像下载不下来，会有提示，根据提示请手动使用镜像，重新启动。
 
 如果实在执行失败，重新来，执行以下命令：
 
 ```
 minikube delete
 
 rm ~/.minikue
 
 ```
 
 启动成功后，可以执行：
 
 ```
 kubectl get nodes
 ```

执行

```
minikube dashboard

```

会自动打开浏览器显示的界面如下：

![WX20180909-214913@2x.png](https://upload-images.jianshu.io/upload_images/2279594-981ae77a6b748527.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 参考资料

https://yq.aliyun.com/articles/221687?spm=a2c4e.11153940.blogcont221687.286.7dd57733A8RwNk&p=2#comments

https://kubernetes.io/docs/tasks/tools/install-kubectl/

https://github.com/kubernetes/minikube



