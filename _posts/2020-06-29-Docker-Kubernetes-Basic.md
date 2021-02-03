---
layout: article
title: Kubernetes 컨테이너 클러스터 기초
tags: [devops, docker, kubernetes]
aside:
    toc: true
---

> 💡 본 글은 한국클라우드컴퓨팅연구조합 'DevOps를 위한 Kubernetes 컨테이너 클러스터 기본 과정' 강의 수강 내용을 바탕으로 정리하였습니다. <br/>
>개인적인 참고 용도로 쓴 글이라 설명이 다소 부족할 수 있습니다.

## 가상화
![가상머신과_도커](/assets/images/etc/vm_docker.png) <br/>
### 서버 가상화(가상 머신)
시스템을 효율적이고 안정적으로 쓰기 위한 기술. 서버를 가상 머신으로 만들어서 사용. <br/>
#### 특징
+ 리소스 효율성 증대 (하드웨어 낭비를 줄임)
+ 안정성 (각 application 분리, 서로 영향 안 받도록)
+ 보안성

#### 한계점
가상 머신 위에 있는 application이 하드웨어를 요청하면, 가상 머신 Guest OS가 직접 하드웨어에 접근하려고 함. 그러다 보니 Host OS와 충돌 남. CPU, 메모리 같은 리소스를 나누어 줄 기술이 필요. <br/>
→ `Hypervisor`는 이 문제를 해결하는 역할. 가상 머신이 직접 하드웨어에 접근하지 못하도록 가로채고 대신 요청. 가상 머신 리소스 관리와 리소스 간 접근 방지 기능 수행. <br/>
→ 이러다 보니 새로운 문제 발생. `Hypervisor`가 가로채는 과정이 들어가서 오버헤드 발생.

### 컨테이너 가상화
앞서 언급한 서버 가상화의 오버헤드 문제를 해결하기 위해 나온 기술이 컨테이너 가상화.
#### 특징
+ 컨테이너에는 가상화 계층, 즉 추가 Guest OS가 없음. Host OS 위에서 각각 application이 독립되어 돌아감.
+ `Hypervisor` 대신 `Container engine`이 있지만 생명 주기 관리, 리소스 제어 정도의 기능만 수행 <br/>
→ 오버헤드 감소, 낭비가 없이 효율적인 배포 및 관리.
+ 이미 리눅스에 있던 `cgroup`과 `namespace` 기술 사용
    - `cgroup`: control group. 프로세스 또는 쓰레드를 그룹화 하여 관리 + 시스템 리소스 사용을 제한하는 기술.
    - `namespace`: 장치, 프로세스, 네트워크 등 각 항목의 오브젝트에 격리된 공간을 내어주는 기술. (ex. 컨테이너 내부에서는 각 오브젝트가 사용하는 PID가 다르도록)

> 💡 가상화 기술 중 하나가 `Docker`, `Docker`의 컨테이너를 손쉽게 관리하도록 해주는 Container Orchestration 중 하나가 `Kubernetes`!

## 도커
### 개요
리눅스 컨테이너를 더 쉽게 관리하기 위한 오픈 소스 플랫폼. 이미지 생성(ex. Dockerfile), 이미지 공유, 컨테이너 생성 등의 기능.

### 구조
+ `image`:  실행 파일. read only. 파일 자체 수정은 안 되고 이미지 위에 레이어를 쌓아서 수정된 상태를 저장.
+ `container`: 이미지를 실행한 형태. 하나의 어플리케이션을 실행하는 서버처럼 동작.
+ `registry`: 저장소. 이미지를 저장하고 공유하기 위한 공간. 공용 Docker hub도 있고 프라이빗 이미지 저장소도 있음.

