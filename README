# SSH 免密登录实验文档（NAT 模式）

## 1. 实验概述

本实验用于配置三台 Ubuntu Server 虚拟机之间的 SSH 免密登录。

本次实验选择 **NAT 模式**。三台虚拟机都连接到 VMware 的 NAT 网络中，因此它们既可以访问外网，也可以在同一个 NAT 网段内互相通信。

## 2. 实验环境

| 主机名 | 系统 | 网络模式 | 实验用户 |
|---|---|---|---|
| jumper | Ubuntu Server | NAT | itheima |
| node1 | Ubuntu Server | NAT | itheima |
| node2 | Ubuntu Server | NAT | itheima |

实验目标：

```text
jumper  <->  node1
jumper  <->  node2
node1   <->  node2
```

最终效果：

三台服务器上的 `itheima` 用户都可以通过 SSH 免密登录到另外两台服务器。

---

## 3. 网络模式选择

三台虚拟机都选择 **NAT 模式**。

在 VMware 中设置：

```text
虚拟机设置
→ 网络适配器
→ NAT 模式
```

选择 NAT 模式的原因：

1. 虚拟机可以访问外网，方便执行 `apt update` 和安装软件。
2. 三台虚拟机通常会处在同一个 VMware NAT 网段中，可以互相通信。
3. 相比桥接模式，NAT 模式更适合实验环境，不容易受真实局域网影响。

---

## 4. 查看 IP 地址

在三台服务器上分别执行：

```bash
ip a
```

记录每台服务器的 IP 地址。

示例：

```text
jumper: 192.168.83.136
node1 : 192.168.83.137
node2 : 192.168.83.138
```

注意：实际 IP 以自己虚拟机显示的结果为准。

---

## 5. 测试网络连通性

在每台服务器上测试是否能 ping 通另外两台服务器。

例如在 `jumper` 上执行：

```bash
ping -c 4 192.168.83.137
ping -c 4 192.168.83.138
```

在 `node1` 上执行：

```bash
ping -c 4 192.168.83.136
ping -c 4 192.168.83.138
```

在 `node2` 上执行：

```bash
ping -c 4 192.168.83.136
ping -c 4 192.168.83.137
```

如果都能 ping 通，说明三台虚拟机之间网络通信正常。

---

## 6. 修改主机名

分别在三台服务器上修改主机名。

在 `jumper` 上执行：

```bash
sudo hostnamectl set-hostname jumper
```

在 `node1` 上执行：

```bash
sudo hostnamectl set-hostname node1
```

在 `node2` 上执行：

```bash
sudo hostnamectl set-hostname node2
```

查看当前主机名：

```bash
hostname
```

或者：

```bash
hostnamectl
```

如果命令行前缀没有立刻变化，可以重新登录，或者重启：

```bash
sudo reboot
```

---

## 7. 配置 `/etc/hosts`

为了后续可以直接使用主机名连接服务器，需要在三台服务器上都配置 `/etc/hosts`。

编辑文件：

```bash
sudo vim /etc/hosts
```

添加下面内容，IP 地址需要替换成自己的真实 IP：

```text
192.168.83.136 jumper
192.168.83.137 node1
192.168.83.138 node2
```

配置完成后测试主机名解析：

```bash
ping -c 4 jumper
ping -c 4 node1
ping -c 4 node2
```

如果能 ping 通，说明主机名解析正常。

---

## 8. 安装并启动 SSH 服务

三台服务器都执行：

```bash
sudo apt update
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
```

查看 SSH 服务状态：

```bash
sudo systemctl status ssh
```

看到类似结果说明正常：

```text
active (running)
```

---

## 9. 创建 `itheima` 用户

三台服务器都执行：

```bash
sudo useradd -m -s /bin/bash itheima
echo "itheima:123456" | sudo chpasswd
```

检查用户是否创建成功：

```bash
id itheima
```

检查家目录：

```bash
ls -ld /home/itheima
```

正常情况下应该能看到：

```text
/home/itheima
```

说明 `itheima` 用户和家目录都已经创建成功。

---

## 10. 切换到 `itheima` 用户

三台服务器都执行：

```bash
su - itheima
```

输入密码：

```text
123456
```

确认当前用户：

```bash
whoami
```

预期输出：

```text
itheima
```

注意：后续生成 SSH 密钥时，一定要在 `itheima` 用户下操作，不要在 root 用户下操作。

---

## 11. 生成 SSH 密钥对

在三台服务器的 `itheima` 用户下分别执行：

```bash
ssh-keygen -t rsa
```

执行后一路回车即可，不需要额外设置密码。

生成的密钥文件位于：

```text
/home/itheima/.ssh/
```

主要文件：

```text
id_rsa      私钥
id_rsa.pub  公钥
```

说明：

```text
私钥：保存在本机，不能随便给别人。
公钥：可以复制到其他服务器，用于配置免密登录。
```

---

## 12. 分发公钥到其他服务器

这一部分是 SSH 免密登录的核心。

### 12.1 在 `jumper` 上执行

