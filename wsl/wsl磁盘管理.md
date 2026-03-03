默认的wsl直接在wsl中删除文件并不会释放磁盘给windows
[Advanced settings configuration in WSL \| Microsoft Learn](https://learn.microsoft.com/en-us/windows/wsl/wsl-config#experimental-settings)

## .wslconfig

需要配置`.wslconfig`

```wslconfig
[wsl2]
networkingMode=mirrored

[experimental]
autoMemoryReclaim=gradual  # 自动回收内存
sparseVhd=true             # 启用稀疏 VHD，允许自动收缩磁盘(但是改配置只针对新建的vhdx, 也就是修改配置之前已有的vhdx无法使用)
```

但是注意, sparseVhd=true, 启用稀疏 VHD，允许自动收缩磁盘(但是改配置只针对新建的vhdx, 也就是修改配置之前已有的vhdx无法使用)

针对已经存在的wsl, 可以通过

`wsl --manage <DistroName> --set-sparse true`

命令将其转换为`sparseVhd`
但是可能会失败, 此时只能手动去进行清理, 不过社区有一个工具`wslcompact`

或者直接强制执行 
```pwsh
wsl --manage Ubuntu --set-sparse true --allow-unsafe
```
说是有损坏数据的风险
## wslcompact
[wslcompact](https://github.com/okibcn/wslcompact)
```
scoop bucket add .oki https://github.com/okibcn/Bucket
scoop install wslcompact
```
执行`wslcompact.cmd`会显示所有的distro, 以及预估的可以压缩的大小

执行`wslcompact -c distro_name`

其原理就是导出镜像, 然后再导入