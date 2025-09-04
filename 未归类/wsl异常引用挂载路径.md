wsl启动时输出
```powershell
(base) PS C:\Users\guxin> wsl -d Ubuntu
<3>WSL (337) ERROR: UtilTranslatePathList:2852: Failed to translate E:\dev\7z2408-extra
<3>WSL (337) ERROR: UtilTranslatePathList:2852: Failed to translate E:\dev\7-Zip
<3>WSL (337) ERROR: UtilTranslatePathList:2852: Failed to translate E:\dev\vcpkg
```
证明有些变量还在windows本地的环境变量里面,或者在注册表里面

对于注册表,需要查找然后删除

```powershell
Get-ChildItem -Path "$env:USERPROFILE" -Recurse -Include *.json,*.ps1,*rc | Select-String -Pattern "E:\\dev"


# output

Documents\PowerShell\Microsoft.PowerShell_profile.ps1:1:$env:VCPKG_ROOT = "E:\dev\vcpkg"
Documents\PowerShell\Microsoft.PowerShell_profile.ps1:3:$env:PATH = "E:\dev\7-Zip;$env:PATH"
Documents\PowerShell\Microsoft.PowerShell_profile.ps1:4:$env:PATH = "E:\dev\7z2408-extra;$env:PATH"
```

删除方法

```powershell
notepad "$PROFILE"

# 然后删除相关的行
```

注意这里的是因为powershell的ps1文件里面我之前写的一个环境配置导致的.