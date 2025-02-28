### SSH 协议

SSH 协议

私钥放在客户端，公钥放在服务端。客户端连接服务端的时候，服务端会判断客户端的私钥是否跟自己的私钥匹配。

### OpenSSH

OpenSSH 是实现 SSH 协议的一套工具

```bash
# 查看 OpenSSH 版本
$ ssh -V
OpenSSH_8.9p1 Ubuntu-3ubuntu0.11, OpenSSL 3.0.2 15 Mar 2022
# 查看 sshd 服务的状态
$ systemctl status sshd
# 通过账号密码连接一个远程服务器
# 客户端连接过的主机信息记录在 `~/.ssh/known_hosts` 文件中
$ ssh <user@example.com:path>

```

### 通过密钥对进行 SSH 连接

通过密钥对连接服务端需要 3 个步骤：

- 在客户端生成密钥对文件
- 将其中的公钥上传到服务端
- 在客户端利用私钥文件连接服务端

#### 生成密钥对

```bash
# 生成客户端私钥
$ ssh-keygen -t <ed25519> -C <annotation> -f <key-path>
```

> `-t`: 指定用于生成密钥对时的算法，目前最常用的是 'ed25519'，此外还有 'sra' 等算法。
>
> `-C`: 添加一些注释信息到输出的公钥文件中，并不会对密钥本身产生任何影响
>
> `-f`: 指定密钥文件的保存路径，默认路径是 `~/.ssh/id_ed25519`

在命令行输入生成私钥的命令并按下回车键后，命令行会提示输入 `passphrase`，这相当于是私钥的密码，属于可选设置。如果设置了 `passphrase`，之后通过私钥连接服务端的时候需要进行 `passphrase` 密码验证。

该命令运行成功后，会生成两个文件。`<key-path>`  是私钥文件， `<key-path>.pub` 是公钥文件。这个公钥文件需要上传到服务端才能实现 SSH 密钥对链接。

#### 将公钥上传到服务器

```bash
$ ssh-copy-id <key-path> -p <port> <user@example.com>
```

命令运行成功后，公钥信息会记录到服务端 `/root/.ssh/authorized_keys` 文件中。

#### 通过私钥连接服务端

##### 通过指定私钥路径连接

```bash
$ ssh -i <private-key-path> <user@example.com>
```

##### 通过配置 `~/.ssh/config` 文件进行连接

在 `~/.ssh/config` 文件中配置客户端信息：

```
Host <host>
  HostName <ip_address>
  IdentityFile <key-path>
  User <user>
```

连接服务器

```bash
$ ssh <host>
```

如果生成私钥的时候设置了 `passphrase`，连接的时候依然需要再次输入。

##### 利用 SSH agent 管理 `passphrase`

```bash
# 启动 SSH agent 服务
$ eval $(ssh-agent)  
# 将 passphrase 提交到 ssh agent 服务
$ ssh-add <key-path>
# ssh-agent 管理了一个密钥列表，ssh 连接服务端进行验证的时候，ssh-agent 会通过枚举的方式判断应该使用哪个密钥文件。
$ ssh <host>
```

### 修改 SSH 配置文件

服务端 SSH 服务的配置文件为：`/etc/ssh/sshd_config`

修改配置文件之后要重启服务

```bash
$ sudo service ssh restart
```

#### 禁止密码登录

```
PasswordAuthentication no
PubkeyAuthentication yes
```

如果上面的修改没有起效，可能是在 `/etc/ssh/ssh_config.d/` 目录下有一些文件的设置允许密码连接，需要将这些文件也进行修改或删除这些文件。