```bash
ssh-copy-id itheima@node1
ssh-copy-id itheima@node2
```

如果主机名解析还没配置好，也可以直接使用 IP：

```bash
ssh-copy-id itheima@192.168.83.137
ssh-copy-id itheima@192.168.83.138
```

第一次连接时可能会提示：

```text
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

输入：

```text
yes
```

然后输入远程服务器上 `itheima` 用户的密码：

```text
123456
```

### 12.2 在 `node1` 上执行

```bash
ssh-copy-id itheima@jumper
ssh-copy-id itheima@node2
```

或者使用 IP：

```bash
ssh-copy-id itheima@192.168.83.136
ssh-copy-id itheima@192.168.83.138
```

### 12.3 在 `node2` 上执行

```bash
ssh-copy-id itheima@jumper
ssh-copy-id itheima@node1
```

或者使用 IP：

```bash
ssh-copy-id itheima@192.168.83.136
ssh-copy-id itheima@192.168.83.137
```

---

## 13. 测试 SSH 免密登录

### 在 `jumper` 上测试

```bash
ssh itheima@node1
exit

ssh itheima@node2
exit
```

### 在 `node1` 上测试

```bash
ssh itheima@jumper
exit

ssh itheima@node2
exit
```

### 在 `node2` 上测试

```bash
ssh itheima@jumper
exit

ssh itheima@node1
exit
```

如果登录时不再要求输入密码，说明 SSH 免密登录配置成功。

---

## 14. 测试远程执行命令

SSH 免密不仅可以用于登录，也可以用于远程执行命令。

在 `jumper` 上执行：

```bash
ssh itheima@node1 hostname
ssh itheima@node2 hostname
```

预期输出：

```text
node1
node2
```

也可以测试其他命令：

```bash
ssh itheima@node1 "uptime"
ssh itheima@node2 "ip a"
```

这说明 `jumper` 可以远程控制 `node1` 和 `node2` 执行命令。

---

## 15. 使用 `scp` 测试文件传输

在 `jumper` 上创建测试文件：

```bash
echo "hello from jumper" > test.txt
```

把文件传到 `node1`：

```bash
scp test.txt itheima@node1:/home/itheima/
```

把文件传到 `node2`：

```bash
scp test.txt itheima@node2:/home/itheima/
```

在 `node1` 上查看：

```bash
ssh itheima@node1 "cat /home/itheima/test.txt"
```

在 `node2` 上查看：

```bash
ssh itheima@node2 "cat /home/itheima/test.txt"
```

预期输出：

```text
hello from jumper
```

说明 SSH 免密和 `scp` 文件传输都正常。

---

## 16. 常见问题排查

### 问题 1：`Could not resolve hostname node1`

报错示例：

```text
Could not resolve hostname node1: Temporary failure in name resolution
```

原因：

```text
当前服务器不知道 node1 这个主机名对应哪个 IP。
```

解决方法：

编辑 `/etc/hosts`：

```bash
sudo vim /etc/hosts
```

添加：

```text
192.168.83.137 node1
```

或者直接使用 IP 连接：

```bash
ssh-copy-id itheima@192.168.83.137
```

---

### 问题 2：`Network is unreachable`

原因：

```text
当前服务器没有到目标网络或外网的路由。
```

检查命令：

```bash
ip a
ip route
```

如果使用 NAT 模式，需要确认 VMware 网络适配器已经选择 NAT。

可以重新获取 IP：

```bash
sudo dhclient -r
sudo dhclient
```

---

### 问题 3：`Permission denied`

可能原因：

1. 密码输入错误。
2. 远程服务器没有创建 `itheima` 用户。
3. SSH 服务没有启动。

检查用户：

```bash
id itheima
```

检查 SSH 服务：

```bash
sudo systemctl status ssh
```

---

### 问题 4：执行 `ssh-copy-id` 后仍然需要密码

可能是远程服务器上的 `.ssh` 目录或 `authorized_keys` 文件权限不正确。

在远程服务器的 `itheima` 用户下执行：

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

然后重新测试：

```bash
ssh itheima@node1
```

---

## 17. 实验总结

本实验基于三台 Ubuntu Server 虚拟机完成了 SSH 免密登录配置。三台虚拟机均使用 NAT 网络模式，并统一创建 `itheima` 普通用户。

实验过程中，首先修改了三台服务器的主机名，并通过 `/etc/hosts` 配置主机名和 IP 地址的映射关系。随后在三台服务器的 `itheima` 用户环境下生成 SSH 密钥对，并使用 `ssh-copy-id` 将公钥分发到其他服务器。

配置完成后，`jumper`、`node1`、`node2` 三台服务器之间可以通过 `itheima` 用户互相 SSH 免密登录。同时，也可以使用 SSH 远程执行命令，并通过 `scp` 进行文件传输。

通过本实验，可以理解 SSH 密钥认证的基本原理，掌握多服务器之间免密登录、远程执行命令和文件传输的基础操作。这为后续学习 `scp`、`rsync`、Ansible、批量运维和自动化部署打下了基础。
