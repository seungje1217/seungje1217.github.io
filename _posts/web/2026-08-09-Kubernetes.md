---
layout: single
title: "📘 Kubernetes?"
toc: true
toc_sticky: true
toc_label: "목차"
categories: web
excerpt: "Kubernetes 기초와 로컬 클러스터 실습"
tag: [Kubernetes, Docker, nginx, kind]
---

## nginx?

nginxs는 웹 브라우저의 요청을 받아 웹페이지나 데이터를 돌려주는 **웹 서버 프로그램**이다.

브라우저에서 http://localhost:8080 이런 주소를 열었을 때

```text
브라우저
  ↓
Kubernetes Service
  ↓
nginx가 실행 중인 Pod
```

와 같은 순서로 요청이 전달된다.

nginx는 브라우저의 요청을 받고 HTML 페이지를 응답한다.

nginx는 Kubernetes에 포함된 기능은 아니지만, Kubernetes를 연습하기 위해 배포한 예제 애플리케이션이다.

실무에서는 HTML, CSS, 이미지와 등 정적파일 제공, 사용자 요청을 백엔드 서버로 전달, HTTPS 연결 관리 등에 사용된다.


## Kubernetes?

Kubernetes(쿠버네티스, K8s)는 여러 컨테이너를 원하는 상태로 실행하고 유지해주는 **컨테이너 관리 시스템**이다.

```bash
docker run nginx
```

이처럼 nginx 컨테이너를 직접 실행하면 컨테이너 하나는 실행할 수 있다.

하지만 실제 서비스에서는 컨테이너가 죽으면 누가 다시 실행할지, 사용자가 늘어나면 컨테이너를 어떻게 늘릴지, 컨테이너가 여러 개면 어디로 요청을 보낼지 등 여러 문제를 해결해야한다.

Kubernetes는 이런 작업을 자동으로 처리한다.

## Docker와 Kubernetes의 차이

| Docker : 컨테이너를 만들어서 실행

| Kubernetes : 여러 컨테이너를 배포, 감시, 복구하고 연결

비유하자면 Docker는 작업자이고 Kubernetes는 작업자를 배치하고 관리하는 관리자라고 생각하면 된다.

우리가 사용하는 kind는 Docker 컨테이너 안에 연습용 Kubernetes 클러스터를 만들어주는 도구이다.

## Kubernetes 가장 중요한 원리

Kubernetes에서는 명령을 하나씩 지시하기보다 원하는 상태를 **선언**한다.

```yaml
replicas: 2
```

YAML에 위와 같이 적는다. 
nginx Pod를 지금 당장 2개 만들어라가 아니라, nginx Pod가 항상 2개인 상태를 유지하라는 뜻이다.

Kubernetes는 계속해서 현재 상태와 원하는 상태를 비교하여 Pod가 1개가 되는 경우 새로운 Pod를 생성하는 등 **선언적 관리**를 한다.

## Kubernetes 핵심 구성 요소

### Cluster

Kubernetes가 동작하는 전체 환경이다.

하나 이상의 Node로 구성되며, 컨테이너를 어느 Node에서 실행할지 관리한다.

이번 실습에서는 `kind create cluster --name k8s-lab`를 사용하여 내 컴퓨터에 로컬 Cluster를 만들었다.

### Node

Pod가 실제로 실행되는 컴퓨터 또는 서버이다.

실무에서는 물리 서버나 AWS의 가상 서버가 Node가 될 수 있다. 이번 실습에서는 Docker 컨테이너 하나가 Node 역할을 담당한다.

### Pod

Kubernetes에서 컨테이너를 실행하는 가장 작은 단위이다.

이번 실습에서는 각 Pod 안에서 nginx 컨테이너가 실행된다.

```text
Pod
└── nginx 컨테이너
```

Pod는 영구적인 존재가 아니다. 삭제되거나 문제가 발생하면 새로운 Pod로 교체될 수 있다.

### Deployment

- 어떤 컨테이너 이미지를 사용?
- Pod 몇 개 실행?
- Pod가 종료됐을 때 어떻게 복구?
- 새로운 버전으로 어떻게 업데이트?

Deployment는 위와 같은 내용을 관리한다.

