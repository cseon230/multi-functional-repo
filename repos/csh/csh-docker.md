# **컨테이너 생성 및 관리의 전체 흐름**

1. **`dockerd`**:
    - 사용자가 CLI 명령(`docker run`)을 통해 보낸 명령어를 처리합니다.
    - Docker 데몬(`dockerd`)은 이를 `containerd`에 전달합니다.
2. **`containerd`**:
    - 고수준 컨테이너 런타임으로, 컨테이너 생명 주기를 관리합니다.
    - gRPC 프로토콜을 통해 `shim` 프로세스를 생성하고, 해당 `shim`을 통해 컨테이너를 실행합니다.
3. **`shim`**:
    - 컨테이너 별로 하나씩 생성되는 경량 프로세스입니다.
    - `runc`를 호출해 실제 컨테이너를 생성합니다.
    - 컨테이너가 실행된 후에는 `shim`이 컨테이너의 **프로세스 및 I/O**를 지속적으로 관리합니다.
4. **`runc`**:
    - `shim`이 호출하는 저수준 컨테이너 런타임입니다.
    - Linux의 Cgroups와 Namespaces를 사용해 실제로 컨테이너 프로세스를 생성하고 시작합니다.
    - 컨테이너 생성이 완료되면 `runc`는 역할을 마치고 종료됩니다.
5. **컨테이너 실행**:
    - 컨테이너는 독립적으로 실행되고, `shim`이 컨테이너의 생명 주기를 전담 관리합니다.

---

### **요약하면:**

- `dockerd`: 명령 해석 및 `containerd`에 전달.
- `containerd`: gRPC를 통해 `shim` 프로세스 생성.
- `shim`: `runc` 호출 → 컨테이너 생성 → 컨테이너 실행 이후 프로세스와 I/O 관리.
- `runc`: 컨테이너를 생성(실행)한 뒤 종료.

---

### **중요한 포인트**

1. **`runc`는 컨테이너 생성 역할만 수행**:
    - 컨테이너를 실행하기 위해 Linux 커널의 Namespaces와 Cgroups를 설정하고 프로세스를 시작합니다.
    - 이후에는 종료되며 컨테이너 관리는 `shim`이 담당합니다.
2. **`shim`은 컨테이너 별로 독립적으로 동작**:
    - 각 컨테이너는 별도의 `shim` 프로세스를 가집니다.
    - 컨테이너의 종료, I/O 스트림 관리 등을 처리합니다.
    - `containerd`와 독립적으로 실행되므로, `containerd`가 재시작되어도 컨테이너는 계속 실행됩니다.

---

### **추가 설명**

### gRPC 통신

- `containerd`와 `shim` 간 통신은 gRPC를 사용합니다.
- gRPC는 빠르고 효율적인 원격 프로시저 호출(RPC) 프로토콜로, `containerd`와 `shim`이 명령 및 상태를 주고받는 데 사용됩니다.

### Docker와 Kubernetes의 활용

- Docker 환경에서는 이 구조가 `dockerd`를 통해 사용됩니다.
- Kubernetes 환경에서는 `containerd`가 CRI(Container Runtime Interface)를 통해 직접 호출됩니다(이때 `dockerd`는 사용되지 않음).

---

### 전체 흐름 그림으로 표현하면:
```
User → dockerd → containerd → shim → runc → Container
```

# 1. Cloud Native Technologies

## 소프트웨어 아키텍처

- SW를 구성하는 요소와 요소 간의 관계 정의
- 전체 구성 관계, 포함 관계, 호출 관계
- 전반적인 프로덕트를 만드는 과정을 지휘
    - 이해관계자들이 시스템을 이해하는 수준은 모두 다르다.
- Antifragile
    - Auto Scaling
    - Microservice
    - Chaos engineering
    - Continuous deployments

### Cloud Native Architecture

- 확장 가능한 아키텍쳐
- 탄력적 아키텍쳐
- 장애 격리
- MSA Application
- DevOps and CI/CD
- const Efficiency
- innovative Technologies


# 2. Docker Essentials - Container

