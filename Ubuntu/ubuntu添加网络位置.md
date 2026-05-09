### 网站挂载到本地目录

创建挂载点

sudo mkdir -p /mnt/at

创建凭据

sudo mkdir -p /etc/samba

sudo nano /etc/samba/at.creds

填入 

username=kdrobot

password=kdrobot2025

添加权限

sudo chmod 600 /etc/samba/at.creds

添加挂载规则 file systems table

sudo nano /etc/fstab

填入

//10.83.1.146/Public  /mnt/at  cifs  credentials=/etc/samba/at.creds,iocharset=utf8,_netdev  0  0

无需重启，直接挂载

sudo mount -a

在文件资源管理器中 /mnt/at 即可查看