이번 실습에서 만든 Deployment는 아까 언급했듯이 nginx Pod가 항상 2개 존재하도록 관리한다.

### Service

Pod에 접근할 수 있도록 고정된 입구를 제공한다.

Pod는 다시 생성될 때 이름과 IP 주소가 바뀔 수 있다. 따라서 사용자가 특정 Pod에 직접 접속하면 안정적으로 서비스를 이용하기 어렵다.

Service는 사용자의 요청을 현재 실행 중인 Pod로 전달한다.

### kubectl

kubectl은 사용자가 Kubernetes Cluster에 명령을 전달하기 위한 도구이다.

```bash
kubectl get pods
```

위는 현재 실행 중인 Pod 목록을 조회하는 명령어이다.

## kind로 로컬 Cluster 만들기

kind는 Docker 컨테이너를 사용하여 로컬 Kubernetes Cluster를 만들어주는 도구이다.

```bash
// k8s-lab 이라는 Cluster 생성
kind create cluster --name k8s-lab --wait 2m 

// Node 정상 준비인지 확인
kubectl get nodes
```

여기서 Node의 상태가 Ready라면 Cluster를 사용할 준비가 된 것이다.

## Deployment와 Service 배포하기

프로젝트의 `k8s` 폴더에는 Deployment와 Service가 작성된 YAML 파일이 있다.

```bash
kubectl apply -f k8s/
```

위의 명령으로 YAML에 선언된 리소스를 Cluster에 적용했다.

실행 결과는 다음과 같.

```text
deployment.apps/k8s-lab created
service/k8s-lab created
```

생성된 리소스의 상태는 다음 명령으로 확인했다.

```bash
kubectl get deploy,pods,svc
```

확인 결과 Deployment가 정상적으로 생성됐으며, nginx Pod 2개가 Running 상태가 됐다. 또한 Pod에 접근하기 위한 Service가 생성됐다.

## Pod 자동 복구 확인하기

Deployment가 Pod를 자동으로 복구하는지 확인하기 위해 Pod 하나를 삭제했다.

```bash
kubectl get pods

kubectl delete pod k8s-lab-756d95d9b5-5bzqz
```

Pod 이름을 조회 후 실제 Pod 이름을 사용하여 삭제했다.

```bash
kubectl get pods -w
```

위의 명령어로 Pod 변화를 계속 관찰하였으며, Ctrl+C를 누르기 전까지 Pod의 상태 변화를 계속 관찰한다.

Pod 하나를 삭제했지만 Deployment가 새로운 Pod를 자동으로 생성했다.

```text
기존 Pod 삭제
    ↓
Pod 개수가 2개에서 1개로 감소
    ↓
Deployment가 새로운 Pod 생성
    ↓
Pod 개수가 다시 2개가 됨
```

이를 통해 Kubernetes의 **자동 복구(Self-healing)** 기능을 확인할 수 있었다.

## 브라우저에서 nginx 확인하기

```bash
kubectl port-forward service/k8s-lab 8080:80
```

위의 명령어를 통해 Kubernetes Service와 내 컴퓨터를 연결했다.

이 명령은 다음 두 포트를 임시로 연결한다.

```text
내 컴퓨터의 8080 포트
          ↓
Kubernetes Service의 80 포트
```

브라우저에서 다음 주소로 접속했다.

```text
http://localhost:8080
```

`Welcome to nginx!` 화면이 나타났으며, 이를 통해 요청이 Service를 거쳐 nginx Pod까지 정상적으로 전달됐다는 것을 확인했다.

`port-forward`는 `Ctrl+C`를 누르면 종료된다. 연결만 종료되는 것이며 Deployment와 Pod가 삭제되는 것은 아니다.

## 정리

- kind를 사용하여 로컬 Kubernetes Cluster 생성
- Deployment와 Service를 YAML로 배포
- nginx Pod 2개가 실행되는 것을 확인
- Pod를 삭제하여 Kubernetes의 자동 복구를 확인
- Service와 port-forward를 통해 브라우저에서 nginx에 접속

즉, Kubernetes는 원하는 상태를 선언하면 컨테이너를 배포하고, 연결하고, 그 상태가 유지되도록 관리하는 시스템이다.
