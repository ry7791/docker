## Docker



- 도커 이미지를 받아오자 

```
docker image pull gihyodocker/echo:latest
```



- 내려받은 이미지를 실행해보자

```
docker run -p 8080 gihyodocker/echo:latest
```



- container 네임 생성 (네임 중복 불가)

```
docker run --name myweb1 -d -p 8080 gihyodocker/echo:latest  //myweb1 이름으로 생성
```

- 실행중인 프로세스 확인

```
docker ps
```

- 모든 프로세스 확인

```
docker ps -a
```



- 프로세스 중지

```
docker stop myweb1   //myweb1 중지
```

- container 제거

```
 docker rm myweb1	//myweb1 제거
```

- 실행중인 프로세스 아이디만 확인

```
docker ps -q
```

- 중지된 모든 프로세스들  삭제

```
docker stop $(docker ps -q) //프로세스 중지
docker rm $(docker ps -qa)
docker container prune
```

- 이미지 보기

  ```
  docker images
  ```

- 이미지 삭제

  ```powershell
  image rmi a717860e38cc // 이미지를 삭제하기 전에는 컨테이너도 삭제해야 한다
  image rmi -f a717860e38cc //이미지랑 컨테이너도 강제 삭제
  ```

  

## docker container 예제

>1. 개발 
>
>   - 경로는 C:\Users\HPE\docker\day01\simpleweb, visual studio code로
>
>   - package.json 생성
>
>     ```powershell
>     {
>         "dependencies": {
>             "express": "*"
>         },
>         "scripts": {
>             "start": "node index.js"
>         }
>     }
>     
>     ```
>
>     
>
>   - index.js 생성
>
>     ```powershell
>     const express =require('express');
>     const app = express();
>     
>     app.get('/', (req,res) =>{
>         res.send('Hi, there!');
>     });
>     
>     app.listen(8080, () => {
>         console.log('Listening on port 8080');
>     });
>     
>     ```
>
>2. npm install
>
>   - 명령창에서 ㄱㄱ
>
>3. npm start 
>
>   -> node index.js 호출됨



### 위의 내용을 docker container로 담아보자

>1. FROM -> Node 사용가능 한 이미지
>
>2. RUN -> NPM INSTALL 실행
>
>3. CMD
>
>   - simpleweb 경로에서 .code로  Dockerfile 만듬
>
>   ```powershell
>   FROM node:alpine   => node 알파인 버전 쓰겠다
>   RUN npm install
>   CMD ["npm", "start"]   => 터미널에서 치는 ***npm start***와 같은 설정
>   ```
>
>4. 명령창에서 이미지 빌드
>
>```
>docker build -t ry7791/simpleweb:latest .    // . 까먹지 말자 현재디렉토리에 있는 도커 파일이라는 뜻   t : tag  // hub.docker 계정하고 연결
>```
>
>- 도커 이미지 확인
>
>  ```
>  docker images
>  ```
>
>- 실행
>
>  ```
>  docker run -d ry7791/simpleweb:latest
>  ```
>
>- docker ps 를 통해 실행 프로그램을 봤더니 아무것도 없다 -> 이미지를 잘못 설치 했다는 것이다

> docker ps로 -a 을 프로세스 확인하고
>
> - 로그 확인 하면서  문제점을 찾아보자
>
> ```
> docker logs d4d2a008c30b
> ```

> - 문제점은 package.json 하고 index.js 가 경로에 없었어서 그랬다
>
> - Dockerfile
>
> ```json
> FROM node:alpine
> 
> COPY ./package.json  ./package.json   //윈도우에 있는 package.json를 컨테이너로 copy 
> 								      
> COPY ./index.js ./index.js			//윈도우에 있는 index.js를 컨테이너로 copy
> 
> 
> RUN npm install
> CMD ["npm", "start"]
> ```
>
> ```powershell
> FROM node:alpine
> 
> WORKDIR /home/node    
> 
> COPY ./package.json  ./package.json
> COPY ./index.js ./index.js
> // 윈도우에 있는 파일을 컨테이너로 옮겨 주는데 컨테이너 경로가 /home/node로 설정 되어 있다
> 따라서 그냥 WORKDIR 를 이용해서 현재 디렉토리를 바꿔준다.
> 
> RUN npm install 
> CMD ["npm", "start"]
> ```
>
> 

> - 실행
>
>   ```
>   docker run -d -p 8080:8080 ry7791/simpleweb:latest
>   ```
>
>   localhost:8080에서 확인가능

>- 이제 이 이미지 값을 올려보자
>
>  ```
>  docker push ry7791/simpleweb:latest
>  ```
>
>- 이미지 값을 받아와 보자 ( 기존 이미지 삭제해주고)
>
>  ```
>  docker pull ry7791/simpleweb:latest
>  ```
>
>- 실행 프로세스의 ip를 확인해보자
>
>  ```
>  docker exec -it silly_jones hostname -i  // silly_jones 는 프로세스 name
>  ```
>
>- shell 로 수행
>
>  ```
>  docker exec -it silly_jones sh // execution iput tty
>  ```
>
>- shell 에서 수정 하고 다시 가동 시키고 싶을 때  컨테이너를 스탑하고 다시 실행하면 다른 컨테이너가 또 생성 된다  그러고 싶지 않으면 `restart` 를 쓰면 된다.
>
>```
>docker restart 404547c5c8ba
>```
>
>



- no cache 설정 빌드

```
docker build --no-cache -t ry7791/simpleweb:latest .
```



--rm

프로세스 종료시 컨테이너 자동으로 제거



- 쓰지 않는 모든 프로세스 종료

  ```
  docker system prune
  ```

  





