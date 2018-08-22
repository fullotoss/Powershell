## Basic tasks on vCenter using PowerCLI:

* [#Run commands on multiple hosts](#Run-commands-on-multiple-hosts)
* [#Add a new PortGroup to an existing vSwitch](#Add-a-new-PortGroup-to-an-existing-vSwitch)


## Run commands on multiple hosts
```powershell
foreach ($vmhost in (Get-VMHost -Name 172.31.14.[1-3]*)){
  #commands
} 
```

## Add a new PortGroup to an existing vSwitch
```powershell
$vs = Get-VirtualSwitch -Name vSwitch0 -VMHost m2vcdesx21201*
New-VirtualPortGroup -VirtualSwitch $vs -Name Customer_24050-V2480-ESXManagement-MXVLN12521001  -VLanID 2480
```

## Start SSH on all the hosts
```powershell
Get-VMHost -name 172.31.14.* | Foreach {Start-VMHostService -HostService ($_ | Get-VMHostService | Where { $_.Key -eq "TSM-SSH"} )}
```









