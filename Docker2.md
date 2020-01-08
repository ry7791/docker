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

2. 

```
 docker build -t ry7791/mymongo:latest .
```

3. 몽고디비 서버 실행

``` 
docker run -p 27017:27017 --name mymongo1 ry7791/mymongo
```

4. 다른 창 켜서 mongo 접속

   ```
   docker exec -it mymongo1 mongo
   ```

   

1. replica set

   ​	ex) mongod --replset myapp --dbpath /폴더지정 --port 27017



ip 확인방법

1. ifconfig
2. ip addr show
3. hostname -i

172.17.0.2

rs.add("172.17.0.2:40003", {arbiterOnly: true})