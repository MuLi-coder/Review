# ssh (windows)
这篇文档来说明一下windows系统从零开始使用ssh协议将本地和github远程仓库建立连接

## 一、创建ssh key
终端中执行:  
> *ssh-keygen -t ed25519 -C "你的邮箱"*

简单解释： 
1. `ssh-keygen` 表示ssh-key的生成命令`(gen:generater)`
2. `-t`表示`type`后面的参数`ed22519`是加密方式
3. `-C`表示注释

然后一路回车即可  
这样就创建完成了一对公钥和密钥，其中公钥给到github账户，私钥留在本地

## 二、查看公钥，上传账户
执行：  
> *cat ~\.ssh\id_ed25519.pub*

简单解释：
1. `cat`表示查看文件内容
2. `~`表示默认家路径`C:\Users\ml`
3. 注意路径：`~\.ssh`是一路回车的默认保存地址，就使用这个默认地址就可以了，不需要更改

然后在GitHub账户中上传ssh公钥就行

## 三、更改配置，更换端口

这个时候我们直接执行链接命令:

> *ssh -T git@github.com*

这个时候可能会有两种情况出现
1. 需要进行yes/no的选择，直接选择yes就可以，然后就会返回hello的内容，表明连接成功
2. 会报链接失败
> *ssh: connect to host github.com port 22: Connection refused*

这个时候很有可能：
- 校园网，企业网的防火墙会拦截向外部ip的22端口传输信息的请求
- 加速器更改DNS解析过程导致返回的ip地址出错，出现bug

所以有几种解决方式