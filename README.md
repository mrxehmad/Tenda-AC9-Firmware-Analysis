i was unable to find any clue becouse i am noob in debugging but i have fine some things that can linked to those files 

blew ti readme of that file 


## Tenda AC9


'''

https://github.com/SC0p30N3/Tenda-AC15-Firmware-V15.03.05.18
https://ea.github.io/blog/2013/10/18/tenda-backdoor/
https://github.com/latonita/tenda-reverse

'''
###File Type
```US_AC9V1.0BR_V15.03.05.15_multi_TDE01.bin: u-boot legacy uImage, \002, Linux/ARM, OS Kernel Image (lzma), 7458816 bytes, Thu Oct 29 06:32:19 2020, Load Address: 0X80000000, Entry Point: 0X80008000, Header CRC: 0XE209FAEB, Data CRC: 0X2FDDD71E
```
 #tell them how to extract framware with binwalk 

hardcode Ips

cloud.tenda.com.cn
182.254.148.51
download.cloud.tenda.com.cn
182.254.218.214
182.254.136.200


### Nmap Scan
Starting Nmap 7.93 ( https://nmap.org ) at 2025-02-21 20:36 PKT
Nmap scan report for wifi.local (10.1.15.1)
Host is up (0.023s latency).
Not shown: 995 closed tcp ports (conn-refused)
PORT      STATE SERVICE
80/tcp    open  http
5500/tcp  open  hotline
8180/tcp  open  unknown
9000/tcp  open  cslistener
10004/tcp open  emcrmirccd

Nmap done: 1 IP address (1 host up) scanned in 0.39 seconds




behavior_manager_api/ip_mac_bind_api.c
/home/tenda_work/ac9v1.0/new/develop_svn2383/cbb/cbb_tpi/include/behavior_manager/bm_common.h

this contain hardcoded ips and domains

cloud.tenda.com.cn
182.254.148.51
download.cloud.tenda.com.cn
182.254.218.214
182.254.136.200
182.254.148.51

and many other sus strings 

like 


./bin/netctrl



./lib/libtpi.so
./usr/bin/app_data_center
/usr/bin/app_data_center
./lib/libupnp.so
./usr/bin/nginx
./bin/miniupnpd
./usr/sbin/wl
./bin/httpd
./bin/upgrade
./bin/3322ip
./lib/libtpi.so
./bin/auto_discover

```
grep -r "taobao" .
grep: ./bin/wan_surf: binary file matches


iptables -D OUTPUT -p udp --dport %d -m state --state NEW -j %s
iptables -F %s
iptables -X %s
iptables -N %s
iptables -A OUTPUT -p udp --dport %d -m state --state NEW -j %s
iptables -A %s -o %s -j DROP
iptables -D %s -o %s -j DROP
www.baidu.com
www.qq.com
www.taobao.com
www.microsoft.com
www.apple.com
www.google.com

```


grep -r "8188" .
./etc_ro/nginx/conf/nginx_init.sh:spawn-fcgi -a 127.0.0.1 -p 8188 /usr/bin/app_data_center
./etc_ro/nginx/conf/nginx.conf:			fastcgi_pass 127.0.0.1:8188;
grep: ./usr/sbin/wl: binary file matches



 grep -r "api.cloud.tenda.com.cn" .
grep: ./bin/auto_discover: binary file matches
grep: ./bin/httpd: binary file matches


## ./bin/auto_discover
it sends data to api.cloud.tenda.com.cn/route/mac/v1 endpoint that contain time and macaddress

POST %s HTTP/1.0
Connection: Keep-Alive
Accept: */*
User-Agent: Mozilla/5.0 (compatible; MSIE 5.01; Windows NT 5.0)
Content-Length: %d
Host: %s
{"mac":["%s"]}
api.cloud.tenda.com.cn
 

## this is from /bin/httpd idont know what it is jsut add explanation for it here 
 get adver url == %s
response from cloud: %s
get link %s from cloud!
get link failed! use %s as default!
api.cloud.tenda.com.cn
/route/adverts/v1
http://www.tenda.com.cn
GET %s HTTP/1.0
Connection: Close
Accept: */*
User-Agent: Mozilla/5.0 (compatible; MSIE 5.01; Windows NT 5.0)
Host: %s
advers
cloud data error, use %s as default!
ERROR!

## /bin/speedtest

## ./bin/88ip


POST /elink/elink.dll/ HTTP/1.1
Connection: Keep-Alive
Pragma: No-Cache
Content-Type: text/xml; Charset=UTF-8
Accept: */*
Accept-Language: zh-cn
User-Agent: Mozilla/4.0 (compatible; Win32; WinHttp.WinHttpRequest.5)
charset: UTF-8
Content-Length: %d
Host: link.dipserver.com
<?xml version="1.0"?>
<ELinkPacket><MsgType>ActiveTestReq</MsgType><Version>1.0</Version><UserName>%s</UserName><UserPwd>%s</UserPwd></ELinkPacket>
Content-Length: 
<Result>
</Result>



## /bin/logserver
send log to server failed


and in last there is telnet binary

but not telnet connetion

