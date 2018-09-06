# 1. OS설치
Centos7.2 <br/>

# 2. Network설정
### 파일
- vi /etc/sysconfig/network-scripts/ifcfg-ensXXX <br/>
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
- vi /etc/clustershell/groups<br/>
====================<br/>
all: 172.16.31.[180-182]<br/>
====================<br/>

### Pssh
- mkdir /var/www/html/Pssh<br/>
- cd /var/www/html/Pssh<br/>
- python setup.py install<br/>
- vi /root/allnodes<br/>
====================<br/>
172.16.31.180<br/>
172.16.31.181<br/>
172.16.31.182<br/>
====================<br/>
- vi /root/datanodes<br/>
====================<br/>
172.16.31.181<br/>
172.16.31.182<br/>
====================<br/>

# 8. 다른 네트워크 세팅
### Yum local repository 구축
- pssh -h /root/datanodes "mkdir /etc/yum.repos.d/.bak; mv /etc/yum.repos.d/CentOS-* /etc/yum.repos.d/.bak"<br/>
- vi /etc/yum.repos.d/Centos7.2.repo<br/>
====================<br/>
baseurl=http://172.16.31.180/Centos7.2<br/>
====================<br/>
- pscp -h /root/allnodes /etc/yum.repos.d/Centos7.2.repo /etc/yum.repos.d/<br/>
- pssh -h /root/allnodes "yum clean all"<br/>

### SELinux, Firewall 설정
- pscp -h /root/allnodes /etc/selinux/config /etc/selinux/config<br/>
- pssh -h /root/allnodes "setenforce 0"<br/>
- pssh -h /root/allnodes "systemctl stop firewalld"<br/>
- pssh -h /root/allnodes "systemctl disable firewalld"<br/>

# 9. Hosts설정
- vi /etc/hosts<br/>
====================<br/>
172.16.31.180 node00.cluster.com<br/>
172.16.31.181 node01.cluster.com<br/>
172.16.31.182 node02.cluster.com<br/>
====================<br/>
- pscp -h /root/allnodes /etc/hosts /etc/hosts<br/>

# 10. NTP설정
- pssh -h /root/allnodes "yum install -y ntp"<br/>

### 다른 네트워크 설정
- vi /etc/ntp.conf<br/>
====================<br/>
server 172.16.31.180<br/>
====================<br/>
- pscp -h /root/allnodes /etc/ntp.conf /etc/ntp.conf<br/>

### Main 설정
- vi /etc/ntp.conf<br/>
====================<br/>
server 127.127.1.0<br/>
====================<br/>
- pssh -h /root/allnodes "systemctl stop ntpd"<br/>

### Syncronize the time 설정
- systemctl restart ntpd<br/>
- pssh -h /root/datanodes "ntpdate 172.16.31.180"
- pssh -h /root/datanodes "systemctl restart ntpd"<br/>
- pssh -h /root/allnodes "systemctl enable ntpd"<br/>

### 확인
- clush -B -a "ntpq -pn"<br/>

# 11. ulimit 설정
- vi /etc/security/limits.conf<br/>
====================<br/>
root soft nofile 64000<br/>
root hard nofile 64000<br/>
====================<br/>
- pscp -h /root/allnodes /etc/security/limits.conf /etc/security/limits.conf<br/>

### 확인
- clush -B -a ulimit -n<br/>

# 12. Transparent Hugepage Compaction 비활성화
- pssh -h /root/allnodes "echo never > /sys/kernel/mm/transparent_hugepage/defrag"<br/>
- pssh -h /root/allnodes "echo never > /sys/kernel/mm/transparent_hugepage/enabled"<br/>
- vi /etc/default/grub<br/>
====================<br/>
- GRUB_CMDLINE_LINUX="rd.lvm.lv=centos/root rd.lvm.lv=centos/swap crashkernel=auto rhgb quiet transparent_hugepage=never"<br/>
====================<br/>
- pscp -h /root/allnodes /etc/default/grub /etc/default/grub<br/>
- pssh -h /root/allnodes "grub2-mkconfig -o /boot/grub2/grub.cfg"<br/>

### 확인
- cat /proc/meminfo<br/>

# 13. vm.swappiness 설정
- pssh -h /root/allnodes "sysctl -w vm.swappiness=1"<br/>
- pssh -h /root/allnodes "sed -i '$ a\vm.swappiness=1' /etc/sysctl.conf"<br/>

# 14. 재부팅
- pssh -h /root/allnodes "shutdown -r now"<br/>

