# VPS 常见问题与故障排查指南（Troubleshooting Guide）

在使用 VPS（Virtual Private Server）的过程中，你可能会遇到网络连接、服务启动失败、域名解析异常、系统崩溃等问题。本指南将帮助你快速定位并解决常见故障，适用于个人博客、开发环境、代理服务等常见用途。

---

## 🧰 一、网络与连接问题

### 1. 无法通过 SSH 登录 VPS

#### ✅ 可能原因：
- SSH 服务未运行
- SSH 端口被防火墙阻挡
- 密钥或密码错误
- 服务器宕机或网络中断

#### 🔧 解决方案：
- **检查 SSH 是否运行**：
  ```bash
  sudo systemctl status ssh
  ```
  如果未运行，尝试重启：
  ```bash
  sudo systemctl restart ssh
  ```

- **检查端口是否开放**：
  ```bash
  sudo ufw status
  ```
  确保 SSH 端口（如 22 或 2222）处于允许状态。

- **使用服务商控制台登录**（如阿里云、腾讯云、DigitalOcean）：
  - 登录平台控制台 → 找到你的 VPS → 使用“VNC”或“Web Console”登录

- **检查 IP 是否被封禁**（如 Fail2Ban）：
  ```bash
  sudo fail2ban-client status sshd
  ```

- **尝试更换 SSH 客户端或网络环境**（如换 WiFi、使用手机热点）

---

### 2. 域名无法访问网站

#### ✅ 可能原因：
- 域名未解析
- 网站服务未运行
- 端口未开放
- 域名未备案（中国大陆服务器）

#### 🔧 解决方案：
- **检查域名解析是否生效**：
  - 使用 `ping` 或 `nslookup` 检查是否指向 VPS IP：
    ```bash
    ping yourdomain.com
    nslookup yourdomain.com
    ```

- **检查 Web 服务是否运行**：
  ```bash
  sudo systemctl status nginx
  sudo systemctl status apache2
  ```

- **检查防火墙设置**：
  ```bash
  sudo ufw status
  ```
  确保 `http` 和 `https` 端口开放。

- **检查 ICP 备案状态**（仅中国大陆服务器）：
  - 登录服务商备案系统查看备案是否通过

- **检查浏览器是否缓存 DNS**：
  - 清除浏览器缓存或使用 `dig` 命令重新查询：
    ```bash
    dig yourdomain.com
    ```

---

### 3. VPS 无法访问外网

#### ✅ 可能原因：
- DNS 配置错误
- 网络接口异常
- 路由表错误
- 服务商限制网络访问

#### 🔧 解决方案：
- **检查 DNS 设置**：
  ```bash
  cat /etc/resolv.conf
  ```
  添加 Google DNS：
  ```bash
  nameserver 8.8.8.8
  nameserver 8.8.4.4
  ```

- **检查网络接口状态**：
  ```bash
  ip a
  ```
  查看 `eth0` 或 `ens3` 是否有公网 IP。

- **测试网络连接**：
  ```bash
  ping 8.8.8.8
  curl -v google.com
  ```

- **重启网络服务**：
  ```bash
  sudo systemctl restart networking
  ```

---

## 🧱 二、服务启动失败问题

### 1. Nginx/Apache 启动失败

#### ✅ 可能原因：
- 配置文件错误
- 端口被占用
- 权限不足
- 文件损坏

#### 🔧 解决方案：
- **查看日志排查问题**：
  ```bash
  sudo journalctl -u nginx
  sudo nginx -t  # 检查配置文件语法
  ```

- **检查端口占用情况**：
  ```bash
  sudo netstat -tuln | grep :80
  sudo lsof -i :80
  ```

- **尝试重新安装**：
  ```bash
  sudo apt purge nginx
  sudo apt install nginx
  ```

---

### 2. MySQL 启动失败

#### ✅ 可能原因：
- 数据库损坏
- 配置文件错误
- 磁盘空间不足
- 权限配置错误

#### 🔧 解决方案：
- **查看日志**：
  ```bash
  sudo journalctl -u mysql
  cat /var/log/mysql/error.log
  ```

- **检查磁盘空间**：
  ```bash
  df -h
  ```

- **尝试修复数据库**：
  ```bash
  sudo mysqld --safe-mode --skip-grant-tables &
  ```

- **重装 MySQL（谨慎操作）**：
  ```bash
  sudo apt purge mysql-server
  sudo apt install mysql-server
  ```

---

## 📦 三、磁盘与资源问题

### 1. 磁盘空间不足

#### ✅ 可能原因：
- 日志文件过大
- 缓存文件未清理
- 系统更新残留文件

#### 🔧 解决方案：
- **查看磁盘使用情况**：
  ```bash
  df -h
  ```

- **查看占用空间最大的目录**：
  ```bash
  sudo du -sh /*
  ```

- **清理 APT 缓存**：
  ```bash
  sudo apt clean
  sudo apt autoclean
  ```

