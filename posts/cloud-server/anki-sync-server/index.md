# Anki同步服务器搭建


## 新服务器配置

我买的是阿里云99计划 99r 一年的服务器。找客服说一下一共能以优惠价买三年。

镜像我选择了 ubuntu24.04。

首先到阿里云控制台重置 root 密码。建议设置一个较为复杂的密码，并保存好。

软件源已经默认设置为了阿里云源，无需手动更换了。

通过控制台的远程连接连接到服务器，安装一些必要软件：

```shell
sudo apt update
sudo apt upgrade
sudo apt install vim gcc g++ gcc-multilib g++-multilib make cmake build-essential python-is-python3 wget gnupg dpkg apt-transport-https lsb-release ca-certificates curl
```

我习惯新建一个用户来做管理，添加新用户：

```shell
# 添加账户 newuser
useradd newuser
# 为账户 newuser 设置密码
passwd newuser
# 将新用户添加到 root 组
adduser newuser sudo
```

修改ssh设置，仅允许密钥登录：

```shell
sudo vim /etc/ssh/sshd_config
# 在配置文件中修改或添加以下两项：
#禁用密码验证
PasswordAuthentication no
#启用密钥验证
PubkeyAuthentication yes
```

接下来添加公钥，如果本机没有生成过公钥，可以搜索 windows ssh公钥生成

```shell
mkdir /home/spirit/.ssh -p
# 将公钥粘贴进去保存即可
vim /home/spirit/.ssh/authorized_keys

# 重启ssh服务
sudo service ssh restart
```

然后就可以在本地使用 ssh 连接服务器了。

修改 `~\.ssh\config` 文件（注意修改自定义内容）：

```shell
Host aliyun-ubuntu24
    HostName your-ip
    User your-user-name
    TCPKeepAlive yes
    ServerAliveInterval 60
    ServerAliveCountMax 30
    IdentityFile "C:\Users\your-local-username\.ssh\id_rsa"
```

修改一下默认 shell：

```shell
ssh aliyun-ubuntu24
# 查看所有 shell
sudo cat /etc/shells
# 修改默认 shell 为 bash
chsh # 输入 bash 路径即可
```



## 安装 docker

1. 添加 Docker 官方 GPG 密钥：

   ```shell
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   ```

2. 添加 Docker 仓库：

   ```shell
   sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
   ```

3. 更新软件包索引

   ```shell
   sudo apt update
   ```

4. 安装最新版本的Docker CE（社区版）：

   ```shell
   sudo apt-get install docker-ce
   ```

5. 查看 Docker 是否正常运行：

   ```shell
   sudo systemctl status docker
   
   docker --version
   ```

6. 将当前用户添加到 docker 组：

   ```shell
   sudo usermod -aG docker ${USER}
   ```

7. 配置镜像加速：在[阿里云容器镜像服务](https://cr.console.aliyun.com/cn-beijing/instances/mirrors) 中可以获取专属 url。根据其中提示配置镜像即可。

8. 重新登录后运行 hello-world 测试：

   ```shell
   docker run hello-world
   ```

## 创建 anki-sync-server

```shell
# 首次添加需要以下命令，其中用户名、密码及端口自定义即可（就是那个23456）
docker run -d \
	-e "SYNC_USER1=sync_name:password" \
    --publish 23456:8080 \
    --volume ./data:/syncserver \
    --name anki-sync-server \
    ghcr.io/yangchuansheng/anki-sync-server:latest
```

docker run 会创建并运行新的容器。创建好容器以后用以下命令管理：

```shell
docker ps [-a] # 加上-a参数是列出所有容器，否则是列出运行中的容器
docker rm CONTAINER # 删除名为 CONTAINER 的容器
# 以下分别为运行/停止/重启容器
docker start CONTAINER
docker stop CONTAINER
docker restart CONTAINER
```

之后需要去阿里云-云服务器ECS-安全组-管理规则-入方向中开放端口。

使用手动添加，填入端口，授权对象为所有 IPv4即可。

同步服务器的地址即为：http://服务器公网IP:23456

## 参考文献

[Ubuntu 24.04 LTS 安装Docker](https://blog.csdn.net/level_level/article/details/139222234)

[Docker命令大全](https://www.runoob.com/docker/docker-command-manual.html)

[anki-sync-server](https://github.com/yangchuansheng/anki-sync-server)



---

> 作者: [Spirit](https://github.com/Ssspirit)  
> URL: https://rxzcloud.top/posts/cloud-server/anki-sync-server/  

