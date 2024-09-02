# Docker Network

## Docker Network Experiment - 24.09.03

### Preparation

- 아주 간단한 Flask App 생성

    - GET요청을 받으면 "Hi! I'm Flask!"라는 응답을 주는 앱

- Dockerization

```sh
docker build -t test_flask_app .
```

### Test1

- 도커 네트워크 생성 없이 컨테이너만 올리기

```sh
docker run -d -p 5000:5000 test_flask_app
```

- 호스트 머신에서 localhost:5000으로 요청 보내기 

    - 결과: 요청O, 응답

```
# 통신 흐름

1. 호스트머신에서 127.0.0.1:5000으로 요청을 보냄
    - 해당 요청은 호스트머신의 5000번 포트로 들어감

2. 포트 매핑 설정에 따라 이 요청은 Docker가 관리하는 네트워크를 통해 컨테이너의 5000번 포트로 전달됨(내부적 라우팅)
    - 컨테이너 안의 Flask App이 요청을 전달 받음

3. Flask App은 요청을 처리하고 나서의 응답을 컨테이너의 5000번 포트로 보냄
    - 이 응답은 다시 도커 네트워크의 내부적 라우팅을 통해 호스트머신의 5000번 포트로 전달됨

4. 호스트머신의 5000번 포트에 응답이 도달했음 !

```

### Test2

- preparation 

    - 명시적으로 같은 네트워크에 할당하지 않고 요청을 보낼 수 있는 컨테이너를 하나 더 띄워보자 -> TestUbuntu

```sh
docker run -d test_ubuntu
```

- test_ubuntu에서 test_flask_app으로 요청 보내기

    - 이때, *Private IP*를 사용!

```sh
# test ubuntu에서
curl http://172.17.0.2:5000
```

- 결과: 응답까지 잘 옴!

```
# 도커 네트워크 기본 할당 방식

- Docker는 기본적으로 bridge 네트워크를 사용하여 컨테이너를 띄움

    - bridge 네트워크는 Docker Host(호스트머신)에 의해 관리되며, 모든 컨테이너가 이 네트워크에 연결되어 서로 통신할 수 있음.

- 도커 네트워크를 명시하지 않고 컨테이너를 올린다면, 이 녀석은 도커의 default bridge network에 배치됨
```

- 그렇다면, Docker Compose의 networks 설정은 어떤 경우에 써먹을 수 있을까?

### Case 1

#### Example Docker Compose

```yml
version: '3'
services:
  web:
    image: nginx
    networks:
      - webnet
  database:
    image: postgres
    networks:
      - dbnet
  backend:
    image: node
    networks:
      - webnet
      - dbnet

networks:
  webnet:
    driver: bridge
  dbnet:
    driver: bridge
```

- 이런 경우, backend 서비스는 webnet, dbnet 두 네트워크 모두 속해있음

    - backend 서비스는 두 네트워크에 속해 있는 모든 컨테이너들과 소통할 수 있음

- 하지만 web컨테이너와 database컨테이너는 도커네트워크가 명시적으로 분리되어 있기에 서로 소통할 수 없음(네트워크 격리)

- 네트워크 격리를 통해서 보안을 강화할 수 있음

- 하지만, 네트워크 간 격리 예외도 존재

    1. 위처럼 두 개 이상의 네트워크에 어떤 컨테이너가 속해있다면, 해당 컨테이너는 각 네트워크 자원과 통신 가능 + 필요에 따라 브릿지 역할도 수행할 수 있음(조금 복잡하긴 하지만)

    2. 네트워크 라우팅 구성
        - Docker Network 라우팅 규칙을 조정해 격리된 네트워크 간의 통신 경로를 설정할 수 있음
        - 이 방법은 진짜 쉽지 않긴 함

- 각 네트워크는 완전히 독립적인 존재라고 생각하면 됨

    - 위 케이스의 web서비스와 database서비스는 서로 다른 네트워크에서 같은 IP를 할당 받고 있을 수도 있다는 것 (ex. 둘 다 10.0.0.2일 가능성도 있음)



