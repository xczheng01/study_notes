* 安装软件及配置

```shell
yum install vsftpd -y

vi /etc/vsftpd/vsftpd.conf

sed -i 's/^userlist_enable=.*/userlist_enable=NO/g'  /etc/vsftpd/vsftpd.conf

sed -i 's/^anonymous_enable=.*/anonymous_enable=YES/g' /etc/vsftpd/vsftpd.conf

systemctl enable vsftpd

systemctl start vsftpd
```



* 将软件目录software移植/var/ftp/pub/下

```shell
createrepo -v /var/ftp/pub/software
```



* 编写yum文件

```shell
cat <<EOF > /etc/yum.repos.d/ftp.repo
[ftp]
name=ftp
baseurl=ftp://192.168.88.11/pub/software
enabled=1
gpgcheck=0
EOF
```