# 15. CM, CDH준비
### 다운로드
- cm 파일 : http://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.15.0/RPMS/x86_64/<br/>
- cdh 파일 : https://archive.cloudera.com/cdh5/parcels/latest/<br/>

### CM repo생성
- yum -y install createrepo<br/>
- mkdir /var/www/html/CM5.15<br/>
- createrepo /var/www/html/CM5.15<br/>
- vi /etc/yum.repos.d/CM5.15.repo<br/>
====================<br/>
[CM5.15]<br/>
name=cm5.15<br/>
baseurl=http://172.16.31.180/CM5.15<br/>
gpgcheck=0<br/>
enabled=1<br/>
====================<br/>
- pscp -h /root/allnodes /etc/yum.repos.d/CM5.15.repo /etc/yum.repos.d/CM5.15.repo<br/>
- pssh -h /root/allnodes "yum clean all"<br/>

### CDH
- mkdir /var/www/html/CDH5.15<br/>

# 16. Java 
### 기존 JDK 삭제
- pssh -h /root/allnodes "yum -y remove java"<br/>

### JDK 설치
- pssh -h /root/allnodes "yum -y install oracle-*"<br/>
- vi ~/.bashrc<br/>
====================<br/>
export JAVA_HOME=/usr/java/jdk1.7.0_67-cloudera<br/>
export PATH=$JAVA_HOME/bin:$PATH<br/>
====================<br/>
- pscp -h /root/allnodes /root/.bashrc /root/.bashrc<br/>
- pssh -h /root/allnodes "source /root/.bashrc"<br/>

### 확인
- clush -B -a "java -version"<br/>

# 17. MariaDB 설치
- yum -y install mariadb-server<br/>
- systemctl restart mariadb<br/>
- systemctl stop mariadb<br/>

### Log파일 백업
- mkdir /var/lib/mysql/.bak<br/>
- mv /var/lib/mysql/ib_logfile* /var/lib/mysql/.bak<br/>

### Option파일 설정
- vi /etc/my.cnf<br/>
====================<br/>
https://www.cloudera.com/documentation/enterprise/latest/topics/install_cm_mariadb.html<br/>
이곳에서 확인<br/>
====================<br/>

### DB시작
- systemctl restart mariadb<br/>
- systemctl enable mariadb<br/>
- /usr/bin/mysql_secure_installation<br/>

### JDBC설치
- pssh -h /root/allnodes "mkdir -p /usr/share/java"
- pscp -h /root/allnodes /var/www/html/JDBC/mysql-connector-java-5.1.47-bin.jar /usr/share/java/mysql-connector-java.jar<br/>

### Table생성
- mysql -u root -p<br/>
====================<br/>
create database scm DEFAULT CHARACTER SET utf8;<br/>
grant all on scm.* TO 'scm'@'%' IDENTIFIED BY 'scm';<br/>
create database amon DEFAULT CHARACTER SET utf8;<br/>
grant all on amon.* TO 'amon'@'%' IDENTIFIED BY 'amon';<br/>
create database rman DEFAULT CHARACTER SET utf8;<br/>
grant all on rman.* TO 'rman'@'%' IDENTIFIED BY 'rman';<br/>
create database metastore DEFAULT CHARACTER SET utf8;<br/>
grant all on metastore.* TO 'hive'@'%' IDENTIFIED BY 'hive';<br/>
create database nav DEFAULT CHARACTER SET utf8;<br/>
grant all on nav.* TO 'nav'@'%' IDENTIFIED BY 'nav';<br/>
create database navms DEFAULT CHARACTER SET utf8;<br/>
grant all on navms.* TO 'navms'@'%' IDENTIFIED BY 'navms';<br/>
create database sentry DEFAULT CHARACTER SET utf8;<br/>
grant all on sentry.* TO 'sentry'@'%' IDENTIFIED BY 'sentry';<br/>
create database oozie DEFAULT CHARACTER SET utf8;<br/>
grant all on oozie.* TO 'oozie'@'%' IDENTIFIED BY 'oozie';<br/>
create database hue DEFAULT CHARACTER SET utf8 ;<br/>
grant all on hue.* to 'hue'@'%' identified by 'hue';<br/>
flush privileges;<br/>
====================<br/>

# 18. Cloudera Manager 패키지 설치
- pssh -h /root/allnodes "yum -y install cloudera-manager-daemons"<br/>
- pssh -h /root/allnodes "yum -y install cloudera-manager-agent"<br/>
- clush -B -a "rpm -qa | grep cloudera"<br/>