- **清理日志文件**：
  ```bash
  sudo rm /var/log/*.log
  sudo rm /var/log/journal/*.journal
  ```

- **扩展磁盘容量**（如服务商支持）：
  - 登录服务商控制台 → 扩容磁盘 → 扩展文件系统（如 `resize2fs` 或 `xfs_growfs`）

---

### 2. 内存不足（OOM）

#### ✅ 可能原因：
- 应用占用内存过高
- 未设置 Swap
- 系统内存配置不合理

#### 🔧 解决方案：
- **查看内存使用情况**：
  ```bash
  free -h
  top
  ```

- **启用 Swap（虚拟内存）**：
  ```bash
  sudo fallocate -l 1G /swapfile
  sudo chmod 600 /swapfile
  sudo mkswap /swapfile
  sudo swapon /swapfile
  echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
  ```

- **优化服务配置**（如 MySQL、Nginx）：
  - 限制最大连接数、调整缓存大小等

---

## 🧨 四、系统崩溃与恢复

### 1. 系统无法启动

#### ✅ 可能原因：
- 系统文件损坏
- 内核升级失败
- 磁盘错误
- 配置错误（如 fstab）

#### 🔧 解决方案：
- **使用服务商控制台进入救援模式**（Rescue Mode）
  - 如阿里云、腾讯云、DigitalOcean 提供救援系统

- **检查文件系统**：
  ```bash
  fsck /dev/vda1
  ```

- **查看启动日志**：
  ```bash
  journalctl -b -1  # 查看上一次启动日志
  ```

- **尝试回滚内核**：
  ```bash
  sudo apt install linux-image-$(uname -r)
  ```

---

## 🧪 五、应用部署问题

### 1. WordPress 无法访问后台

#### ✅ 可能原因：
- .htaccess 文件损坏
- 插件冲突
- 数据库连接失败

#### 🔧 解决方案：
- **检查数据库连接信息**：
  ```bash
  nano /var/www/html/wp-config.php
  ```

- **禁用所有插件**（重命名插件目录）：
  ```bash
  mv wp-content/plugins wp-content/plugins.bak
  ```

- **重建 .htaccess 文件**：
  ```bash
  cd /var/www/html
  rm .htaccess
  touch .htaccess
  chmod 644 .htaccess
  ```

---

### 2. Docker 容器无法访问

#### ✅ 可能原因：
- 端口未映射
- 容器未运行
- 网络配置错误

#### 🔧 解决方案：
- **查看容器状态**：
  ```bash
  docker ps -a
  ```

- **检查端口映射**：
  ```bash
  docker inspect container_name
  ```

- **进入容器调试**：
  ```bash
  docker exec -it container_name bash
  ```

- **重启 Docker 服务**：
  ```bash
  sudo systemctl restart docker
  ```

---

## 📌 六、常见错误代码与含义

| 错误码 | 含义 | 解决方案 |
|--------|------|----------|
| `Connection refused` | 服务未运行或端口未开放 | 检查服务状态和防火墙 |
| `Permission denied` | 权限不足或密钥错误 | 检查用户权限和 SSH 配置 |
| `502 Bad Gateway` | Nginx 无法连接后端服务 | 检查 PHP、FastCGI、后端服务是否运行 |
| `504 Gateway Timeout` | 请求超时 | 检查后端服务响应时间或网络延迟 |
| `403 Forbidden` | 权限限制或文件权限错误 | 检查文件权限和 Nginx/Apache 配置 |
| `404 Not Found` | 页面不存在或路径错误 | 检查 URL、Nginx/Apache 配置 |
| `413 Request Entity Too Large` | 上传文件过大 | 修改 Nginx 配置 `client_max_body_size` |

---

## 🧪 七、常用排查命令速查表

| 功能 | 命令 |
|------|------|
| 查看 IP 地址 | `ip a` |
| 查看系统日志 | `journalctl -u service_name` |
| 查看进程占用 | `top` 或 `htop` |
| 查看端口占用 | `netstat -tuln` 或 `lsof -i :端口号` |
| 查看磁盘使用 | `df -h` |
| 查看文件占用 | `du -sh /*` |
| 测试网络连接 | `ping`, `curl`, `traceroute` |
| 查看防火墙状态 | `ufw status` |
| 查看 SSH 登录记录 | `last` 或 `cat /var/log/auth.log` |

---

## 📚 八、下一步推荐

- [setup-guide.md](setup-guide.md)：了解 VPS 初次配置流程
- [security.md](security.md)：学习 VPS 安全加固方法
- [providers.md](providers.md)：查看主流 VPS 服务商对比
- [faq.md](faq.md)：查看常见问题解答

---

欢迎你根据自己的运维经验，补充更多实用排查技巧或脚本，帮助新手更高效地定位和解决 VPS 故障。

> 📌 如果你有特定用途（如搭建博客、科学上网、企业网站等），可以在 Issues 中留言，我们可以为你定制排查方案。
