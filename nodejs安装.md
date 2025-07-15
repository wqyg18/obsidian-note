```bash
# 1. 通过 Gitee 镜像安装 nvm
curl -o- https://gitee.com/mirrors/nvm/raw/v0.39.7/install.sh | bash

# 2. 让 nvm 立即生效
source ~/.bashrc
```

```bash
# 临时指定国内镜像，再安装所需版本（示例 20.17.0）
NVM_NODEJS_ORG_MIRROR=https://mirrors.aliyun.com/nodejs-release \
nvm install 20.17.0

# 完成后设为默认版本
nvm alias default 20.17.0
nvm use 20.17.0
```

```bash
# 全局设置为淘宝镜像（npmmirror）
npm config set registry https://registry.npmmirror.com

# 验证
npm config get registry
```

```bash
# 1. 用刚才的国内镜像安装 nrm
npm i -g nrm --registry https://registry.npmmirror.com

# 2. 添加/更新国内源
nrm use taobao

# 3. 查看当前可用源
nrm ls
```