## swarm 을 이용한 실전 애플리케이션 개발 - todo app



### 1. MySQL 서비스 구축

>Master/slave 이미지생성
>
>컨테이너의 설정 파일 및 스크립트 다루는 방법
>
>데이터베이스 초기화
>
>Master/slave 간의 Replication 설정



- Master/slave 구조 구축
  - Docker hub의 Mysql:5.7 이미지로 생성
  - Master/slave 컨테이너는 두 역할을 모두 수행할 수 있는 하나의 이미지로 생성
  - MYSQL_MASTER 환경 변수의 유무에 따라 Master, slave 결정
  - replicas 값을 설정하여 slave 개수 조정



쓰고자하는 도커보다 선행 작업이 필요할 때 entrykit을 쓴다

mysql  같은 경우 서버 아이디를 CMD ["mysqld"] 보다 먼저 설정해야 마스터 슬레이브를 정해 줄 수 있다.

따라서 entrykit을 써준다

```
ENTRYPOINT [ \
  "prehook", \
    "add-server-id.sh", \
    "--", \
  "docker-entrypoint.sh" \
]
```

- add-server-id.sh 실행되며 서버 아이디 자동생성

  ```
  #!/bin/bash -e
  OCTETS=(`hostname -i | tr -s '.' ' '`)
  
  MYSQL_SERVER_ID=`expr ${OCTETS[2]} \* 256 + ${OCTETS[3]}`
  # 127.0.27.3 이라 하면 2번째 3번째 27 3 으로 아이디 생성함 => 아이디 겹칠 일이 없음
  echo "server-id=$MYSQL_SERVER_ID" >> /etc/mysql/mysql.conf.d/mysqld.cnf
  
  ```

  

1. 이미지 빌드

```
C:\Users\HPE\docker\day03\swarm\todo\tododb> docker build -t localhost:5000/ch04/tododb:latest .
```

2. 

```
docker push localhost:5000/ch04/tododb:latest
```

3. http://localhost:5000/v2/_catalog 에서 확인

   ```
   curl http://localhost:5000/v2/_catalog
   ```

4. 

```
docker exec -it manager sh
docker network create --driver=overlay --attachable todoapp
```

5. //manager  쉘에서ㅇ

```
 docker stack deploy -c /stack/todo-mysql.yml todo_mysql
```

6. 잘 됐는지 확인

```
docker stack ls
docker service ls
docker service logs todo_mysql_master  확인해보니 오류남;

docker service ps todo_mysql_master

문제는 윈도우하고 리눅스 캐리지 리턴할때의 문제

모든 sh 파일에서 CRLF를 LF로 바꿔주자 그리고 이미지 빌드부터 다시 해주자
```

- 다 됐으면 worker01 02 03 들어가서 각각의 ip 확인

  ```
  docker exec -it worker03 sh
  hostname -i
  docker ps 
  docker exec -it 21996b105865 hostname -i      
  ```

  

| worker01            | 172.25.0.6 |      | worker02           | 172.25.0.5 |      | worker03           | 172.25.0.4 |
| ------------------- | ---------- | ---- | ------------------ | ---------- | ---- | ------------------ | ---------- |
| todo_mysql_master.1 | 10.0.5.98  |      | todo_mysql_slave.1 | 10.0.5.101 |      | todo_mysql_slave.2 | 10.0.5.100 |
|                     |            |      |                    |            |      |                    |            |



- master, slave에 쉽게 들어가자

```
docker exec -it manager `
docker service ps todo_mysql_master --no-trunc `
--filter "desired-state=running" `
--format "docker container exec -it {{.Node}} docker exec -it {{.Name}}.{{.ID}} bash"
```

- 들어가서 init-data.sh 해서 초기화 시켜주자

- 그리고 mysql -uroot -p  비번 : gihyo

- 마스터랑 슬레이브 둘 다 들어가서 show slave status\G 로 연동 됐는지 확인해보자






복습

docker ps 로 worker01 02 03 manager register 있는지 확인

docker exec -it manager sh 들어감

docker service ls

docker stack services todo_mysql  이걸로 마스터 슬레이브 이름 확인 가능

docker service ps todo_mysql_master



마스터가 worker03 sh 이면

docker exec -it worker03 sh

docker ps 

docker exec -it 이름  sh

mysql -uroot -p 





```
$ docker-compose up 
$ docker ps (registry, manager, worker01, worker02, worker03)
$ docker exec -it manager sh
	(M) $ docker swarm init  (-> join token 생성 됨)
$ docker exec -it worker01 sh  (worker02, worker03에서도 실행)
	(W1) $ docker swarm join (with join token) 
$ docker exec -t manager sh	
	(M) $ docker node ls (worker01~worker03, manager 확인)
	(M)	$ docker network create --driver=overlay --attachable todoapp
	(M) $ docker stack deploy -c /stack/visualizer.yml visualizer
	(M) $ docker stack deploy -c /stack/todo-mysql.yml todo_mysql
$ docker exec -t worker01 (or worker02, worker03) sh -> MASTER DB 접속
	ex, W1) $ docker exec -it [MASTER DB Container] bash
	ex, W1, Master Container) $ init-data.sh 
	ex, W1, Master Container) $ mysql -uroot -p tododb
	ex, W1, Master Container) mysql> select * from todo;
$ docker push localhost:5000/ch04/todoapi:latest 	
```



- 포트번호 확인

apt-get update

apt-get install -y net-tools

netstat-ntpl 