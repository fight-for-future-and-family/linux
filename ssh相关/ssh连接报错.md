## Received disconnect from host: 2: Too many authentication failures for root
> [click here](https://serverfault.com/questions/36291/how-to-recover-from-too-many-authentication-failures-for-user-root)  

```
If you get the following SSH Error:

$ Received disconnect from host: 2: Too many authentication failures for root
This could happen if you have (default on my system) five or more DSA/RSA identity files stored in your .ssh directory. In this case if the -i option isn't specified at the command line the ssh client will first attempt to login using each identity (private key) and next prompt for password authentication. However, sshd drops the connection after five bad login attempts (again default may vary).

大致意思是：如果在默认的  .ssh  目录下，存在好几个 DSA/RSA 认证文件，如果在连接的时候没有在命令行显示的指定 -i ，那么ssh客户端将会尝试用每一个私钥进行登录，那么 sshd 服务将会在达到一定失败次数之后 丢掉 这个连接。 参数为 ** MaxAuthTries **

So if you have a number of private keys in your .ssh directory you could disable Public Key Authentication at the command line using the -o optional argument.

For example:

$ ssh -o PubkeyAuthentication=no root@host
```

```
Host    lvsDR
        HostName 192.168.0.66
        PubkeyAuthentication no
        User root
        Port 22

```