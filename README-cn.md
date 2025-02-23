```markdown
<div align="right">
  <a href="README.md">English (EN)</a> | 
  <a href="README-cn.md">中文 (CN)</a>
</div>

---

# Tenda AC9 固件分析

本仓库包含对 Tenda AC9 固件版本 `US_AC9V1.0BR_V15.03.05.15_multi_TDE01.bin` 的分析。目标是探索其结构、网络行为以及潜在的安全问题，包括硬编码的 IP 地址、域名和开放端口。

## 相关资源
- [Tenda AC15 固件分析](https://github.com/SC0p30N3/Tenda-AC15-Firmware-V15.03.05.18)
- [Tenda 后门博客](https://ea.github.io/blog/2013/10/18/tenda-backdoor/)
- [Tenda 逆向工程](https://github.com/latonita/tenda-reverse)

## 固件详情

### 文件类型
```
US_AC9V1.0BR_V15.03.05.15_multi_TDE01.bin: u-boot legacy uImage, \002, Linux/ARM, OS 内核镜像 (lzma), 7458816 字节, Thu Oct 29 06:32:19 2020, 加载地址: 0X80000000, 入口点: 0X80008000, 头部 CRC: 0XE209FAEB, 数据 CRC: 0X2FDDD71E
```

### 使用 Binwalk 提取固件
要分析固件，请使用 `binwalk` 提取其组件：
1. **安装 binwalk**:
   ```bash
   sudo apt install binwalk
   ```
2. **分析结构**:
   ```bash
   binwalk US_AC9V1.0BR_V15.03.05.15_multi_TDE01.bin
   ```
   - 输出显示 TRX 头部、LZMA 压缩内核和 SquashFS 文件系统。
3. **提取内容**:
   ```bash
   binwalk -e US_AC9V1.0BR_V15.03.05.15_multi_TDE01.bin
   ```
   - 创建 `_US_AC9V1.0BR_V15.03.05.15_multi_TDE01.bin.extracted/` 文件夹，包含内核 (`5C.LZMA`) 和文件系统 (`18EB48.squashfs`)。
4. **解压文件系统**:
   ```bash
   unsquashfs -f -d squashfs-root 18EB48.squashfs
   ```
   - 访问 `squashfs-root/` 中的文件。

## 硬编码的 IP 和域名
固件中包含对外部服务器的硬编码引用，可能用于云服务或更新：
- **域名**:
  - `cloud.tenda.com.cn`
  - `download.cloud.tenda.com.cn`
  - `api.cloud.tenda.com.cn`
- **IP 地址**:
  - `182.254.148.51`
  - `182.254.218.214`
  - `182.254.136.200`

在二进制文件中还发现了其他域名：
- `www.baidu.com`, `www.qq.com`, `www.taobao.com`, `www.microsoft.com`, `www.apple.com`, `www.google.com`（可能用于测速或互联网连接检测）。

## Nmap 扫描结果
在路由器 `10.1.15.1` 上执行的扫描结果：
```
Starting Nmap 7.93 (https://nmap.org) at 2025-02-21 20:36 PKT
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
```

## 关键发现
### 源代码引用
- **固件中的路径**:
  - `behavior_manager_api/ip_mac_bind_api.c`
  - `/home/tenda_work/ac9v1.0/new/develop_svn2383/cbb/cbb_tpi/include/behavior_manager/bm_common.h`
  - 这些表明硬编码的 IP 和域名来自 Tenda 的开发环境。

### 二进制文件和库
提取的文件揭示了与网络相关的组件：
- `./bin/netctrl`: 网络控制工具。
- `./lib/libtpi.so`: Tenda 专有库（可能处理云通信）。
- `./usr/bin/app_data_center`: 为 Luci 提供支持的 FastCGI 后端，运行在端口 8188。
- `./lib/libupnp.so`, `./bin/miniupnpd`: UPnP 支持（可能是端口 9000）。
- `./usr/bin/nginx`: 运行在端口 8180 的 Web 服务器。
- `./bin/httpd`: 运行在端口 80 的 Web 服务器。

### 可疑字符串
#### `./bin/auto_discover`
向 `api.cloud.tenda.com.cn/route/mac/v1` 发送 MAC 地址和时间戳：
```http
POST %s HTTP/1.0
Connection: Keep-Alive
Accept: */*
User-Agent: Mozilla/5.0 (compatible; MSIE 5.01; Windows NT 5.0)
Content-Length: %d
Host: %s
{"mac":["%s"]}
```
- **用途**: 设备跟踪或注册。

#### `./bin/httpd`
处理 Web 请求，可能获取广告：
```
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
```
- **解释**: 此二进制文件提供路由器的 Web 界面（端口 80），并查询 `api.cloud.tenda.com.cn/route/adverts/v1` 获取广告 URL。如果请求失败，则回退到默认链接（例如 `http://www.tenda.com.cn`）。过时的 User-Agent 表明此组件几乎没有更新。

#### `./bin/speedtest`
包含测试域名（例如 `www.taobao.com`），可能用于带宽测试。

#### `./bin/88ip`
向 `link.dipserver.com` 发送 XML 请求：
```http
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
```
- **用途**: 动态 DNS 或服务认证。

#### `./bin/logserver`
尝试将日志发送到外部服务器（如果连接被阻止则失败）。

#### `./usr/sbin/telnetd`
Telnet 服务器二进制文件存在，但默认配置中未启用。检查 `/etc/init.d/` 中的启动脚本（例如 `S50telnetd`）。

### iptables 规则
硬编码的防火墙规则：
```
iptables -D OUTPUT -p udp --dport %d -m state --state NEW -j %s
iptables -F %s
iptables -X %s
iptables -N %s
iptables -A OUTPUT -p udp --dport %d -m state --state NEW -j %s
iptables -A %s -o %s -j DROP
iptables -D %s -o %s -j DROP
```
- 可能用于控制出站流量（例如到云服务器的流量）。

### 端口 8188
来自 `nginx_init.sh` 和 `nginx.conf`:
- `spawn-fcgi -a 127.0.0.1 -p 8188 /usr/bin/app_data_center`
- Nginx 将 `/cgi-bin/luci/` 代理到 `127.0.0.1:8188`。
- **用途**: 为 Luci Web 界面提供后端支持。

## 安全注意事项
- **硬编码的 IP 和域名**: 可能存在中间人攻击或跟踪的风险。
- **非标准端口（5500、10004）**: 可能指示调试后门。
- **Telnet**: 存在但默认未启用——请验证运行时行为。

## 贡献
欢迎 fork、使用 Ghidra 等工具进一步分析，并通过 pull request 提交您的发现！