## 컨테이너 가상화 기술과 Docker

### 가상화 기술의 이해

서버 가상화 방식

: 운영체제 위에 Hypervisor라는 프로그램을 띄우고 그 하이퍼바이저가 우리가 필요로 하는 VM을 실행시켜준다. 즉 호스트PC위에 게스트OS를 가동하는데, 그 가동을 하이퍼바이저가 해준다.

이미지 : 운영체제 자체를 하나의 압축파일처럼 묶어서 사용하는 단위.

컨테이너 가상화

: 가상화를 실행시켜주는 단위가 컨테이너라는 단위로 분리. 컨테이너가 운영체제일 수도 있고 데이터베이스 같은 미들웨어일 수도 있고 파이썬이나 자바같은 웹 서버 같은 형태로 제공될 수도 있다.

컨테이너 런타임은 운영체제가 따로 없다. 따라서 적은 리소스를 가지고 빠른 VM 형태를 운영할 수 있다는 장점이 있다.

도커 그룹끼리 묶어서 네트워크를 함께 쓸 수 있고 두 개의 네트워크 그룹을 격리시킬 수도 있고 두 개의 네트워크를 묶는 세번째 네트워크도 만들 수 있다.

도커 daemon = 도커 서버

도커 컨테이너 = 사용 가능한 인스턴스. 실체화되어 있는 상태.

도커 볼륨 = 데이터 저장소

도커는 리눅스 태생이고, 리눅스 베이스로 실행된다.

### Docker Image 구조

docker image는 변경 불가능한 형태의 단일 파일이다. 이미지를 컨테이너 라는 이름으로 실체화해서 사용해야한다. Docker Hub 사이트에 등록되어 있는걸 사용하거나 독립적인 Docker Registry 저장소를 직접 만들어서 관리한다.

이미지는 다양한 계층구조로 나뉘어 구성되어있다. 이 계층구조를 Layer방식이라고 한다. 여러 개의 Layer를 하나의 파일 시스템으로 사용 가능하다.

도커 이미지를 만들기 위해 확장자가 없는 상태에서 Dockerfile 이라는 파일을 만들어준다. 이름을 바꿔도 상관없지만 기본은 Dockerfile임

Docker image를 생성하기 위한 스크립트 파일

예시

```docker
FROM node:alpine as builder
WORKDIR /app
COPY ./package.json .
RUN npm install
COPY ..
RUN npm run build

FROM nginx
COPY --from=builder /app/build /usr/share/nginx/html/
```

dockerfile 로 이미지를 만들고 그 이미지로 컨테이너화 시킨다

### Docker Container 명령어 - 1

도커 컨테이너의 이해

- 도커 이미지를 실행한 상태를 “컨테이너” 라고 부른다. (메모리에 올라간 인스턴스를 뜻함)
- 격리 된 시스템 자원 및 네트워크를 사용할 수 있는 독립적인 실행 단위
- 읽기 전용 상태인 이미지에 변경 된 사항을 저장할 수 있는 컨테이너 계층에 저장

이미지가 준비되어 있는 상태에서 `docker run`을 입력하면 컨테이너화 된다. 이 컨테이너는 고유의 ID가 부여된다. 컨테이너 이름을 지정할 수도 있다.

컨테이너의 라이프 사이클

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/6b922040-a4b7-4630-98cc-0c4bd79a53ca/969719ea-f910-475f-bc10-50777ac2ceae/image.png)

mysql 컨테이너 생성.

```java
docker run -d -p 3306:3306 -e MYSQL_ALLOW_EMPTY_PASSWORD=true --name mysql mysql:5.7
```

- -p 3306:3306 의미 : Docker컨테이너는 외부에서 접근할 수 없는 격리된 네트워크 환경에서 실행된다. -p 옵션을 사용하면 컨테이너 내부의 포트를 호스트 시스템(사용자 컴퓨터)과 연결하여 외부에서 접근 가능하게 만든다. `-p <호스트 포트>:<컨테이너 포트>`
- 3306 표준 포트는 데이터베이스에서 mysql과 mariadb가 기본적으로 사용하는 표준 포트이다

