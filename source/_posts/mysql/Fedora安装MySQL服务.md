---
layout: post
title: MySQL安装
categories: [MySQL]
permalink: mysql/install.html
---



## 一、 Fedora 系统安装 MySQL Server

### 1. **更新软件包索引**
首先，确保你的软件包索引是最新的：

```bash
sudo dnf update
```

### 2. **安装 MySQL Server**
MySQL 在 Fedora 上提供了官方的 `mysql-server` 包。运行以下命令来安装 MySQL：

```bash
sudo dnf install mysql-server
```

### 3. **启动 MySQL 服务**
安装完成后，需要启动 MySQL 服务。运行以下命令：

```bash
sudo systemctl start mysqld
```

### 4. **启用 MySQL 在系统启动时自动启动**
为了确保 MySQL 服务在系统启动时自动启动，可以使用以下命令：

```bash
sudo systemctl enable mysqld
```

### 5. **查看 MySQL 服务状态**
确认 MySQL 服务正在运行：

```bash
sudo systemctl status mysqld
```

如果 MySQL 正常运行，你会看到服务状态为 `active (running)`。

```shell
● mysqld.service - MySQL 8.0 database server
     Loaded: loaded (/usr/lib/systemd/system/mysqld.service; disabled; preset: disabl>
    Drop-In: /usr/lib/systemd/system/service.d
             └─10-timeout-abort.conf
     Active: active (running) since Wed 2024-11-06 16:07:03 CST; 5s ago

```

### 6. **运行 MySQL 安全配置**
运行 MySQL 的安全配置脚本以设置 root 密码、删除默认的匿名用户、禁用远程 root 登录等安全选项：

```bash
sudo mysql_secure_installation
```

按提示设置 MySQL root 用户的密码，并选择是否删除匿名用户、禁用远程 root 登录等。



### 8. **登录到 MySQL**
使用设置的 root 密码登录到 MySQL：

```bash
mysql -u root -p
```

系统会提示你输入密码，输入你在安全配置过程中设置的 root 密码即可。

### 9. **验证 MySQL 安装**
登录后，你可以运行以下 SQL 查询，验证 MySQL 是否正确安装并正常运行：

```sql
SELECT VERSION();
```

如果返回 MySQL 版本号，说明 MySQL 安装成功。

