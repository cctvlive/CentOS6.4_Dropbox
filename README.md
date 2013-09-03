CentOS6.4_Dropbox
=================

CentOS6.4_Dropbox

Public : CentOS使用Dropbox定时同步备份方案详解

欢迎交流: cctv_243028755@live.cn chinacodeigniter@gmail.com




    一直以来，使用美帝VPS建站，由于网速等多方面原因，面临着备份困难、下载困难的难题，而且还要面临故障啊、商家跑路啊等等数据丢失的风险，前段时间DS不是数据都木有了么。现在就来讲一下解决这个问题的方法，早些时候有用脚本通过FTP来备份的，例如使用godaddy域名附带的免费空间或者另外单独购买备份空间，但有时并不好用；也有两台VPS同步备份的，这个花费有点高。而现在使用DropBox来进行同步备份，全自动化，非常好用，去年就有过类似的介绍，但很多都不详细，搜集了网上的方法，特别整理出来，所有过程都有详细注明。

1、
安装Dropbox Linux客户端：

    SSH登陆，进入root目录，输入以下命令(注：下面的命令中已包含cd到root根目录的命令，而且只有在root根目录下后续步骤才能生效)，视版本不同而选择：

32位CentOS：

cd ~ && wget -O - "http://www.dropbox.com/download?plat=lnx.x86" | tar xzf -

64位CentOS： 先切换到  # linux64   /////First switch to # linux64

cd ~ && wget -O - "http://www.dropbox.com/download?plat=lnx.x86_64" | tar xzf -

2.0    下载后已自动解压，不需要再执行解压命令。~/.dropbox-dist/dropboxd 安装执行,弹出安装引导
 
2、
#Dropbox与机器绑定：

执行以下命令

~/.dropbox-dist/dropboxd &   后台运行

  //#  第一次执行会生成“host_id”，这机器与Dropbox进行绑定的唯一字符串，提示的信息是一个链接，而且会重复滚动出现直到绑定完成。复制这个链接在浏览器里访问，输入Dropbox帐户和密码就可以了，输入后会自动跳转到Dropbox主界面并且会有绑定成功的提示，此时在ssh客户端里也会有提示并且停止滚动，再按回车键就完成绑定。

//#    (注：官方的运行命令结尾没有“&”这个符号，在centos下运行会出现ssh冻结无反应的情况，据称Debian也会。实际上守护进程已经在运行了。)

3、
建立目录软链接：

    在root目录下生成的“Dropbox”文件夹(linux文件夹名称区分大小写的)，就是Windows里叫做“同步目录”的文件夹，只要把文件放置在里面就会同步。在未同步之前，里面有一个文件夹“.dropbox.cache”和一个文件“.dropbox”。当然我们不可能把网站放置到这里，因此我们需要在里面建立软链接就行了，使用ln命令建立软链接(软链接其实就是windows里的快捷方式)，格式是：ln –s 源文件 目标文件，我可以先进入“Dropbox”文件夹，免去每次都需要输入目标文件的麻烦。过程如下：

cd ~/Dropbox
ln -s /home/wwwroot

    释义：进入“Dropbox”文件夹，建立/home/wwwroot/ 文件夹的软链接。运行这两个命令后会在“Dropbox”文件夹下生成一个名为“wwwroot”的软链接。如果网站放在不同地方的话，那么就建立多个软链接就好。

4、
运行同步守护进程，同步网站数据：

    输入以下命令（就是第二步绑定“host_id”的命令）

~/.dropbox-dist/dropboxd &

    运行此命令后，视数据大小和网络环境而定，反正美国的VPS同步都很快，会在浏览器里的Dropbox文件管理界面里看到同步的文件夹。同时，在“Events(活动)”里看到同步记录，记录里有文件数量和文件夹数量，机器与Dropbox帐户的绑定日志也会记录在里面，这个其实就是Dropbox的帐户活动记录。

5、
定时同步，节约内存资源：

a）关闭Dropbox进程

killall dropbox

b）回到root目录下，创建定时运行脚本backup.sh

编写定时同步脚本：

vi backup.sh

    用vi编辑器新建backup.sh目录，运行后会进入vi编辑器，此时按“I”键进入编辑模式，复制以下代码粘贴进去，按ESC键退出编辑模式，开启大写锁定状态(按“Caps Lock”键)，再按两次“Z”键即自动保存并退出vi编辑器。

#!/bin/sh
start() {
echo starting dropbox
/root/.dropbox-dist/dropboxd &
}
stop() {
echo stoping dropbox
pkill dropbox
}
case "$1" in
start)
start
;;
stop)
stop
;;
restart)
stop
start
;;
esac

    继续运行以下命令，用“chmod”命令为“backup.sh”添加可执行权限：

