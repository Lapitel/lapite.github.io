---
title: "도커를 이용한 ELK-Stack 오답노트"
date: 2020-08-31 18:15:00 +0900
categories: 도커 ELK ELK-Stack 엘라스틱서치 filebeat
---
#엘라스틱서치 오류
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
ulimit -n 65536
https://www.elastic.co/guide/en/elasticsearch/reference/current/file-descriptors.html

[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
sudo /sbin/sysctl -w vm.max_map_count=262144
https://taetaetae.github.io/2019/02/10/access-log-to-elastic-stack/
https://linux.systemv.pe.kr/elasticsearch-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EC%84%A4%EC%A0%95/

[3]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured
vim elasticsearch/conf/elasticsearch.yml
아래내용 설정
node.data : true
network.host : 0.0.0.0
discovery.seed_hosts : []
cluster.initial_master_nodes : [자신의 IP]
https://stackoverflow.com/questions/59350069/elasticsearch-7-start-up-error-the-default-discovery-settings-are-unsuitable-f

#리눅스 방화벽 설정 해제
​```
$ firewall-cmd --permanent --zone=public --add-port=9200/tcp
$ firewall-cmd --reload
$ firewall-cmd --list-ports
​```
https://msyu1207.tistory.com/entry/Elasticsearch-%EC%84%A4%EC%B9%98-%EB%B0%8F-%EC%99%B8%EB%B6%80-%ED%97%88%EC%9A%A9

#Logstash 필터 grok패턴확인
http://grokdebug.herokuapp.com/
http://grokconstructor.appspot.com/do/match
출처: https://sjh836.tistory.com/69 [빨간색코딩]

LogFormat "%h \"%{X-Forwarded-For}i\" \"%{X-Forwarded-Proto}i\" %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
39.7.46.22 "-" "-" - - [31/Aug/2020:14:11:31 +0900] "GET /download.do?ftp=10&fnm=1d1a424a2f804e0c88f1a33d9b966653.png HTTP/1.1" 200 - "http://103.244.111.135/fsec/dataProd/generalDataProdDetail.do?cmnx=44&goods_id=ff262e50-dd3c-11ea-8c99-f765e0d1a10a" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.125 Safari/537.36"

최종: 
filter {
        grok {
                match => { "message" => "%{IPORHOST:proxy_ip} \"(?:%{IPORHOST:client_ip}|-)\" \"(?:%{WORD:http}|-)\" %{USER:ident} %{USER:auth} \[%{HTTPDATE:apache_timestamp}\] \"%{WORD:method} %{DATA:request_uri} HTTP/%{NUMBER:http_version}\" %{NUMBER:status} (?:%{NUMBER:byte_response}|-) \"%{DATA:referrer}\" \"%{DATA:agent}\"" }
        }
}

#elastic search 503 에러
cluster.initial_master_nodes 설정
http://kangmyounghun.blogspot.com/2019/05/elasticsearch-70.html

#filebeat 다운로드 경로
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.8.0-linux-x86_64.tar.gz

#filebeat 오작동시 ( 이틀 헤맴.. )
filebeat 설정파일에서 enable=true 확인

#docker 중지된 컨테이너 모두 삭제
docker rm -v $(docker ps -a -q -f status=exited)
https://subicura.com/2017/01/19/docker-guide-for-beginners-2.html
