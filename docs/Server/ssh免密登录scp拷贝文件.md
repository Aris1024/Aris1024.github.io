# ssh 免密登录 SCP 拷贝文件

1. 进入~/.ssh目录,拷贝id_rsa.pub内容
2. pbcopy < ~/.ssh/id_rsa.pub
3. 登录服务器
4. 进入 ~/.ssh
5. 在 authorized_keys文件中添加刚刚的公钥即可
6. 注意,要是没有authorized_keys 就创建  vi authorized_keys
7. 添加完成后,需要修改权限 (-rw-r--r--)

```shell
chmod 644 ~/.ssh/authorized_keys
ls -ll
-rw-r--r-- 1 bhjr bhjr 1539 2月  20 15:42 authorized_keys
```

[参考地址](https://blog.csdn.net/u013197629/article/details/73608613)



测试拷贝文件,不用输入密码即可成功

```shell
scp ./TF_redeem_pupup.png bhjr@10.20.200.101:/app/public/file/common/app/test/zjl_test/
```



可选: 配置服务器

在本地 ~/.ssh目录下,查找config文件,没有就vi创建

添加一下配置(参考)

```shell
Host 101
   User bhjr
   Hostname 10.20.200.101
   IdentityFile ~/.ssh/id_rsa

Host 111
   User bhjr
   Hostname 10.20.200.111
   IdentityFile ~/.ssh/id_rsa

Host 121
   User bhjr
   Hostname 10.20.200.121
   IdentityFile ~/.ssh/id_rsa
Host vps
   User root
   Hostname 23.105.222.131
   Port 27953
   IdentityFile ~/.ssh/id_rsa
```

然后拷贝文件的时候就可以不用输入IP, 直接输入机器Host别名就行,例如

```shell
scp ./TF_redeem_pupup.png 101:/app/public/file/common/app/test/zjl_test/
```