chmod +x backup.sh

c）继续在root目录下，创建数据库备份脚本bakmysql.sh

编写定时同步脚本：

vi bakmysql.sh

    按“I”键进入编辑模式，复制以下代码并粘贴(文字部分填写需填写完好才行)，按“ESC”退出编辑模式，开启大写锁定状态，再按两次“Z”键即自动保存并退出vi编辑器。

#!/bin/bash
DBName=修改为数据库名
DBUser=修改为数据库用户名
DBPasswd=修改为数据库密码
BackupPath=/root/Dropbox/
LogFile=/root/db.log
DBPath=/usr/local/mysql/var/ 
//#备份的数据库目录
//#BackupMethod=mysqldump
//#BackupMethod=mysqlhotcopy
//#BackupMethod=tar
NewFile="$BackupPath"db$(date +%y%m%d).tgz
DumpFile="$BackupPath"db$(date +%y%m%d)
OldFile="$BackupPath"db$(date +%y%m%d --date='5 days ago').tgz  #自动删除5天前的备份
echo "-------------------------------------------" &gt;&gt; $LogFile
echo $(date +"%y-%m-%d %H:%M:%S") &gt;&gt; $LogFile
echo "--------------------------" &gt;&gt; $LogFile
//#Delete Old File
if [ -f $OldFile ]
then
rm -f $OldFile &gt;&gt; $LogFile 2&gt;&1
echo "[$OldFile]Delete Old File Success!" &gt;&gt; $LogFile
else
echo "[$OldFile]No Old Backup File!" &gt;&gt; $LogFile
fi
if [ -f $NewFile ]
then
echo "[$NewFile]The Backup File is exists,Can't Backup!" &gt;&gt; $LogFile
else
case $BackupMethod in
mysqldump)
       if [ -z $DBPasswd ]
       then
	       mysqldump -u $DBUser --opt $DBName &gt; $DumpFile
       else
	       mysqldump -u $DBUser -p$DBPasswd --opt $DBName &gt; $DumpFile
       fi
       tar czvf $NewFile $DumpFile &gt;&gt; $LogFile 2&gt;&1
       echo "[$NewFile]Backup Success!" &gt;&gt; $LogFile
       rm -rf $DumpFile
       ;;
mysqlhotcopy)
       rm -rf $DumpFile
       mkdir $DumpFile
       if [ -z $DBPasswd ]
       then
	       mysqlhotcopy -u $DBUser $DBName $DumpFile &gt;&gt; $LogFile 2&gt;&1
       else
	       mysqlhotcopy -u $DBUser -p $DBPasswd $DBName $DumpFile &gt;&gt;$LogFile 2&gt;&1
       fi
       tar czvf $NewFile $DumpFile &gt;&gt; $LogFile 2&gt;&1
       echo "[$NewFile]Backup Success!" &gt;&gt; $LogFile
       rm -rf $DumpFile
       ;;
*)
       service mysql stop &gt;/dev/null 2&gt;&1
       tar czvf $NewFile $DBPath$DBName &gt;&gt; $LogFile 2&gt;&1
       service mysql start &gt;/dev/null 2&gt;&1
       echo "[$NewFile]Backup Success!" &gt;&gt; $LogFile
       ;;
esac
fi
echo "-------------------------------------------" &gt;&gt; $LogFile


    记得修改上面代码中的中文部分为自己的参数。接下来，为bakmysql.sh添加可执行权限

chmod +x bakmysql.sh

6、
编写周期性执行指令：

crontab -e

    “crontab”命令运行后会自动调用内置的vi编辑器进行编辑，按“I”键进入编辑模式，复制以下四行指令代码并粘贴

0 4 * * * sh /root/backup.sh restart
0 5 * * * sh /root/backup.sh stop
0 2 * * * sh /root/bakmysql.sh restart
0 3 * * * sh /root/bakmysql.sh stop

    上面的意思是在（backup.sh）每天4点开始同步，5点关闭同步，（bakmysql.sh）每天2点开始同步，3点关闭同步，一个小时一般都够用，除非网站特别大。完成后按“ESC”退出编辑模式，开启大写锁定状态，再按两次“Z”键即自动保存并退出vi编辑器。附：“crontab -l” 列出目前的时程表，“crontab -r” 删除目前的时程表，查看系统当前时间的命令是“date”。

7、
卸载dropbox方法：

    停止守护进程

killall dropbox

    删除目录

rm -rf .dropbox .dropbox-dist Dropbox dropbox.tar.gz dbmakefakelib.py dbreadconfig.py