:::tips
一、检查本地docker环境
:::
```
[root@server001 data]# cd
[root@server001 ~]# cat /etc/os-release 
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"

```
```
[root@server001 ~]# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2022-11-06 06:02:26 CST; 1 weeks 6 days ago
     Docs: https://docs.docker.com
 Main PID: 9869 (dockerd)
    Tasks: 103
   Memory: 3.2G
   CGroup: /system.slice/docker.service
           ├─  9869 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
           ├─ 88493 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 3222 -container-ip 192.168.80.2 -container-port 5032
           ├─ 88500 /usr/bin/docker-proxy -proto tcp -host-ip :: -host-port 3222 -container-ip 192.168.80.2 -container-port 5032
           ├─ 91124 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 10000 -container-ip 172.17.0.3 -container-port 10000
           ├─ 91130 /usr/bin/docker-proxy -proto tcp -host-ip :: -host-port 10000 -container-ip 172.17.0.3 -container-port 10000
           ├─ 92154 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 5032 -container-ip 172.17.0.4 -container-port 5032
           ├─ 92160 /usr/bin/docker-proxy -proto tcp -host-ip :: -host-port 5032 -container-ip 172.17.0.4 -container-port 5032
           ├─106882 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 3316 -container-ip 192.168.192.2 -container-port 3306
           ├─106888 /usr/bin/docker-proxy -proto tcp -host-ip :: -host-port 3316 -container-ip 192.168.192.2 -container-port 3306
           ├─107032 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 8233 -container-ip 192.168.192.3 -container-port 80
           ├─107037 /usr/bin/docker-proxy -proto tcp -host-ip :: -host-port 8233 -container-ip 192.168.192.3 -container-port 80
           ├─116625 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 8765 -container-ip 172.17.0.2 -container-port 80
           ├─116630 /usr/bin/docker-proxy -proto tcp -host-ip :: -host-port 8765 -container-ip 172.17.0.2 -container-port 80
           ├─120228 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 3660 -container-ip 172.17.0.5 -container-port 3000
           └─120233 /usr/bin/docker-proxy -proto tcp -host-ip :: -host-port 3660 -container-ip 172.17.0.5 -container-port 3000

Nov 18 23:10:18 server001 dockerd[9869]: time="2022-11-18T23:10:18.138714642+08:00" level=info msg="ignoring event" container=8380ebc7d020...kDelete"
Nov 18 23:10:19 server001 dockerd[9869]: time="2022-11-18T23:10:19.327219758+08:00" level=info msg="ignoring event" container=2e878e0b5acc...kDelete"
Nov 18 23:14:37 server001 dockerd[9869]: time="2022-11-18T23:14:37.073651406+08:00" level=info msg="ignoring event" container=ef5ca6380386...kDelete"
Nov 18 23:14:38 server001 dockerd[9869]: time="2022-11-18T23:14:38.313154636+08:00" level=info msg="ignoring event" container=cfc52962d8d3...kDelete"
Nov 18 23:19:15 server001 dockerd[9869]: time="2022-11-18T23:19:15.961130166+08:00" level=warning msg="Failed to allocate and map port 330... in use"
Nov 18 23:19:15 server001 dockerd[9869]: time="2022-11-18T23:19:15.988282221+08:00" level=error msg="a0eb94f3c58843ee478e275ee36338c38f7da...ntainer"
Nov 18 23:19:15 server001 dockerd[9869]: time="2022-11-18T23:19:15.993476870+08:00" level=error msg="Handler for POST /v1.25/containers/a0eb94f3c5...
Nov 19 00:47:49 server001 dockerd[9869]: time="2022-11-19T00:47:49.553244119+08:00" level=info msg="Attempting next endpoint for pull afte...unknown"
Nov 19 00:48:20 server001 dockerd[9869]: time="2022-11-19T00:48:20.063669869+08:00" level=info msg="Download failed, retrying (1/5): net/h...timeout"
Nov 19 00:53:28 server001 dockerd[9869]: time="2022-11-19T00:53:28.417569916+08:00" level=error msg="Not continuing with pull after error:...anceled"
Hint: Some lines were ellipsized, use -l to show in full.
```
:::tips
二、安装docker-compose工具
:::
```
 curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```