## 도커 명령
[Docker Reference Document](https://docs.docker.com/reference/) <br/>
> 💡 `docker` 명령어는 관리자만 실행 가능. `sudo` 명령어를 사용하거나, `docker` 명령을 사용할 수 있도록 설정(`sudo usermod -aG docker $USER`)

### 공통
+ `docker inspect NAME` 오브젝트 정보 확인

### 이미지 명령어
+ `docker pull NAME:TAG` hub에서 다운로드. `TAG` 부분에 버전을 명시하는 습관을 갖기!
+ `docker images` (== `docker image ls`). 다운로드해 둔 이미지 전체 조회
+ `docker rmi` 이미지 지우기(컨테이너로 실행 중이면 에러. 컨테이너 먼저 삭제해야 함.)
+ `docker save -o SAVE_NAME IMG_NAME` 호스트에 저장된 이미지를 아카이브 파일로 복사
+ `docker load -i FILE_NAME` 아카이브 파일을 불러옴

> 💡 `inspect` 명령에서 봐야하는 부분: `cmd`와 `entrypoint`. 이 이미지를 실행하면 어떤 프로세스가 동작하는지 알 수 있음. 우선순위는 `entrypoint` > `cmd`.

### 컨테이너 명령어
#### 실행
+ `docker create` 컨테이너 만들기
+ `docker start` 컨테이너 실행
+ `docker run` create + start (이미지를 실행해서 컨테이너로 만들어주는 명령어. 나한테 이미지 파일이 없으면 docker hub에서 이미지를 찾아서 다운로드해서 실행)
    - `-it` shell 이미지. `-i` 는 interactive(표준 입력 유지), `-t`는 terminal
    - `-d` detached mode. 지금 당장 동작할 건 아니고 background에 실행. 이후에 `attach` 명령 가능.
    - `-e` 이미지에 따라 환경 변수를 추가로 입력
    - `--name` 이름 지정
    - `--cpus` CPU 할당. Core의 개수나 %로 제한 가능.
    - `--memory` Mem 할당
    - `-h` hostname 지정
    - `-l` link
    - `--rm` 필요한 작업하고 종료되면 알아서 삭제되는 옵션
+ `docker ps` 정상 동작 중인 컨테이너 목록.
    - `-a` 동작 중지된 목록 포함 전부 조회.

#### 관리
+ `docker attach` 쉘 프로그램을 실행하고 있는 컨테이너에 연결. (단, 쉘 프로그램을 실행하고 있지 않은 컨테이너는 `attach`하면 에러. `exec` 명령어를 사용!)
+ `docker exec` cmd에 정의되어 처음 실행하는 명령 외의 다른 명령 실행. 동작 중인 컨테이너에 attach하지 않고도 잠깐 그 컨테이너의 프로세스를 동작. (ex. `exec bash` 따로 쉘을 띄워서 동작)
+ `docker top NAME [ps option]` 해당 컨테이너 내에서 현재 동작 중인 프로세스. 필요 없는 프로세스는 일반 리눅스 명령어 `kill PID` 로 중단 가능.
+ `docker stats NAME` 컨테이너 상태 확인. 이름 안 쓰면 전체 조회.
+ `docker logs NAME` 에러 원인을 찾을 때
+ `docker cp SRC_PATH DEST_PATH` 호스트의 파일 → 컨테이너 또는 컨테이너 파일 → 호스트 복사.
+ `docker diff` 변경 상태 체크.
+ `docker container prune` 동작 안 하고 있는 컨테이너 한 번에 지우기

> 💡 `attach` 명령으로 컨테이너의 쉘을 실행한 상태에서 `exit` 명령을 실행하면 컨테이너가 종료됨. 컨테이너를 종료하지 않고 이전 쉘로 돌아가려면 `CTRL + P + Q`

## 도커 볼륨
### 개요
기본적으로 컨테이너의 데이터는 컨테이너의 `layer`에 저장. 따라서 컨테이너 삭제 시 전부 삭제됨. → 컨테이너의 데이터를 영구 보관하려면 `volume`을 사용해야 함!
+ 로컬의 볼륨을 컨테이너에 연결하면 컨테이너의 데이터를 로컬에 쌓게 되고 서로 공유. 파일을 매번 `cp` 명령으로 복사하지 않아도 그 볼륨에 저장해 두면 서로 공유. (때로는 이미지 자체에 이미 볼륨이 있기도 함.)
+ 방식은 `bind mount`와 `volume` 두 가지. `volume`을 쓰기를 추천! (∵ `volume`은 관리를 docker 명령어로 할 수 있음.)

### 명령어
+ `docker volume create NAME`
+ `docker volume ls` 목록 조회
> 💡 `docker run` 할 때 `-v`로 생성과 동시에 붙여줄 수 있음. `:ro`로 읽기 전용 설정 가능. 이미 실행한 컨테이너에 볼륨을 붙이기는 어렵고 처음 동작할 때부터 붙여야.
> `inspect` 명령어로 도커 볼륨이 연결되어 있는 호스트 디렉토리 확인 가능. 기본 경로는 `/var/lib/docker/volumes`

## 도커 네트워크
[Docker Network Reference Document](https://docs.docker.com/network/) <br/>

### 네트워크 드라이버 유형
#### bridge
해석하자니 애매해서 그대로 가져오자면 <br/>
> The default network driver. If you don’t specify a driver, this is the type of network you are creating. `Bridge` networks are usually used when your applications run in standalone containers that need to communicate.
> In terms of Docker, a `bridge` network uses a software bridge which allows containers connected to the same `bridge` network to communicate, while providing isolation from containers which are not connected to that `bridge` network.
+ 컨테이너 생성 시 기본으로 사용하는 네트워크. 가상의 인터페이스. 같은 `bridge`에 연결되어 있으면 컨테이너 IP로 통신 가능.
+ 내부 → 외부: NAPT 통신 사용
+ 외부 → 내부: 포트포워딩을 사용해야 함

+ 명령어
    - 네트워크 생성 `docker network create --subnet SUBNET --gateway GATEWAY NAME`
    - `docker run --network NAME`

#### host
호스트 인터페이스에 정의된 설정을 그대로 사용. 포트 번호도 못 바꿈. 기본이 같은 포트로 설정된 다른 컨테이너를 host하면 실행 안 됨.

#### null
네트워크 기능 사용 안 하는 서비스일 때

#### overlay
서로 다른 도커 호스트를 연결하여 통신할 수 있도록. 도커에서는 사용 안 하고 클러스터 환경에서 사용.

#### macvlan
물리적 인터페이스를 매개로 통신. 외부 → 내부로 direct하게 패킷을 전달할 수 있음.
> Some applications, especially legacy applications or applications which monitor network traffic, expect to be directly connected to the physical network.
> In this type of situation, you can use the `macvlan` network driver to assign a MAC address to each container’s virtual network interface, making it appear to be a physical network interface directly connected to the physical network.

1. 호스트의 인터페이스 중 하나의 네트워크 주소 알아내기(ex. `ip a show enp0s3` 또는 `enp0s8` 등)
2. 사용할 인터페이스에 무작위 모드 활성화 ( `ip link set NAME promisc on` )
3. `macvlan` 네트워크 생성 후 해당 드라이버를 사용하는 컨테이너 생성

### 컨테이너 간 통신
#### 링크
기본적으로 같은 `bridge` 드라이버 네트워크(IP 주소)를 사용해서 통신. 또는 이름이나 별칭으로 통신하기도. → 링크 기능 사용!
`docker run --link NAME`
> 💡 사실 `--link`는 내가 해당 컨테이너의 /etc/hosts 파일에 수동으로 다른 컨테이너의 주소를 등록하는 것과 별 다를 게 없음!

#### 포트 포워딩
`bridge`를 사용하는 모든 컨테이너는 외부에서 접근할 때 포트포워딩을 사용해야 함. `-p` 명령어 사용.
`docker run -p hostPort:containerPort` 만약 8080:80이면 8080포트로 접근했을 때 컨테이너의 80 포트로 전달.

## 도커 이미지 제작
### 제작 및 업로드
+ `docker tag IMG_NAME TARGET_NAME[:TAG]` 이미지 이름 자체를 변경하는 건 불가능, 태그 기능으로 이름 하나를 추가하는 형태. 이름은 "허브ID/저장소이름:태그" 형태가 좋음.
+ `docker login [SERVER]` 내가 이미지를 push할 저장소에 로그인. 뒤에 서버 안 써주면 기본은 docker hub. 로그인 후 `docker push`로 업로드.
+ `docker commit` 컨테이너를 이미지로 만드는 명령어. 잘 쓰이지는 않음. 이미 중지된 컨테이너의 정보를 확인하고 싶거나 컨테이너에 변화가 생겼을 때 이미지로 만들어서 정보를 확인하기도 함.
+ `docker export` 컨테이너를 아카이브 파일로 생성. `import`로 이미지 파일로 불러오기. 단, 불러온 이미지는 cmd 같은 기본 설정이 하나도 안 되어 있는 상태. 깨끗한 상태에서 새로 쓸 수 있음.

### Dockerfile
#### 개요
여러 가지 지시어를 사용하여 이미지 제작. 이 안에 CMD, ENTRYPOINT 등을 코드 형태의 텍스트 문서로 작성.
파일 이름은 "Dockerfile"로 하기를 권장. 아니면 빌드할 때 경로를 따로 설정 해야 함.

#### 지시어
[Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) <br/>
+ `FROM` 베이스 이미지.
+ `RUN` 이미지 "생성"할 때 베이스 이미지가 어떤 동작을 하면서 생성하게 할 건지.
+ `CMD` 실제 컨테이너로 "동작"할 때. 우선순위가 낮은 편. `ENTRYPOINT`에 있으면 그게 먼저 실행, 혹은 run 할 때 지정한 프로세스가 있으면 그거 먼저.
+ `ENTRYPOINT` 실제 컨테이너로 "동작"할 때. 우선순위 높음.

RUN, CMD, ENTRYPOINT 작성하는 방법 두 가지
+ `exec` 바로 실행. 좀 더 안정적. docker 파일 작성할 때는 이 방식을 사용하기를 권장.
+ `shell` shell을 먼저 실행하고 그 shell에서 실행. 인자 값을 전달 받는 데 한계가 있을 수 있음.

#### Dockerfile 이미지 제작
+ `docker build` 명령어
    - `-t` 이미지의 이름을 지정. 반드시 지정하기를 권장. (∵이름을 지정하지 않으면 이미지를 사용할 때마다 ID를 써야 해서 번거로움.)

## 프라이빗 이미지 저장소
### 개요
로그인할 때 server를 지정하지 않으면 이미지를 `pull`, `push` 할 때 공용 docker hub를 사용. 공용 hub를 쓰고 싶지 않으면 따로 프라이빗 저장소를 구성해야 함.
프라이빗 저장소 유형은 여러가지가 있지만 수업에서는 Harbor 사용.

### Harbor 저장소 구축
1. Docker Comose 설치 후 파일 실행 권한 부여
``` shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```
2. 최신 버전 다운 ([harbor Github](https://github.com/goharbor/harbor/releases)에서 다운로드 링크 알아내서), 압축 해제
``` shell
wget https://github.com/goharbor/harbor/releases/download/v2.0.1-rc1/harbor-offline-installer-v2.0.1-rc1.tgz
tar zxf harbor-offline-installer-v2.0.1-rc1.tgz
```
3. harbor.yml 파일, daemon.json 설정
``` shell
vi harbor.yml
vi /etc/docker/daemon.json
```
> 우리 저장소에 안전하게 https로 접근하게 하려면 관련 문서 참조. 중요한 점은 yml 문서 상에 작성한 certificate, private_key 경로에 crt, key 파일을 복사(`cp`)해주기.   https://goharbor.io/docs/2.0.0/install-config/configure-https/

4. `install.sh` 실행하고 로그인. `push`, `pull`로 이미지 업로드, 다운로드.

## Kubernetes
### 개요
[표준 용어집](https://kubernetes.io/ko/docs/reference/glossary/?all=true#term-control-plane) <br/>
쿠버네티스는 분산 시스템을 탄력적으로 실행하기 위한 프레임 워크를 제공. 서비스 디스커버리, 로드 밸런싱, 스토리지 오케스트레이션, 자동화된 롤아웃과 롤백 등 제공. <br/>
+ `pod`: 쿠버네티스에서 생성하고 관리할 수 있는 최소 단위. 하나 이상의 컨테이너를 담는 컨테이너 그룹. (대부분 하나의 컨테이너만 담음)

### 구성 요소
![쿠버네티스_구성요소](/assets/images/etc/k8e.png) <br/>
#### 마스터
클러스터의 컨트롤 플레인
+ `kube-apiserver`: 요청 수신, 내부 구성요소(리소스) 관리.
    - API 버전 규칙
        - 알파(Alpha) 수준: 버전 이름에 `alpha` 포함. 버그 위험성 있음. 기술 지원이 공지 없이 중단 가능. 단기간 테스트 클러스터에서만 사용하기를 권장.
        - 베타(Beta) 수준: 버전 이름에 `beta` 포함. 테스트 완료, 활성화 시켜도 안전. 기술 지원 중단 없음. 베타가 끝나면 많은 변경이 어려우므로, 베타 기능을 사용한 후 피드백 제공을 권장.
        - 안정화(stable) 수준: 버전 이름이 `vX`(X는 정수). 이후 여러 버전에 걸쳐 소프트웨어 릴리스에 포함.
+ `etcd`: 쿠버네티스 클러스터의 모든 정보 데이터를 저장하는 키-값 저장소.
+ `kube-controller-manager`: 클러스터 상태 감시. `Node controller`, `Replication`(파드 복제 관리), `Endpoint`(서비스와 파드 연결), `Service account`(인증) 등
+ `cloud-controller-manager`: 클라우드 공급자(AWS, GCP 등)의 기능과 클라우드에서 동작하는 쿠버네티스 구성 요소가 상호작용 할 수 있도록 관리.

#### 노드
컨테이너 런타임 환경 제공(컨테이너는 노드의 리소스를 사용)
+ `kubelet`
+ `kube-proxy`
+ `runtime`

### 쿠버네티스 오브젝트
#### 개요
+ 쿠버네티스 시스템에서 영속성을 가지는 개체. 클러스터의 상태를 나타내기 위해 이 개체를 이용.
→ 쿠버네티스 오브젝트를 동작시키려면 쿠버네티스 API를 이용해야 함.
+ 오브젝트의 구성을 결정해주는 오브젝트 필드 두 가지 (쿠버네티스 컨트롤 플레인은 이 두 상태를 일치시키기 위해 지속적으로 관리!)
    - `spec`: 오브젝트를 생성할 때 리소스에 원하는 특징(*의도한 상태*)
    - `status`: 쿠버네티스 시스템과 컴포넌트에 의해 제공되고 업데이트된 오브젝트의 *현재 상태*

#### 쿠버네티스 오브젝트 기술
오브젝트 생성 시 주의 사항
+ 기본적인 정보(ex. 이름) + `spec`(의도한 상태)를 제시해야 함.
+ API 요청 내용 안에 JSON 형식으로 정보를 포함시켜 줘야 함.
+ JSON으로 정보를 직접 제공해줄 수도 있지만, 대부분 정보를 `.yaml` 파일로 `kubectl`에 제공. (∵ YAML 파일이 사용자 친화적) `kubectl`은 API 요청이 이루어질 때, JSON 형식으로 정보를 변환시켜 줌. `kubectl apply -f <directory>`
+ 의미상 맞다면 가능한 연관된 오브젝트들을 하나의 파일에 모아 작성. (`---`로 구분 가능)

##### 예시

1. 설정 파일 작성(`application/nginx-app.yaml`) <br/>

~~~ yaml
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment # What kind of object you want to create
metadata: # Data that helps uniquely identify the object (name string, UID, and optional namespace)
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec: # What state you desire for the object
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
--- # Management of multiple resources can be simplified by grouping them together
apiVersion: v1
kind: Service
metadata:
  name: my-nginx-svc
  labels:
    app: nginx
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: nginx
~~~

2-1. 작성한 YAML 파일로 리소스 생성
```shell
kubectl apply -f https://k8s.io/examples/application/nginx-app.yaml
```

2-2. 파일 대신 디렉토리 지정 가능. 디렉토리 안의 `.yml` 또는 `.json` 파일 읽어서 작업
```shell
kubectl apply -f https://k8s.io/examples/application/nginx/
```

> 위의 YAML 예시에서는 `label`이 하나지만, 여러 `label`을 사용해야 하는 경우가 많음. `label`은 이름이나 UID와 달리 고유하지 않고 중복 사용 가능함. 레이블 셀렉터로 오브젝트 식별.

<!--more-->
---
