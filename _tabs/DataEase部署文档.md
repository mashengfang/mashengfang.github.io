# DataEase 2.0 部署文档

## 一、部署思路概述

DataEase 2.0 是基于 Docker 和 Docker Compose 的数据可视化平台，部署流程主要包括以下几个阶段：

1. **环境准备阶段**：检查系统环境，安装必要的依赖工具
2. **目录初始化阶段**：创建运行目录和配置文件目录
3. **工具安装阶段**：安装 dectl 命令行管理工具
4. **容器环境准备**：确保 Docker 和 Docker Compose 已安装并运行
5. **镜像加载阶段**：加载 DataEase 和 MySQL 镜像
6. **服务配置阶段**：配置 DataEase 服务和开机自启动
7. **服务启动阶段**：启动 DataEase 服务并验证

## 二、系统要求

### 2.1 硬件要求
- CPU: 2核及以上
- 内存: 4GB 及以上（推荐 8GB）
- 磁盘: 20GB 及以上可用空间

### 2.2 软件要求
- 操作系统: Linux（CentOS 7+、Ubuntu 18+、openEuler 等）
- Docker: 20.10 及以上版本
- Docker Compose: 1.29 及以上版本
- 系统服务管理工具: systemd（systemctl）或 service

### 2.3 必需的系统工具
- `envsubst`: 用于环境变量替换（通常包含在 gettext 包中）
- `systemctl`: 用于系统服务管理（systemd 系统）

## 三、详细部署步骤

### 步骤 1: 检查安装环境并初始化环境变量

**操作内容：**
- 检查系统版本和架构
- 检查磁盘空间
- 检查网络连接
- 初始化安装所需的环境变量

**执行命令：**
```bash
# 检查系统信息
uname -a
cat /etc/os-release

# 检查磁盘空间
df -h

# 检查网络连接
ping -c 3 www.baidu.com
```

**注意事项：**
- 确保系统时间正确
- 确保有 root 或 sudo 权限

---

### 步骤 2: 设置运行目录

**操作内容：**
- 创建 DataEase 运行目录：`/opt/dataease2.0`
- 创建配置文件目录：`/opt/dataease2.0/conf`

**执行命令：**
```bash
# 创建运行目录
mkdir -p /opt/dataease2.0
mkdir -p /opt/dataease2.0/conf

# 设置目录权限
chmod 755 /opt/dataease2.0
chmod 755 /opt/dataease2.0/conf
```

**目录结构：**
```
/opt/dataease2.0/
├── conf/          # 配置文件目录
├── data/          # 数据目录
├── logs/          # 日志目录
└── ...
```

---

### 步骤 3: 初始化运行目录

**操作内容：**
- 复制安装文件到运行目录
- 调整配置文件参数（使用环境变量替换）

**执行命令：**
```bash
# 复制安装文件（假设安装包在 /tmp/dataease-installer）
cp -r /tmp/dataease-installer/* /opt/dataease2.0/

# 调整配置文件参数
cd /opt/dataease2.0/conf
# 使用 envsubst 替换配置文件中的环境变量
```

**常见问题处理：**

**问题 1: `envsubst: command not found`**

**原因：** 系统缺少 `gettext` 包，该包提供了 `envsubst` 命令。

**解决方案：**

```bash
# CentOS/RHEL/openEuler 系统
yum install -y gettext

# Ubuntu/Debian 系统
apt-get update && apt-get install -y gettext-base

# 验证安装
which envsubst
envsubst --version
```

---

### 步骤 4: 安装 dectl 命令行工具

**操作内容：**
- 安装 dectl 工具到系统路径
- 安装位置：`/usr/local/bin/dectl` 和 `/usr/bin/dectl`

**执行命令：**
```bash
# 复制 dectl 到系统路径
cp /opt/dataease2.0/bin/dectl /usr/local/bin/dectl
cp /opt/dataease2.0/bin/dectl /usr/bin/dectl

# 设置执行权限
chmod +x /usr/local/bin/dectl
chmod +x /usr/bin/dectl

# 验证安装
dectl version
```

**dectl 常用命令：**
```bash
dectl status      # 查看服务状态
dectl start       # 启动服务
dectl stop        # 停止服务
dectl restart     # 重启服务
dectl logs        # 查看日志
```

---

### 步骤 5: 修改操作系统相关设置

**操作内容：**
- 配置系统参数优化
- 设置文件描述符限制
- 配置防火墙规则（如需要）

**执行命令：**
```bash
# 设置文件描述符限制
cat >> /etc/security/limits.conf << EOF
* soft nofile 65535
* hard nofile 65535
EOF

# 配置内核参数（可选）
cat >> /etc/sysctl.conf << EOF
vm.max_map_count=262144
net.core.somaxconn=65535
EOF
sysctl -p

# 配置防火墙（如需要）
# CentOS/RHEL
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --reload

# Ubuntu
ufw allow 80/tcp
ufw allow 443/tcp
```

