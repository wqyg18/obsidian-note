```powershell
# 查看当前dns服务器
nslookup whoami.ds.akahelp.net 2>NUL | findstr /C:"Name:" /C:"Address:"

# 如果不对,更换为正确服务器
Get-NetAdapter -Physical | Format-Table Name,Status

Set-DnsClientServerAddress -InterfaceAlias "WLAN" -ServerAddresses 223.5.5.5,223.6.6.6
```