---
title: SSH指令&WSL安装
date: 2024-09-08 21:57:23
tags:
- original
categories:
- The Missing Semester
---

### SSH指令

#### 什么是SSH

**SSH（Secure Shell）**是一种网络协议，加密两台计算机之间的通信，并且支持各种身份验证机制。现实中，它主要用于保证远程登录和远程通信的安全，任何网络服务都可以用这个协议来加密。

SSH 密钥登录采用的是**非对称加密**。非对称加密需要两个密钥成对使用，分为**公钥（public key）**和**私钥（private key）**。每个用户通过自己的密钥登录。其中，私钥必须私密保存，不能泄漏；公钥则是公开的，可以对外发送。它们的关系是，公钥和私钥是一一对应的，每一个私钥都有且仅有一个对应的公钥，反之亦然。

如果数据使用公钥加密，那么只有使用对应的私钥才能解密，其他密钥都不行；反过来，如果使用私钥加密（这个过程一般称为“签名”），也只有使用对应的公钥解密。

SSH 的软件架构是**服务器-客户端模式（Server-Client）**。在这个架构中，SSH 软件分成两个部分：

- 向服务器发出请求的部分，称为客户端（client），OpenSSH 的实现为 `ssh`。
- 接收客户端发出的请求的部分，称为服务器（server），OpenSSH 的实现为 `sshd`。

另外，OpenSSH 还提供一些辅助工具软件（比如 `ssh-keygen` 、`ssh-agent`）和专门的客户端工具（比如 `scp` 和 `sftp` ），这个教程也会予以介绍。

---

#### SSH客户端

- **安装SSH客户端**：

  ```bash
  # Ubuntu 和 Debian（Linux系统一般自带ssh）
  $ sudo apt install openssh-client
  ```

- **登录服务器**：

  `hostname` 是主机名，它可以是域名，也可能是 IP 地址，或局域网内部的主机名。

  ```bash
  $ ssh hostname
  ```

  不指定用户名的情况下，将使用客户端的当前用户名，作为远程服务器的登录用户名。

  ```bash
  $ ssh user@hostname
  ```

  用户名也可以使用 `ssh` 的 `-l` 参数指定，这样的话，用户名和主机名就不用写在一起了。

  ```bash
  $ ssh -l username host
  ```

  ssh 默认连接服务器的22端口，`-p`参数可以指定其他端口。

  ```bash
  $ ssh -p 8821 foo.com
  ```

- **配置文件**：

  >  配置文件的格式是：
  >
  >    1. 每一行是一个配置命令。每行都是配置项和对应的值，配置项的大小写不敏感，与值之间使用空格分隔。
  >    2. `#` 开头的行表示注释，`#` 只能放在一行的开头，不能放在一行的结尾。
  
  SSH 客户端的**全局配置文件**是 `/etc/ssh/ssh_config`。
  
  **用户个人的配置文件**在`~/.ssh/config`，优先级高于全局配置文件。
  
  除了配置文件，`~/.ssh`目录还有一些用户个人的密钥文件和其他文件。下面是其中一些常见的文件。
  
  | 文件                  | 用途                      |
  | :-------------------- | :------------------------ |
  | `~/.ssh/id_ecdsa`     | 用户的 ECDSA 私钥         |
  | `~/.ssh/id_ecdsa.pub` | 用户的 ECDSA 公钥         |
  | `~/.ssh/id_rsa`       | **用于 SSH2 的 RSA 私钥** |
  | `~/.ssh/id_rsa.pub`   | **用于SSH2 的 RSA 公钥**  |
  | `~/.ssh/identity`     | 用于 SSH1 的 RSA 私钥。   |
  | `~/.ssh/identity.pub` | 用于 SSH1 的 RSA 公钥     |
  | `~/.ssh/known_hosts`  | 包含 SSH 服务器的公钥指纹 |
  
  用户个人的配置文件`~/.ssh/config`，可以按照不同服务器，列出各自的连接参数，从而不必每一次登录都输入重复的参数。下面是一个例子。（可以写入多个服务器信息。`*`是通配符，表示所有服务器。）
  
  ```
  Host GitHub
    User git
    Hostname github.com
    IdentityFile ~/.ssh/id_rsa_keats
    
  Host *
    Port 2222
    IdentityFile ~/.ssh/id_rsa_keats
  ```
  
  以后，登录 `remote.example.com` 时，只要执行 `ssh remoteserver` 命令，就会自动套用 config 文件里面指定的参数。
---

#### SSH服务器

- **安装SSH服务器**：

  ```bash
  $ sudo apt install openssh-server
  ```

- **启动sshd**：

  ```bash
  # 启动
  $ sudo systemctl start sshd.service
  
  # 停止
  $ sudo systemctl stop sshd.service
  
  # 重启
  $ sudo systemctl restart sshd.service
  
  # 让 sshd 在计算机下次启动时自动运行
  $ sudo systemctl enable sshd.service
  ```

- **配置文件**：

  sshd 的配置文件在 `/etc/ssh` 目录，主配置文件是 `sshd_config`，此外还有一些安装时生成的密钥。

  | 文件                              | 用途                  |
  | --------------------------------- | --------------------- |
  | `/etc/ssh/sshd_config`            | 配置文件              |
  | `/etc/ssh/ssh_host_ecdsa_key`     | ECDSA 私钥            |
  | `/etc/ssh/ssh_host_ecdsa_key.pub` | ECDSA 公钥            |
  | `/etc/ssh/ssh_host_key`           | 用于 SSH1 的 RSA 私钥 |
  | `/etc/ssh/ssh_host_key.pub`       | 用于 SSH1 的 RSA 公钥 |
  | `/etc/ssh/ssh_host_rsa_key`       | 用于 SSH2 的 RSA 私钥 |
  | `/etc/ssh/ssh_host_rsa_key.pub`   | 用于 SSH2 的 RSA 公钥 |
  | `/etc/pam.d/sshd`                 | PAM 配置文件          |

  配置文件修改以后，并不会自动生效，必须重新启动 sshd。

