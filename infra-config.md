# 4/29 기록

날짜: 2025년 4월 29일

- 프로젝트 정리 및 발표 연습

# 프로젝트 역할과 기여

- 2개의 컴퓨터로 DMZ 단과 백 단 의 인프라 환경 구성
- DMZ 단의 DNS 구성, 방화벽 구성, DMS 단과 백 단의 라우팅 구성, 로드벨런서 구성
- 백 단의 인프라 환경 구축 및 구성, DNS 구성, 방화벽 구성, 라우팅 구성
- 아키텍쳐 보완 및 재구성

## DMZ 단 구성

### 1. 내부 DNS 서버 구성

- vi /var/named/mini.flix.db
    
    ![image.png](4%2029%20%E1%84%80%E1%85%B5%E1%84%85%E1%85%A9%E1%86%A8%201e45283b0f4a807cbce8d2dc63188424/image.png)
    
    - 도메인 주소를 mini.flix로 고정
- vi /var/named/mini.flix.zones
    
    ![image.png](4%2029%20%E1%84%80%E1%85%B5%E1%84%85%E1%85%A9%E1%86%A8%201e45283b0f4a807cbce8d2dc63188424/image%201.png)
    
- vi /var/named/1.168.192.in-addr.arpa.db
    
    ![image.png](4%2029%20%E1%84%80%E1%85%B5%E1%84%85%E1%85%A9%E1%86%A8%201e45283b0f4a807cbce8d2dc63188424/image%202.png)
    

// systemctl restart named 

- vi /etc/sysconfig/selinux
    - SELINUX=disabled : SELinux 비활성화
- vi /etc/sysconfig/network-scripts/ifcfg-ens160 (외부 인터페이스 카드)
    
    ![image.png](4%2029%20%E1%84%80%E1%85%B5%E1%84%85%E1%85%A9%E1%86%A8%201e45283b0f4a807cbce8d2dc63188424/image%203.png)
    
    - DNS1=192.168.1.1 (라우터 주소)
    - DNS2=168.126.63.1

// systemctl stop NetworkManager (트러블 슈팅때 restart로 초기화가 안됨)

// systemctl start NetworkManager

### 2. 방화벽 구성

- firewalld로 방화벽 구성
- 외부 인터페이스 ens160 , 내부 인터페이스 ens224로 구성
    - 외부
        
        ![image.png](4%2029%20%E1%84%80%E1%85%B5%E1%84%85%E1%85%A9%E1%86%A8%201e45283b0f4a807cbce8d2dc63188424/image%204.png)
        
        - http(웹페이지 접속), mountd(NFS 마운트), nfs, rpc-bind(NFS 마운트, 외부통신), ssh
    - 내부
        
        ![image.png](4%2029%20%E1%84%80%E1%85%B5%E1%84%85%E1%85%A9%E1%86%A8%201e45283b0f4a807cbce8d2dc63188424/image%205.png)
        
        - 9092/tcp 포트를 열어두어 백단에 로그를 전송할 수 있도록 함
    - direct 규칙
        
        ![image.png](4%2029%20%E1%84%80%E1%85%B5%E1%84%85%E1%85%A9%E1%86%A8%201e45283b0f4a807cbce8d2dc63188424/image%206.png)
        
        - 트랜스 코딩 서버와 스트림 서버를 연동
        - 상세한 네트워크 시나리오를 설정할 수 있음

### 3. 라우팅 구성

- 192.168.0/24 요청이 들어오면 ens160인터페이스에 10.128.0.177 주소로 보내도록 설정
    
    ![image.png](4%2029%20%E1%84%80%E1%85%B5%E1%84%85%E1%85%A9%E1%86%A8%201e45283b0f4a807cbce8d2dc63188424/image%207.png)
    

### 4. 로드벨런서 구성

- vi /etc/haproxy/haproxy.cfg
    
    ![image.png](4%2029%20%E1%84%80%E1%85%B5%E1%84%85%E1%85%A9%E1%86%A8%201e45283b0f4a807cbce8d2dc63188424/image%208.png)
    
    - 80번 포트로 들어오는 http 요청을 라운드로빈으로 두 웹서버에 교차하여 배정

## 백 단 구성

