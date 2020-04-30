## Hands On 3 - Associators 

#### Get information about a particular process:
`Get-WmiObject -query "select * from Win32_Process where name = 'notepad++.exe'"`

#### Get information about all the processes:
`Get-WmiObject -class Win32_Process | select __RELPATH,Handle,ProcessName | ft`

#### Get information about Process Owner ID and Process ID:
`Get-WmiObject Win32_SessionProcess | select Antecedent,Dependent | ft`

#### Get the classes were information is pulled from:
`Get-WmiObject -query "Associators of {Win32_Process.Handle=7692} where classdefsonly"`

#### Get the "Associators of" information:
`Get-WmiObject -query "Associators of {Win32_Process.Handle=7692}"`

#### Get the "References of" informaiton:
`Get-WmiObject -query "References of {Win32_Process.Handle=7692}"`

#### Using Process ID, get the Owner of the process (this will only work if the process has an Associator class Win32_LogonSession)
`Get-WmiObject -query "Associators of {Win32_Process.Handle=$proc_handle} where resultclass=Win32_LogonSession"`

#### Using the Owner ID, attempt to get the name of the owners
`Get-WmiObject -Query "ASSOCIATORS OF {Win32_LogonSession.LogonId=$login_id} WHERE ResultClass=Win32_UserAccount"`

#### Get Interesting information about running processes
```
$processObjects = @() 

$procs = Get-WmiObject -query "select * from win32_process"
foreach($proc in $procs){

    [hashtable]$objectProperty = @{}

    $objectProperty.Add('ExecutablePath', $proc.ExecutablePath)
    $objectProperty.Add('Handle', $proc.Handle)
    $objectProperty.Add('ProcessName', $proc.ProcessName)

    #Set default values
    $objectProperty.Add('SID', "UNK")
    $objectProperty.Add('UserName', "UNK")
    $objectProperty.Add('Domain', "UNK")

    $proc_handle = $proc.Handle
    $logon_sessions = Get-WmiObject -query "Associators of {Win32_Process.Handle=$proc_handle} where resultclass=Win32_LogonSession"

    foreach($logon_session in $logon_sessions){
        
        $logon_id = $logon_sessions.LogonId
        $user_account = Get-WmiObject -Query "ASSOCIATORS OF {Win32_LogonSession.LogonId=$logon_id} WHERE ResultClass=Win32_UserAccount"
        $objectProperty['SID'] = $user_account.SID
        $objectProperty['UserName'] = $user_account.Name
        $objectProperty['Domain'] = $user_account.Domain
        
    }

    $processObjects += New-Object -TypeName psobject -Property $objectProperty
}

$processObjects | Select Handle,ProcessName,ExecutablePath,UserName,Domian,SID | ft
```

### List ownwers of all the files in a specific directory
```
$files = Get-WmiObject -query "select * from CIM_DataFile where Path='<insert path>'"

$files | foreach-object {

  $file_name = $_.Name

  $owner = (Get-wmiobject -query "Associators of {Win32_LogicalFileSecuritySetting='$file_name'} where AssocClass=Win32_LogicalFileOwner ResultRole=Owner").AccountName

  Write-Output "$file_name $owner"
}
```