도커는 이미지로 컨테이너를 실행시켰을 때 프로세스가 돌아가는데 없으면 종료된다. 프로세스를 계속 유지하기위해 위해 `-it` 라는 옵션을 작성한다. interactive tty 라는 건데, 입력을 계속 받도록 세션을 열여두는 것이다.

컨테이너의 Name을 지정해주지 않으면 무작위로 Name이 생성된다.

```java
docker run -d -e MARIADB_ALLOW_EMPTY_ROOT_PASSWORD=true —name my-mariadb mariadb
```

- —name my-mariadb : 컨테이너 이름을 지정하는 옵션
- mariadb : 이 컨테이너가 사용할 이미지를 지정

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/6b922040-a4b7-4630-98cc-0c4bd79a53ca/4f2b7cef-ecd5-4c93-b7de-59cae87a927d/image.png)

- docker container ls : docker ps 와 동일. 컨테이너의 목록을 나타낸다
- docker container stop : 컨테이너 중지
- docker container rm : 컨테이너 삭제
- docker container logs : 특정 컨테이너의 로그를 확인. -f 는 로그를 실시간으로 출력
- docker container exec : 실행 중인 컨테이너 내부에서 명령어 실행
- docker container inspect : 특정 컨테이너에 대한 자세한 정보 출력
- docker image ls : 다운로드된 모든 이미지 조회
- docker image rm : 특정 이미지 삭제
- docker image pull : Docker hub 또는 레지스트리에서 이미지 다운로드
- docker system prune : 사용하지 않는 Docker 리소스 정리. 삭제 대상은 중지된 컨테이너, 사용하지 않는 네트워크, Dangling이미지(태그 없는 이미지), 사용하지 않는 볼륨

### Port Mapping

도커 컨테이너에서 사용하고자 하는 Port를 자유롭게 설정 가능함

호스트 시스템에서 도커 컨테이너 Port를 사용하기 위해서는 Port Mapping이 필요하다.

```docker
$ docker run -p 80:8080 <IMAGE NAME>
```

-p : Port Mapping 의 줄임

호스트PC의 포트를 80으로 설정하고 컨테이너포트를 8080으로 설정함.

```docker
$ docker run -p 80:80 nginx
```

nginx는 기본적으로 80 이라는 포트를 가지고 서비스되고 있다. 주소창에 포트번호를 입력하지 않으면 기본적으로 80이라는 포트값이 설정된다.

**HTTP 프로토콜 포트 : 80**

**HTTPS 프로토콜 포트 : 443**

```docker
docker run nginx
```

위 명령어를 입력하면 nginx 이미지를 다운로드 받고 컨테이너를 실행한다. 기본적으로 nginx 는 80번 포트에서 HTTP 서버를 실행한다.

```docker
docker exec -it [containerID] bash
```

위 명령어를 입력하여 컨테이너 내부로 접속한다. 컨테이너 내부는 리눅스 환경과 유사하며, nginx 웹 서버가 이미 실행 중이다.

```docker
curl http://127.0.0.1
```

curl 명령어는 특정 URL로 HTTP 요청을 보내고 그 응답 내용을 출력한다. http://127.0.0.1 은 **컨테이너 내부에서 자신**을 의미하고, 현재 실행 중인 nginx 서버에 요청을 보낸 것이다.

컨테이너 내부는 기본적으로 외부에서 접근이 불가능하다. 외부에서 nginx서버에 접속하려면, 컨테이너의 포트를 호스트 머신에 매핑해야 한다.

```docker
docker run -d -p 8080:80 nginx
```

위 명령어는 호스트 8080 포트를 컨테이너의 80 포트에 연결한다. 따라서, 브라우저에서 http://localhost:8080으로 접속하면 nginx 기본 페이지를 볼 수 있다.

localhost는 기본적으로 127.0.0.1 로 매핑된다. 두 주소는 같은 의미를 가진다.

**127.0.0.1 주소는 루프백 IP 주소다. 컴퓨터 자신을 가리킨다.**

