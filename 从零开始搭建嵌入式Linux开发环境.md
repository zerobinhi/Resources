# 将用户添加到sudoers 组中

```bash
su
```

```bash
sudo usermod -aG sudo,dialout,uucp $USER
```

# 在使用 `sudo` 命令时无需输入密码

```bash
sudo bash -c 'echo "zuozhongkai ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers'
```

重启系统

# 安装 VIM

```bash
sudo apt install vim -y
```

配置 VIM

```bash
sudo bash -c 'echo "set tabstop=4      \" 设置 Tab 字符的宽度为 4 个字符" >> /etc/vim/vimrc'
sudo bash -c 'echo "set number         \" 在每一行的左边显示行号" >> /etc/vim/vimrc'
sudo bash -c 'echo "set noexpandtab    \" 使用实际的 Tab 字符，而不是空格" >> /etc/vim/vimrc'
sudo bash -c 'echo "set shiftwidth=4   \" 设置缩进时使用的字符数为 4" >> /etc/vim/vimrc'
```

# FTP 服务(文件互传)

安装 FTP 服务

```bash
sudo apt install vsftpd -y
```

配置 FTP 服务

```bash
sudo vim /etc/vsftpd.conf
```

删除 write_enable=YES 前面的的 '#'

重启FTP 服务

```bash
sudo /etc/init.d/vsftpd restart
```

# NFS 服务开启

```bash
sudo apt install nfs-kernel-server rpcbind -y
```

配置 NFS

```bash
sudo bash -c 'echo "/home/zuozhongkai/linux/nfs *(rw,sync,no_root_squash)" >> /etc/exports'
```

重启NFS 服务

```bash
sudo /etc/init.d/nfs-kernel-server restart
```

# 搭建TFTP

安装 tftp-hpa 和tftpd-hpa，命令如下：

```bash
sudo apt install tftp-hpa tftpd-hpa xinetd -y
```

创建一个文件夹

```bash
mkdir -p /home/zuozhongkai/linux/tftpboot
sudo chmod 777 /home/zuozhongkai/linux/tftpboot
```

配置tftp

```bash
sudo vim /etc/xinetd.d/tftp
```

输入如下内容

```tex
server tftp
{
	socket_type = dgram
	protocol = udp
	wait = yes
	user = root
	server = /usr/sbin/in.tftpd
	server_args = -s /home/zuozhongkai/linux/tftpboot/
	disable = no
	per_source = 11
	cps = 100 2
	flags = IPv4
}
```

启动 tftp 服务，命令如下：

```bash
sudo service tftpd-hpa start
```

打开/etc/default/tftpd-hpa 文件

```bash
sudo vim /etc/default/tftpd-hpa
```

替换为以下内容

```tex
# /etc/default/tftpd-hpa

TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/home/zuozhongkai/linux/tftpboot"
TFTP_ADDRESS=":69" 
TFTP_OPTIONS="-l -c -s"
```

重启 tftp 服务器

```bash
sudo service tftpd-hpa restart
```



# 其他重要文件

```bash
sudo apt install git zlib1g-dev unzip gcc g++ aptitude make build-essential libncurses-dev u-boot-tools traceroute openssh-server rsync libacl1-dev bc python3 python3-pip python3-setuptools python3.12-venv gcc-arm-none-eabi bison flex libssl-dev dpkg-dev lzop gcc-arm-linux-gnueabihf g++-arm-linux-gnueabihf -y
```

安装交叉编译工具链

```bash
sudo mkdir /usr/local/arm
sudo cp -f arm-buildroot-linux-gnueabihf.tar.gz /usr/local/arm/
cd /usr/local/arm/
sudo tar zxvf arm-buildroot-linux-gnueabihf.tar.gz
```

```bash
sudo vim /etc/profile
```


在末尾输入如下内容

```tex
export PATH=$PATH:/usr/local/arm/arm-buildroot-linux-gnueabihf/bin
```





