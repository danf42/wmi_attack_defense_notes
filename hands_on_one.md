## Hands On 1
### Retrieve the following information from your local machine using WMI/CIM cmdlets

#### To get information on install antivirus product
`Get-WmiObject -Namespace "root\SecurityCenter" -class AntiVirusProduct`

`Get-CimInstance -ClassName AntiVirusProduct -Namespace root\SecurityCenter`

`Get-WmiObject -Namespace "root\SecurityCenter" -Query "select * from AntiVirusProduct"`

#### To get a listing of all files on a computer
`Get-WmiObject -Namespace "root\cimv2" -query "select * from CIM_DataFile" | select Name`

`Get-CimInstance -Namespace "root\cimv2" -query "select * from CIM_DataFile" | select Name`

#### To get a list of all services
`Get-WmiObject -Namespace "root\cimv2" -query "select * from Win32_Service" | Select-Object name,processid,state | ft`

#### To get a list of all running services
`Get-WmiObject -Namespace "root\cimv2" -query "select * from Win32_Service where state = 'Running'" | Select-Object name,processid,state | ft`

#### To get a list of all properties associated with the Processor
`get-wmiobject -namespace root\cimv2 -query "select * from Win32_Processor" | Format-List *`

#### To get the architecture of the Processor 
`get-wmiobject -namespace root\cimv2 -query "select architecture,addresswidth from Win32_Processor"`

#### To get logged on users 
`get-wmiobject -namespace root\cimv2 -class Win32_LoggedOnuser`

`Get-WmiObject -Namespace root\cimv2 -query "select * from Win32_LoggedOnuser"`

`Get-WmiObject -Namespace root\cimv2 -query "select * from Win32_LoggedOnuser" | select Antecedent,Dependent`

#### To get Logon session information (this can be correlated with logged on users)
`Get-WmiObject -Namespace root\cimv2 -query "select * from Win32_logonsession" | select LogonId,LogonType,StartTime`	

#### To Get a list of patches installed
`Get-WmiObject -Namespace root\cimv2 -class "win32_quickfixengineering"`

#### To get security logs
`Get-WmiObject -Namespace root\cimv2 -Class Win32_NTEventLogFile`

`Get-WmiObject -Class Win32_NTEventLogFile | Where-Object {$_.LogFileName -eq 'security'}`

#### To Get command line aruments to start processes
`Get-WmiObject -Namespace root\cimv2 -Class win32_process | Select-Object Caption,Processid,Commandline |ft`

#### To Get list of services and the path to their executable
`Get-WmiObject -Namespace root\cimv2 -Class win32_service | Select-Object Name,Displayname,state,pathname`
