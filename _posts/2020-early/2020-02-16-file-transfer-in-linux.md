---
title: Linux里的文件传输
categories: [Tech]
tags: [Linux, shell]
date: 2020-02-16
---

如果要和 Linux 交换文件怎么办？

<!-- more -->

## scp

命令全称 Secure copy， 用于 ssh 主机间的文件复制，也称为远程拷贝。

```sh
# Copy a local file to a remote host:
scp path/to/local_file remote_host:path/to/remote_file

# Copy a file from a remote host to a local directory:
scp remote_host:path/to/remote_file path/to/local_directory

# Recursively copy the contents of a directory from a remote host to a local directory:
scp -r remote_host:path/to/remote_directory path/to/local_directory

# Copy a file between two remote hosts transferring through the local host:
scp -3 host1:path/to/remote_file host2:path/to/remote_directory

# Use a specific username when connecting to the remote host:
scp path/to/local_file remote_username@remote_host:path/to/remote_directory

# Use a specific ssh private key for authentication with the remote host:
scp -i ~/.ssh/private_key local_file remote_host:/path/remote_file
```

​ 参数说明：

```
-1： 强制scp命令使用协议ssh1
-2： 强制scp命令使用协议ssh2
-4： 强制scp命令只使用IPv4寻址
-6： 强制scp命令只使用IPv6寻址
-B： 使用批处理模式（传输过程中不询问传输口令或短语）
-C： 允许压缩。（将-C标志传递给ssh，从而打开压缩功能）
-p：保留原文件的修改时间，访问时间和访问权限。
-q： 不显示传输进度条。
-r： 递归复制整个目录。
-v：详细方式显示输出。scp和ssh(1)会显示出整个过程的调试信息。这些信息用于调试连接，验证和配置问题。
-c cipher： 以cipher将数据传输进行加密，这个选项将直接传递给ssh。
-F ssh_config： 指定一个替代的ssh配置文件，此参数直接传递给ssh。
-i identity_file： 从指定文件中读取传输时使用的密钥文件，此参数直接传递给ssh。
-l limit： 限定用户所能使用的带宽，以Kbit/s为单位。
-o ssh_option： 如果习惯于使用ssh_config(5)中的参数传递方式，
-P port：注意是大写的P, port是指定数据传输用到的端口号
-S program： 指定加密传输时所使用的程序。此程序必须能够理解ssh(1)的选项。
```

## rsync

`rsync`基本上就是用来替代`scp`的命令，功能更强大，参数也更复杂。支持增量备份，压缩拷贝，删除同步，软链复制等等。

```sh
# Transfer file from local to remote host:
rsync path/to/local_file remote_host:path/to/remote_directory

# Transfer file from remote host to local:
rsync remote_host:path/to/remote_file path/to/local_directory

# Transfer file in [a]rchive (to preserve attributes)
# and compressed ([z]ipped) mode
# with [v]erbose and [h]uman-readable [p]rogress:
rsync -azvhP path/to/local_file remote_host:path/to/remote_directory

# Transfer a directory and all its children from a remote to local:
rsync -r remote_host:path/to/remote_directory path/to/local_directory

# Transfer directory contents (but not the directory itself) from a remote to local:
rsync -r remote_host:path/to/remote_directory/ path/to/local_directory

# Transfer a directory [r]ecursively, in [a]rchive to preserve attributes
# resolving contained soft[l]inks , and ignoring already transferred files [u]nless newer:
rsync -rauL remote_host:path/to/remote_file path/to/local_directory

# Transfer file over SSH and delete local files that do not exist on remote host:
rsync -e ssh --delete remote_host:path/to/remote_file path/to/local_file

# Transfer file over SSH and show global progress:
rsync -e ssh --info=progress2 remote_host:path/to/remote_file path/to/local_file
```

参数说明：

