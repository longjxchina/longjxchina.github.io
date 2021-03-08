# ubuntu 18.0.4安装bind9

## 安装

```bash
apt install -y bind9 bind9-host dnsutils bind9-doc
```



## 配置

配置jjb.com解析，本机ip地址`172.22.124.136`，配置分3步，如下：



### 配置named.conf.local

```bash
zone "jjb.com" {
	type master;
	file "/etc/bind/zones/jjb.com.zone";
	allow-update { 172.22.124.136; };
};
```



### 配置named.conf.options

通过acl设置允许访问dns的ip，可以设置网段。

```bash
acl "trusted" {
	 172.22.0.0/16;
};

options {
	directory "/var/cache/bind";

	// If there is a firewall between you and nameservers you want
	// to talk to, you may need to fix the firewall to allow multiple
	// ports to talk.  See http://www.kb.cert.org/vuls/id/800113

	// If your ISP provided one or more IP addresses for stable 
	// nameservers, you probably want to use them as forwarders.  
	// Uncomment the following block, and insert the addresses replacing 
	// the all-0's placeholder.

	// forwarders {
	// 	0.0.0.0;
	// };

	//========================================================================
	// If BIND logs error messages about the root key being expired,
	// you will need to update your keys.  See https://www.isc.org/bind-keys
	//========================================================================
	listen-on port 53 { 172.22.124.136; };	
	allow-transfer { none; };
	
	dnssec-enable no;
	dnssec-validation no;

	auth-nxdomain no;    # conform to RFC1035
	forwarders {
		180.76.76.76;
		172.22.222.1;
		223.5.5.5;
	};
	recursion yes;
	allow-recursion { trusted; };
};
```



### 配置jjb.com.zone

```bash
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     dns.jjb.com. root.dns.jjb.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      dns.jjb.com.
@       IN      A       127.0.0.1
@       IN      AAAA    ::1
dns     IN      A       172.22.124.136
www     IN      A       172.22.124.136
oa      IN      A       172.22.124.122
```



## 检查配置、启动服务

### 检查配置

```bash
named-checkconf
```



### 启动服务

#### 启动

```bash
systemctl start bind9
```



#### 查看服务状态

```bash
systemctl status bind9
```



#### 设置自动启动

```bash
systemctl enable bind9
```