---

#### SSH密钥登录

> SSH 密钥登录分为以下的步骤：
>
> 0. 客户端通过 `ssh-keygen` 生成自己的公钥和私钥。
> 1. 手动将客户端的公钥放入远程服务器的指定位置。
> 2. 客户端向服务器发起 SSH 登录的请求。
> 3. 服务器收到用户 SSH 登录的请求，发送一些随机数据给用户，要求用户证明自己的身份。
> 4. 客户端收到服务器发来的数据，使用私钥对数据进行签名，然后再发还给服务器。
> 5. 服务器收到客户端发来的加密签名后，使用对应的公钥解密，然后跟原始数据比较。如果一致，就允许用户登录。

- **生成密钥**：

  ```bash
  # 直接输入ssh-keygen，程序会询问一系列问题（直接回车即可），然后生成密钥。
  $ ssh-keygen
  
  # 使用-t参数，指定密钥的加密算法。如果省略该参数，默认使用 RSA 算法。
  $ ssh-keygen -t dsa
  ```

- **上传公钥**：

  ```bash
  # 生成密钥以后，公钥必须上传到服务器，才能使用公钥登录。
  # OpenSSH 规定，用户公钥保存在服务器的~/.ssh/authorized_keys文件。
  # 你要以哪个用户的身份登录到服务器，密钥就必须保存在该用户主目录的~/.ssh/authorized_keys文件。
  # 只要把公钥添加到这个文件之中，就相当于公钥上传到服务器了。每个公钥占据一行。如果该文件不存在，可以手动创建。
  # 用户可以手动编辑该文件，把公钥粘贴进去；也可以在本机计算机上，执行ssh-copy-id 命令，自动上传公钥。
  # 只要公钥上传到服务器，下次登录时，OpenSSH 就会自动采用密钥登录，不再提示输入密码。
  
  # OpenSSH 自带一个ssh-copy-id命令，可以自动将公钥拷贝到远程服务器的~/.ssh/authorized_keys文件。
  # 如果~/.ssh/authorized_keys文件不存在，ssh-copy-id命令会自动创建该文件。
  # 公钥文件可以不指定路径和.pub后缀名，ssh-copy-id会自动在~/.ssh目录里面寻找。
  
  $ ssh-copy-id -i key_file user@host
  ```

---

### WSL安装

*详见[WSL+VScode安装教程](https://www.bilibili.com/video/BV1KX4HewEBx/)。*

#### 启用 Windows 功能：

![打开虚拟机相关功能](https://ref.xht03.online/202412101932859.png)

---

#### 下载 WSL：

```bash
# 下载
$ wsl --install

# 查看版本
$ wsl --version
```

---

#### 下载 Ubuntu：

![在Microsoft Store随便选一个版本的Linux内核](https://ref.xht03.online/202412101932174.png)



---

#### 在 Ubuntu 中配置 SSH 服务器

- **安装 ssh 服务器**

  ```bash
  $ sudo apt-get install openssh-server
  ```

- **运行 ssh 服务器**

  ```bash
  $ sudo service ssh start
  ```

- **检查 ssh 服务器是否已经开启成功**

  ```bash
  $ systemctl status sshd
  ```

  如果有 `active (running)` 表示已经运行，否则则执行安装步骤

- **修改配置文件**

  ```bash
  $ sudo vim /etc/ssh/sshd_config
  
  # 增加 `Port 22`
  # 增加 `PermitRootLogin yes`
  #如果配置文件中已经有上述两项配置，则修改
  ```

- **查看虚拟机的 ip**

  ```bash
  $ ifconfig
  ```



---

#### Windows 中 ssh 登录

- 打开 powershell，ssh 连接虚拟机

  ```bash
  $ ssh <username>@<ip_address>
  ```

- 如果登录成功，表示 linux 配置成功。否则根据输入的错误日志，重新排查。



---

#### VScode 配置

- **安装 SSH 插件**

  ![安装以上三个插件](https://ref.xht03.online/202412101933724.png)

- **SSH登录**

  ![点击加号添加服务器，在上方长栏输入ssh命令，并配置客户端](https://ref.xht03.online/202412101933082.png)

  ```
  Host Ubuntu(自定义)
      HostName 10.245.68.242	（虚拟机内部系统ip地址）
      User <user_name>				（虚拟机内部系统登录账号）
      IdentityFile "C:\Users\<User_name>\.ssh\id_rsa"
  ```

- **配置 SSH 密钥免密登录**

  1. **制造密钥**：

     ```bash
     $ ssh-keygen -t rsa -C "<email>
     ```

  2. **复制密钥**：

     ```bash
     # 一般位于 C:\Users\<User_name>\.ssh
     cat /rsa_id.pub
     ```

  3. **粘贴到 Ubuntu 虚拟机**：

     ```bash
     $ vim ~/.ssh/authorized_keys
     ```

- **最终效果**

  ![在vscode终端直接访问Ubuntu虚拟机](https://ref.xht03.online/202412101933432.png)

---

### 参考文章

[SSH 密钥登录](https://wangdoc.com/ssh/key)

[vscode 连接本地虚拟机 Linux 系统](https://www.cnblogs.com/wanghao-boke/p/17859096.html)