```
-v, --verbose          详细模式输出。
-q, --quiet            精简输出模式。
-c, --checksum         打开校验开关，强制对文件传输进行校验。
-a, --archive          归档模式，表示以递归方式传输文件，并保持所有文件属性，等于 -rlptgoD。
-r, --recursive        对子目录以递归模式处理。
-R, --relative         使用相对路径信息。
-b, --backup           创建备份，也就是对于目的已经存在有同样的文件名时，将老的文件重新命名为 ~filename。可以使用 --suffix 选项来指定不同的备份文件前缀。
--backup-dir           将备份文件（~filename）存放在在目录下。
-suffix=SUFFIX         定义备份文件前缀。
-u, --update           仅仅进行更新，也就是跳过所有已经存在于 DST，并且文件时间晚于要备份的文件。（不覆盖更新的文件。）
-l, --links            保留软链结。
-L, --copy-links       想对待常规文件一样处理软链结。
--copy-unsafe-links    仅仅拷贝指向 SRC 路径目录树以外的链结。
--safe-links           忽略指向 SRC 路径目录树以外的链结。
-H, --hard-links       保留硬链结。
-p, --perms            保持文件权限。
-o, --owner            保持文件属主信息。
-g, --group            保持文件属组信息。
-D, --devices          保持设备文件信息。
-t, --times            保持文件时间信息。
-S, --sparse           对稀疏文件进行特殊处理以节省 DST 的空间。
-n, --dry-run          显示哪些文件将被传输（新增、修改和删除）。
-W, --whole-file       拷贝文件，不进行增量检测。
-x, --one-file-system  不要跨越文件系统边界。
-B, --block-size=SIZE  检验算法使用的块尺寸，默认是 700 字节。
-e, --rsh=COMMAND      指定使用 rsh, ssh 方式进行数据同步。
--rsync-path=PATH      指定远程服务器上的 rsync 命令所在路径信息。
-C, --cvs-exclude      使用和 CVS 一样的方法自动忽略文件，用来排除那些不希望传输的文件。
--existing             仅仅更新那些已经存在于 DST 的文件，而不备份那些新创建的文件。
--delete               删除那些 DST 中 SRC 没有的文件。
--delete-excluded      同样删除接收端那些被该选项指定排除的文件。
--delete-after         传输结束以后再删除。
--ignore-errors        即使出现 IO 错误也进行删除。
--max-delete=NUM       最多删除 NUM 个文件。
--partial              保留那些因故没有完全传输的文件，以便实现断点续传。
--force                强制删除目录，即使不为空。
--numeric-ids          不将数字的用户和组 ID 匹配为用户名和组名。
--timeout=TIME         IP 超时时间，单位为秒。
-I, --ignore-times     不跳过那些有同样的时间和长度的文件。
--size-only            当决定是否要备份文件时，仅仅察看文件大小而不考虑文件时间。
--modify-window=NUM    决定文件是否时间相同时使用的时间戳窗口，默认为 0。
-T --temp-dir=DIR      在 DIR 中创建临时文件。
--compare-dest=DIR     同样比较 DIR 中的文件来决定是否需要备份。
--progress             显示传输过程。
-P                     等同于 -partial -progress。
-z, --compress         对备份的文件在传输时进行压缩处理。
--exclude=PATTERN      指定排除不需要传输的文件模式。
--include=PATTERN      指定不排除而需要传输的文件模式。
--exclude-from=FILE    排除 FILE 中指定模式的文件。
--include-from=FILE    不排除 FILE 指定模式匹配的文件。
--version              打印版本信息。
--address              绑定到特定的地址。
--config=FILE          指定其他的配置文件，不使用默认的 rsyncd.conf 文件。
--port=PORT            指定其他的 rsync 服务端口。
--blocking-io          对远程 shell 使用阻塞 IO。
--stats                给出某些文件的传输状态。
--log-format=formAT    指定日志文件格式。
--password-file=FILE   从 FILE 中得到密码。
--bwlimit=KBPS         限制 I/O 带宽，KBytes per second。
-h, --help             显示帮助信息。
```

## vsftpd

`vsftpd`是 Linux 的 ftp 服务端程序，一般需要单独安装，比如：

```
yum install vsftpd -y
```

修改配置文件前记得备份。

```sh
cp /etc/vsftpd.conf /etc/vsftpd.conf.bak      //备份配置文件
vim /etc/vsftpd.conf
```

如果需要匿名访问，参考以下配置文件。

```
listen=YES                       //FTP处于独立启动模式
anonymous_enable=YES             //是否允许匿名访问,匿名帐户为 ftp和 anonymous
local_enable=YES                 //是否允许本地用户访问
write_enable=YES                 //允许本地用户访问时,是否允许他们有写入的权限
local_umask=022                  //本地用户在写入文件时,这些文件默认的权限
anon_upload_enable=YES           //是否允许匿名用户上传
anon_mkdir_write_enable=YES      //是否允许匿名用户创建目录
dirmessage_enable=YES            //使用者进入某个目录时是否显示由message_file指定的文件内容
xferlog_enable=YES               //是否启用日志
connected_from_port_20=YES       //是否允许从20的连接请求
xferlog_file=/var/log/vsftpd.log //日志文件的位置
xferlog_std_format=YES           //是否用标准格式存储日志
secure_chroot_dir=/var/run/vsftpd/empty
pam_service_name=vsftpd          //设置PAM认证服务的配置文件名,该文件位于/etc/pam.d目录下
rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
ssl_enable=NO
utf8_filesystem=YES
anon_root=/home/www              //匿名用户访问的目录
```

修改配置后重启服务。

```sh
# restart
systemctl restart vsftpd
# check
netstat -an | grep 21
```

默认的匿名访问目录就是`ftp`用户的家目录，可以查 `/etc/passwd` 文件。

```sh
cat /etc/passwd | grep ftp
ftp:x:110:115:ftp daemon,,,:/srv/ftp:/bin/false

```

如果文件写入失败，有可能是权限的问题，owner 改成`ftp`并下放权限。

```
chown -R ftp /srv/ftp
```

## http

如果只是为了读文件，直接用 Python 开个 http 服务即可。

```
cd /path/to/shared_dir

# python2，系统自带
python -m SimpleHTTPServer 8000

# python3 的用法不一样
python3 -m http.server 8000
```

然后用浏览器访问 `http://linux-ip:8000` 即可下载该目录的文件。
