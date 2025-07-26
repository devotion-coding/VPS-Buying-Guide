
# VPS 安全加固指南（Security Guide）

VPS（Virtual Private Server）作为你个人或企业服务的基础设施，安全性至关重要。一旦被攻击或入侵，可能导致数据泄露、服务中断、甚至法律风险。本指南将为你提供一套完整的 VPS 安全加固方案，帮助你从系统、网络、应用、权限等多个层面提升服务器安全性。

---

## 🔐 一、基础安全设置

### 1. 修改默认 SSH 端口
默认的 SSH 端口是 `22`，容易成为自动化攻击的目标。

**修改方法：**
```bash
sudo nano /etc/ssh/sshd_config
```
修改如下配置：
```bash
Port 2222  # 更改为一个非知名端口（1024~65535）
```
重启 SSH 服务：
```bash
sudo systemctl restart sshd
```

**注意**：修改端口后，务必在防火墙中开放新端口，并测试能否重新连接。

---

### 2. 禁用 root 登录
root 用户权限过高，直接登录存在较大风险。

**修改方法：**
```bash
sudo nano /etc/ssh/sshd_config
```
修改如下配置：
```bash
PermitRootLogin no
```
重启 SSH 服务：
```bash
sudo systemctl restart sshd
```

### 3. 使用 SSH 密钥登录（推荐）
禁用密码登录，使用 SSH 密钥可以大幅提升安全性。

**生成密钥对（本地执行）：**
```bash
ssh-keygen -t rsa -b 4096
```

**上传公钥到服务器：**
```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub yourusername@your_vps_ip -p [your_ssh_port]
```

**禁用密码登录：**
```bash
sudo nano /etc/ssh/sshd_config
```
修改如下配置：
```bash
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM no
```
重启 SSH：
```bash
sudo systemctl restart sshd
```

---

### 4. 创建普通用户并授权 sudo
避免使用 root 执行日常操作，创建普通用户并授予 sudo 权限。

**创建用户：**
```bash
sudo adduser yourusername
```

**添加 sudo 权限：**
```bash
sudo usermod -aG sudo yourusername
```

---

## 🛡️ 二、配置防火墙（推荐使用 UFW）

防火墙是防止外部攻击的第一道防线。

### 1. 安装 UFW（Ubuntu/Debian）
```bash
sudo apt install ufw -y
```

### 2. 设置默认规则
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### 3. 开放常用端口
```bash
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
sudo ufw allow 2222/tcp  # 替换为你的 SSH 端口
```

### 4. 启用防火墙
```bash
sudo ufw enable
```

**查看状态：**
```bash
sudo ufw status
```

---

## 🧱 三、安装入侵防御工具（Fail2Ban）

Fail2Ban 可以监控日志，自动封禁尝试暴力破解的 IP。

### 1. 安装 Fail2Ban
```bash
sudo apt install fail2ban -y
```

### 2. 配置 Fail2Ban
创建本地配置文件：
```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

修改以下内容：
```ini
[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 5
bantime = 86400  # 封禁时间（秒）
```

重启 Fail2Ban：
```bash
sudo systemctl restart fail2ban
```

**查看封禁状态：**
```bash
sudo fail2ban-client status sshd
```

---

## 🔒 四、系统安全加固

### 1. 定期更新系统和软件包
```bash
sudo apt update && sudo apt upgrade -y
```

### 2. 删除不必要的服务和用户
```bash
sudo apt purge telnet rsh-server rlogin-server -y
sudo userdel -r nobody
```

### 3. 限制用户登录权限
编辑 PAM 配置文件：
```bash
sudo nano /etc/ssh/sshd_config
```
设置允许登录的用户组或用户：
```bash
AllowGroups admin
# 或
AllowUsers yourusername
```

### 4. 设置密码策略（可选）
安装 `libpam-pwquality`：
```bash
sudo apt install libpam-pwquality -y
sudo nano /etc/pam.d/common-password
```
设置密码复杂度策略。

---

## 📦 五、Web 服务安全加固（Nginx/Apache）

### 1. 禁止目录浏览
在 Nginx 配置中添加：
```nginx
location / {
    autoindex off;
}
```

### 2. 隐藏版本号
```nginx
server_tokens off;
```

### 3. 配置访问限制
```nginx
location /admin {
    allow 192.168.1.0/24;
    deny all;
}
```

### 4. 使用 HTTPS（推荐）
使用 Let's Encrypt 免费证书：
```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx
```

---

## 📁 六、文件权限管理

### 1. 设置合理权限
```bash
sudo find /var/www/html -type f -exec chmod 644 {} \;
sudo find /var/www/html -type d -exec chmod 755 {} \;
sudo chown -R www-data:www-data /var/www/html
```

### 2. 定期检查敏感文件
如 `.env`、`.git`、`.ssh`、数据库配置文件等，避免被外部访问。

---

## 🧹 七、日志审计与监控

### 1. 查看关键日志
- SSH 登录日志：
  ```bash
  sudo cat /var/log/auth.log
  ```
- Web 访问日志：
  ```bash
  sudo cat /var/log/nginx/access.log
  ```

### 2. 安装日志分析工具（如 GoAccess）
```bash
sudo apt install goaccess -y
sudo goaccess /var/log/nginx/access.log -o /var/www/html/report.html --real-time-html
```

### 3. 使用监控工具
- **Netdata**：实时监控资源使用情况
- **Prometheus + Grafana**：适用于多服务器监控
- **Logwatch**：每日日志分析报告

---

## 💾 八、定期备份策略

### 1. 使用 rsync 备份
```bash
rsync -avz /var/www/html user@backup_server:/backup/
```

### 2. 使用 cron 定时任务
```bash
crontab -e
```
添加如下内容（每天凌晨 2 点备份）：
```bash
0 2 * * * rsync -avz /var/www/html user@backup_server:/backup/
```

### 3. 使用快照（Snapshots）
如果你使用的是云平台（如阿里云、AWS、DigitalOcean），建议定期创建系统快照。

---

## 📬 九、ICP 备案与合规安全

如果你使用的是中国大陆服务器，必须完成 ICP 备案，否则网站将无法正常访问。

- 确保域名实名认证
- 填写真实网站信息
- 不得提供违法、色情、赌博等非法内容
- 定期检查备案信息是否更新

---

## 📌 十、安全加固清单（Checklist）

| 安全措施 | 是否完成 |
|----------|----------|
| 修改 SSH 端口 | ❌ |
| 禁用 root 登录 | ❌ |
| 使用 SSH 密钥登录 | ❌ |
| 配置防火墙（UFW） | ❌ |
| 安装 Fail2Ban | ❌ |
| 定期更新系统 | ❌ |
| 配置 HTTPS | ❌ |
| 设置文件权限 | ❌ |
| 定期备份数据 | ❌ |
| 安装监控工具 | ❌ |
| 完成 ICP 备案（如需） | ❌ |

---

## 📚 十一、后续学习推荐

- [setup-guide.md](setup-guide.md)：了解 VPS 初次配置流程
- [faq.md](faq.md)：查看常见问题解答

---

欢迎你根据自己的运维经验，补充更多实用安全加固技巧或脚本，帮助新手更高效地提升 VPS 安全性。

> 📌 如果你有特定用途（如搭建博客、科学上网、企业网站等），可以在 Issues 中留言，我们可以为你定制安全加固方案。
