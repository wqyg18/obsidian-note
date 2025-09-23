## 1. 更新包索引

`sudo apt update sudo apt upgrade -y`

---

## 2. 安装 PostgreSQL

直接安装官方的 Ubuntu 包：

`sudo apt install -y postgresql postgresql-contrib`

安装完成后，PostgreSQL 会自动作为一个服务启动。

---

## 3. 检查服务状态

`sudo service postgresql status`