네트워크 요청을 컴퓨터 외부로 보내지 않고 자신에게 돌려보내는 역할을 한다.

외부에서 컨테이너 DB로 접속

```docker
$ docker run -d -p 3307:3306 -e MARIADB_ALLOW_EMPTY_ROOT_PASSWORD=true --name my-mariadb mariadb
```

사용자가 3307 포트로 요청을 보내면 docker daemon이 그것을 3306으로 포워딩해준다. 컨테이너의 응답도 3306 포트에서 3307 포트로 반환된다.

### 표준 포트

- HTTP : 80
- HTTPS : 403
- MySQL / MariaDB : 3306

### 비표준 포트

표준 포트를 사용하지 않고, 개발자나 시스템이 필요에 따라 지정한 포트 번호. 개발 중 다른 서비스와 충돌을 피하기 위해 주로 사용

- 리액트 개발 서버 : 3000 (리액트 개발 도구에서 기본적으로 사용하는 포트)

### 요청 흐름 : 네트워크 요청 → 호스트 → 컨테이너

```docker
매핑이-p 3000:80 으로 되어있다고 가정
```

1. 네트워크 요청
    - 클라이언트(브라우저or네트워크APP)에서 네트워크 요청이 들어온다
    - ex ) `http://localhost:3000`
2. 호스트 머신의 3000번 포트
    - 도커는 -p 3000:80 으로 설정된 매핑 정보를 참조하여 호스트 머신의 3000번 포트로 들어온 요청을 처리함
3. 호스트 머신과 컨테이너 간 네트워크
    - 도커는 호스트 머신과 컨테이너 간의 가상 네트워크를 통해 요청을 컨테이너 내부로 전달
    - 요청은 컨테이너 내부의 80번 포트로 포워딩 됨
4. 컨테이너 내부의 80번 포트
    - 컨테이너 내부에서 80번 포트를 수신 대기하고 있는 App(NGINX, Apache 등)이 요청을 처리.
5. 응답 전달
    - 애플리케이션이 생성한 응답은 컨테이너 내부의 80번 포트에서 다시 호스트의 3000번 포트로 전달되고, 최종적으로 클라이언트에 반환된다.



# 3. Docker Essentials - Image

운영 체제를 가지고 있는 단일 파일 = 이미지

이미지는 컨테이너를 만드는 데에 필요한 ‘읽기 전용’ 상태의 템플릿

컨테이너 실행에 필요한 파일과 설정 값 등을 포함하고 있지만 상태 값을 가지고 있지 않기 때문에 컨테이너화 해서 사용해야 한다.

도커 이미지가 **제공**되는 곳을 **Registry** 라고 부른다.

이미지가 **저장되는 저장소**를 **Repository** 라고 부른다.

이미지를 가지고 와서 컨테이너 작업을 하기 위한 **Docker daemon**

사용자는 **Docker CLI에서 명령어를 작성**하여 **Docker daemon 에게 명령**한다.

### Docker Image 작성

Dockerfile : 이미지 빌드용 DSL (Domain Specific Language) → 특정 도메인에 특화된 언어

```docker
$ touch Dockerfile
$ vi Dockerfile
```

- touch : 비어있는 파일을 만드는 명령어
- vi : 텍스트 편집 에디터 사용

```docker
FROM ubuntu:16.04
```

```docker
$ docker build --tag [도커이미지명:태그명(버전,특징..)] -f Dockerfile .
```

도커 이미지를 빌드하는 명령어. 마지막에 들어가는 . 는 현재 디렉토리에 도커 파일이 있으니 여기서 찾아라 라는 의미.

```docker
$ docker image ls
```

```docker
$ docker run -it first-image:0.1 bash
```

이미지로 도커 컨테이너 실행

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/6b922040-a4b7-4630-98cc-0c4bd79a53ca/38c10305-cc32-40b1-93c9-f876da0a7cb4/image.png)

