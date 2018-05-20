### 允许某个人从某个地方进行登录
#####  /etc/ssh/sshd_config
#####  只允许root从 1.2.3.4 和 cptest1从任何地方进行登录
```
AllowUsers  root@1.2.3.4 cptest1
AllowGroups root

DenyUsers sk ostechnix
DenyGroups root
```

---
### 针对某个IP做特定的限制可以登录
```
# general config
PermitRootLogin no 

# the following overrides the general config when conditions are met. 
Match Address  192.168.0.*
    PermitRootLogin yes
```