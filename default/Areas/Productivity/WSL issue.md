
In powershell,
```
Restart-Service -Name vmcompute
```




[Orgul Eroglu] 10-01-2024 09:50
Restart-Service -Name vmms

[이세환] 10-01-2024 09:50
Ok, I'll do

[이세환] 10-01-2024 09:51
Cannot find any service with service name "vmms"

[이세환] 10-01-2024 09:51
Error message like that

[이세환] 10-01-2024 09:51
I run like
> Restart-Service -Name vmms

[Orgul Eroglu] 10-01-2024 09:51
Get-Service

[Orgul Eroglu] 10-01-2024 09:52
 Get-Service|findstr -i vm

[이세환] 10-01-2024 09:52
there is vmcompute

[이세환] 10-01-2024 09:52
and there are many vm* services

[Orgul Eroglu] 10-01-2024 09:52
Can you restart that   ?

[이세환] 10-01-2024 09:52
vmcompute ?

[Orgul Eroglu] 10-01-2024 09:53
yes

[Orgul Eroglu] 10-01-2024 09:53
maybe it is not the one

[Orgul Eroglu] 10-01-2024 09:53
The vmcompute.exe service, also known as the Hyper-V Host Compute Service, is a Windows service that allows users to launch and manage Windows containers.

[이세환] 10-01-2024 09:53
Oh, it looks ok