# 1. OS설치
Centos7.2 <br/>

# 2. Network설정
### 파일
vi /etc/sysconfig/network-scripts/ifcfg-ensXXX <br/>

### 설정
BOOTPROTO=none<br/>
ONBOOT=yes<br/>
IPADDR=172.16.31.180<br/>
GATEWAY=172.16.31.1<br/>
NETMASK=255.255.255.0<br/>

### 재시작
systemctl restart network<br/>

# 3. SSH Key
### Key생성
ssh-keygen<br/>

### Key
for IP in {180..181}; do echo -n "$IP -> "; ssh-copy-id -i ~/.ssh/id_rsa.pub 172.16.31.$IP; done<br/>
