## [原创]自动修改同一天从30个IP发送邮件账号的密码

本文原创：**中国科学技术大学 张焕杰**

修改时间：2017.10.16

## 一、目的

有人专门通过尝试或撞库的方式获取邮件系统密码，并利用这些账号发送垃圾邮件。

本脚本定时运行，统计coremail系统当日每个用户发信的IP来源个数，如果超过30，这种一般是发送垃圾邮件的行为，直接修改用户的密码。

## 二、相关代码

假定程序放在`/home/coremail/auto_stop_spam_send`目录下，脚本名字是`auto_stop_spam_send.sh`

```
#!/bin/bash

cd /home/coremail/auto_stop_spam_send

#管理员的邮箱，以便发送通知邮件
OPMAIL=*****@ustc.edu.cn
#修改的密码
NEWPWD=xyz12345

MFILE=spamsend.eml

#如果存在文件norun，直接退出
if [ -f norun ]
then
  exit
fi

date >> run.log

#统计当日用户的发信来源IP个数，格式是 数量  邮件地址
grep -h cmd:DATA /home/coremail/logs/mtatrans/mta`date +%Y_%m_%d`/* | grep Local:1 | grep ",Result:Deliver," \
 | sed -e "s/.*,ClientIp://" -e "s/,FreeIP.*Sender:/ /" -e "s/,SenderEmail.*//"  | sort | uniq  \
 | cut -f2 -d' ' | sort | uniq -c | sort -n -r |awk  '$1>30 {print $2}' | sort > send30.txt

#如果单日超过20个帐号需要禁止，可能有问题，给管理员发邮件，并退出不再执行
cnt=`cat send30.txt | wc -l`
if [ $cnt -gt 20 ]
then
  echo "From: <autotun@xxx.edu.cn>" > $MFILE
  echo "Subject: $cnt mailaccounts will be blocked due to mail send, need you check" >> $MFILE
  echo >> $MFILE
  echo "Dear admin user:" >> $MFILE
  echo >> $MFILE
  echo "    too many accounts was sending mails from 30+ IPs one day, I will not block them." >> $MFILE
  echo "    please login and check the reason, delete norun, and I will run next time." >> $MFILE
  cat send30.txt >> $MFILE
  /home/coremail/bin/userutil --put-msg $OPMAIL $MFILE
  touch norun
  date >> blockuser.log
  echo "too many users" >> blockuser.log
  exit
fi

#对每个新的异常帐号修改密码

#blocked.txt 存放的是已经修改过密码的用户，修改一次后就不再修改了
sort blocked.txt > sortblocked.txt

for u in `comm -13 sortblocked.txt send30.txt`; do
  date >> blockuser.log
  echo $u >> blockuser.log
  echo $u >> blocked.txt
  echo "From: <autorun@xxx.edu.cn>" > $MFILE
  echo "Subject: mail account blocked due to mail send" >> $MFILE
  echo >> $MFILE
  echo "Dear user $u:" >> $MFILE
  echo >> $MFILE
  echo "    Your mail account was blocked due to send mails from 30+ IPs one day." >> $MFILE
  /home/coremail/bin/userutil --put-msg $u $MFILE
  /home/coremail/bin/userutil --put-msg $OPMAIL $MFILE
  /home/coremail/bin/userutil --set-user-attr $u password=$NEWPWD
done

````

## 三、程序的运行

在 `crontab -e`中增加以下内容，每10分钟运行一次即可
````
*/10 * * * * /home/coremail/auto_stop_spam_send/auto_stop_spam_send.sh
````

***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
