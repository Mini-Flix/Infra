# MiniFLIX 인프라 상세 구성 가이드

## 프로젝트 역할과 기여

- 2개의 컴퓨터로 DMZ 단과 백 단의 인프라 환경 구성
- DMZ 단의 DNS, 방화벽, 라우팅, 로드밸런서 구성
- 백 단의 인프라 환경 구축 및 DNS, 방화벽, 라우팅 구성
- 아키텍처 보완 및 재구성

## DMZ 단 구성

### 1. 내부 DNS 서버 설정

#### DNS 설정 파일 편집
```bash
vi /var/named/mini.flix.db
vi /var/named/mini.flix.zones
vi /var/named/1.168.192.in-addr.arpa.db
```

#### SELinux 비활성화
```bash
vi /etc/sysconfig/selinux
# SELINUX=disabled 설정
```

### 2. 방화벽 구성

#### Firewalld 설정
```bash
# 외부 인터페이스 (ens160) 구성
firewall-cmd --zone=external --list-all
firewall-cmd --zone=external --add-service=http
firewall-cmd --zone=external --add-service=mountd
firewall-cmd --zone=external --add-service=nfs
firewall-cmd --zone=external --add-service=rpc-bind
firewall-cmd --zone=external --add-service=ssh
```

### 3. 라우팅 구성

```bash
# 192.168.0/24 요청을 ens160 인터페이스의 10.128.0.177로 라우팅
ip route add 192.168.0/24 via 10.128.0.177 dev ens160
```

### 4. 로드밸런서 구성

#### HAProxy 설정
```bash
vi /etc/haproxy/haproxy.cfg

# 80번 포트 로드밸런싱 설정
frontend http-in
    bind *:80
    default_backend web-servers

backend web-servers
    balance roundrobin
    server web01 192.168.1.103:80 check
    server web02 192.168.1.106:80 check
```

## 백 단 구성

### 1. 인프라 환경 구축

#### DHCP 서버 구성
```bash
vi /etc/dhcp/dhcpd.conf

# DHCP 설정 예시
subnet 192.168.0.0 netmask 255.255.255.0 {
    range 192.168.0.100 192.168.0.200;
    option routers 192.168.0.1;
    option domain-name-servers 192.168.0.104, 8.8.8.8;
}
```

### 2. DNS 구성

```bash
vi /var/named/mini.flix.db
vi /var/named/mini.flix.zones
vi /var/named/0.168.192.in-addr.arpa.db

# DNS 서비스 재시작
systemctl restart named
```

## 아키텍처 개선 사항

1. 물리적 제약으로 인해 스위치 대신 브리지로 연결된 2대의 PC 아키텍처로 변경
2. 각 PC를 다른 색상으로 표현하여 가시성 향상
3. HAProxy 서버의 로드밸런싱 표현
4. 백엔드의 각 분야별 책임 소재 범위 제한