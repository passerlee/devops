安装虚拟机master，httpd和mysql，对其基本环境进行配置，配置ip地址并进行连通性测试，测试成功后使用远程工具对虚拟机进行操控。<br />对连通性测试进行自动化配置，编写shell脚本ping.sh并进行自动化连通性测试。<br />首先编写iplist文件以供脚本进行所测试ip的读取，如下所示：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672731938508-f7a0bee2-0e5c-4af2-b37f-25ad9d575c3f.png#averageHue=%23253f52&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=98&id=u0bf0a9f4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=123&originWidth=315&originalType=binary&ratio=1&rotation=0&showTitle=false&size=155428&status=done&style=none&taskId=u586b4d08-e912-496c-9eda-0631cab896a&title=&width=252)
```
#!/bin/bash
ips=$(cat /root/testsh/iplist)
for ip in $ips
do
    echo "正在ping $ip 请稍等..."
    ping -c 2 -i 0.5 -W 2 $ip &>/dev/null
    if [ $? -eq 0 ]
        then
            echo "地址$ip是通的"
            echo $ip >>/root/testsh/ok.txt
    else
        echo "地址$ip是不通的"
        echo $ip >>/root/testsh/notok.txt
    fi
done
echo "所有的ip地址已经检测完毕"
```
执行结果如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672731991254-22bd7de0-5c57-4c50-9c08-71ddc379edc5.png#averageHue=%23294355&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=129&id=ue73bc870&margin=%5Bobject%20Object%5D&name=image.png&originHeight=161&originWidth=333&originalType=binary&ratio=1&rotation=0&showTitle=false&size=215044&status=done&style=none&taskId=u8d98870a-9344-4637-9a1e-e4afbd4d8da&title=&width=266.4)<br />在此基础上，对本地master上的文件希望进行远程分发至httpd和mysql以及执行，编写脚本fenfa.sh以实现该功能。<br />首先创建fenfa文件夹，将需要进行分发的文件复制装在该文件夹，以便脚本运行时进行访问。
```
#!/bin/bash
ips=$(cat /root/testsh/iplist)
dir=/root/testsh/fenfa/
files=$(ls /root/testsh/fenfa)
for ip in $ips
do
    for file in $files
    do
        scp ${dir}${file} root@${ip}:/root &>/dev/null
        if [ $? -eq 0 ]
            then
                echo "往${ip}分文件${dir}${file}分发成功..."
        else
            echo "往${ip}分文件${dir}${file}分发失败，请检测网络"
        fi
    done
done
```
编写完成进行脚本执行结果如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672732168291-b0e8f602-4dc3-4445-b23e-c657c7f3d5dd.png#averageHue=%232d495c&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=245&id=ue918b232&margin=%5Bobject%20Object%5D&name=image.png&originHeight=306&originWidth=483&originalType=binary&ratio=1&rotation=0&showTitle=false&size=592541&status=done&style=none&taskId=u5ad7e2f0-7697-46f2-bef9-0bc9a9f94af&title=&width=386.4)<br />在第一天的进行网络连通和远程分发操作的基础上，希望实现对系统的初始化管理，包括修改本地主机的主机名，关闭主机的selinux服务，隐藏主机的登录信息以及修改主机的dns。编写脚本init.sh以实现这些基本功能，脚本具体如下：
```
#!/bin/bash
ip=$(ifconfig | grep "192.168.10" | cut -d'n' -f2 | cut -d' ' -f2)
lastip=$(ifconfig | grep "192.168.10" | cut -d'n' -f2 | cut -d' ' -f2 | cut -d"." -f4)
hostname=host${lastip}
hostnamectl set-hostname $hostname
echo "地址为$ip的主机名修改为$hostname"

echo "关闭地址为$ip的主机的selinux"
setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux

echo "隐藏地址为$ip的主机的登录信息"
echo "欢迎登录该主机" >/etc/issue

echo "配置地址为$ip的主机的DNS为114.114.114.114"
echo "nameserver 114.114.114.114" >/etc/resolv.conf
```
在对以上脚本进行运行的具体结果如下所示：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672732415344-f0058060-6f7f-4061-8c15-469fcd22a7ba.png#averageHue=%232e4d62&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=138&id=u97d6da20&margin=%5Bobject%20Object%5D&name=image.png&originHeight=173&originWidth=484&originalType=binary&ratio=1&rotation=0&showTitle=false&size=335720&status=done&style=none&taskId=uee8f58f2-5cf6-47d8-8d84-42a15dd8b19&title=&width=387.2)<br />脚本使用hostnamectl set-hostname命令将主机的hostname永久性更改，以实现对主机的可靠性标识。<br />使用setenforce 0命令关闭当前状态的selinux，然后更改/etc/sysconfig/selinux文件进行永久性的selinux服务关闭。替换/etc/issue文件中的信息实现用户登录信息的隐藏。<br />更改网络配置文件/etc/resolv.conf中的dns项目实现dns的更改。<br />使用以上操作实现对主机的初始化。<br />在初始化完成后希望进行用户的动态增加和删除，编写两个脚本以实现该服务。首先编写一个users.txt文件以供脚本进行读取操作。<br />Users.txt文件如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672732487014-95f1ac70-4072-4119-9727-5a6367da9689.png#averageHue=%23244053&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=192&id=u8ce6dbff&margin=%5Bobject%20Object%5D&name=image.png&originHeight=240&originWidth=350&originalType=binary&ratio=1&rotation=0&showTitle=false&size=336871&status=done&style=none&taskId=ua45d9353-020a-4b6d-9bfa-e3edc1c7d4b&title=&width=280)
```
#!/bin/bash
users=$(cat /root/testsh/users.txt)
for user in $users
do
    id $user &>/dev/null
    if [ $? -eq 0 ]
        then
            echo "用户$user已存在，不用添加了"
            continue
    else
        useradd $user &>/dev/null
        if [ $? -eq 0 ]
            then
                echo "${user}123" | passwd --stdin $user &>/dev/null
                chage -d 0 $user &>/dev/null
                echo "用户$user增加成功"
             else
                echo "用户$user增加失败"
        fi
    fi
done
```
查看文件users.txt中的添加用户是否已经存在，存在则返回用户存在信息，不存在则创建之后返回创建成功或者失败信息，所创建的用户密码为用户名加上数字123。<br />脚本执行结果如下所示：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672732552617-a2a42f2b-e665-4b05-9378-fbb97e9cdeff.png#averageHue=%23284357&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=179&id=u21c98235&margin=%5Bobject%20Object%5D&name=image.png&originHeight=224&originWidth=358&originalType=binary&ratio=1&rotation=0&showTitle=false&size=321594&status=done&style=none&taskId=u7ca4f9c6-574c-4572-872c-0ab24d12680&title=&width=286.4)<br />实现用户创建之后进行用户删除脚本userdel.sh编写，同样需要编写nousers.txt文件使脚本读取，实现对某些用户的删除。<br />nousers.txt具体内容如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672732697137-4d6aed72-eb81-40f1-ac94-1c78f3c93156.png#averageHue=%2320374a&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=59&id=ua955de26&margin=%5Bobject%20Object%5D&name=image.png&originHeight=74&originWidth=333&originalType=binary&ratio=1&rotation=0&showTitle=false&size=98885&status=done&style=none&taskId=ud4579de3-e4c1-4c72-92b7-d6dbbafe983&title=&width=266.4)
```
#!/bin/bash
users=$(cat /root/testsh/nousers.txt)
for user in $users
do
    id $user &>/dev/null
    if [ $? -eq 0 ]
        then
            userdel -r $user &>/dev/null
            if [ $? -eq 0 ]
                then
                    echo "用户$user删除成功"
                else
                echo "用户$user删除失败"
            fi
     else
        echo "用户$user不存在，请检查"
     fi

done
```
查看文件nousers.txt中需要删除的用户，查看该用户是否存在，不存在返回不存在信息，反之则进行userdel删除用户操作，执行完毕后返回成功或者失败信息。<br />执行脚本后结果如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672732740748-2686ac3b-b9c7-49c4-8b63-53cda0d9df1c.png#averageHue=%23233b4e&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=61&id=ubabc2848&margin=%5Bobject%20Object%5D&name=image.png&originHeight=76&originWidth=365&originalType=binary&ratio=1&rotation=0&showTitle=false&size=111296&status=done&style=none&taskId=ubbec2468-ed37-4892-9736-21a8ad6a55c&title=&width=292)<br />自此实现本地用户的自动化创建和删除。<br />在此基础上还需要实现对远程主机的自动化初始化和用户添加删除操作，将今天的本地主机的操作脚本与昨天的远程分发脚本相结合，实现对远程主机的相同操作。
```
#!/bin/bash
sh /root/testsh/fenfa.sh
ips=$(cat /root/testsh/iplist)
for ip in $ips
do
    echo "主机${ip}正在系统初始化"
    ssh root@${ip} chmod a+x /root/init.sh
    ssh root@${ip} sh /root/init.sh
done
```
执行结果如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672732835988-217a1e57-ef67-4518-95af-b1fbb84cddb3.png#averageHue=%232c485c&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=371&id=u0248892f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=464&originWidth=503&originalType=binary&ratio=1&rotation=0&showTitle=false&size=935624&status=done&style=none&taskId=ubdfe732f-3d62-4888-8a30-2340a1098bd&title=&width=402.4)
```
#!/bin/bash
sh /root/testsh/fenfa.sh
ips=$(cat /root/testsh/iplist)
for ip in $ips
do
    echo "主机${ip}正在增加用户"
    ssh root@${ip} mkdir /root/testsh &>/dev/null
    ssh root@${ip} cp /root/users.txt /root/testsh
    ssh root@${ip} chmod a+x /root/useradd.sh
    ssh root@${ip} sh /root/useradd.sh
done
```
执行结果如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672732893791-dc65e87f-1cf1-4029-887d-6ea46acc3df8.png#averageHue=%232b465a&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=282&id=uea5ca6d2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=353&originWidth=331&originalType=binary&ratio=1&rotation=0&showTitle=false&size=468568&status=done&style=none&taskId=u4aa313a7-7cd0-4152-a004-0d71a72d9d4&title=&width=264.8)
```
#!/bin/bash
sh /root/testsh/fenfa.sh
ips=$(cat /root/testsh/iplist)
for ip in $ips
do
    echo "主机${ip}正在删除用户"
    ssh root@${ip} mkdir /root/testsh &>/dev/null
    ssh root@${ip} cp /root/nousers.txt /root/testsh
    ssh root@${ip} chmod a+x /root/userdel.sh
    ssh root@${ip} sh /root/userdel.sh
done
```
执行结果如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672732967317-aef3a611-a835-42cf-bcd8-b18431a402a2.png#averageHue=%232c495c&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=356&id=u7b48270a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=445&originWidth=455&originalType=binary&ratio=1&rotation=0&showTitle=false&size=811730&status=done&style=none&taskId=u33b489b7-9436-45d6-9195-999d86da299&title=&width=364)<br />远程操作均为将本地脚本添加到fenfa目录并运行fenfa.sh进行远程分发，再给脚本添加上可执行权限，最后执行实现。<br />在系统运行过程中，有许多指标，希望实现对这些指标的自动收集。
```
#!/bin/bash
hostname=${hostname}
echo "以下是$hostname的系统数据"
#进程数
ps=$(ps aux | wc -l)
if [ $ps -gt 200 ]
    then
        echo "进程数为$ps，大于200，多留意"
else
    echo "进程数为$ps,正常"
fi
#内存占用率
memused=$(sar -r |grep "平均时间" | awk '{print $4}' | cut -d '.' -f1)
if [ $memused -gt 75 ]
    then
        echo "内存占用率过高，为${memused}%"
else
    echo "内存占用率正常，为${memused}%"
fi
#cpu占用率
cpuidle=$(sar -u | grep "平均时间" | awk '{print $8}' | cut -d '.' -f1)
cpuused=$((100-$cpuidle))
if [ $cpuused -gt 75 ]
    then
        echo "cpu占用率过高，为${cpuused}%"
else
    echo "cpu占用率正常"
fi
#用户数
users=$(who | wc -l)
if [ $users -gt 20 ]
    then
        echo "用户数过多，为$users"
else
    echo "用户数正常"
fi
#文件系统
rootused=$(df -hT | grep "xfs" | grep "root"  | awk '{print $6}' | cut -d'%' -f1)
homeused=$(df -hT | grep "xfs" | grep "home"  | awk '{print $6}' | cut -d'%' -f1)
bootused=$(df -hT | grep "xfs" | grep "boot"  | awk '{print $6}' | cut -d'%' -f1)
totalused=$(($rootused+$homeused+$bootused))
if [ $totalused -gt 80 ]
    then
        echo "文件系统占用太高，为${totalused}%"
else
    echo "文件系统占用正常，为${totalused}%"
fi
#网络流量
netflow=$(sar -n DEV | grep "平均时间" | grep "ens33" | awk '{print $5,$6}')
echo "网卡ens33的平均速率和发送速率为$netflow，单位为k/s"
```
脚本执行结果如下:<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672733148062-c1d23d47-67c9-422d-bf32-e66a58c22603.png#averageHue=%23284457&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=157&id=u275bae60&margin=%5Bobject%20Object%5D&name=image.png&originHeight=196&originWidth=496&originalType=binary&ratio=1&rotation=0&showTitle=false&size=389768&status=done&style=none&taskId=u6573cfcb-20d7-4af7-a992-17ba03cb947&title=&width=396.8)<br />系统数据主要有以下组成：进程数，内存占用率，CPU占用率，文件系统以及网卡的速率。<br />ps aux | wc -l命令输出进程数，判断是否小于200，大于则给予提示并输出，不大于则只输出进程数；<br />sar -r |grep "平均时间" | awk '{print $4}' | cut -d '.' -f1命令输出内存的占用率，同样的操作判断是否大于75，大于则给予提示并输出，不大于正常输出；<br />sar -u | grep "平均时间" | awk '{print $8}' | cut -d '.' -f1命令相同的操作输出cpu的占用率；<br />who | wc -l命令判断系统中的用户数，判断是否大于20，进行同样的提示方法；<br />df -hT | grep "xfs"分别输出root,home以及boot的内存占用率，将三个数值相加输出总的内存占用率，判断是否大于80，80以上提示占用过高，以下则正常；<br />sar -n DEV命令查询ens33的平均速率和发送速率，打印输出并注释单位。<br />以上即该脚本中所有的指标监测功能，shell脚本实现了对系统这些常用指标的快捷收集，方便了管理员的管理。<br />将该脚本分发到远程主机并进行远程主机的指标收集:
```
#!/bin/bash
sh /root/testsh/fenfa.sh
ips=$(cat /root/testsh/iplist)
for ip in $ips
do
    echo "主机${ip}正在进行系统指标监控..."
    ssh root@${ip} chmod a+x /root/system.sh
    ssh root@${ip} sh /root/system.sh
done
```
执行结果如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672733219172-65d70657-b0d4-4bb9-9d01-c59eac9b7229.png#averageHue=%232a475b&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=315&id=u93f31a45&margin=%5Bobject%20Object%5D&name=image.png&originHeight=394&originWidth=520&originalType=binary&ratio=1&rotation=0&showTitle=false&size=821323&status=done&style=none&taskId=u2ce291c0-5033-454f-9c89-938b2602ef5&title=&width=416)<br />在对服务器进行操作的同时通常需要进行备份操作，编写脚本实现备份的自动化
```
#!/bin/bash
hostname=${hostname}
echo "主机$hostname正在做全量备份，请稍后..."
backup_time=$(date +%Y_%m_%d_%H)
weekday=$(date +%w)
if [ $weekday -eq 5 ]
    then
        tar -cvjpf /root/etc_backup_${backup_time}.tar.bz2 /etc --exclude /etc/abrt --exclude /etc/aliases.db &>/dev/null
        if [ $? -eq 0 ]
            then
                size=$(du -sh /root/etc_backup_${backup_time}.tar.bz2)
                echo "备份成功，备份文件为/root/etc_backup_${backup_time}.tar.bz2，大小为$size"
        else
            echo "备份不成功"
        fi
else
    echo "没有到做全量备份的时间，不需要备份"
fi
```
执行结果如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672733285203-bb206dc3-8b04-4ba4-a762-7dfa26fbe114.png#averageHue=%23233c4e&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=52&id=u3e5389a6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=65&originWidth=550&originalType=binary&ratio=1&rotation=0&showTitle=false&size=143378&status=done&style=none&taskId=u7b580d13-0d2b-40f5-a755-62765aa9faa&title=&width=440)<br />对系统中/etc目录进行备份，除去该目录里的abrt以及aliases.db文件，tar -cvjpf命令实现该文档的tar工具备份。date +%Y_%m_%d_%H命令实现对当前时间日期的输出，以date +%w命令判断当前是周几，判断其值是否为5，是则备份，否则提示，实现指定的周五时期备份的需求。最后{backup_time}.tar.bz2命名文件实现对文件备份日期的可视化。
```
#!/bin/bash
sh /root/testsh/fenfa.sh
ips=$(cat /root/testsh/iplist)
for ip in $ips
do
    echo "主机${ip}正在进行备份..."
    ssh root@${ip} chmod a+x /root/backup.sh
    ssh root@${ip} sh /root/backup.sh
done
```
执行结果如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672733323992-8bf87460-65cb-4060-943b-91419dd9de24.png#averageHue=%23253e51&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=95&id=u9dc58a0d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=119&originWidth=563&originalType=binary&ratio=1&rotation=0&showTitle=false&size=268615&status=done&style=none&taskId=u7897e75f-740d-465e-934b-cd98e0a2028&title=&width=450.4)<br />最后需要实现软件的自动化部署，实现对远程主机httpd和mysql的服务部署，本地编写install.sh脚本分发至两台主机。
```
#!/bin/bash
ip=$(ifconfig | grep "192.168.10" | awk '{print $2}')
if [ $ip == "192.168.10.2" ]
then
echo "在主机$ip上安装httpd服务，请稍后..."
yum list &>/dev/null
if [ $? -eq 0 ]
then
yum -y install httpd &>/dev/null
echo "htppd软件安装完毕，或者已安装"
sed -i 's/#ServerName www.example.com:80/ServerName www.longdong.com/g' /etc/httpd/conf/httpd.conf
systemctl restart httpd &>/dev/null
systemctl enable httpd &>/dev/null
netstat -antulp | grep :80 &>/dev/null
if [ $? -eq 0 ]
then
echo "httpd服务已经启动成功"
echo "www.longdong.com" >/var/www/html/index.html
else
echo "httpd服务没有启动成功，请检查..."
fi
else
echo "yum源不可用"
fi
elif [ $ip == "192.168.10.3" ]
then
echo "主机$ip上安装mariadb服务..."
yum list &>/dev/null
if [ $? -eq 0 ]
then
yum -y install mariadb-server.x86_64 &>/dev/null
echo "mariadb软件安装完毕，或者已安装"
systemctl restart mariadb.service &>/dev/null
systemctl enable mariadb.service &>/dev/null
netstat -antulp | grep :3306 &>/dev/null
if [ $? -eq 0 ]
then
echo "mariadb服务已经启动成功"
else
echo "mariadb服务没有启动成功，请检查..."
fi

else
echo "yum源不可用"
fi
else
echo "不可安装"
fi
远程分发脚本如下所示：
#!/bin/bash
sh /root/testsh/fenfa.sh
ips=$(cat /root/testsh/iplist)
for ip in $ips
do
echo "主机${ip}正在进行软件安装"
ssh root@${ip} chmod a+x /root/install.sh
ssh root@${ip} sh /root/install.sh
done
```
执行结果如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672733459669-37c4ca3d-2fa9-4a6d-adb2-b7e3716ce680.png#averageHue=%23294559&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=220&id=u157392c0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=275&originWidth=558&originalType=binary&ratio=1&rotation=0&showTitle=false&size=615142&status=done&style=none&taskId=u00f8fb4e-bbb8-48bf-b8b0-8c4e5cbd953&title=&width=446.4)<br />ifconfig命令打印当前主机的IP地址，判断是否是192.168.10.2（httpd）或者192.168.10.3（mysql）,是则安装相应的服务，不是则打印输出错误信息。在安装服务之后进行对应服务的启动以及开机自启，再用netstat -antulp过滤相应端口查看相应服务是否启动成功，打印输出相应的信息，如果相应的服务未被安装则打印输出相应服务yum源不可用的提示指令。脚本执行成功则将httpd以及mariadb服务安装到相应的主机。<br />对于环境中的mysql服务器，我们希望它不会因为一些环境的变化而产生较大的变化，为实现这一目标就需要管理员进行对数据库服务器中的数据库做必要的备份，以实现在某些重大事故中可以有效地挽回损失。<br />在本次脚本编写中希望实现对数据库有效的备份，因此设计对其在周一进行全量备份，而其他时间去做在此基础上的增量备份。
```
#!/bin/bash
ip=$(ifconfig | grep "192.168.10" | awk '{print $2}')
weekday=$(date +%w)
if [ $ip == "192.168.10.3" ] && [ $weekday -eq 1 ]
    then
        echo "主机$ip 正在进行数据库的全量备份，请稍后..."
        mkdir /root/mybackup &>/dev/null
        #全量备份
        backupfile=mybackup_$(date +%Y_%m_%d).sql
        mysqldump -uroot -p123456 --all-databases -R>/root/mybackup/$backupfile
        if [ $? -eq 0 ]
            then
                echo "主机$ip 数据库全量备份成功"
        else
            echo "主机$ip 数据库全量备份不成功"
        fi
elif [ $ip == "192.168.10.3" ]
    then
        #增量备份
        echo "主机$ip 正在进行增量备份，请稍后..."
        tar -jcvf /root/mybackup/mysql_bin_$(date +%Y_%m_%d).tar.bz2 /var/lib/mysql/mysql-bin.* &>/dev/null
        if [ $? -eq 0 ]
           then
                echo "主机$ip 进行数据库增量备份成功"
        else
            echo "主机$ip 进行数据库增量备份不成功"
        fi
else
    echo "$ip 不是数据库主机，无需备份！！！"
fi
```
基本功能如上所述，实现不同时间的全量与增量备份，实现过程为首先使用ifconfig命令寻找数据库服务器所对应的主机ip,是该ip则进行备份，不是则提示不是该服务器；创建备份文件目录/root/mybackup储存备份的文件；判断日期是否为周一，是则用mysqldump命令进行数据库的全量备份，并命名为有时间标识的数据库文件储存在备份目录；备份完成后输出提示信息。判断不是周一的话同样的操作进行增量备份，将数据库文件做tar包压缩并保存在备份目录。根据该功能完成对数据库的备份操作，基本实现对数据库备份功能的配置。当然，相同的需要master主机来进行脚本的分发操作，实现脚本farmybackup.sh有如下：
```
#!/bin/bash
sh /root/testsh/fenfa.sh
ips=$(cat /root/testsh/iplist)
for ip in $ips
do
    echo "主机${ip}正在进行数据库备份..."
    ssh root@${ip} chmod a+x /root/mybackup.sh
    ssh root@${ip} sh /root/mybackup.sh
done
```
自此可以实现远程的文件分发至数据库服务器，并在数据库服务器执行返回执行结果。脚本具体执行结果有如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672733648110-879d5139-f8fc-4375-a04a-4de63466bdde.png#averageHue=%232f4e62&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=119&id=u9ea4a2d8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=149&originWidth=564&originalType=binary&ratio=1&rotation=0&showTitle=false&size=336919&status=done&style=none&taskId=u9d2cece0-151f-4475-91cd-cd58e8a5721&title=&width=451.2)<br />在实现对httpd和mysql服务器对应服务安装之后，我们还在希望对其具体服务的监控，以保证该服务在该服务器上可以有效地运行。为此需要具体服务监控脚本对这两个服务器进行实时监控。<br />具体实现脚本service.sh有如下：
```
#!/bin/bash
ip=$(ifconfig | grep "192.168.10" | awk '{print $2}' )
if [ $ip == "192.168.10.2" ]
    then
        echo "正在监控$ip的httpd进程情况，请稍等..."
        httpd_num=$(pgrep -l httpd | wc -l)
        if [ $httpd_num -eq 5 ]
            then
                echo "httpd的进程数量为$httpd_num,过高，请注意..."
                echo "httpd的进程数量为$httpd_num,过高，请注意..." | mail -s "httpd alert" root@localhost
                systemctl restart httpd &>/dev/null
         fi
        echo "正在监控$ip的httpd的访问情况..."
        cat /var/log/httpd/access_log | awk '{print $1, $4, $9}' | sed 's/\[//g'  >/var/log/httpd/log.txt
        cat /var/log/httpd/log.txt | uniq -c | sort -r | head -10 >/var/log/httpd/danger.txt
        cat /var/log/httpd/danger.txt | mail -s "DDOS alert" root@localhost
        max_access=$(cat /var/log/httpd/danger.txt | head -1 | awk '{print $1}')
        if [ $max_access -gt 4 ]
            then
                echo "有DDOS，请自卫..."
                echo "有DDOS，请自卫..." | mail -s "DDOS DDOS DDOS" root@localhost
        fi
elif [ $ip == "192.168.10.3" ]
    then
        echo "正在进行$ip数据库监控"
        netstat -antulp | grep :3306 &>/dev/null
        if [ $? -eq 0 ]
        then
            echo "mariadb工作正常"
        else
            systemctl restart mariadb
                if [ $? -eq 0 ]
                    then
                        echo "mariadb重启成功"
                fi
        fi
        conn=$(mysqladmin -uroot -p123456 status | awk '{print $4}')
        if [ $conn -gt 20 ]
            then
                echo "mariadb连接数太多了，为$conn"
                echo "mariadb连接数太多了，为$conn" | mail -s "mariadb" root@localhost
        else
            echo "mariadb连接数正常"
        fi
else
    echo "$ip不是监控主机，不需要监控"
fi
```
在本环境中有两个服务器分别运行httpd以及mariadb服务，需要判断具体服务器的服务，还是用ifconfig 过滤出具体的ip地址，判断是否符合httpd服务器（192.168.10.2）或者mysql服务器（192.168.10.3），符合则进行监控操作，反之给予信息。<br />然后判断ip，对httpd服务器需要监测其进程情况，以pgrep -l httpd | wc -l将所运行的进程打印出来，判断其是否大于峰值，大于则给予提示输出当前进程数，并以httpd alert为题为本地服务器发送邮件通知。再者需要对其访问情况进行监测，cat /var/log/httpd/access_log | awk '{print $1, $4, $9}' | sed 's/\[//g'命令实现对htppd访问情况的过滤并保存输出信息到/var/log/httpd/log.txt文件，cat /var/log/httpd/log.txt | uniq -c | sort -r | head -10 则是将过滤的信息项前十项输出到同目录下的danger.txt，将该文件地信息命名为DDOS alert发送给本地服务器，并在该文件中判断访问次数是否超过峰值，超过则提示检查系统是否被攻击，并发送本地邮件提醒管理员。<br />判断ip，对mysql服务器的服务状态进行监测，netstat命令过滤出对应端口3306的信息，判断数据是否正常运行，非运行状态则对数据库进行重启服务。再者对数据库的连接情况进行监测，mysqladmin命令判断连接数量，如果大于峰值则提示输出并向数据库服务器本地发送邮件mariadb提示管理员管理连接。<br />在此服务基础上实现master对服务脚本的分发脚本farservice.sh如下：
```
#!/bin/bash
sh /root/testsh/fenfa.sh
ips=$(cat /root/testsh/iplist)
for ip in $ips
do
    echo "主机${ip}正在进行服务监控"
    ssh root@${ip} chmod a+x /root/service.sh
    ssh root@${ip} sh /root/service.sh
done
```
在两个脚本的协助下实现对两个服务的监控，结果如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672733816929-3ecc0e1d-7f2a-4bff-aef8-a68960e3e365.png#averageHue=%23274255&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=125&id=u6efe2eb8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=156&originWidth=449&originalType=binary&ratio=1&rotation=0&showTitle=false&size=280869&status=done&style=none&taskId=ud2897a4a-e7f2-40fe-bb96-b206e76bc75&title=&width=359.2)<br />最后希望实现对两个服务器所产生日志的自动备份，编写脚本logbackup.sh实现该基本操作，具体代码如下：
```
#!/bin/bash
ip=$(ifconfig | grep "192.168.10" | awk '{print  $2}')
backup_time=$(date +%Y_%m_%d)
if [ $ip == "192.168.10.2" ]
    then
        echo "正在进行主机$ip日志备份，请稍后..."
        if [ ! -e /logbackup/httpd/httpd_log_$backup_time.tar.bz2 ]
            then
            tar -cjvf /logbackup/httpd/httpd_log_$backup_time.tar.bz2 /var/log/messages /var/log/secure /var/log/httpd/ &>/dev/null
            if [ $? -eq 0 ]
                then
                    echo "主机$ip日志备份成功"
            else
                echo "主机$ip日志备份不成功，请检查"
            fi
        else
            echo "主机$ip已经进行过日志备份"
        fi
elif [ $ip == "192.168.10.3" ]
    then
        echo "正在进行主机$ip日志备份，请稍后..."
        if [ ! -e /logbackup/mariadb/mariadb_log_$backup_time.tar.bz2 ]
           then
            tar -jcvf /logbackup/mariadb/mariadb_log_$backup_time.tar.bz2 /var/log/messages /var/log/secure /var/lib/mysql/mysql-bin.* &>/dev/null
            if [ $? -eq 0 ]
                then
                echo "主机$ip日志备份成功"
            else
                 echo "主机$ip日志备份不成功，请检查"
            fi
        else
            echo "主机$ip已经进行过日志备份"
        fi
else
    echo "主机$ip不是目标主机，不需要进行日志备份"
fi
```
跟上面同样的操作首先判断ip地址和日期。<br />对httpd服务器判断有没有今日份备份，备份过了则提示信息，没有则用tar命令创建一个备份文件包含备份具体日期的tar包，里面包含日志文件目录/var/log下的messages，secure ，httpd文件，打包完成之后输出成功与否的信息。<br />对mysql服务器则是备份日志文件目录/var/log下的messages，secure 和mysql-bin开头的日志文件，操作方式与httpd相似。<br />如果判断ip地址为非httpd服务器或者mysql服务器的ip，则提示信息表示无需备份。<br />最后让本地master主机进行脚本分发，分发脚本如下：
```
#!/bin/bash
sh /root/testsh/fenfa.sh
ips=$(cat /root/testsh/iplist)
for ip in $ips
do
    echo "主机${ip}正在进行日志备份"
    ssh root@${ip} chmod a+x /root/logbackup.sh
    ssh root@${ip} sh /root/logbackup.sh
done
```
至此已经可以实现对服务日志的自动备份，具体执行结果如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672733968247-6a4dc059-34b6-46db-bb48-3b41b57d4345.png#averageHue=%23263f51&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=130&id=u3efd0347&margin=%5Bobject%20Object%5D&name=image.png&originHeight=163&originWidth=466&originalType=binary&ratio=1&rotation=0&showTitle=false&size=304573&status=done&style=none&taskId=u4e0a6781-0de2-4929-be59-f19b4be4991&title=&width=372.8)<br />在前几天的学习中，对shell脚本编程实现服务控制以及远程管理各项功能有了一定了解和实践。对于前面的脚本需要实现选择性执行，因此需要进行编写一个整合脚本对前面的脚本进行梳理和贯通。在对项目整合之前对其功能进行分析，需要使用选择性语句进行脚本的选择执行，而选择性语句有if-else语句和case分支语句。故编写两种脚本实现整合。
```
#!/bin/bash
while [ True ]
do
echo "请选择要完成的功能,q退出:
    1--检查网络连通性
    2--文件自动分发
    3--批量用户增加
    4--批量用户删除
    5--服务器初始化
    6--系统指标收集
    7--系统自动备份
    8--自动部署
    9--数据库备份
    10--服务监控
    11--日志自动备份"
read -p "请输入： " answer
if [ $answer == 'q' ]
    then
        break
fi
if [ $answer -eq 1 ]
then
    sh /root/testsh/ping.sh
elif [ $answer -eq 2 ]
then
    sh /root/testsh/fenfa.sh
elif [ $answer -eq 3 ]
then
    sh /root/testsh/faruseradd.sh
elif [ $answer -eq 4 ]
then
    sh /root/testsh/faruserdel.sh
elif [ $answer -eq 5 ]
then
    sh /root/testsh/farinit.sh
elif [ $answer -eq 6 ]
then
    sh /root/testsh/farsystem.sh
elif [ $answer -eq 7 ]
then
    sh /root/testsh/farbackup.sh
elif [ $answer -eq 8 ]
then
    sh /root/testsh/farinstall.sh
elif [ $answer -eq 9 ]
then
    sh /root/testsh/farmybackup.sh
elif [ $answer -eq 10 ]
then
    sh /root/testsh/farservice.sh
elif [ $answer -eq 11 ]
then
    sh /root/testsh/farlogbackup.sh
else
    echo "未知功能选项，请重新输入"
fi
done
```
脚本用一个while循环语句进行周期性执行。首先用echo语句打印输出可以输如的编号，供脚本使用者选择输入。然后先用一个if语句判断输入内容是否为退出选项，是则break退出循环，反之则用用if-else语句判断输入内容，根据输入内容执行相应的脚本，完成相应的任务。<br />脚本执行结果如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672734060202-7afef05d-1206-4df2-904f-2930124e24b7.png#averageHue=%23274053&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=198&id=uab6892d1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=248&originWidth=190&originalType=binary&ratio=1&rotation=0&showTitle=false&size=189118&status=done&style=none&taskId=uf34aea23-3198-4394-8724-9e5fb0a7c6b&title=&width=152)![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672734066563-e9bec8ee-0829-47c2-bcc6-f0ab63e6a6f0.png#averageHue=%232b465a&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=118&id=u8d854872&margin=%5Bobject%20Object%5D&name=image.png&originHeight=148&originWidth=297&originalType=binary&ratio=1&rotation=0&showTitle=false&size=176338&status=done&style=none&taskId=ub998c459-f394-4c4e-b00a-6c3c2efc4a4&title=&width=237.6)<br />第二种case分支语句整合脚本main-v2.sh如下所示：
```
#!/bin/bash
while [ True ]
do
echo "请选择要完成的功能,q退出:
    1--检查网络连通性
    2--文件自动分发
    3--批量用户增加
    4--批量用户删除
    5--服务器初始化
    6--系统指标收集
    7--系统自动备份
    8--自动部署
    9--数据库备份
    10--服务监控
    11--日志自动备份"
read -p "请输入： " answer
if [ $answer == 'q' ]
then break
fi
case $answer  in
1)
sh /root/testsh/ping.sh
2)
sh /root/testsh/fenfa.sh
;;
3)
sh /root/testsh/faruseradd.sh
;;
4)
sh /root/testsh/faruserdel.sh
;;
5)
sh /root/testsh/farinit.sh
;;
6)
sh /root/testsh/farsystem.sh
;;
7)
sh /root/testsh/farbackup.sh
;;
8)
sh /root/testsh/farinstall.sh
;;
9)
sh /root/testsh/farmybackup.sh
;;
10)
sh /root/testsh/farservice.sh
;;
11)
sh /root/testsh/farlogbackup.sh
;;
esac
done
```
与v1一样，脚本用一个while循环语句进行周期性执行。首先用echo语句打印输出可以输如的编号，供脚本使用者选择输入。然后先用一个if语句判断输入内容是否为退出选项，是则break退出循环，不是则判断输入的内容，执行相应的case语句对应脚本，执行相关任务。<br />输出结果如下所示：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672734131753-a9ab460d-595b-4e29-8f88-131bc26be4c3.png#averageHue=%23264155&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=290&id=u8b2bceec&margin=%5Bobject%20Object%5D&name=image.png&originHeight=362&originWidth=322&originalType=binary&ratio=1&rotation=0&showTitle=false&size=467444&status=done&style=none&taskId=ua9aa96c5-ccc9-413a-856c-06eda59a59a&title=&width=257.6)![image.png](https://cdn.nlark.com/yuque/0/2023/png/34381447/1672734137214-bc7da3fc-85fa-42e5-8877-a0d1de45a49e.png#averageHue=%232d4b5e&clientId=u4bbb355b-6b3f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=180&id=ua5df87bb&margin=%5Bobject%20Object%5D&name=image.png&originHeight=225&originWidth=387&originalType=binary&ratio=1&rotation=0&showTitle=false&size=349168&status=done&style=none&taskId=ub197da4c-3daf-41da-9680-eeb0e3b1c51&title=&width=309.6)
