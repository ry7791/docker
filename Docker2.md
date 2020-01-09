## Docker  2



### 볼륨 마운트

앞에 있는 폴더를 컨테이너의 폴더에 연결

볼륨 마운트 쓰면 윈도우에서 내용 바꿔도 컨테이너도 바뀜  연동

- 폴더를 마운트 시켜줌

```
-v /my/datadir:/var/lib/mysql
```

> 개발단계에서는 좋으나 다른 환경에서 작업할 때 앞에 있는 폴더를 사용할 수 없다





- 컨테이너 자체를 마운트 시켜줌

  - 데이터 볼륨 컨테이너 역할을 할 이미지를 Dockerfile 로 생성

    ```
    FROM busybox
    
    VOLUME /var/lib/mysql
    
    CMD ["bin/true"]
    ```

  - 이미지 빌드

    ```
    docker image build -t example/mysql-data:latest
    ```

  - 데이터 볼륨 테이너 실행 // cmd 에서 셸을 실행하는 것이 전부라 실행 끝나면 컨테이너 바로 종료됨

    ```
    docker container run -d --name mysql-date example/mysql-date:latest
    ```

  - 

```
docker run -d --rm --name mysql `
    -e "MYSQL_ALLOW_EMPTY_PASSWORD=yes" `
    -e "MYSQL_DATABASE=volume_test" `
    -e "MYSQL_USER=example" `
    -e "MYSQL_PASSWORD=example" `
     --volumes-from mysql-data `
     mysql:5.7
```



### Docker-compose

> - 한 번에 여러개의 컨테이너 생성 가능
> - 커맨드 or 복잡한 설정을 쉽게 관리

```
docker container run -d -p 9000:8080 example/echo:latest
```

컨테이너를 여러 개 만들거면 이런 명령어를 여러번 수행하기 귀찮지 않은가

그때 도커 컴포즈를 써서 여러개의 컨테이너를 한번에 생성해보자

```

정리~
```



### Docker-compose 파일로 mogoDB 설치하기

1. mongodb를 Docker container로 실행

2. Dockerfile 작성 (mongodb 설치를 위한 이미지 생성)

3. Dockerfile의 Image build

   	- ry7791/mymongodb:latest

4.  Mongodb container 생성 ->  실행

5.  Client에서 mongodb 테스트

   ```
   $ mongo -h<IP> -p<PORT>
   mogo> show dbs;
   mogo> use bookstore;
   mogo> db.books.save('{"title":"Docker compose sample"}');
   mogo> db.books.find();
   ```



### ** mongoDB 3대 설치 PRIMARY 1, SECONDARY 2

1. code . 

```
FROM mongo

CMD ["mongod","--dbpath","/data","--replSet","myapp"]
```

2.  이미지 실행

```
 docker build -t ry7791/mymongo:latest .
```

3. 몽고디비 서버 실행

``` 
docker run -p 27017:27017 --name mymongo1 ry7791/mymongo
docker run -p 27017:27017 이름 ry7791/mymongo
```

4. 다른 창 켜서 mongo 접속

   ```
   docker exec -it mymongo1 mongo
   mongo --host 172.17.0.2 --port 27017
   mongo 172.17.0.2:27017
   ```

   

1. replica set

   ​	ex) mongod --replset myapp --dbpath /폴더지정 --port 27017



서버를 기동은 mongod 

클라이언트는 mongo



- ip 확인방법

  ```
  1. ifconfig
  2. ip addr show
  3. hostname -i
  ex) docker exec -it 이름 hostname -i
  => 172.17.0.2
  ```

  

172.17.0.2

172.17.0.3

172.17.0.4



rs.add("172.17.0.3:40002")

: 27021 를 마스터 디비에 추가



docker inspect 컨테이너 : 컨테이너에 대한 정보



## docker-compose 로 replication 만드는 법

- Dockerfile

  ```
  FROM mongo
  
  RUN mkdir /usr/src/configs
  WORKDIR /usr/src/configs
  COPY replicaSet.js .
  # COPY setup.sh .
  
  CMD ["mongo", "mongodb://mongo1:27017", "./replicaSet.js"]
  ```

- docker-compose.yml

  ```
  version: "3"
  services:
      mongo1:
          image: "mongo"
          ports:
              - "27017:27017"
          volumes: 
              - $HOME/mongoRepl/mongo1:/data/db
          networks:
              - mongo-networks
          command: mongod --replSet myapp
              
      mongo2:
          image: "mongo"
          ports:
              - "27018:27017"
          volumes: 
              - $HOME/mongoRepl/mongo2:/data/db
          networks:
              - mongo-networks
          command: mongod --replSet myapp
          depends_on: 
              - mongo1
  
      mongo3:
          image: "mongo"
          ports:
              - "27019:27017"
          volumes: 
              - $HOME/mongoRepl/mongo3:/data/db
          networks:
              - mongo-networks
          command: mongod --replSet myapp 
          depends_on: 
              - mongo2
              
      mongodb_setup:
          image: "mongo_repl_setup"
          depends_on: 
              - mongo3
          networks:
              - mongo-networks
          
  networks: 
      mongo-networks:
          driver: bridge        
  ```

