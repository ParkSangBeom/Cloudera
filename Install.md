# 1.OS설치
- Centos7.2 <br/>

# 2.Network설정
vi /etc/sysconfig/network-scripts/ifcfg-ensXXX <br/>

### 설정
BOOTPROTO=none<br/>
ONBOOT=yes<br/>
IPADDR=172.16.31.180<br/>
GATEWAY=172.16.31.1<br/>
NETMASK=255.255.255.0<br/>
