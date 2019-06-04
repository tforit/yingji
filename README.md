# yingji
当企业被攻击者入侵，系统被挂暗链、内容遭到恶意篡改，服务器出现异常链接、卡顿等情况时，需要进行紧急处理，使系统在最短时间内恢复正常。由于应急处理往往时间紧，所以尝试将应急中常见处理方法整合到脚本中，可自动化实现部分应急工作。应急脚本采用python2.0完成，由于所有需要执行的命令都是依靠ssh进行远程链接，所以在运行脚本之前，需要输入正确的主机ip地址、ssh远程连接端口、ssh远程登录账户、ssh远程登录密码。

一、脚本实现的主要功能

1、获取主机信息
获取的主机信息包括：主机ip地址、主机名、当前系统内核版本、当前系统版本、系统当前时间；
2、获取异常进程
获取异常进程主要是采用两种方式，第一种，通过执行netstat -antp获取当前主机存在哪些链接，并通过判断外部链接地址归属地，如果归属地不是中国，则会提取相关pid，并根据pid定位出文件所在位置。第二种，通过cpu占有率，一旦发现cpu占用率高于%15时，会提取对应程序的pid，根据pid定位异常文件位置。
3、判断常见命令是否被篡改
在之前的应急响应中出现过常见命令被非法篡改情况，如ps、netstat命令被恶意替换，利用stat查看文件详细信息，通过比对时间的方式判断命令是否被篡改。
4、查看系统启动项
很多恶意程序会修改系统启动项，这样即使系统进行重启时，恶意程序也能自动启动，查看init.d目录下的启动文件，根据修改时间提取最近被修改的启动文件，并根据时间排序列出前5个。
5、查看历史命令
查看.bash_history历史命令，通过匹配关键字，如wget、cur等，来查看系统在被入侵后是否被执行了恶意操作。
6、判断非系统默认账户
恶意程序可能会在系统中新建账户，通过查看login.defs文件获取最小uid，从而根据uid查看passwd文件，获取之后新建的系统用户。
7、获取当前登录用户
通过调用who，查看当前登录用户（tty为本地登录，pts为远程登录），判断是否存在异常用户登录情况。
8、查看系统当前用户
通过查看etc/passwd,查看相关用户信息，确定是否存在异常用户。
9、查看crontab定时任务
查看/etc/crontab定时任务，并将输出结果保存到log中
10、查看、保存最近三天系统文件修改情况
通过find命令，查找最近三天修改过的文件，由于修改的系统文件较多，所以修改文件被单独保存在本地file_edit文件中
11、查找特权用户。
查看passwd文件，查找用户id为0的特权用户
12、secure日志分析
日志分析是应急的重头工作，尤其是在应急后期的溯源阶段，日志分析更显得尤为重要，由于日志种类包括服务器日志、应用日志，此处只是分析了secure服务器日志，提取日志的ip地址进行判断，并对ip归属地进行判断，查看的secure日志单独保存在本地secure中。

脚本整体的思路比较简单，就是远程登录到linux执行常见的应急命令，脚本中的命令在centos下都是可正常运行的，可以在根据实际环境自行在对命令做调整。上面的部分功能如果有好的实现方法也可灵活调整，如判断常见命令是否被篡改，脚本中是根据时间进行判断，在实际应用中也可根据文件大小进行判断，代码整体写的比较渣，有意见欢迎大家多多指出，希望通过改进能让功能更加完善。
