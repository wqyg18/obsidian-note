```powershell
Get-ChildItem "HKCU:\Software\Microsoft\Windows\CurrentVersion\Lxss"
```
```powershell
(base) PS C:\Users\guxin> Get-ChildItem "HKCU:\Software\Microsoft\Windows\CurrentVersion\Lxss"

    Hive: HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Lxss

Name                           Property
----                           --------
{4fa9cb6c-a75d-472b-a005-27637 State             : 1
e3d7a77}                       DistributionName  : Ubuntu
                               Version           : 2
                               BasePath          : C:\Users\guxin\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu_7
                               9rhkp1fndgsc\LocalState
                               Flags             : 15
                               DefaultUid        : 1000
                               PackageFamilyName : CanonicalGroupLimited.Ubuntu_79rhkp1fndgsc
                               Flavor            : ubuntu
                               OsVersion         : 24.04
{79e2f42a-9722-44c9-a7f6-d9d5b State             : 1
aaf0ceb}                       DistributionName  : Ubuntu-24.04
                               Version           : 2
                               BasePath          : C:\Users\guxin\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu24
                               .04LTS_79rhkp1fndgsc\LocalState
                               Flags             : 15
                               DefaultUid        : 1000
                               PackageFamilyName : CanonicalGroupLimited.Ubuntu24.04LTS_79rhkp1fndgsc
                               Flavor            : ubuntu
                               OsVersion         : 24.04
{d025f9ad-1745-4654-bf6b-91711 State               : 1
31760ff}                       DistributionName    : Ubuntu-20.04
                               Version             : 2
                               BasePath            : \\?\D:\code\linux\wsl\ubuntu_2004
                               Flags               : 15
                               DefaultUid          : 0
                               RunOOBE             : 0
                               VhdFileName         : ext4.vhdx
                               Flavor              : ubuntu
                               OsVersion           : 20.04
                               Modern              : 1
                               ShortcutPath        : C:\Users\guxin\AppData\Roaming\Microsoft\Windows\Start Menu\Ubuntu
                               -20.04.lnk
                               TerminalProfilePath : C:\Users\guxin\AppData\Local\Microsoft\Windows Terminal\Fragments\
                               Microsoft.WSL\{a454f65a-02dd-58c2-85d7-6f4889727213}.json
```

## 简洁版
```powershell
Get-ChildItem "HKCU:\Software\Microsoft\Windows\CurrentVersion\Lxss" |
  ForEach-Object {
    [PSCustomObject]@{
      DistributionName = (Get-ItemProperty $_.PsPath).DistributionName
      BasePath         = (Get-ItemProperty $_.PsPath).BasePath
    }
  }
```

```powershell
DistributionName BasePath
---------------- --------
Ubuntu           C:\Users\guxin\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu_79rhkp1fndgsc\LocalState
Ubuntu-24.04     C:\Users\guxin\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu24.04LTS_79rhkp1fndgsc\LocalState
Ubuntu-20.04     \\?\D:\code\linux\wsl\ubuntu_2004
```