:::tips
安装docker-compose工具
:::
```
[root@node docker-compose]#  curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   423  100   423    0     0    362      0  0:00:01  0:00:01 --:--:--   362
100 16.2M  100 16.2M    0     0  8568k      0  0:00:01  0:00:01 --:--:-- 8568k
```
```
chmod +x /usr/local/bin/docker-compose 
```
```
[root@node docker-compose]# docker-compose version
docker-compose version 1.25.0, build 0a186604
docker-py version: 4.1.0
CPython version: 3.7.4
OpenSSL version: OpenSSL 1.1.0l  10 Sep 2019
```
:::tips
三、下载Homepage镜像
:::
```
[root@server001 ~]# docker pull ghcr.io/benphelps/homepage:latest 
latest: Pulling from benphelps/homepage
213ec9aee27d: Pull complete 
4cf055c45671: Pull complete 
bb15f8897be6: Pull complete 
8d429ea46e68: Pull complete 
db9b463faf84: Pull complete 
628af1bfc622: Pull complete 
f9b6ea337cea: Pull complete 
7342375bc9bb: Pull complete 
739199b897e4: Pull complete 
Digest: sha256:4e0bf58f98920c99d060e86b490cf037f2eec78c2d0b966fd76306b91f132cbc
Status: Downloaded newer image for ghcr.io/benphelps/homepage:latest
ghcr.io/benphelps/homepage:latest
```
:::tips
四、使用docker-cli部署
:::
```
docker run -d --name homepage \
-p 3660:3000 \
-v /data/homepage/data:/app/config \
-v /var/run/docker.sock:/var/run/docker.sock \
--restart always \
ghcr.io/benphelps/homepage:latest 
```
:::tips
五、使用docker-compose部署
:::
```
[root@server001 ~]# mkdir -p /data/homepage/data
[root@server001 ~]# chmod -R 777 /data/homepage/
[root@server001 ~]# cd /data/homepage/
```
```
[root@server001 homepage]# cat docker-compose.yaml 
version: '3.3'
services:
    homepage:   
        container_name: homepage
        image: ghcr.io/benphelps/homepage:latest
        ports:
            - 3555:3000
        environment:
            - PUID=0    
            - PGID=0  
            - TZ=Asia/Shanghai  
        restart: always    
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock
            - /data/homepage/data:/app/config
```
```
[root@server001 homepage]# docker-compose up -d
Creating homepage ... done
```
```
[root@server001 homepage]# docker ps
CONTAINER ID   IMAGE                               COMMAND                  CREATED          STATUS                    PORTS                                                     NAMES
4f7952bbd228   ghcr.io/benphelps/homepage:latest   "docker-entrypoint.s…"   52 seconds ago   Up 51 seconds (healthy)   0.0.0.0:3555->3000/tcp, :::3555->3000/tcp                 homepage
```
```
[root@server001 homepage]# docker-compose logs 
Attaching to homepage
homepage    | Listening on port 3000
```
:::tips
六、访问Homepage首页
:::
![da2f3c73c19442ee9e624583b7cda8ec.png](https://cdn.nlark.com/yuque/0/2022/png/34381447/1669210120055-d5cec3b3-2a74-4889-b8d5-f8bc700e849d.png#averageHue=%23202b3c&clientId=u6fccbce5-0f56-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u4bd685a0&margin=%5Bobject%20Object%5D&name=da2f3c73c19442ee9e624583b7cda8ec.png&originHeight=1329&originWidth=2486&originalType=binary&ratio=1&rotation=0&showTitle=false&size=62449&status=done&style=none&taskId=u7d835462-e9ac-4df2-ba88-c28849c3358&title=)
:::tips
七、修改Homepage相关配置文件
:::
```
[root@server001 data]# ls
bookmarks.yaml  docker.yaml  services.yaml  settings.yaml  widgets.yaml
[root@server001 data]# 
```
```
[root@server001 data]# cat docker.yaml 
---
# For configuration options and examples, please see:
# https://github.com/benphelps/homepage/wiki/Docker-Integration

# my-docker:
#   host: 127.0.0.1
#   port: 2375

my-docker:
  socket: /var/run/docker.sock
```
```
[root@server001 data]# cat services.yaml 
---
# For configuration options and examples, please see:
# https://github.com/benphelps/homepage/wiki/Services

- My First Group:
    - Snipe-IT:
        href: http://192.168.3.166:8233
        description:  snipe-IT固定资产平台
        server: my-docker
        container: snipe

- My Second Group:
    - webssh:
        href: http://192.168.3.166:3222
        description: WEBSSH
        server: my-docker
        container: my-webssh

- My Third Group:
    - My Third Service:
        href: http://localhost/
        description: Homepage is muzilee-test
```
:::tips

:::
```
[root@server001 data]# cat widgets.yaml 
---
# For configuration options and examples, please see:
# https://github.com/benphelps/homepage/wiki/Information-Widgets

- resources:
    cpu: true
    memory: true
    disk: /

- search:
    provider: duckduckgo
    target: _blank
```
```
[root@server001 data]# docker restart  homepage 
homepage
```
八、访问Homepage首页<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/34381447/1669210424435-00083fb1-30f4-4fe7-887e-42ae6594fcdf.png#averageHue=%232c3748&clientId=u6fccbce5-0f56-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=595&id=ufbe0924b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=744&originWidth=1911&originalType=binary&ratio=1&rotation=0&showTitle=false&size=74050&status=done&style=none&taskId=u056ef364-ce3c-42db-b89a-e9bbb23c04a&title=&width=1528.8)
