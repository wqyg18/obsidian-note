[GitHub - duzyn/scoop-cn: 中国用户能用的 Scoop 应用库，每日同步 Scoop 的官方库，加速应用的下载速度](https://github.com/duzyn/scoop-cn)

首先
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```
然后
```powershell
irm https://cdn.jsdelivr.net/gh/duzyn/scoop-cn/install.ps1 | iex
```

scoop安装完毕, 接下来进行make和tree的安装

首先

```powershell
scoop install make
scoop install eza
```

```powershell
notepad $PROFILE
```

在其中编辑(前面是git相关, 终端输出中文不会乱码)
```txt
# 设置环境编码为 UTF-8
$OutputEncoding = [System.Text.Encoding]::UTF8
[Console]::InputEncoding = [Console]::OutputEncoding = [System.Text.Encoding]::UTF8

# 让 Git 明确使用 UTF-8 输出
$env:LESSCHARSET = "utf-8"

function tree {
    eza --tree --group-directories-first $args
}
```

然后就可以直接使用`make`或者`tree`命令了
```bash
tree -d
tree -f
tree -L 2
tree -a -L 1 -I .gitignore
tree -a -L 1 -I ".gitignore|.git"
```

可以安装一个`fzf`, 非常好用, 直接替换`Ctrl+r`
```
scoop install fzf
```

注册到`Ctrl+r`
```powershell
notepad $PROFILE
```
```ps1
Set-PSReadLineKeyHandler -Chord Ctrl+r -ScriptBlock {
    $historyPath = (Get-PSReadLineOption).HistorySavePath

    if (Test-Path $historyPath) {
        $command = Get-Content $historyPath |
            Where-Object { $_.Trim() -ne "" } |
            Get-Unique -AsString |
            fzf --tac --height 40%

        if ($command) {
            [Microsoft.PowerShell.PSConsoleReadLine]::RevertLine()
            [Microsoft.PowerShell.PSConsoleReadLine]::Insert($command)
        }
    }
}
```
```powershell
. $PROFILE
```

