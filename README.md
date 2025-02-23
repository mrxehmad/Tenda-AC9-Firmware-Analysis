<div align="right">
  <a href="README.md">English (EN)</a> | 
  <a href="README-cn.md">中文 (CN)</a>
</div>


# Tenda AC9 Firmware Analysis

This repository contains an analysis of the Tenda AC9 firmware version `US_AC9V1.0BR_V15.03.05.15_multi_TDE01.bin`. The goal is to explore its structure, network behavior, and potential security concerns, including hardcoded IPs, domains, and open ports.

## Related Resources
- [Tenda AC15 Firmware Analysis](https://github.com/SC0p30N3/Tenda-AC15-Firmware-V15.03.05.18)
- [Tenda Backdoor Blog](https://ea.github.io/blog/2013/10/18/tenda-backdoor/)
- [Tenda Reverse Engineering](https://github.com/latonita/tenda-reverse)
- [**api.cloud.tenda.com.cn**](https://github.com/mrxehmad/api.cloud.tenda.com.cn)

## Firmware Details
### File Type
```
US_AC9V1.0BR_V15.03.05.15_multi_TDE01.bin: u-boot legacy uImage, \002, Linux/ARM, OS Kernel Image (lzma), 7458816 bytes, Thu Oct 29 06:32:19 2020, Load Address: 0X80000000, Entry Point: 0X80008000, Header CRC: 0XE209FAEB, Data CRC: 0X2FDDD71E
```

### Extracting the Firmware with Binwalk
To analyze the firmware, extract its components using `binwalk`:
1. **Install binwalk**:
   ```bash
   sudo apt install binwalk
   ```
2. **Analyze the structure**:
   ```bash
   binwalk US_AC9V1.0BR_V15.03.05.15_multi_TDE01.bin
   ```
   - Output shows a TRX header, LZMA-compressed kernel, and SquashFS filesystem.
3. **Extract contents**:
   ```bash
   binwalk -e US_AC9V1.0BR_V15.03.05.15_multi_TDE01.bin
   ```
   - Creates `_US_AC9V1.0BR_V15.03.05.15_multi_TDE01.bin.extracted/` with kernel (`5C.LZMA`) and filesystem (`18EB48.squashfs`).
4. **Decompress the filesystem**:
   ```bash
   unsquashfs -f -d squashfs-root 18EB48.squashfs
   ```
   - Access files in `squashfs-root/`.

## Hardcoded IPs and Domains
The firmware contains hardcoded references to external servers, likely for cloud services or updates:
- **Domains**:
  - `cloud.tenda.com.cn`
  - `download.cloud.tenda.com.cn`
  - `api.cloud.tenda.com.cn`
- **IPs**:
  - `182.254.148.51`
  - `182.254.218.214`
  - `182.254.136.200`

Additional domains found in binaries:
- `www.baidu.com`, `www.qq.com`, `www.taobao.com`, `www.microsoft.com`, `www.apple.com`, `www.google.com` (possibly for speed tests or internet conectivity).

## Nmap Scan Results
Performed on the router at `10.1.15.1`:
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

## Key Findings
### Source Code References
- **Paths in Firmware**:
  - `behavior_manager_api/ip_mac_bind_api.c`
  - `/home/tenda_work/ac9v1.0/new/develop_svn2383/cbb/cbb_tpi/include/behavior_manager/bm_common.h`
  - These suggest hardcoded IPs/domains originate from Tenda’s development environment.

### Binaries and Libraries
Extracted files reveal network-related components:
- `./bin/netctrl`: Network control utility.
- `./lib/libtpi.so`: Tenda proprietary library (likely handles cloud communication).
- `./usr/bin/app_data_center`: FastCGI backend for Luci on port 8188.
- `./lib/libupnp.so`, `./bin/miniupnpd`: UPnP support (possibly port 9000).
- `./usr/bin/nginx`: Web server on port 8180.
- `./bin/httpd`: Web server on port 80.
- `./bin/upgrade`, `./bin/3322ip`, `./bin/auto_discover`: Firmware update, dynamic DNS, and discovery tools.

### Suspicious Strings
#### `./bin/auto_discover`
Sends MAC address and timestamp to `api.cloud.tenda.com.cn/route/mac/v1`:
```http
POST %s HTTP/1.0
Connection: Keep-Alive
Accept: */*
User-Agent: Mozilla/5.0 (compatible; MSIE 5.01; Windows NT 5.0)
Content-Length: %d
Host: %s
{"mac":["%s"]}
```
- **Purpose**: Device tracking or registration.

- I have created a **backed server** to handle and track requests from that api [**api.cloud.tenda.com.cn**](https://github.com/mrxehmad/api.cloud.tenda.com.cn)

#### `./bin/httpd`
Handles web requests, possibly fetching advertisements:
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
- **Explanation**: This binary serves the router’s web interface (port 80) and queries `api.cloud.tenda.com.cn/route/adverts/v1` for ad URLs. If the request fails, it falls back to a default link (e.g., `http://www.tenda.com.cn`). The outdated User-Agent suggests minimal updates to this component.

#### `./bin/speedtest`
Contains test domains (e.g., `www.taobao.com`), likely for bandwidth testing.

#### `./bin/88ip`
Sends XML requests to `link.dipserver.com`:
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
- **Purpose**: Dynamic DNS or service authentication.

#### `./bin/logserver`
Attempts to send logs to an external server (fails if connectivity is blocked).

#### `./usr/sbin/telnetd`
Telnet server binary exists but no active connection found in the default config. Check `/etc/init.d/` for startup scripts (e.g., `S50telnetd`).

### iptables Rules
Hardcoded firewall rules:
```
iptables -D OUTPUT -p udp --dport %d -m state --state NEW -j %s
iptables -F %s
iptables -X %s
iptables -N %s
iptables -A OUTPUT -p udp --dport %d -m state --state NEW -j %s
iptables -A %s -o %s -j DROP
iptables -D %s -o %s -j DROP
```
- Likely controls outbound traffic (e.g., to cloud servers).

### Port 8188
From `nginx_init.sh` and `nginx.conf`:
- `spawn-fcgi -a 127.0.0.1 -p 8188 /usr/bin/app_data_center`
- Nginx proxies `/cgi-bin/luci/` to `127.0.0.1:8188`.
- **Purpose**: Backend for Luci web interface.

## Security Notes
- **Hardcoded IPs/Domains**: Potential for man-in-the-middle attacks or tracking.
- **Non-Standard Ports (5500, 10004)**: Could indicate debug backdoors.
- **Telnet**: Present but not enabled by default—verify runtime behavior.

## Contributing
Feel free to fork, analyze further with tools like Ghidra, and submit findings via pull requests!
