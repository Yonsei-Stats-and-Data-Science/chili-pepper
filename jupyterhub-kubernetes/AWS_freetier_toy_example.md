---
title: JupyterHub with Kubernetes toy example on AWS free-tier T2 micro instances
author: Jongmin Mun
---


- AWS 프리티어(t2 micro)에서 진행 - kops 세팅용 인스턴스 하나, master 인스턴스 하나, node 인스턴스 하나, 총 3개 EC2 인스턴스 사용
- 뭔가를 설치하는 스텝은 전반적으로 매우 느림. 30분-1시간 걸린 것도 있음.
- EKS는 과금이 되는 것 같아서, kops 사용
- 하루 동안 설치 연습을 해 본 결과, 프리티어 S3 사용량의 40퍼센트, EC2 사용량의 10퍼센트를 이미 사용함. →12월 7일 쯤에 삭제하기

클러스터 이름: ysbk21.k8s.local

helm chart release: ysbk21jh

jupyterhub namespace: ysbk21jhk8s 

접속 주소: [a8f8cb6c1d30f4f9f990af162f5c85e0-585791944.ap-northeast-2.elb.amazonaws.com](http://a8f8cb6c1d30f4f9f990af162f5c85e0-585791944.ap-northeast-2.elb.amazonaws.com/)

순서

1. Setup kubernetes on AWS using kops
    
    1.1. 인스턴스 생성, kops 설치
    
    1.2. 클러스터 생성
    
    1.3. 클러스터 테스트
    
2. Setting up helm
3. Install Jupyterhub
    
    3.1. Initialize a Helm chart configuration file
    
4. Future plan:
- kubernetes 전반적인 개념에 대해 더 공부
- reverse proxy를 쓰면 쿠버네티스 위에서 도는 다른 서비스(Rstudio server?)에 ip 한개로 다른 url 줄 수 있을 것 같은데, 이에 대해 공부

전반적으로 아래 내용을 따름.

[Zero to JupyterHub with Kubernetes - Zero to JupyterHub with Kubernetes documentation](https://zero-to-jupyterhub.readthedocs.io/en/latest/)

# 1. Setup kubernetes on AWS using kops

## 1.1. 인스턴스 생성, kops 설치

다음 3개의 자료를 참고하고, 각 자료에 있는 명령어를 취합하여 진행

[Kubernetes on Amazon Web Services (AWS) - Zero to JupyterHub with Kubernetes documentation](https://zero-to-jupyterhub.readthedocs.io/en/latest/kubernetes/amazon/step-zero-aws.html)

[AWS에서 kops로 Kubernetes 클러스터 구축하기](https://essem-dev.medium.com/aws%EC%97%90%EC%84%9C-kops%EB%A1%9C-kubernetes-%ED%81%B4%EB%9F%AC%EC%8A%A4%ED%84%B0-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0-376d71baef9a)

[Setup Kubernetes on AWS | Kubernetes Cluster on AWS Using Kops | Kubernetes AWS Kops](https://www.youtube.com/watch?v=YwNlzpreqmw)

### **1.1.1.** AWS 콘솔에서 EC2 인스턴스 하나 만들기

이름을 Kops라 하자.

### **1.1.2.** IAM role(역할) 만들기

유튜브 링크(3분 20초 쯤)에 있는 대로. 일단 간단히 해보기 위해 AdministratorAccess policy로 role을 만든다.

### **1.1.3.** Kops에 IAM role 추가. (작업>보안>IAM 역할 추가)

[AWS | EC2에 IAM 역할 추가 및 콘솔 접속](https://kitty-geno.tistory.com/68)

### **1.1.4.** Kops 인스턴스에 ssh로 접속. 아래 커맨드 실행하여 EC2에 Kops 설치

```bash
#download kops binary
curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64

#give execute permissions to this binary
chmod +x ./kops

#move this binary to /usr/local/bin/ so that this command will bein the path
sudo mv ./kops /usr/local/bin/

kops version #check install
```

### **1.1.5.** 이어서 kubectl 설치

```bash
#download
curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

# add execute permission
chmod +x ./kubectl

#move
sudo mv ./kubectl /usr/local/bin/kubectl
```

참고) 각각의 역할은 아래와 같다.

- **kops**: to maintain the cluster
- **ctl**: to interact with the cluster(ex. create port, service, do deployment...)

### **1.1.6** Install the AWS CLI:

```bash
sudo apt**-**get update
sudo apt**-**get install awscli
```

또는

[Linux에서 AWS CLI 버전 2 설치, 업데이트 및 제거](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/install-cliv2-linux.html)

aws 환경 설정을 위해 aws configure를 입력 후 필요한 항목들을 입력하자.

```
aws configure
AWS Access Key ID [None]: <Your access key id>
AWS Secret Access Key [None]: <Your secret access key>
Default region name [None]: ap-northeast-2
Default output format [None]:
```

나에게 발생한 오류:  region name 잘못 입력해서 configure 파일에 region name 항목이 2개가 되자, 갑자기 aws configure 파일을 못 불러오기 시작했다.  nano로 중복된 항목을 제거하니 다시 잘 작동했다.

제대로 설정이 되었는지 다음과 같은 명령어를 통해 확인해 보자.

```
aws ec2 describe-instances
aws iam list-users
```

### 1.1.7. Setup an ssh keypair to use with the cluster:

```bash
ssh**-**keygen
```

## 1.2. **클러스터 생성**

이제 작업 환경이 구축되었으니 클러스터를 생성해보자.

### 1.2.1. **S3 버킷 생성**

[[AWS] S3 bucket 생성하기](https://zamezzz.tistory.com/298)

- Create an S3 bucket to store your cluster configuration
- Since we are on AWS we can use a S3 backing store.
- It is recommended to enabling versioning on the S3 bucket.
- **We don’t need to pass this into the KOPS commands. It is automatically detected by the kops tool as an env variable.**

```bash
aws s3api create-bucket \
    --bucket ysbk21 \
    --region ap-northeast-2 \
    --create-bucket-configuration LocationConstraint=ap-northeast-2
aws s3api put-bucket-versioning \
    --bucket ysbk21  \
    --versioning-configuration Status=Enabled
```

### 1.2.2. **환경 변수 설정(.bashrc에 써 놓지 않으면 ssh 재접속시마다 해줘야 함)**

kops가 AWS API를 쓰기 위한 환경 변수를 설정한다. 

```
export AWS_ACCESS_KEY_ID=$(aws configure get aws_access_key_id)
export AWS_SECRET_ACCESS_KEY=$(aws configure get aws_secret_access_key)
```

작업을 편리하게 하기 위해 아래 환경 변수도 생성한다. **Gossip 기반 클러스터를 사용할 계획이기 때문에 클러스터 이름은 .k8s.local로 끝나야 한다.** (예: myfirstcluster.k8s.local)

Since we are not using pre-configured DNS we will use the suffix “.k8s.local”. Per the docs, if the DNS name ends in .k8s.local the cluster will use internal hosted DNS.

```bash
export KOPS_CLUSTER_NAME=ysbk21.k8s.local
export KOPS_STATE_STORE=s3://ysbk21 

vi .bashrc #SSH 다시 접속하면 환경 변수가 사라져서 cluster validation이 안되므로.
```

### 1.2.3. create the "definition" of the cluster

cluster 자체를 만드는 것이 아니라 definition만 만드는 것,

요금을 생각해서 t2.micro 타입으로 1개씩만 만들도록 하자.

클러스터를 생성한 Availability Zone을 지정해야 하는데 여기서는 간단하게 ap-northeast-2c만 지정했다. 다음 명령어로 클러스터 생성을 시작해 보자. (클러스터가 바로 생성되는 것은 아니고 생성을 위한 설정들이 만들어진다.)

```bash
kops create cluster \
--state=${KOPS_STATE_STORE} \
--node-count=1 \
--master-size=t2.micro \
--node-size=t2.micro \
--zones=ap-northeast-2c \
--name=${KOPS_CLUSTER_NAME} \

```

아래 명령어를 통해 진짜로 생성

```bash
kops update cluster --name ysbk21.k8s.local --yes --admin
```

## 1.3. **클러스터 테스트**

다음 명령어들로 클러스터 상태를 확인할 수 있다.

```
kops validate cluster            # 클러스터 상태 확인
kubectl get nodes --show-labels  # 노드 목록 가져오기
kubectl -n kube-system get po    # kube-system 네임스페이스 안의 Pod 목록
```

### Wait for the cluster to start-up

Running the `kops validate cluster` command will tell us what the current state of setup is. If you see “can not get nodes” initially, just be patient as the cluster can’t report until a few basic services are up and running.

Keep running `kops validate cluster` until you see “Your cluster $NAME is ready” at the end of the output:

```bash
time until kops validate cluster**;** do sleep **15;** done
```

### cluster 지우기

```bash
kops delete cluster --name= ysbk21.k8s.local --yes
```

지우고 같은 이름으로 다시 만들 때 UNAUTHORIZED error 해결법:

[kOps 1.19 reports error "Unauthorized" when interfacing with AWS cluster](https://stackoverflow.com/questions/66341494/kops-1-19-reports-error-unauthorized-when-interfacing-with-aws-cluster)

# 2. setting up helm

별 문제 없음

[Setting up helm - Zero to JupyterHub with Kubernetes documentation](https://zero-to-jupyterhub.readthedocs.io/en/latest/kubernetes/setup-helm.html)

```bash
curl https://raw.githubusercontent.com/helm/helm/HEAD/scripts/get-helm-3 | bash
helm version
```

# 3. Install Jupyterhub

[Installing JupyterHub - Zero to JupyterHub with Kubernetes documentation](https://zero-to-jupyterhub.readthedocs.io/en/latest/jupyterhub/installation.html)

## 3.1. Initialize a Helm chart configuration file

```bash
nano config.yaml
```

한 다음에, 아래 comment를 붙여넣는 것으로 시작.

```bash
# This file can update the JupyterHub Helm chart's default configuration values.
#
# For reference see the configuration reference and default values, but make
# sure to refer to the Helm chart version of interest to you!
#
# Introduction to YAML:     https://www.youtube.com/watch?v=cdLNKUoMc6c
# Chart config reference:   https://zero-to-jupyterhub.readthedocs.io/en/stable/resources/reference.html
# Chart default values:     https://github.com/jupyterhub/zero-to-jupyterhub-k8s/blob/HEAD/jupyterhub/values.yaml
# Available chart versions: https://jupyterhub.github.io/helm-chart/
#
```

## 3.2. Install JupyterHub[¶](https://zero-to-jupyterhub.readthedocs.io/en/latest/jupyterhub/installation.html#install-jupyterhub)

Make Helm aware of the [JupyterHub Helm chart repository](https://jupyterhub.github.io/helm-chart/) so you can install the JupyterHub chart from it without having to use a long URL name.

```bash
helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
```

나는 아래와 같은 output이 나왔다... docs에 있는 `...Successfully got an update from the "stable" chart repository` 가 없다.

```bash
ubuntu@ip-172-31-36-1:~$ helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
"jupyterhub" has been added to your repositories
ubuntu@ip-172-31-36-1:~$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "jupyterhub" chart repository
Update Complete. ⎈Happy Helming!⎈
```

이제 jupyterhub라는 chart를 install한다.

`config.yaml` 에 나온 설정에 따라 install하는 것이다(코멘트 말곤 아무것도 안적었지만)

`config.yaml` 가 있는 디렉토리에서 아래 명령을 실행한다

```bash
helm upgrade --cleanup-on-fail \
  --install <helm-release-name> jupyterhub/jupyterhub \
  --namespace <k8s-namespace> \
  --create-namespace \
  --version=<chart-version> \
  --values config.yaml
```

내 설정을 넣은 명령어(실행에 30분 이상 걸림, output 출력도 없음):

```bash
helm upgrade --cleanup-on-fail \
  --install ysbk21jh jupyterhub/jupyterhub \
  --namespace ysbk21jhk8s \
  --create-namespace \
  --version=1.2.0 \
  --values config.yaml \
  --timeout 60m
```

[Glossary](https://helm.sh/docs/glossary/#release)

위 명령어에서 내가 정해 줘야 하는 내용은 4개다.

### 1. `**<helm-release-name>`**

`**<helm-release-name>**` refers to a [Helm release name](https://helm.sh/docs/glossary/#release), an identifier used to differentiate chart installations. You need it when you are changing or deleting the configuration of this chart installation. If your Kubernetes cluster will contain multiple JupyterHubs make sure to differentiate them. You can list your Helm releases with `helm list`.

**release란?**

When a chart is installed, the Helm library creates a *release* to track that installation.

**A single chart may be installed many times into the same cluster, and create many different releases.** For example, one can install three PostgreSQL databases by running `helm install` three times with a different release name.

### 2. `**<k8s-namespace>**`

`**<k8s-namespace>**` refers to a [Kubernetes namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/), an identifier used to group Kubernetes resources, in this case all Kubernetes resources associated with the JupyterHub chart. You’ll need the namespace identifier for performing any commands with `kubectl`.

### 3. `**-version**`

The `**-version**` parameter corresponds to the *version of the Helm chart*, not the version of JupyterHub. Each version of the JupyterHub Helm chart is paired with a specific version of JupyterHub. E.g., `0.11.1` of the Helm chart runs JupyterHub `1.3.0`. For a list of which JupyterHub version is installed in each version of the JupyterHub Helm Chart, see the [Helm Chart repository](https://jupyterhub.github.io/helm-chart/).

- 위 링크를 보고 1.2.0으로 설정

### 4. `**-timeout**`

**(나에게 발생했던 오류)** If you’re pulling from a large Docker image you may get a `Error: timed out waiting for the condition` error, add a `-timeout=<number-of-minutes>m` parameter to the `helm` command.

### 문제 발생시:

- If you get a `release named <helm-release-name> already exists` error, then you should delete the release by running `helm delete <helm-release-name>`. Then reinstall by repeating this step. If it persists, also do `kubectl delete namespace <k8s-namespace>` and try again.
- In general, **if something goes *wrong* with the install step**, delete the Helm release by running `helm delete <helm-release-name>` before re-running the install command.

### enable autocompletion for kubectl

To remain sane we recommend that you enable autocompletion for kubectl (follow [the kubectl installation instructions for your platform](https://kubernetes.io/docs/tasks/tools/#kubectl) to find the shell autocompletion instructions)

[bash auto-completion on Linux](https://kubernetes.io/docs/tasks/tools/included/optional-kubectl-configs-bash-linux/)

and set a default value for the `--namespace` flag:

```bash
kubectl config set-context $(kubectl config current-context) --namespace ysbk21jhk8s
```

Wait for the *hub* and *proxy* pod to enter the `Running` state. (내 경우, hub가 running이 될 때까지 거의 60분이 걸림)

```bash
kubectl get pod --namespace ysbk21jhk8s

NAME                              READY   STATUS             RESTARTS        AGE
continuous-image-puller-rbjss     1/1     Running            0               102m
hub-7546b468f4-dzptz              1/1     Running            0               83m
proxy-5485447bd7-tbt6d            1/1     Running            0               83m
user-scheduler-79748fbdfd-kz8nx   0/1     CrashLoopBackOff   12 (62s ago)    83m
user-scheduler-79748fbdfd-zsglv   1/1     Running            13 (118s ago)   83m
```

## 3.3. 외부 ip를 복사해서 접속해 보기.

- Find the IP we can use to access the JupyterHub.
- Run the following command until the `EXTERNAL-IP` of the `proxy-public` [service](https://kubernetes.io/docs/concepts/services-networking/service/) is available like in the example output.

```bash
kubectl get service **--**namespace ysbk21jhk8s
```

```bash
NAME           TYPE           CLUSTER**-**IP      EXTERNAL**-**IP     PORT**(**S**)**        AGE
hub            ClusterIP      **10.51.243.14**    **<**none**>**          **8081/**TCP       **1**m
proxy**-**api      ClusterIP      **10.51.247.198**   **<**none**>**          **8001/**TCP       **1**m
proxy**-**public   LoadBalancer   **10.51.248.230**   **104.196.41.97**   **80:31916/**TCP   **1**m
```

- 쿠버네티스에서 말하는 **서비스**라는 자원은 **POD의 집합**
- 동일한 서비스 동작을 구현하는 여러가지 POD를 하나로 묶어 관리하는 객체.
- 서비스(Service)의 종류 : ClusterIP, NodePort, Loadbalancer

If the IP for `proxy-public` is too long to fit into the window, you can find the longer version by calling:

```bash
kubectl describe service proxy**-**public **--**namespace ysbk21jhk8s
```

계속 pending이다가, 무려 30분 정도 지나니 proxy_public의 external ip가 나왔음. 여기로 접속하면 jupyterhub 로그인 페이지가 나옴

```bash
[a8f8cb6c1d30f4f9f990af162f5c85e0-585791944.ap-northeast-2.elb.amazonaws.com](http://a8f8cb6c1d30f4f9f990af162f5c85e0-585791944.ap-northeast-2.elb.amazonaws.com/)
```



- To use JupyterHub, enter the external IP for the `proxy-public` service in to a browser.
- JupyterHub is running with a default *dummy* authenticator so entering any username and password combination will let you enter the hub.

### 문제상황

user-scheduler가 번갈아서 계속 crashLoopBackOff 상태에 빠짐 

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5385dd33-6ea7-423d-a489-08e26ea98f03/Untitled.png)

로그인해 보니

2021-12-05T06:43:00.306429Z [Warning] 0/2 nodes are available: 2 Insufficient memory

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ffdbd510-e1d8-49e5-ac7e-a6d35e17faa5/Untitled.png)

아마도 T2 micro의 메모리(고작 1기가) 때문인 것 같음

[누가 Kubernetes 클러스터에 있는 나의 사랑스러운 Prometheus 컨테이너를 죽였나! - LINE ENGINEERING](https://engineering.linecorp.com/ko/blog/prometheus-container-kubernetes-cluster/)

pod들의 상태를 확인

```bash
kubectl -n ysbk21jhk8s get pods -o wide
#       cluster 이름 #get명령 #넓은 출력포맷
```

scheduler pod 두 개가 crassLoopBackoff 상태인 것을 확인했다.

각 pod의 상태를 자세히 보자.

```bash
kubectl -n ysbk21jhk8s describe pods user-scheduler-79748fbdfd-kz8nx
#       cluster 이름 #describe명령 #pod대상   #대상 pod 이름
```

- `CrashLoopBackOff` 상태란 파드(pod)가 시작과 비정상 종료를 연속해서 반복하는 상태를 말함.
- 종료 코드(exit code)는 `255`. (내 경우)
- 컨테이너 상태를 확인한 뒤 서버 공간이 충분한 지 확인했습니다. 과거 경험에 비추어 볼 때, Prometheus가 대상에서 읽어오는 수집 지표(metrics)들의 크기가 가용 디스크 공간을 초과해서 비정상 종료되는 경우가 종종 있었습니다. Prometheus 파드의 [PVC](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)를 기준으로 저장 공간의 사용량을 확인했습니다.

```bash
kubectl get pvc -o wide 
```

```bash
kubectl logs user-scheduler-79748fbdfd-kz8nx -f -c ysbk21jhk8s 
```