curl 명령어는 터미널 상태에서 http에 있는 특정 uri를 호출해서 결과를 볼 수 있는 명령어

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/6b922040-a4b7-4630-98cc-0c4bd79a53ca/bf547c0b-4298-48d7-9b78-a9231ef7c75e/image.png)

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/6b922040-a4b7-4630-98cc-0c4bd79a53ca/4276ffb7-c264-452a-9db2-d3a98ad4ef2e/image.png)

```
$ docker run -d -p 5000:5000 --restart always --name registry registry:2
```

```
$ docker pull ubuntu
$ docker tag ubuntu localhost:5000/ubuntu
$ docker push localhost:5000/ubuntu
```

http://localhost:5000/v2/_catalog 로 접속하면 ubuntu를 확인 가능함

도커 이미지를 docker hub에 업로드 하기 전에 도커허브에서 레포지토리를 만들어주고

이미지 태그명을 [도커허브아이디]/[이미지명]으로 작성해줘야한다

```
docker tag nodejs-demo:latest cseon230/nodejs-demo:latest
```

위와 같이 작성하면 cseon230/nodejs-demo 라는 이름을 가진 도커 이미지가 새로 생성된다

```
docker push cseon230/nodejs-demo:latest
```

이렇게 작성하면 docker hub에 해당 이미지가 업로드 되고,
이미지를 다운받으려면, 아래와 같이 docker hub 아이디와 해당 이미지 명, 태그명까지 포함하여 pull 을 작성한다

```
docker pull cseon230/nodejs-demo:latest
```


# 4. Docker Network and Storage

## Docker Network 구조 (1) - docker0와 container network 구조

## 1. bridge

도커 네트워크 중 bridge 네트워크는 가상 네트워크 인터페이스인 docker0 를 구현하여 네트워크 통신을 한다.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/6b922040-a4b7-4630-98cc-0c4bd79a53ca/1085e1cb-0a67-4e2b-8fe5-db2044db4f4f/image.png)

위 사진은 Docker의 network 구조를 간단히 도식화 한 것이다.

### Docker0 interface

docker를 설치 한 후 host의 network interface를 살펴보면, **docker0** 라는 virtual interface가 있는 것을 볼 수 있다. 아래는 docker를 설치한 host의 interface를 확인한 정보이다. 아래와 같이 docker0라는 interface를 확인할 수 있다.

### Docker0 interface의 특징

- IP는 자동으로 `172.17.0.1` 로 설정되며 16 bit netmask(255.255.0.0)로 설정됨
- 이 IP는 DHCP를 통해 할당 받는 것은 아니며, docker 내부 로직에 의해 자동 할당 받는 것임
(Docker 브리지는 기본적으로 `172.17.0.0/16` 범위에서 IP를 자동으로 할당함)
- **docker0는 일반적인 interface가 아니며, virtual ethernet bridge이다.**

> DHCP란?
**Dynamic Host Configuration Protocol** 의 약자로, 네트워크 상에서 IP주소와 기타 네트워크 설정(서브넷 마스크, 게이트웨이, DNS서버)을 자동으로 할당해주는 프로토콜

1. 컴퓨터 또는 디바이스가 네트워크에 연결되면 DHCP 서버가 그 디바이스에 사용할 수 있는 IP 주소를 임시로 할당함
2. 이렇게 할당된 IP 주소는 일정 시간 동안 유효하며, 만료되면 갱신하거나 새 IP를 요청해야 한다

DHCP의 장점
1. 사용자가 네트워크 설정(IP 주소 등)을 수동으로 입력할 필요가 없어 간편함
2. 네트워크 관리자가 IP 주소를 효율적으로 관리할 수 있음
>

Docker 네트워크 상태 확인

```
$ docker network ls
```

네트워크 상세 정보 보기

```
$ docker network inspect [network_name]
```

Docker 컨테이너가 생성되면 컨테이너 내부 네트워크와 docker0 브리지 네트워크를 연결하기 위해 veth(Virtual Ethernet) 페어가 생성된다.

### veth 페어란?

