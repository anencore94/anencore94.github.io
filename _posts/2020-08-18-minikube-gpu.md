---
title: "Minikube GPU 사용 가이드"
date: 2020-08-18 15:49:00
tags: k8s
---

# 1. 개요

쿠버네티스 환경에서 pod 과 같은 쿠베 리소스가 노드의 GPU 를 사용하기 위해서는 설정해주어야 하는 몇 가지 작업이 있습니다. 본 문서에서는 특별히 <strong>Minikube</strong> 환경에서 GPU 를 사용하기 위해서 필요한 설정들을 순서대로 설명합니다. 일반적인 쿠버네티스 환경에서는 설정 방법이 다소 다른 것으로 알고 있으며, 이에 대해서는 추후 다루도록 하겠습니다.

# 2. 서론

Minikube 에서 NVDIA GPU 지원 방식에 대한 [공식 문서](https://minikube.sigs.k8s.io/docs/tutorials/nvidia_gpu/)를 보면, 안타깝게도 kvm2 와 none 를 제외한 driver 옵션들은 GPU passthrouh 를 지원하지 않기 때문에 macOS 와 Windows 에서는 minikube 로 gpu 사용은 할 수 없다고 합니다.

따라서 본 문서에서는 Ubuntu 16.04 환경을 기준으로 하며, `--driver=kvm2` 옵션은 추가적인 설정이 다소 필요하기에, 보다 쉬운 방법인 `--driver=none` 옵션으로 Minikube 를 생성하여 호스트의 GPU 를 사용하는 방법을 다루었습니다.

> 참고한 자료는 다음과 같습니다.
- https://minikube.sigs.k8s.io/docs/tutorials/nvidia_gpu/
- https://docs.nvidia.com/datacenter/kubernetes/kubernetes-upstream/index.html
- https://ssaru.github.io/2019/07/25/20190725-Connect_GPU_to_Minikube/
- https://github.com/NVIDIA/nvidia-docker
- https://github.com/NVIDIA/k8s-device-plugin
- https://www.tensorflow.org/install/docker#gpu_support

# 3. Prerequisites

본 문서는 다음과 같은 환경에서 검증되었습니다.
- OS : Ubuntu 16.04
- minikube : v1.12.3
- docker-ce : 19.03.9
- GPU : NVIDIA GeForce GTX 1070
- nvidia graphics driver :
  - NVIDIA-SMI : 430.64
  - Driver Version: 430.64
  - CUDA Version: 10.1
- nvidia-docker
  - nvidia-container-runtime
  - nvidia-container-toolkit


# 4. 시작하기

minikube 의 [NVIDIA GPU Support 공식 문서](https://minikube.sigs.k8s.io/docs/tutorials/nvidia_gpu/)를 보면 `none` driver 를 사용할 경우의 설치 및 사용 방법이 굉장히 간단하게 설명되어있지만, 실제로 설치를 시도해보면 생각처럼 쉽지 않습니다.

예를 들면 `configure docker with nvidia as the default runtime` 와 같이 굉장히 간략하게 설명된 부분으로 인하여 설치하는 데 어려운 부분이 있기 때문에 본 문서에서는 각 단계를 하나하나 조금 더 자세히 설명하겠습니다.

## 1) Install minikube

Minikube 바이너리를 설치하는 방법은 [이전 포스트](https://anencore94.github.io/2020/08/15/minikube-usage.html)에서 확인하실 수 있습니다.

## 2) NVIDIA driver, NVIDIA docker 설치 및 설정

### a) NVIDIA Driver 설치

본 글을 읽는 독자는 GPU 를 로컬에서는 이미 잘 사용하고 계신 분을 대상으로 하기에, 사용하시는 GPU 에 맞는 NVIDIA Driver 와 CUDA 는 이미 로컬에 잘 설치되었다고 가정하겠습니다.

혹시 GPU 를 처음 사용하시는 분이라면 [다음 문서](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#package-manager-installation) 등을 참고하시어 OS 와 GPU 에 맞는 driver 를 설치하시기 바랍니다.

정상적으로 설치되었다면 다음과 같은 결과가 출력됩니다.
```shell
$ nvidia-smi

 NVIDIA-SMI 430.64       Driver Version: 430.64       CUDA Version: 10.1     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 1070    Off  | 00000000:01:00.0  On |                  N/A |
|  0%   40C    P8     7W / 170W |    881MiB /  8116MiB |      2%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0      1160      G   /usr/lib/xorg/Xorg                           496MiB |
|    0      2922      G   compiz                                        69MiB |
+-----------------------------------------------------------------------------+
```

### b) NVIDIA-docker 설치

Ubuntu 16.04 의 경우에는 다음 명령어를 실행하여 설치합니다.

```shell
$ distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
$ curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
$ curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit nvidia-container-runtime
sudo systemctl restart docker
```

정상적으로 설치되었는지 확인하기 위해 다음 명령어를 실행합니다.
- cuda 9.x 대 GPU 를 사용하신다면, 아래 `docker run ...` 명령에서 `cuda:10.0-base` 대신 `cuda:9.0-base` 로 실행해야 합니다.

```shell
$ docker run --gpus all nvidia/cuda:10.0-base nvidia-smi

...
 NVIDIA-SMI 430.64       Driver Version: 430.64       CUDA Version: 10.1     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 1070    Off  | 00000000:01:00.0  On |                  N/A |
|  0%   40C    P8     7W / 170W |      0MiB /  8116MiB |      2%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|   No running processes found                                                |
+-----------------------------------------------------------------------------+

$ nvidia-container-runtime --version
runc version spec: 1.0.1-dev
```

해당 명령어의 출력 결과에서 현재 GPU 를 사용하고 있는 Process 가 하나도 없다고 나오는 것이 이상하게 생각되실 수 있지만, docker 입장에서는 독립된 자기 자신의 내부 프로세스만 확인할 수 있기 때문에 이는 정상적인 상황입니다.

#### nvidia-docker2

다른 문서를 찾아보면 다음 명령어가 실행되어야 한다는 문서가 존재합니다. 하지만 본 가이드대로 nvidia-docker 를 설치한 경우는 다음 명령어가 아래와 같은 에러를 내며 실행되지 않더라도 문제되지 않습니다.

```shell
$ nvidia-docker version
nvidia-docker: command not found
```
`$ nvidia-docker` 는 depreated 된 버전인 `nvidia-docker2 pkg` 에서만 실행가능한 커맨드이며, `nvidia-container-toolkit` 과 같은 newest nvidia-docker 를 설치하였다면 실행되지 않는 게 정상이며 실행될 필요가 없다고 합니다.
- [nvidia-docker2 관련 내용](https://github.com/NVIDIA/nvidia-docker/issues/1028#issuecomment-554345662)


### c) docker default-runtime 변경

Minikube 는 docker default-runtime 을 Docker-CE 로 사용하지만, docker container 내에서 NVIDIA GPU 를 사용하기 위해서는 nvidia-docker 라는 docker-runtime 으로 docker container 를 생성해야 합니다.
- nvidia-docker : 간단히 말하면 Docker-CE 의 확장판

따라서 Minikube 가 nvidia-docker 로 pod 내부의 container 를 생성할 수 있도록 호스트의 default-runtime 설정을 변경해주어야 합니다.

docker daemon config file 을 다음과 같이 수정합니다.
```shell
$ vi /etc/docker/daemon.json
{
  "default-runtime": "nvidia",
  "runtimes": {
      "nvidia": {
          "path": "nvidia-container-runtime",
          "runtimeArgs": []
   }
  }
}
```

수정 후 docker 를 재시작합니다.
```shell
$ sudo systemctl daemon-reload
$ sudo service docker restart
```

## 3) Start Minikube

root user 로 변경한 뒤, minikube 를 생성합니다. kubernetes-version 은 v1.15 이상이어야 합니다.

```shell
$ sudo su

$ minikube start --driver=none --kubernetes-version=v1.16.3
```

`feature-gates=DevicePlugins=true` parameter 를 추가해주어야 한다는 문서도 존재하지만, DevicePlugin 은 k8s v1.10 부터 beta version 으로 변경되었으므로 v1.16.3 버전에서는 따로 추가해주지 않아도 괜찮습니다.
  - 시스템 하드웨어 resource 를 kubelet 에 알리는 데 사용하는 device plugin framework 로 구현된 [nvidia-device-plugin](https://github.com/NVIDIA/k8s-device-plugin) 의 사용을 위한 기능입니다.

[minikube GPU 공식 문서](https://minikube.sigs.k8s.io/docs/tutorials/nvidia_gpu/)에서 사용하는 `apiserver-ips, apiserver-name` parameter 또한 반드시 필요한 parameter 는 아니며, 특별히 필요한 경우에만 사용하시면 됩니다.

### a) coredns loop plugin 비활성화

`minikube start --driver=none` 으로 생성 시 <strong>로컬의 dns 설정에 따라</strong> coredns pod 이 무한 loop 에 빠지는 경우가 발생할 수 있습니다.

`minikube start` 이후 `kubectl get pod -A` 로 coredns pod 이 `CrashLoopBackOff` 상태인 것을 확인하였다면, 여러 가지 해결 방법([링크](https://stackoverflow.com/questions/53075796/coredns-pods-have-crashloopbackoff-or-error-state/53414041#53414041))이 있지만, 로컬 머신의 설정을 건드리지 않고, 간단하게 해결할 수 있는 방법은 다음과 같습니다.

coredns pod 이 참고하는 configmap 을 수정하기 위해 다음을 입력합니다.
```shell
$ kubectl -n kube-system edit configmap coredns
```

`data.Corefile` 에서 `loop` 라고 써진 줄 전체를 지우고 `:wq` 를 통해 변경 사항을 저장합니다.
```shell
configmap/coredns edited
```

coredns pod 을 지우면, 자동으로 다시 생성되는 coredns pod 이 변경된 configmap 을 가지고 생성되게 됩니다.
```shell
$ kubectl -n kube-system delete pod -l k8s-app=kube-dns
pod "coredns-xxxxxxxxxx" deleted
```

이제 coredns pod 이 RUNNING 상태로 뜨는지 확인합니다.
```shell
$ kubectl -n kube-system get pod -l k8s-app=kube-dns
```


## 4) Install NVIDIA's device plugin

nvidia 에서 DevicePlugin Framework 에 맞춰 gpu 사용을 위해 개발한 nvidia-device-plugin 을 배포합니다.
```shell
$ kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/1.0.0-beta4/nvidia-device-plugin.yml
```

nvidia-device-plugin 은 daemonset 으로 생성되지만, minikube 를 1 node 로 생성했으므로 1 개의 pod 이 `RUNNING` 상태로 생성되었는지 확인합니다.
```shell
$ kubectl get pod -A | grep nvidia
```

node 정보에 gpu 가 사용가능하도록 설정되었는지 확인합니다.
```shell
$ kubectl get nodes "-o=custom-columns=NAME:.metadata.name,GPU:.status.allocatable.nvidia\.com/gpu"
>>>
NAME       GPU
minikube   1
```
- 설정되지 않은 경우, GPU 의 value 가 `<none>` 으로 표시됩니다.

# 5. GPU pod 사용하기

이제 k8s 환경에 nvidia gpu setting 을 끝냈으니, gpu 를 pod 이 사용할 수 있는지를 테스트해봅니다.

다음과 같이 <strong>cuda lib 를 포함하고 있는</strong> `nvidia/cuda:10.0-runtime` 이미지로 pod 을 생성합니다.
- 로컬의 GPU 에 맞는 cuda version 이미지로 생성해야 하는 것에 주의 바랍니다.
- 적합한 cuda 를 포함한 이미지라면 어떤 이미지든 사용 가능합니다.
  - 예) tensorflow/tensorflow:2.2.0rc2-gpu-py3 등
  - cuda 를 포함하지 않은 일반 nginx 등의 이미지로 생성된 pod 의 경우에는 gpu 를 정상적으로 사용하지 못함에 주의 바랍니다.

`spec.resources.requests` 와 `spec.resources.limits` 에 `nvidia.com/gpu` 를 포함해야 pod 내에서 GPU 사용이 가능합니다.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu
spec:
  containers:
  - name: gpu-container
    image: nvidia/cuda:10.0-runtime
    command:
      - "/bin/sh"
      - "-c"
    args:
      - nvidia-smi && tail -f /dev/null
    resources:
      requests:
        nvidia.com/gpu: 1
      limits:
        nvidia.com/gpu: 1
```

`kubectl logs` 를 통해 방금 생성한 pod 내에서 `nvidia-smi` 가 정상적으로 수행된 것을 확인합니다.

```shell
 NVIDIA-SMI 430.64       Driver Version: 430.64       CUDA Version: 10.1     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 1070    Off  | 00000000:01:00.0  On |                  N/A |
|  0%   40C    P8     7W / 170W |      0MiB /  8116MiB |      2%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|   No running processes found                                                |
+-----------------------------------------------------------------------------+

```