---

### 步骤 6: 安装 Docker

**操作内容：**
- 检查 Docker 是否已安装
- 如未安装，执行安装步骤
- 启动 Docker 服务

**执行命令：**
```bash
# 检查 Docker 是否已安装
docker --version

# 如果未安装，执行安装（以 CentOS 为例）
# 安装依赖
yum install -y yum-utils device-mapper-persistent-data lvm2

# 添加 Docker 仓库
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 安装 Docker
yum install -y docker-ce docker-ce-cli containerd.io

# 启动 Docker
systemctl start docker
systemctl enable docker

# 验证 Docker 安装
docker ps
```

**注意事项：**
- 如果检测到 Docker 已安装，会跳过安装步骤
- 确保 Docker 服务已启动并设置为开机自启

---

### 步骤 7: 安装 Docker Compose

**操作内容：**
- 检查 Docker Compose 是否已安装
- 如未安装，执行安装步骤

**执行命令：**
```bash
# 检查 Docker Compose 是否已安装
docker-compose --version

# 如果未安装，执行安装
# 方法1: 使用 pip 安装
pip3 install docker-compose

# 方法2: 直接下载二进制文件
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

# 验证安装
docker-compose --version
```

**注意事项：**
- 如果检测到 Docker Compose 已安装，会跳过安装步骤
- 确保版本符合要求（1.29+）

---

### 步骤 8: 加载 DataEase 镜像

**操作内容：**
- 加载 DataEase 主镜像：`dataease:v2.10.17`
- 加载 MySQL 数据库镜像：`mysql:8.4.5`

**执行命令：**
```bash
# 进入镜像目录（假设镜像文件在安装包中）
cd /opt/dataease2.0/images

# 加载 DataEase 镜像
docker load -i dataease-v2.10.17.tar

# 加载 MySQL 镜像
docker load -i mysql-8.4.5.tar

# 验证镜像加载
docker images | grep -E "dataease|mysql"
```

**预期输出：**
```
REPOSITORY   TAG        IMAGE ID       CREATED         SIZE
dataease     v2.10.17   xxxxxxxxxxxx   2 weeks ago     2.5GB
mysql        8.4.5      xxxxxxxxxxxx   1 month ago     500MB
```

**注意事项：**
- 确保镜像文件完整且未损坏
- 镜像加载可能需要几分钟时间，请耐心等待

---

### 步骤 9: 配置 DataEase 服务

**操作内容：**
- 配置 dataease 系统服务
- 配置开机自启动

**执行命令：**
```bash
# 创建 systemd 服务文件
cat > /etc/systemd/system/dataease.service << 'EOF'
[Unit]
Description=DataEase Service
After=docker.service
Requires=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/opt/dataease2.0
ExecStart=/usr/local/bin/dectl start
ExecStop=/usr/local/bin/dectl stop
ExecReload=/usr/local/bin/dectl restart

[Install]
WantedBy=multi-user.target
EOF

# 重新加载 systemd 配置
systemctl daemon-reload

# 设置开机自启动
systemctl enable dataease.service
```

**常见问题处理：**

**问题 2: `systemctl: command not found`**

**原因：** 系统未使用 systemd，或 systemctl 命令不在 PATH 中。

**解决方案：**

**方案 A: 如果系统使用 systemd 但命令不在 PATH**
```bash
# 查找 systemctl 位置
which systemctl
# 或
find /usr -name systemctl

# 添加到 PATH
export PATH=$PATH:/usr/bin
```

**方案 B: 如果系统使用 SysV init（如较老的系统）**
```bash
# 使用 service 命令代替
# 创建 init.d 脚本
cat > /etc/init.d/dataease << 'EOF'
#!/bin/bash
# chkconfig: 2345 90 10
# description: DataEase Service

case "$1" in
    start)
        /usr/local/bin/dectl start
        ;;
    stop)
        /usr/local/bin/dectl stop
        ;;
    restart)
        /usr/local/bin/dectl restart
        ;;
    status)
        /usr/local/bin/dectl status
        ;;
    *)
        echo "Usage: $0 {start|stop|restart|status}"
        exit 1
        ;;
esac
EOF

chmod +x /etc/init.d/dataease
chkconfig --add dataease
chkconfig dataease on
```

**方案 C: 手动管理服务（不推荐）**
```bash
# 创建启动脚本
cat > /opt/dataease2.0/start.sh << 'EOF'
#!/bin/bash
cd /opt/dataease2.0
/usr/local/bin/dectl start
EOF

chmod +x /opt/dataease2.0/start.sh

# 添加到 /etc/rc.local（如果存在）
echo "/opt/dataease2.0/start.sh" >> /etc/rc.local
chmod +x /etc/rc.local
```

---

### 步骤 10: 启动 DataEase 服务

