## Solve not enough entropy on virtual machine

GPG와 같은 키를 생성 시 entropy가 부족해서 생성 시 시간이 많이 소모되는 경우가 있다.

이를 해결하기 위해 대부분 rngd 서비스(rng-tools 패키지에 포함)를 이용하여 해결을 하는 경우가 많은데

이로도 VM에서는 처리할 수 없는 경우가 가끔 생긴다.

이는 hypervisor에서 hardware random number generator를 사용할 수 있게 설정되어있지 않아서 인 경우가 대부분이다.



### 일반적인 방식

다음은 기존 방식인 RNG를 사용하는 방법을 간단히 보여준다.

```bash
yum install rng-tools -y
```

#### 데몬 시작
Systemd를 사용하는 경우(CentOS 7):
```bash
mkdir -p /etc/systemd/system/rngd.d
cat <<EOF > /etc/systemd/system/rngd.d/customexec.conf
[Service]
ExecStart=
ExecStart=/sbin/rngd -f -r /dev/urandom
EOF
systemctl restart rngd.service
```

System V를 사용하는 경우(CentOS 6):
```bash
sed -i 's|^\(EXTRAOPTIONS=\).*$|\1"-r /dev/urandom -o /dev/random -t 5"|' /etc/sysconfig/rngd
service rngd restart
```

### 다른 방법
HAVEGE(HArdware Volatile Entropy Gathering and Expansion)을 이용하여 RNG 데몬을 대체가능하다.

http://www.irisa.fr/caps/projects/hipsor/

#### 설치
havege는 EPEL 저장소에 있으므로, 먼저 저장소를 등록해주어야 한다.
```bash
yum install epel-release -y
yum install havege -y
```
#### 데몬 시작
Systemd를 사용하는 경우(CentOS 7):
```bash
systemctl start havege
```

System V를 사용하는 경우(CentOS 6):
```bash
service havege start
```

#### Entropy 확인
**Before:**
```bash
[root@localhost ~]# cat /proc/sys/kernel/random/entropy_avail
27
```
**After:**
```bash
[root@localhost ~]# cat /proc/sys/kernel/random/entropy_avail
1885
```