### 1. 인프라 환경 구축 및 구성

- vi /etc/dhcp/dhcpd.conf
    
    ![image.png](4%2029%20%E1%84%80%E1%85%B5%E1%84%85%E1%85%A9%E1%86%A8%201e45283b0f4a807cbce8d2dc63188424/image%209.png)
    
    - DHCP 서버 구성
    - name서버 설정

### 2. DNS 구성

- vi /var/named/mini.flix.db
    
    ![image.png](4%2029%20%E1%84%80%E1%85%B5%E1%84%85%E1%85%A9%E1%86%A8%201e45283b0f4a807cbce8d2dc63188424/image%2010.png)
    
    - 도메인 주소를 mini.flix로 고정
- vi /var/named/mini.flix.zones
    
    ![image.png](4%2029%20%E1%84%80%E1%85%B5%E1%84%85%E1%85%A9%E1%86%A8%201e45283b0f4a807cbce8d2dc63188424/image%2011.png)
    
- vi /var/named/0.168.192.in-addr.arpa.db
    
    ![image.png](4%2029%20%E1%84%80%E1%85%B5%E1%84%85%E1%85%A9%E1%86%A8%201e45283b0f4a807cbce8d2dc63188424/image%2012.png)
    

// systemctl restart named 

- vi /etc/sysconfig/network-scripts/ifcfg-ens160 (외부 인터페이스 카드)
    
    ![image.png](4%2029%20%E1%84%80%E1%85%B5%E1%84%85%E1%85%A9%E1%86%A8%201e45283b0f4a807cbce8d2dc63188424/image%2013.png)
    
    - DNS1=192.168.0.104 (라우터 주소)
    - DNS2=8.8.8.8

// systemctl stop NetworkManager (트러블 슈팅때 restart로 초기화가 안됨)

// systemctl start NetworkManager

### 3. 방화벽 구성

- firewalld로 방화벽 구성
    - 외부
        
        ![image.png](4%2029%20%E1%84%80%E1%85%B5%E1%84%85%E1%85%A9%E1%86%A8%201e45283b0f4a807cbce8d2dc63188424/image%2014.png)
        
    - 내부
        
        ![image.png](4%2029%20%E1%84%80%E1%85%B5%E1%84%85%E1%85%A9%E1%86%A8%201e45283b0f4a807cbce8d2dc63188424/image%2015.png)
        
    - direct 규칙
        
        ![image.png](4%2029%20%E1%84%80%E1%85%B5%E1%84%85%E1%85%A9%E1%86%A8%201e45283b0f4a807cbce8d2dc63188424/image%2016.png)
        
        - 1,2 줄 : 로그스트림 허용
        - 3, 4줄 : 트랜스코딩 서버와 스트리밍 서버 연결
        - 5, 6줄 : NFS 포트 개방

### 4. 라우팅 구성

- ip route
    
    ![image.png](4%2029%20%E1%84%80%E1%85%B5%E1%84%85%E1%85%A9%E1%86%A8%201e45283b0f4a807cbce8d2dc63188424/image%2017.png)
    
    - 192.168.1.0/24 주소는 10.128.0.66의 DMZ 단의 라우터로 보냄

## 아키텍쳐 보완 및 재구성

### 기존의 아키텍쳐

![image.png](4%2029%20%E1%84%80%E1%85%B5%E1%84%85%E1%85%A9%E1%86%A8%201e45283b0f4a807cbce8d2dc63188424/image%2018.png)

### 수정한 아키텍쳐

![image.png](4%2029%20%E1%84%80%E1%85%B5%E1%84%85%E1%85%A9%E1%86%A8%201e45283b0f4a807cbce8d2dc63188424/image%2019.png)

### 수정 내용

1. 첫 번째 인프라 설계 시점에서는 각 서버를 스위치를 통해 연결할 수 있다고 판단하였지만 상황상 불가능
    1. 물리적으로 연결돼있지 않아 bridge로 연결된 2대의 pc 사용하는 아키텍쳐로 변경
2. 각 pc를 다른 색으로 표현하여 가시성 향상
3. haproxy 서버의 로드벨런싱 표현
4. 백단의 각 분야별 책임 소재 범위 제한