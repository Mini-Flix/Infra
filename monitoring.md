#### 모니터링 파이프라인  
  ## 맡은 역할: 각 서버로부터 로그를 수집하는 데이터 파이프라인을 구축

  ## 컴포넌트별 상세 흐름
  -------------------------------------------------------------------------------------------------
- Filebeat (로그수집)
  - 각 서버(web, WAS)에 설치되며 로그 파일을 실시간으로 모니터링
  - 새로운 로그가 기록되면 이를 읽어 설정된 kafka 토픽으로 로그를 송신

- kafka(메시지 브로커)
  - filebeat가 보내준 로그를 토픽(topic) 별로 분류해서 임시 저장하는 곳.
  - kafka는 실시간 스트리밍 메시지 큐 역할을 수행함
  - producer역할을 하는  filebeat가 로그를 보내면 kafka가 중계기 역할을 하며 consumer인 logstash로 전송
- Logstash
  - kafka에서 메시지를 구독하고 필터링 및 파싱 처리 후 Elasticsearch에 전송
  - 앞단에서 전송되는  로그는 아무처리도 되지 않았고 서버마다 로그 형식이 다름
  - 데이터 가공(필드 추가, 포맷 통일)

- Elasticsearch
  - logstash가 가공한 로그를 받아서 적재하는 NoSQL 검색엔진
  - 매우 빠른 검색기능을 제공함

- Kibana
  - Elasticsearch에 쌓인 로그 데이터를 시각화 해주는 웹UI
  - 로그 검색, 대시보드 생성, 시계열 분석 등이 가능함
  - 안전한 접속을 위해 SSH 터널링을 사용해 접속
  
```bash
ssh터널링으로 [로컬PC] --5601--> [라우터:15601] --5601--> [Kibana 서버] 순으로 접속
```
 ---------------------------------------------------------------------------------------------------
 서버에 filebeat 설치하기
 ```bash
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
cat <<EOF | sudo tee /etc/yum.repos.d/elastic.repo
[elastic-7.x]
name=Elastic 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
enabled=1
autorefresh=1
type=rpm-md
EOF

# Filebeat 설치
sudo dnf install filebeat -y
```
