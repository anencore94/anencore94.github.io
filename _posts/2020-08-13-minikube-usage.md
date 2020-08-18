---
title: "Minikube 사용 가이드"
date: 2020-08-14 15:50:00
tags: k8s
---

# 1. 개요

쿠버네티스를 처음 공부할 때, 가장 먼저 접하게 되는 쿠버네티스 환경인 [Minikube](https://github.com/kubernetes/minikube) 를 사용하는 기본적인 방법을 다룹니다. Minikube 공식 문서 중 주로 사용하는 기능들을 위주로 작성되었습니다.

Minikube 라는 단어에서 유추할 수 있듯이, 나만의 작은 쿠버네티스 클러스터를 로컬에 구축하고 사용하고자 하는 분들을 위해, CNCF 에서 직접 개발, 관리하는 오픈소스 프로젝트입니다.

쿠버네티스를 처음 접하는 분들은 쿠버네티스 환경을 직접 구성하는 것 자체가 사실 매우 어려운 일이기 때문에, 쿠버네티스 공식 튜토리얼을 포함한 대부분의 쿠버네티스 입문서는 minikube 환경에서의 테스트를 바탕으로 하고 있습니다.

> 본 문서는 release v1.11.0 을 기준으로 작성되었으며, 자세한 내용은 Minikube github 의 [README.md](https://github.com/kubernetes/minikube) 와 [공식 사이트](https://minikube.sigs.k8s.io/docs/)에 작성되어 있으니 참고바랍니다.

-----

# 2. Prerequisite

Minikube 는 linux, macos, windows 를 모두 호환하는 프로젝트이지만, 본 문서에서는 linux kernel 기반 os 를 기준으로 작성되었습니다. 특히 windows 의 경우에는 다른 부분이 다소 있을 수 있으니 주의바랍니다.

## 하드웨어

- cpu >= 2
- free memory >= 2GB
- free disk >= 20GB
- 외부망 접속 가능

## 소프트웨어

- 지원하는 Container or virtual machine manager 중 최소 하나
  - Docker, Hyperkit, Hyper-V, KVM, Parallels, Podman, VirtualBox, or VMWare
- curl
- kubectl

-----

# 3. 설치

설치 방법은 굉장히 간단합니다. 먼저 원하는 작업 경로로 이동합니다.

```
$ cd {$WorkSpace}
```

원하는 버전의 바이너리를 현재 경로에 다운받습니다. (예: url 에서 v1.11.0 대신 v1.10.1 로 변경하면 minikube v1.10.1 를 받을 수 있습니다.)

```
$ curl -LO https://storage.googleapis.com/minikube/releases/v1.11.0/minikube-linux-amd64
```

바이너리를 \$PATH 로 이동시켜 모든 경로에서 사용할 수 있게 합니다.

```
$ sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

다음을 입력하여 정상 설치된 것을 확인합니다.
```
$ minikube version
```
- minikube version: v1.11.0 와 같이 출력되면 정상 설치된 것입니다.

-----

# 4. 사용 방법

## 시작하기

minikube 를 실행합니다.
```
$ minikube start
```

- os locale 설정이 ko_KR 로 설정되어 있다면, minikube 가 생성되고 있다는 안내 메시지들이 부분적으로 한글로 출력될 것입니다. 다만, 아직 한글 번역이 완성되지 않아 대부분의 메시지는 영어로 출력될 것입니다. [minikube-translations](https://minikube.sigs.k8s.io/docs/contrib/translations/) 를 참고하여 직접 번역을 반영해주실 수 있습니다.

다음과 같이 `minikube start` 시 여러 옵션을 지정하여 생성할 수도 있습니다.

```
$ minikube start --cpus=4 --memory='3000mb' --disk-size='30000mb' --driver=virtualbox --kubernetes-version=v1.16.3
```
- cpu 4 개, memory 3000 mb, disk size 30000 mb 를 가지는 minikube 를 virtualbox hypervisor 로 생성하며, 쿠버네티스 버전은 v1.16.3 으로 생성하라는 명령입니다.
- 자세한 option 은 `minikube start --help` 를 통해 확인할 수 있습니다.

반복하여 사용하는 옵션은 다음과 같이 로컬에 정보를 저장하여, 매번 입력하지 않아도 설정되도록 할 수 있습니다.

- cpu, memory, disk-size 와 같이 보통 자주 변경하지 않는 옵션은 지정해놓고 사용하는 것을 추천드립니다.

```
$ minikube config view  # 로컬에 저장된 config 를 확인할 수 있습니다.
$ minikube config set memory 2048  # 이후로 생성되는 minikube 는 모두 memory 2048 mb 로 생성됩니다.
$ minikube config set memory 3072  # 덮어쓰기 방식으로 지정된 config 를 변경할 수도 있습니다.
```

### Driver option

driver option 은 주로 다음과 같은 option 을 사용합니다. (특정 버전 이전에서는 `vm-driver` parameter 와 동일합니다.)

각각의 driver 마다 prerequisite 이 다릅니다. 자세한 내용은 다음 [링크](https://minikube.sigs.k8s.io/docs/drivers/)를 통해 확인바랍니다.

주로 사용하는 driver 에 대한 간략한 설명은 다음과 같습니다.
- docker : minikube 를 docker container 로 띄웁니다. 내부적으로 docker in docker 기능을 사용하여 여러 pod 들을 구동하게 됩니다. 대부분의 
- virtualbox : virtualbox package 를 사용해 vm 을 구축합니다.
- kvm2 : kvm2 를 사용해 vm 을 구축합니다. host 의 gpu 등을 minikube 내부에서 사용하고 싶은 경우 등에 사용합니다.
- none : root 권한으로만 사용 가능하며, host 의 docker 등의 자원들을 사용하여 Minikube 를 생성합니다.

## 접근하기

Minikube 를 생성하면 kubectl 이 참고하는 KUBECONFIG 경로(변경한 적이 없다면 `~/.kube/config`)에 Minikube 접근을 위한 config 파일이 생성된 것을 확인하실 수 있습니다.

```
$ kubectl config view
```
- current-context 가 Minikube 를 바라보도록 설정된 것을 확인하실 수 있습니다.

```
$ kubectl get nodes
$ kubectl get cs
$ kubectl get pod -A
```
- 위 명령이 정상적으로 수행된다면, 이제 kubectl 을 사용한 k8s resource 관리를 자유롭게 할 수 있습니다.

Minikube 인스턴스 내부에 직접 접근하고 싶은 경우에는, 다음 명령을 통해 접근 가능합니다. (사용한 driver option 에 따라 `minikube ssh` 를 사용할 수 없을 수도 있습니다.)

```
$ minikube ssh
```

## 삭제하기

`minikube start` 로 생성한 인스턴스 및 관련 config 를 모두 삭제합니다.

```
$ minikube delete
```

## 일시 정지하기

사용 중인 minikube 인스턴스의 리소스 사용을 잠시 중지하고 싶은 경우, 일시 정지 기능을 사용할 수 있습니다.

```
$ minikube pause
```

-----

# 5. 기타 기능

## 1) minikube 내부로 파일 보내기

minikube 내부로 특정 파일이나 폴더를 옮기고 싶은 경우에는 다음 두 방법 중 하나를 사용하시면 됩니다.

### A) scp 활용
minikukbe 의 default user name 은 docker 이며, ssh-key 를 통한 접근이 필요합니다. 따라서 minikube 의 내장 기능인 `minikube ssh-key` 와 `minikube ip` 커맨드를 사용하여 접속정보를 얻은 후 scp 를 진행합니다.
```
$ scp -i $(minikube ssh-key) <local-path> docker@\$(minikube ip):<remote-path>
```

예)
```
$ scp -i $(minikube ssh-key) /home/myuser/sourceFolder docker@$(minikube ip):/home/docker/destiationFolder
```

### B) minikube mount 활용

`minikube mount` 기능을 사용하여 호스트의 특정 디렉토리를 minikube 와 동기화해버리는 방법도 있습니다.
```
$ minikube mount [flags] <source directory>:<target directory>
```

예)
```
$ minikube mount /home/myuser/sourceFolder:/home/docker/destiationFolder
```

## 2) minikube 에 insecure-registry 설정하기

`minikube start` 시에 `--insecure-registry` flag 를 사용하면, minikube 내부에서 insecure-registry 등록을 따로 하지 않아도 자동으로 설정됩니다.

예)
```
$ minikube start --insecure-registry "192.168.7.55:5000"
```
