

# 1 :book: prepare



> 代码是针对 `3062e5de8d9bb1d86afa0895f1219357c04353a9`  提交与2021-03-21日



## 1.1 :bookmark:  install docker



```shell
sudo apt-get update

sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo apt-key fingerprint 0EBFCD88
# 验证输出(9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88)

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt-get install docker-ce docker-ce-cli containerd.io

sudo docker run hello-world
```



## 1.2 :bookmark:  get apollo



```shell
#文件太大，一次下载下来不可能，所以先拉取最表层
git clone --depth=1 https://gitee.com/ApolloAuto/apollo.git
```

> 按照上面执行后，可能还是会报错
>
> [Apollo学习(一): git clone的坑](https://zhuanlan.zhihu.com/p/113396470)
>
> * 报错一：The TLS connection was non-properly terminated
>
>   > 详细报错：
>   >
>   > ```shell
>   > error: RPC failed; curl 56 GnuTLS recv error (-110): The TLS connection was non-properly terminated.
>   > ```
>   >
>   > 解决办法：
>   >
>   > ```shell
>   > #1. 安装一些必要的环境和依赖
>   > sudo apt-get install build-essential fakeroot dpkg-dev
>   > #2. 创建一个名为git-rectify的路径
>   > mkdir ~/git-rectify
>   > #3. 进入路径，获取git的源文件
>   > cd ~/git-rectify
>   > apt-get source git
>   >     #注意：如果提示错误：您必须在 sources.list 中指定代码源（deb-src）URI，解决方法：
>   >     sudo software-properties-gtk
>   >     #弹出的软件和更新->ubuntu 软件中的源代码选项需要勾选上，选中“源代码”，保存之后就可以下载源码包了。
>   >     sudo apt update
>   >     apt-get source git
>   > #4. 安装依赖
>   > sudo apt-get build-dep git
>   > #5. 安装libcurl的依赖文件
>   > sudo apt-get install libcurl4-openssl-dev
>   > #6. 进入目录
>   > cd git-2.25.1/    #路径名后面2.*是版本号，需要看一下自己的版本
>   > #7. 修改文件内容，需要修改两个文件
>   > gedit ./debian/control # 把libcurl4-gnutls-dev 修改为 libcurl4-openssl-dev
>   > gedit ./debian/rules # 把TEST =test整行删除
>   > #8. 编译和构建安装包
>   > sudo dpkg-buildpackage -rfakeroot -b
>   > #有可能这一步执行错误，参考链接 [https://zhuanlan.zhihu.com/p/113396470]知道可以使用下面一行代码
>   > sudo dpkg-buildpackage -rfakeroot -b -uc -us
>   > 
>   > #9. 退回上一级目录，安装编译好的安装包
>   > cd ..
>   > sudo dpkg -i git_2.25.1-1ubuntu3_amd64.deb
>   > 用git进行clone时提示“服务器验证失败”，在命令行下输入：
>   > export GIT_SSL_NO_VERIFY=1
>   > ```
>   >
>   > 



```shell
#恢复出来其他分支
#git fetch --all git fetch origin
git fetch --unshallow   
# git fetch --all git fetch origin  这个命令应该也行
```

**如果没办法恢复其他分支，执行如下**

> 1. vim .git/config
> 2. 按照如下示例修改：
>
> ```shell
> [core]
>     repositoryformatversion = 0
>     filemode = true
>     bare = false
>     logallrefupdates = true
>     ignorecase = true
>     precomposeunicode = true
> [remote "origin"]
>     url = https://github.com/ReactiveX/rxjs
>     fetch = +refs/heads/master:refs/remotes/origin/master
> [branch "master"]
>     remote = origin
>     merge = refs/heads/master
> ```
>
> 修改成：
>
> ```shell
> [core]
>     repositoryformatversion = 0
>     filemode = true
>     bare = false
>     logallrefupdates = true
>     ignorecase = true
>     precomposeunicode = true
> [remote "origin"]
>     url = https://github.com/ReactiveX/rxjs
>     fetch = +refs/heads/*:refs/remotes/origin/*
> [branch "master"]
>     remote = origin
>     merge = refs/heads/master
> ```
>
> **最后一步**
>
> ```shell
> git fetch -v
> ```
>
> 



```shell
v6.0.0 = e79f9d6765  # 2020-09-21  e79f9d6765981c704cfb44c01175ad5707fc90c4
git checkout v6.0.0
```





## 1.3 :bookmark:  compile



### 1-预处理（屏蔽nvidia）

**使用`nvidia`的方法，没有通过，把电脑给灭了，所以注释掉nvidia相关**



**修改dockerfile**

```shell
cd apollo/docker/scripts
# 1 注释掉dev_start.sh下面两行
# info "Determine whether host GPU is available ..."
determine_gpu_use_host
info "USE_GPU_HOST: ${USE_GPU_HOST}"

# 2 删除掉下面两行（由于是多行命令，只是注释掉还不行，必须删除）
-e USE_GPU_HOST="${USE_GPU_HOST}" \
```

### 2-设置Apollo编译环境

**设置Apollo编译环境**

```shell
# a.设置环境变量，在终端输入以下命令：
cd ~
echo "export APOLLO_HOME=$(pwd)" >> ~/.bashrc && source ~/.bashrc
source ~/.bashrc
# b.将当前账户加入docker账户组中并赋予其相应权限，在终端输入以下命令：
sudo gpasswd -a $USER docker  
sudo usermod -aG docker $USER  
sudo chmod 777 /var/run/docker.sock
```

命令执行完成后，重新启动一下计算机。

### 3-编译Apollo源码

**编译Apollo源码**

```shell
bash docker/setup_host/install_docker.sh
bash docker/scripts/dev_start.sh 
# 如果所需镜像满足要求，可以使用下面命令，免除镜像比较和更新，加快速度
#bash docker/scripts/dev_start.sh -n
```



**进入docker内并完成编译**

```shell
#进入docker
bash docker/scripts/dev_into.sh
#编译apollo
bash apollo.sh build_opt
```



### 4-运行DreamView



```shell
# a.若您已经在docker环境中，请忽略此步骤，否则请执行以下命令进入docker环境：
cd ~/apollo
bash docker/scripts/dev_start.sh
bash docker/scripts/dev_into.sh
# b.启动apollo 在终端输入以下命令：
bash scripts/bootstrap.sh
#如果启动成功，在终端会输出以下信息：
    nohup: appending output to 'nohup.out'
    Launched module monitor.
    nohup: appending output to 'nohup.out'
    Launched module dreamview.
    Dreamview is running at http://localhost:8888
```

在浏览器中输入以下地址,可以访问DreamView:

**http://localhost:8888**

```shell
# 回放数据包 在终端输入以下命令下载数据包：
python docs/demo_guide/rosbag_helper.py demo_3.5.record
#输入以下命令可以回放数据包，在浏览器DreamView中应该可以看到回放画面。
cyber_recorder play -l -f demo_3.5.record
# 如果成功在浏览器中看到回放画面，则表明您的apollo系统已经部署成功！
```







