# 1. OS설치
Centos7.2 <br/>

# 2. Network설정
### 파일
- vi /etc/sysconfig/network-scripts/ifcfg-ensXXX <br/>

### 설정
====================<br/>
BOOTPROTO=none<br/>
ONBOOT=yes<br/>
IPADDR=172.16.31.180<br/>
GATEWAY=172.16.31.1<br/>
NETMASK=255.255.255.0<br/>
====================<br/>

### 네트워크 재시작
- systemctl restart network<br/>

# 3. SSH Key
### Key생성
- ssh-keygen<br/>

### Key복사
- for IP in {180..182}; do echo -n "$IP -> "; ssh-copy-id -i ~/.ssh/id_rsa.pub 172.16.31.$IP; done<br/>

# 4. Yum local repository 구축
### Centos Package 파일복사
- mkdir -p /var/www/html/Centos7.2<br/>
- mkdir /media/cdrom<br/>
- mount /dev/cdrom /media/cdrom/<br/>
- cp -r /media/cdrom/* /var/www/html/Centos7.2/<br/>

### 기존 repo파일 백업
- mkdir /etc/yum.repos.d/.bak<br/>
- mv /etc/yum.repos.d/CentOS-* /etc/yum.repos.d/.bak/<br/>

### Centos repo파일 생성
- vi /etc/yum.repos.d/Centos7.2.repo<br/>
====================<br/>
[Centos7.2]<br/>
name=centos7.2<br/>
#baseurl=http://172.16.31.180/Centos7.2<br/>
baseurl=file:///var/www/html/Centos7.2<br/>
gpgcheck=0<br/>
enabled=1<br/>
====================<br/>
- yum clean all<br/>

# 5. Http 데몬 설치
### 설치
- yum -y install httpd

### 세팅
- vi /etc/httpd/conf/httpd.conf<br/>
====================<br/>
ServerName 172.16.31.180:80<br/>
====================<br/>
- chcon -R -t httpd_sys_content_t /var/www/html/Centos7.2<br/>

### 서비스 시작
- systemctl restart httpd<br/>
- systemctl enable httpd<br/>

# 6. SELinux, Firewall 설정
### SELinux
- vi /etc/selinux/config<br/>
====================<br/>
SELINUX=disabled<br/>
====================<br/>
- setenforce 0<br/>

### Firewall
- systemctl stop firewalld<br/>
- systemctl disable firewalld<br/>

# 7. Clush, Pssh설치
### Cluster
- mkdir /var/www/html/Cluster<br/>
- cd /var/www/html/Cluster<br/>
- yum -y install python2-clustershell-1.8-1.el7.noarch.rpm<br/>
- yum -y install clustershell-1.8-1.el7.noarch.rpm<br/>

### Pssh
- mkdir /var/www/html/Pssh<br/>
- cd /var/www/html/Pssh<br/>
- python setup.py install<br/>

# 8. 다른 네트워크 세팅
