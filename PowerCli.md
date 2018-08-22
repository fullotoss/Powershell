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

## Stop SSH on all the host
```powershell
Get-VMHost -name 172.31.14.* | Foreach {Stop-VMHostService -HostService ($_ | Get-VMHostService | Where { $_.Key -eq "TSM-SSH"} )}
```


## Modify gateway
```powershell
$vmHostNetworkInfo = Get-VmHostNetwork -Host m2vcdesx010101
Set-VmHostNetwork -Network $vmHostNetworkInfo -VMKernelGateway 172.31.14.62
```

## Remove vSwitch
```powershell
$vswitch =  Get-VirtualSwitch -VMHost $vmhost -Name vSwitch1
Remove-VirtualSwitch -VirtualSwitch $vswitch -confirm:$false
```

## Adding static routes
```powershell
New-VMHostRoute -VMHost $vmhost -Destination 10.135.2.72 -Gateway 172.31.14.62 -PrefixLength 32 -Confirm:$false
New-VMHostRoute -VMHost $vmhost -Destination 10.125.8.120 -Gateway 172.31.14.62 -PrefixLength 32 -Confirm:$false
New-VMHostRoute -VMHost $vmhost -Destination 10.117.0.10 -Gateway 172.31.14.62 -PrefixLength 32 -Confirm:$false
```

## Removing static routes
```powershell
$destIpList = ('10.117.0.10', '10.125.8.120')
foreach ($vmhost in (Get-VMHost -Name 172.31.14.[1-3]*))
{
	$routes = Get-VMHostRoute -VMHost $vmhost | where {$destIpList -contains $_.Destination.IPAddressToString}
	Remove-VMHostRoute -VMHostRoute $routes -Confirm:$false
}
```

## Check route table
```powershell
Get-VMHostRoute -VMHost 172.31.14.21
```

## Create vSwitch1 on the ESXi host
```powershell
$vs = New-VirtualSwitch -Name vSwitch1 -VMHost $vmhost
New-VirtualPortGroup -VirtualSwitch $vs -Name Customer_24050-558-Management-M1VLN12521001 -VLanID 558
New-VirtualPortGroup -VirtualSwitch $vs -Name Customer_24050-2305-Tst-MXVLN12521001 -VLanID 2305
New-VirtualPortGroup -VirtualSwitch $vs -Name Customer_24050-2311-Database-MXVLN12521001 -VLanID 2311
New-VirtualPortGroup -VirtualSwitch $vs -Name Customer_24050-2304-Dev-MXVLN12521001 -VLanID 2304
New-VirtualPortGroup -VirtualSwitch $vs -Name Customer_24050-2303-App-MXVLN12521001 -VLanID 2303
New-VirtualPortGroup -VirtualSwitch $vs -Name Customer_24050-2336-ZCCC-MXVLN12521001 -VLanID 2336
New-VirtualPortGroup -VirtualSwitch $vs -Name Customer_24050-2302-DMZ-MXVLN12521001 -VLanID 2302
```


## Remove vmnic3 from VDS
```powershell
Get-VMhost -Name $vmhost | Get-VMHostNetworkAdapter -Physical -Name vmnic3 | Remove-VDSwitchPhysicalNetworkAdapter -confirm:$false
```

```powershell
$vs = Get-VirtualSwitch -name vSwitch1 -VMHost $vmhost
Set-VirtualSwitch -VirtualSwitch $vs -Nic vmnic3 -confirm:$false
```


## Add vmnic3 to vSwitch1
```powershell
$vs = Get-VirtualSwitch -name vSwitch1 -VMHost mtvcdesx02033
Set-VirtualSwitch -VirtualSwitch $vs -Nic vmnic3 
#if you need to add more than 1 adapter
#Set-VirtualSwitch -VirtualSwitch $vs -Nic vmnic3,vmnic2
```


## Remove vmnic3 from VDS and add it to vSwutch1
```powershell
Get-VMhost -Name $vmhost | Get-VMHostNetworkAdapter -Physical -Name vmnic3 | Remove-VDSwitchPhysicalNetworkAdapter -confirm:$false
$vs = Get-VirtualSwitch -name vSwitch1 -VMHost $vmhost
Set-VirtualSwitch -VirtualSwitch $vs -Nic vmnic3  #confirm is a must!
```



## Create management network on vSwitch0
```powershell
$hostname = "m2vcdesx02169*"
$IP = "172.31.14.25"
$Mask = "255.255.255.192"
$VLAN = "2111"
$vswitch0 =  Get-VirtualSwitch -VMHost $hostname -Name vSwitch0
New-VMHostNetworkAdapter -VMHost $hostname -VirtualSwitch $vswitch0 -PortGroup ManagementNetwork -IP $IP -SubnetMask $Mask 
$portGroup = Get-VirtualPortgroup -Name "ManagementNetwork" -VirtualSwitch $vswitch0
Set-VirtualPortGroup -VirtualPortGroup $portGroup -VlanId $VLAN
Get-VMHost -Name $hostname | Get-VMHostNetworkAdapter -VMKernel | where { $_.IP -eq $IP } | Set-VMHostNetworkAdapter -ManagementTrafficEnabled $true -confirm:$false
```








