# VPS 初次配置指南（Setup Guide）

本指南将带你完成 VPS（Virtual Private Server）的初次配置流程，包括注册、实名认证、系统选择、SSH 登录、基础环境搭建、域名绑定、备案等内容，适用于个人博客、网站、开发测试、代理等常见用途。

---

## 🚀 一、注册账号与实名认证

### 1. 注册账号
- 访问你选择的 VPS 服务商官网（如阿里云、腾讯云、DigitalOcean 等）
- 注册账号，建议使用真实邮箱，便于接收通知和找回密码

### 2. 实名认证（尤其重要）
- 如果你在中国大陆购买 VPS 或域名，必须完成实名认证
- 个人用户：上传身份证照片
- 企业用户：上传营业执照、法人身份证、授权书等
- 实名信息将用于 ICP 备案、域名绑定等

📌 **提示**：实名认证后不可更改，请确保信息准确。

---

## 💻 二、选择合适的 VPS 套餐

### 1. 配置选择
- **CPU**：1~2核起步
- **内存**：1~2GB 起步
- **硬盘**：20GB SSD 起步
- **带宽**：1Mbps 起步（建议 2Mbps 以上）
- **流量**：选择“不限流量”或高流量套餐更安全

### 2. 地区选择
- **中国大陆用户**：选择中国大陆、香港、新加坡等地区
- **海外用户**：选择美国、欧洲等地

### 3. 操作系统选择
- **Linux**（推荐）：
  - Ubuntu（社区强大，适合新手）
  - CentOS（企业常用）
  - Debian（稳定）
- **Windows Server**：适合运行 .NET、SQL Server 等服务
- **其他系统**：如 FreeBSD、CoreOS 等（进阶用户）

📌 **提示**：如无特殊需求，推荐使用 Ubuntu 22.04 LTS。

---

## 🔐 三、远程登录配置

### 1. SSH 登录（Linux / macOS / Windows WSL）

#### 方法一：使用密钥登录（推荐）
- 在购买 VPS 时生成 SSH 密钥对（或上传已有公钥）
- 保存好私钥文件（如 `id_rsa`）
- 登录命令：
  ```bash
  ssh root@your_vps_ip -i path/to/id_rsa
  ```

#### 方法二：使用密码登录
- 首次登录后建议修改 root 密码：
  ```bash
  passwd
  ```

#### 方法三：创建普通用户并授权 sudo
```bash
adduser yourusername
usermod -aG sudo yourusername
```

### 2. Windows 远程桌面（RDP）
- 使用远程桌面工具（如 Windows 自带的 `mstsc`）
- 输入 VPS 的公网 IP 和登录账号密码

📌 **提示**：首次登录建议更改默认密码，关闭 root 登录权限以提高安全性。

---

## 🛠️ 四、基础环境搭建

### 1. 更新系统
```bash
apt update && apt upgrade -y  # Ubuntu/Debian
yum update -y                 # CentOS
```

### 2. 安装常用工具
```bash
apt install curl wget git unzip net-tools ufw -y
```

### 3. 安装 Web 服务（可选）
#### 安装 Nginx
```bash
apt install nginx -y
systemctl start nginx
systemctl enable nginx
```

#### 安装 Apache
```bash
apt install apache2 -y
systemctl start apache2
systemctl enable apache2
```

#### 安装数据库（MySQL/MariaDB）
```bash
apt install mysql-server -y
mysql_secure_installation
```

#### 安装 PHP（如需运行 WordPress）
```bash
apt install php php-fpm php-mysql -y
```

### 4. 使用 Docker（进阶）
安装 Docker：
```bash
curl -fsSL https://get.docker.com | sh
```
安装 Docker Compose：
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/v2.23.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

---

## 🌐 五、绑定域名与 SSL 证书

### 1. 购买并解析域名
- 在域名注册商（如阿里云、腾讯云、Namecheap）购买域名
- 登录域名管理后台，添加 A 记录，指向你的 VPS IP 地址

### 2. 配置 Nginx/Apache 绑定域名
编辑 Nginx 配置文件：
```bash
nano /etc/nginx/sites-available/default
```
修改 `server_name` 为你自己的域名：
```nginx
server_name yourdomain.com www.yourdomain.com;
```
重启 Nginx：
```bash
systemctl restart nginx
```

### 3. 申请 SSL 证书（HTTPS）
安装 Certbot：
```bash
apt install certbot python3-certbot-nginx -y
```
申请证书：
```bash
certbot --nginx -d yourdomain.com -d www.yourdomain.com
```
证书会自动续期。

📌 **提示**：如果你使用中国大陆服务器，记得完成 ICP 备案后再绑定域名。

---

## 📦 六、部署你的应用

### 1. 搭建个人博客（WordPress）
- 安装 LAMP（Linux + Apache + MySQL + PHP）
- 下载 WordPress：
  ```bash
  wget https://wordpress.org/latest.tar.gz
  tar -xzvf latest.tar.gz
  mv wordpress /var/www/html/
  ```
- 配置数据库，设置权限，访问域名完成安装

### 2. 搭建 Git 服务器
- 安装 Git：
  ```bash
  apt install git -y
  ```
- 创建 Git 用户：
  ```bash
  adduser git
  ```
- 配置 SSH 公钥登录，搭建私有仓库

### 3. 搭建代理服务（Shadowsocks、V2Ray、Trojan）
- 安装对应软件包
- 配置服务端配置文件
- 开放防火墙端口

---

## 🔒 七、安全加固建议

### 1. 配置防火墙
使用 UFW：
```bash
ufw allow OpenSSH
ufw allow http
ufw allow https
ufw enable
```

### 2. 禁用 root 登录
编辑 SSH 配置：
```bash
nano /etc/ssh/sshd_config
```
修改为：
```bash
PermitRootLogin no
```
重启 SSH：
```bash
systemctl restart ssh
```

### 3. 安装 Fail2Ban（防止暴力破解）
```bash
apt install fail2ban -y
systemctl enable fail2ban
systemctl start fail2ban
```

### 4. 定期备份
- 使用 `rsync`、`tar` 或云平台快照功能定期备份
- 使用 `cron` 定时任务自动备份

---

## 📋 八、ICP 备案（仅限中国大陆服务器）

### 1. 准备材料
- 域名实名认证
- 企业营业执照（企业备案）
- 身份证（个人备案）
- 服务器提供商（如阿里云、腾讯云）提供的备案服务号

### 2. 提交备案
- 登录服务器提供商的备案系统
- 填写网站信息、主办者信息、负责人信息
- 上传材料
- 提交审核（通常需 20 个工作日）

### 3. 备案成功后
- 在网站底部显示备案号（如：京ICP备12345678号）
- 域名可以正常解析并访问

📌 **提示**：备案期间网站无法访问，建议提前规划。

---

## 📚 九、后续维护建议

- 定期更新系统和软件
- 监控服务器资源使用情况（CPU、内存、磁盘）
- 使用监控工具（如 Netdata、Prometheus）
- 设置自动更新和备份机制
- 关注服务商通知，避免欠费停机

---

## 📌 十、下一步推荐

- [security.md](security.md)：学习 VPS 安全加固方法
- [faq.md](faq.md)：查看常见问题解答

---

欢迎你根据自己的部署经验，补充更多实用教程或脚本，帮助新手更高效地完成 VPS 初次配置。

> 📌 如果你有特定用途（如搭建博客、科学上网、企业网站等），可以在 Issues 中留言，我们可以为你定制部署方案。
