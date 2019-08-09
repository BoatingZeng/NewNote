## Linux命令
1. 查看进程：ps -ef | grep mysql
2. 杀进程：kill -s 9 1827
3. 查看文件尾：tail file.log -n 300
4. 后台运行：nohup command &
5. 复制目录时保持权限等：rsync -a 或 cp -a

## unbuntu开机启动sh
svn为例：
1. 在/etc/init.d写个脚本startsvn.sh
```
    #!/bin/bash
    svnserve -d -r /home/svn
```
2. `chmod 777 startsvn.sh`  设置启动权限
3. `update-rc.d startsvn.sh defaults`

## unbuntu桌面版亮度
```
sudo su
echo 50 > /sys/class/backlight/intel_backlight/brightness
```

## 添加PATH
编辑/etc/profile，末尾添加
`export PATH=$PATH:/home/boating/program/node-v8.1.3-linux-x64/bin`
等号两边不能有空格，如果要立刻生效，执行`source profile`

## 挂载windows共享文件夹
```
sudo apt-get install samba
sudo  apt-get install cifs-utils
```

uid和gid可以用id username来查
```
sudo mount.cifs //192.168.18.102/photoscanTasks  ~/share/win10/test -o username=boating,password=boating,uid=1000,gid=1000
```

强行umount
```
umount -l /PATH/OF/BUSY-DEVICE
umount -f /PATH/OF/BUSY-NFS(NETWORK-FILE-SYSTEM)
```

## 挂载ftp
```
sudo apt-get install curlftpfs
mkdir /mnt/my_ftp
curlftpfs ftp-user:ftp-pass@host:port /mnt/my_ftp/
```