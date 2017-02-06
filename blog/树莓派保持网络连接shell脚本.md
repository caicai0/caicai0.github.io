######故事背景
树莓派3b现在有了板载无线网卡，可以很方便的连接到wifi。但是无线路由器不能保证永远正常工作，由于各种原因会重启路由。这个时候，raspberry3b就会有一定概率断开网络连接。系统级功能比如像手机一样有网就连，断网后自动重连的功能目前没有能力实现。所以就想搞一个脚本检查网络连接状态，如果没有网络重启一下。
######shell脚本
#! /bin/bash
#检测网络连接
log=/root/network.log
#判断输出日志文件是否存在
if [ ! -f ${log} ]
then
touch ${log}
fi
ping -c 1 192.168.1.1 > /dev/null 2>&1
if [ $? -eq 0 ];then
#echo `date` 检测网络正常 >> ${log}
else
echo `date` 检测网络异常 >> ${log}
reboot
fi
这个脚本基本上是ping路由地址（192.168.1.1）来判断无线的连接状态，$?是上一条命令执行的return值。ping通返回0，ping不通返回不是0。当ping不通的时候写下日志，然后重启系统（为什么不重启网络？我是真的不在家的时间有点长，所以没有胆量尝试）。
######定时运行
起初想用nodejs脚本定时运行，自己都觉得小题大作了。转而寻找用shell脚本实现的思路。查资料后发现linux系统本身有一个cron服务（俗称定时任务）。
修改/etc/crontab文件，添加一行 

*/5 *   * * *   root    bash /root/network.sh
当系统时间的分钟整除5的时候就会以root身份调用一个后面的bash /root/network.sh命令。
