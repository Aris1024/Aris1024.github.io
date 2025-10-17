# acme.sh 配置管理SSL证书

> 背景: 腾讯云已购买域名,并且已经解析到VPS的IP地址,现在在要使用https服务,需要配置SSL证书.

1. 工具: acme.sh
        [文档地址: https://docs.dnspod.cn/dns/acme-sh](https://docs.dnspod.cn/dns/acme-sh)
        需要腾讯云准备子账号和自定义策略,然后让二者关联即可,需要获取子账号的SecretId和SecretKey,具体参考上面的文档,很详细.

2. 安装:
        因为要从GitHub下载包,所以需要开启代理
            `curl https://get.acme.sh | sh -s email=youremail@xx.com`
        安装以后,根据提示,
            安装的目录: Installing to /root/.acme.sh
            安装的位置: Installed to /root/.acme.sh/acme.sh
            配置好的alias: Installing alias to '/root/.bashrc'
        下一步重启终端: Close and reopen your terminal to start using acme.sh

3. 申请证书:
        这个提示网络连接错误,开启代理再执行(proxyon)
            Please refer to https://curl.haxx.se/libcurl/c/libcurl-errors.html for error code: 35
            Please refer to https://curl.haxx.se/libcurl/c/libcurl-errors.html for error code: 28
        执行命令:
            `acme.sh --issue --force --dns dns_tencent -d yourdomain.com -d *.yourdomain.com --log`

4. 使用证书:
        执行命令:
            `acme.sh --install-cert -d 'yourdomain.com' --key-file /app/ssl/yourdomain.com.key --fullchain-file /app/ssl/yourdomain.com.fullchain.cer  --reloadcmd "systemctl restart nginx"`

5. 强制更新证书:
        执行命令:
            `acme.sh --renew -d yourdomain.com --force`
        更新完成后记得执行使用证书的命令
            `acme.sh --install-cert -d 'yourdomain.com' --key-file /app/ssl/yourdomain.com.key --fullchain-file /app/ssl/yourdomain.com.fullchain.cer  --reloadcmd "systemctl restart nginx"`