- veth 페어는 두 개의 가상 네트워크 인터페이스로 구성된 일종의 “가상 네트워크 케이블”이다.
- 하나의 인터페이스는 컨테이너에 연결되고 다른 하나는 호스트(즉, docker0 브리지)에 연결된다.
- 이 두 인터페이스는 서로 연결되어 있어, 한쪽에서 보낸 데이터를 다른 쪽에서 받을 수 있다.

### Docker 컨테이너가 네트워크를 연결하는 과정

1. 컨테이너 생성 → Docker는 컨테이너가 사용할 네트워크 네임스페이스를 생성함
2. veth 페어 생성 → Docker는 하나의 veth 페어를 생성. (veth의 한쪽을 veth123, 다른 한쪽은 eth0라고 가정)
3. veth 연결 → veth123은 호스트 네트워크의 docker0 브리지에 연결, eth0는 컨테이너의 네트워크 네임스페이스로 이동.
4. IP 주소 할당 → Docker는 컨테이너의 eth0 인터페이스에 IP 주소를 할당.
이 IP 주소는 docker0 브리지의 서브넷(172.17.0.0/16) 범위 내에서 할당
5. 라우팅 설정 → 컨테이너 내부에서 eth0은 docker0 브리지를 게이트웨이로 사용하도록 라우팅이 설정됨.

컨테이너 1: eth0 <-> veth12345 <-> docker0
컨테이너 2: eth0 <-> veth67890 <-> docker0

여기서, 컨테이너가 생성될때마다 veth 페어가 생성된다고 했는데 어째서 eth0가 공통으로 사용되는것처럼 보일까? eth0는 각 컨테이너 내부에서만 존재하며 독립적이기 때문에 모든 컨테이너 동일한 이름을 사용할 수 있는 것이다.

네임스페이스와 veth를 활용한 격리 덕에 Docker 컨테이너는 독립적인 네트워크 환경을 유지할 수 있다.

---

## 2. host

host 네트워크는 Docker 컨테이너가 호스트 네트워크 네임스페이스를 직접 사용하는 방식.

host 방식으로 컨테이너를 생성하면, 컨테이너가 독립적인 네트워크 영역을 갖지 않고 host와 네트워크를 함께 사용하게 된다. 즉, 컨테이너 내부에서 설정된 IP주소가 없으며, 호스트의 IP 주소를 공유한다.

컨테이너 생성 시 —net=host 옵션을 이용하면 된다.

```
$ docker run --network host [image]
```

Host 네트워크에서는 포트 매핑이 필요 없다. 컨테이너에서 사용하는 포트는 바로 호스트에서 사용된다.

**성능 이점**

- 네트워크 브리징, NAT 처리 등의 가상화 계층이 제거되므로 네트워크 성능이 더 우수함
- 대용량 트래픽을 처리하거나 네트워크 성능이 중요한 어플리케이션에서 적합하다

**격리성 감소**

- 컨테이너와 호스트의 네트워크가 동일하므로, 컨테이너 내부에서 네트워크 설정을 변경하면 호스트 네트워크에도 영향을 미칠 수 있다
- 보안상 더 주의가 필요하다

host 네트워크를 사용하려면 컨테이너를 생성(run)할 때부터 네트워크를 설정해야 한다.

이미 생성된 컨테이너에 대해서는 네트워크 모드를 변경할 수 없기 때문에, 새로운 컨테이너를 생성해야 한다.

```
$ docker run -d -it --network host ubuntu
```

```
$ docker network inspect host
```

```
[
    {
        "Name": "host",
        "Id": "6d65e5ba10d615a8f02fbe54fc159480815bbcd4b1821b080200e9be58d447d0",
        "Created": "2022-08-19T05:22:45.255646496Z",
        "Scope": "local",
        **"Driver": "host",**
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": null
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        **"Containers": {
            "87da86d1b853173aa72f4510fcba84c04166f70d68c039dab8177eccf9776d81": {
                "Name": "amazing_pike",
                "EndpointID": "d816baad6c4cd2e5eeaf6f4478871a0aba92ea9e8808636d9b309f8069c70099",
                "MacAddress": "",
                "IPv4Address": "",
                "IPv6Address": ""
            }
        },**
        "Options": {},
        "Labels": {}
    }
]

```

