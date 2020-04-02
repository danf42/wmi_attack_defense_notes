### Hands On 2: Using Methods:
#### List all the methods named "Create" in the root\cimv2 namespace
`Get-CimClass -namespace root\cimv2 -MethodName create`
		
#### List all the methods named "Create" in all the namespaces
Get-WmiNamespace: [PowerShell Magazine #PSTip List all WMI namespaces on a system](https://www.powershellmagazine.com/2013/10/18/pstip-list-all-wmi-namespaces-on-a-system/)
```
$namespaces = Get-WmiNamespace

foreach ($ns in $namespaces){
  Get-CimClass -namespace $ns -MethodName create
}
```
		
#### List owners of all processes running on your machine
```
$processes = Get-WmiObject -class win32_process
foreach ($process in $processes){
  $name = ($process).Name
  try{
    $owner = ($process).GetOwner().user
    Write-Output "$name   $owner"
  } catch { continue }
}
```