**操作内容：**
- 启动 DataEase 服务
- 验证服务状态

**执行命令：**
```bash
# 启动服务（使用 systemctl）
systemctl start dataease

# 或直接使用 dectl
dectl start

# 查看服务状态
dectl status

# 查看服务日志
dectl logs

# 或查看 Docker 容器状态
docker ps
docker-compose -f /opt/dataease2.0/docker-compose.yml ps
```

**验证服务启动：**
```bash
# 检查容器是否运行
docker ps | grep dataease

# 检查服务端口（默认 80）
netstat -tlnp | grep 80
# 或
ss -tlnp | grep 80

# 访问 Web 界面
curl http://localhost
```

**预期结果：**
- DataEase 容器正常运行
- MySQL 容器正常运行
- 80 端口监听正常
- Web 界面可以访问

---

## 四、安装后验证

### 4.1 服务状态检查

```bash
# 检查所有容器状态
dectl status

# 检查容器日志
dectl logs

# 检查特定容器日志
docker logs dataease
docker logs mysql
```

### 4.2 Web 界面访问

1. 打开浏览器访问：`http://服务器IP地址`
2. 默认管理员账号：
   - 用户名：`admin`
   - 密码：`DataEase@123456`（请参考实际安装文档）

### 4.3 功能验证

- [ ] 登录界面正常显示
- [ ] 可以成功登录
- [ ] 数据源连接测试正常
- [ ] 可以创建数据集
- [ ] 可以创建仪表板

---

## 五、常见问题排查

### 5.1 环境变量替换失败

**症状：** `envsubst: command not found`

**解决：** 安装 gettext 包（见步骤 3）

### 5.2 服务管理命令不存在

**症状：** `systemctl: command not found`

**解决：** 
- 检查系统是否使用 systemd
- 如不使用 systemd，使用 service 或手动管理（见步骤 9）

### 5.3 Docker 容器无法启动

**排查步骤：**
```bash
# 查看容器日志
docker logs <container_name>

# 检查 Docker 服务状态
systemctl status docker

# 检查磁盘空间
df -h

# 检查内存使用
free -h
```

### 5.4 端口被占用

**排查步骤：**
```bash
# 检查端口占用
netstat -tlnp | grep 80
# 或
lsof -i :80

# 修改配置文件中的端口
vi /opt/dataease2.0/conf/dataease.yml
```

### 5.5 镜像加载失败

**排查步骤：**
```bash
# 检查镜像文件完整性
md5sum dataease-v2.10.17.tar

# 检查磁盘空间
df -h

# 手动加载镜像
docker load -i <image_file>
```

---

## 六、卸载步骤

如需卸载 DataEase，可按以下步骤操作：

```bash
# 1. 停止服务
dectl stop
# 或
systemctl stop dataease

# 2. 禁用开机自启
systemctl disable dataease

# 3. 删除服务文件
rm -f /etc/systemd/system/dataease.service
systemctl daemon-reload

# 4. 删除 dectl 工具
rm -f /usr/local/bin/dectl /usr/bin/dectl

# 5. 删除运行目录（谨慎操作，会删除所有数据）
# rm -rf /opt/dataease2.0

# 6. 删除 Docker 镜像（可选）
docker rmi dataease:v2.10.17
docker rmi mysql:8.4.5
```

---

## 七、维护建议

### 7.1 定期备份

```bash
# 备份数据目录
tar -czf dataease-backup-$(date +%Y%m%d).tar.gz /opt/dataease2.0/data

# 备份配置文件
tar -czf dataease-conf-backup-$(date +%Y%m%d).tar.gz /opt/dataease2.0/conf
```

### 7.2 日志管理

```bash
# 查看日志
dectl logs

# 清理旧日志（谨慎操作）
find /opt/dataease2.0/logs -name "*.log" -mtime +30 -delete
```

### 7.3 性能监控

```bash
# 监控容器资源使用
docker stats

# 监控磁盘使用
df -h /opt/dataease2.0
```

---

## 八、参考信息

- DataEase 官方文档：https://dataease.io/docs/
- Docker 官方文档：https://docs.docker.com/
- Docker Compose 文档：https://docs.docker.com/compose/

---

## 附录：快速安装检查清单

- [ ] 系统环境检查完成
- [ ] 运行目录创建完成
- [ ] 安装文件复制完成
- [ ] envsubst 工具已安装
- [ ] dectl 工具安装完成
- [ ] Docker 已安装并运行
- [ ] Docker Compose 已安装
- [ ] DataEase 镜像加载完成
- [ ] MySQL 镜像加载完成
- [ ] 系统服务配置完成
- [ ] systemctl 可用或已使用替代方案
- [ ] 服务启动成功
- [ ] Web 界面访问正常

---

**文档版本：** v1.0  
**最后更新：** 2024年  
**适用版本：** DataEase 2.10.17