### **`eth0`가 Docker에서 어떤 역할을 하나요?**

Docker 컨테이너 내부에서도 `eth0`는 네트워크 인터페이스로 사용됩니다. 컨테이너가 네트워크에 연결되면, Docker는 내부적으로 `eth0`라는 네트워크 인터페이스를 생성합니다.

### **Bridge 네트워크에서:**

- 컨테이너의 `eth0`는 호스트의 `docker0` 브리지에 연결됩니다.
- 데이터 흐름:

    ```
    컨테이너 eth0 <-> veth <-> docker0 <-> 외부 네트워크
    ```


### **Host 네트워크에서:**

- 컨테이너의 `eth0`는 호스트의 네트워크 인터페이스를 그대로 사용합니다.
- 데이터 흐름:

    ```
    컨테이너 eth0 = 호스트 eth0
    ```


**컨테이너의 `eth0`와 호스트의 `eth0`가 동일한 네트워크 스택을 공유합니다.**

- 컨테이너의 `eth0`는 **독립적인 인터페이스가 아니며**, 실제로는 호스트의 `eth0`와 완전히 동일한 네트워크 인터페이스를 사용합니다.
- 즉, 컨테이너 내부에서 `eth0`를 통해 네트워크 통신을 하면, 이는 곧바로 호스트의 `eth0`를 통해 네트워크로 나가게 됩니다.


### host network 사용 시 발생할 수 있는 문제점

(참고 : 사용할 수 있는 포트의 개수는 1번부터 65535까지로 정해져있음)

host network 사용 시 호스트PC와 컨테이너가 같은 포트를 사용한다면 충돌이 날 수 있다.

예를 들어, 호스트에서 mysql 을 사용중이면 기본적으로 3306포트를 사용하고 컨테이너 자체에 mysql을 설치하여 연결한다면 이것 또한 3306포트를 사용하므로 포트 충돌이 일어난다

이를 해결하기 위해서는 컨테이너에서 mysql 실행 시 포트를 3307로 변경하거나,

```docker
$ docker run --network host -e MYSQL_ROOT_PASSWORD=root -d mysql --port=3307
```

또는 컨테이너 혹은 호스트의 mysql 설정 파일에서 포트를 변경한다. (mysql 설정파일 my.cnf 을 수정하여 port 값 변경

```docker
[mysqld]
port=3307
```

---

## 3. none Network

none 네트워크 모드는 도커에서 제공하는 가장 단순한 네트워크 모드다. 이 모드는 컨테이너를 완전히 네트워크에서 격리하는데 사용된다. 컨테이너는 자체적으로 네트워크 인터페이스를 가지지만, **외부 네트워크와 통신하지 않고 다른 컨테이너와도 연결되지 않는다.**

### none network의 주요 특징

1. 네트워크 비활성화
    - 컨테이너는 기본 네트워크 설정(bridge, host 등)을 사용하지 않는다
    - 컨테이너 내부에 네트워크 인터페이스는 있지만, 외부 네트워크와 연결되지 않는다
2. 격리된 환경
    - 네트워크 격리가 필요할 때 유용하다
    - 컨테이너는 외부와 통신하지 않으며, 컨테이너 간 네트워크 연결도 차단된다.
3. 네트워크 구성
    - 컨테이너는 **loopback 인터페이스(`io`)만 활성화**되어 있다.
    - `eth0` 이나 다른 네트워크 인터페이스는 생성되지 않는다

# loopback 인터페이스란?

## 루프백 주소(loopback)란

A네트워크(0.0.0.0~127.255.255.255)에서 마지막 네트워크인 이 127.0.0.0 네트워크와 그 안에 속하는 주소들**(127.0.0.1~127.255.255.255)**은 **루프백이라는 가상의 인터페이스(데이터 전송 통로)에 사용하기 위해 예약해 놓은 주소이다.**

loopback 인터페이스는 그 의미대로 **내 컴퓨터에서 나간 신호가 다시 내 컴퓨터로 돌아오기** 때문에 붙여진 이름이다.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/6b922040-a4b7-4630-98cc-0c4bd79a53ca/1cc0d527-a05d-434e-b26e-7e257d2ffd98/image.png)