- replicaSet.js

  ```
  config = {
      _id: "myapp",
      members: [
          {_id:0, host: "mongo1:27017"},
          {_id:1, host: "mongo2:27017"},
          {_id:2, host: "mongo3:27017"},
      ]
  }
  
  rs.initiate(config);
  rs.conf();
  ```

- 위에 3개가 있는 폴더 경로에서 도커 이미지 빌드

  ```
  docker build --no-cache -t mongo_repl_setup .
  ```

- 도커 컴포즈

  ```
  docker-compose up
  ```

- 몽고디비 실행

  ```
  docker exec -it mysql_mongo1_1 mongo mongodb://mongo1:27017
  docker exec -it mysql_mongo2_1 mongo
  docker exec -it mysql_mongo3_1 mongo
  ```

- 프라이머리가 된 곳에서 데이터를 만들어보자

  ```
  show dbs;
  
  use bookstore;
  
  db.books.find();
  
  db.books.save({"tittle":"Docker compose files"})
  
  db.books.find();
  ```

- 프라이머리가 만들 데이터가        다른 세컨더리에  있나 확인해보자

  ```
  세컨더리에서
  use bookstore;
  db.books.find(); => 슬레이브 오케이 해주자
  rs.slaveOk()  => 이제 데이터 가져올거임 대문자 O
  db.books.find();
  ```

- 프라이머리를 죽이면 다른 세컨더리가 프라이머리가 됨

- 죽인 프라이머리를 다시 시작하면 세컨더리가 됨 slaveOk(); 해줘서 데이터 연동시키자

docker 

1. docker-compse
2. docker swarm
3. kubernetes



### docker 네트워크

- none network
  - 네트워크를 사용하지 않음
  - lo 네트워크만 사용, 외부와 단절
- container network
  - 다른 컨테이너의 네트워크 환경 공유
  - --net container: [다른 컨테이너의 id or name]
  - 두 컨테이너 사이의 네트워크 정보가 동일
- overlay network
  - 다른 호스트들 간에 네트워크 공유





## Docker Swarm

- 여러 호스트를 클러스터로 묶어주는 컨테이너 오케스트레이션

| 이름    | 역할                                                         | 대응하는 명령어 |
| ------- | ------------------------------------------------------------ | --------------- |
| Compose | 여러 컨테이너로 구성된 도커 애플리케이션을 관리(주로 단일 호스트) | docker-compose  |
| Swarm   | 클러스터 구축 및 관리 (주로 멀티 호스트)                     | docker swarm    |
| Service | 스웜에서 클러스트 안의 서비스(컨테이너 하나 이상의 집합)를 관리 | docker service  |
| Stack   | 스웜에서 여러 개의 서비스를 합한 전체 애플리케이션을 관리    | docker stack    |

### swarm 사용 해보자

-  dind 이미지 다운

  ```
  docker pull docker:19.03.5-dind
  ```

  

- docker-compose.yml 파일을 만들자

  ```
  version: "3"
  services: 
    registry:
      container_name: registry
      image: registry:latest
      ports: 
        - 5000:5000
      volumes: 
        - "./registry-data:/var/lib/registry"
  
    manager:
      container_name: manager
      image: docker:19.03.5-dind
      privileged: true
      tty: true
      ports:
        - 8000:80
        - 9000:9000
      depends_on: 
        - registry
      expose: 
        - 3375
      command: "--insecure-registry registry:5000"
      volumes: 
        - "./stack:/stack"
  
    worker01:
      container_name: worker01
      image: docker:19.03.5-dind
      privileged: true
      tty: true
      depends_on: 
        - manager
        - registry
      expose: 
        - 7946
        - 7946/udp
        - 4789/udp
      command: "--insecure-registry registry:5000"
  
    worker02:
      container_name: worker02
      image: docker:19.03.5-dind
      privileged: true
      tty: true
      depends_on: 
        - manager
        - registry
      expose: 
        - 7946
        - 7946/udp
        - 4789/udp
      command: "--insecure-registry registry:5000"
  
    worker03:
      container_name: worker03
      image: docker:19.03.5-dind
      privileged: true
      tty: true
      depends_on: 
        - manager
        - registry
      expose: 
        - 7946
        - 7946/udp
        - 4789/udp
      command: "--insecure-registry registry:5000"
  ```

- 컴포즈업하자

  ```
  docker-compose up
  ```

- manager를 swarm init 하자

  ```
  docker exec -it manager docker swarm init
  ```

- swarm init 하고 나온 토큰으로 나머지 worker1 2 3 들 등록하자

  ```
  docker exec -it worker01 토큰~
  docker exec -it worker02 토큰~
  docker exec -it worker03 토큰~
  ```

- manager 로 들어가서 ps 해보면 다 연결 된걸 알 수 있음







- 도커 레지스트리용 이미지 생성

  ```
  docker tag gihyodocker/echo:latest localhost:5000/gihyodocker/echo:latest
  ```

- 도커 레지스트리에 이미지 등록

  ```
  docker push localhost:5000/gihyodocker/echo
  ```

  - manager 컨테이너에 이미지 설치

  ```
  docker exec -it manager sh
  docker pull registry:5000/gihyodocker/echo
  ```

  