데이터가 흐르는 통로(인터페이스)에 데이터를 보내면 당연히 어느 목적지에 도달을 한다. 위 사진에서 **컴퓨터가 라우터로 이어진 이더넷 인터페이스(보통 자기 IP주소)로 데이터를 보내면 게이트웨이인 라우터에 도달한다.**

하지만 **루프백 인터페이스로 데이터를 보낼 시 위 사진처럼 자기 컴퓨터로 돌아온다.**

### 루프백 주소를 사용하는 이유

특정 IP 주소를 적으면 그 IP주소로 메세지를 보내는 프로그램이 있다고 가정할 때, 다른 컴퓨터 IP 주소를 적어 테스트하기 전, 먼저 내 컴퓨터를 대상으로 실행하여 프로그램이 잘 작동하나 테스트하기 위해 loopback 주소를 사용한다.

하지만 *‘그냥 자기 IP 주소를 적으면 되는 것 아닌가?’* 라는 의문이 들 수 있다.

당연히 되지만, 자기 컴퓨터의 IP를 일일이 다 외우고 다니지 않을 뿐더러. IP 주소를 알아내는 것 조차 귀찮기 때문에, **루프백 인터페이스 주소로 많이 쓰이는** `127.0.0.1` 이란 숫자를 외우면 내 컴퓨터에 어떤 IP 주소가 할당되어 있나 확인할 필요가 없다

### 127.0.0.1과 127.0.0.0 네트워크

루프백의 이름은 주소 `lo[숫자]`형태이며, 보통 `127.0.0.1~127.255.255.254` 중 첫 번째 IP 주소인 127.0.0.1을 제일 많이 쓴다.

그리고 **127.0.0.1이 할당된 인터페이스는 127.0.0.0 네트워크의 첫 번째 주소니까 주로 `lo0` (loopback0) 이란 이름이 붙여진다.**

### 정리

**루프백(loopback)주소 == 갔다가 나에게 돌아오는 주소 == 내 컴퓨터 IP주소**

**127.0.0.0 네트워크(127.0.0.1~127.255.255.254) == 127.0.0.1 == 내 컴퓨터 IP 주소**

namespace 과 cgroup 의 차이 ->
namespace는 해당 프로세스가 볼 수 있는 범위를 제한한다
cgroup은 해당 프로세스가 쓸 수 있는 사용량을 제한한다


# Docker Volume Storage

도커는 개별적인 가상화 환경이기 때문에 모든 데이터는 컨테이너 내부에 존재한다. 그 말은, 컨테이너가 삭제되면 작업했던 모든 데이터도 삭제된다는 뜻이다.

**컨테이너 내부의 데이터를 외부로 연결시켜 주는 기능이다**

Docker volume은 기본적으로 `/var/lib/docker/volumes/ (Docker Desktop) 디렉토리에 저장

볼륨 리스트 보기

```
$ docker volume ls
```

볼륨 생성

```
$ docker volume create my-volume
```

볼륨 정보 

```
$ docker volume inspect my-volumeㅜ 
```

컨테이너 생성 시 volume 연결

```
$ docker run -v [생성볼륨명]:[컨테이너 내부 디렉토리 경로] --name [컨테이너명] [이미지명]
```

컨테이너 생성 시 생성한 볼륨과 연결

```
$ docker run --rm -it --network my-network -v ./docker_volume_test:/app/test ubuntu:16.04
```

`-v` : volume 생성을 의미

`./docker_volume_test:/app/test` : 현재 디렉토리에 docker_volume_test 폴더를 만들고, 생성할 컨테이너 내부에 /app/test 를 만들어서 볼륨을 연결한다.

볼륨을 연결해놓고 컨테이너를 삭제한 다음, 컨테이너를 다시 생성할 때 기존의 볼륨을 연결해도 데이터를 다시 불러올 수 있다.