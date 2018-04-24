---
layout: post
title:  "Container Wars: Kubernetes vs. Docker Swarm vs. Amazon ECS"
date:   2018-04-23 11:25:01
author: Mars
categories: tools
---

원본 https://caylent.com/containers-kubernetes-docker-swarm-amazon-ecs/  



### Container Orchestration: A Quick Introduction

2013년 Docker가 나왔고 곧 대세가 되었다.
그 시점부터 컨테이너 기술은 DevOps의 지형과 
앱의 빌드, 배포, 운영을 극적으로 변화시키고 있다.

3개의 주요 컨테이너 기술을 살펴보고 어떤 것이 좋을지 알아보자.

### Synopsis
Kubernetes(K8s or Kube)은 container orchestration 영역의 떠오르는 강자이며 구글에 의해서 시작된 오픈 소스 프로젝트이다.
K8s는 구글 클라우드 플렛폼에서 잘 동작하지만 거의 대부분의 인프라스트럭처(AWS 같은)와도 잘 동작한다.


Swarm은 Docker가 제공하는 orchestration tool 이며 
이제는 Docker Engine과 완전히 통합되어 있어서 쉽게 사용할 수 있다. 
서비스를 배포하는 일은 `docker service create` 명령 만큼이나 쉽다. 

  
Amazon Elastic Container Service (ECS)는 AWS services과 당신의 서비스를 연동하기 위한 기술이다. 
AWS 중심의 솔루션들과 쉽게 통합되기에, AWS를 사용하지 않고 있다면 별로일거다.


### Kubernetes
Kube는 다양한 프로덕셔 환경에서 잘 동작한다: 독립 서버(bare metal) 환경, 
자기 소유 VM(on-premise VMs), 
대부분의 클라우드 프로바이더(AWS, Google Cloud Platform, Microsoft Azure 등등), 
이들의 조합과 하이브리드.



Clusters는 몇몇 주요 컴포넌트를 포함한다:
- Pods: 하나의 노드에 함께 배포되어야 할 하나 이상의 컨테이너의 그룹.
- Labels: Key-Value 태그. 파드(Pods), 서비스, Replication Controller에 식별자로 할당된다.
- Services: 파드의 그룹에 제공되는 이름, 컨테이너 트레픽을 보내는 로드 밸런서로써 동작한다.
- Replication controllers: 지정된 수의 복제를 보장해주는 프레임웍


Kube는 가장 복잡하지만 가장 강력한 기능을 제공하고, 프로세스를 단순화시켜주는 좋은 툴들이 많다.



### Docker Swarm
Docker Swarm은 개발자들에게 빠르고 쉽게 여러 컨테이너와 마이크로서비스를 배포하도록 해준다.
이 3개의 툴 중에 가장 가볍고 Docker Engine에 통합되어 있으므로 가장 쉽다. 


### Elastic Container Service (Amazon ECS)
AWS의 container management service로써, Amazon ECS는 Docker-compatible 서비스이다. 
Containerized applications을 EC2 인스턴스에서 실행할 수 있도록 해주는 Kube와 Swarm의 대안이다.

그런데 서울 리전에서는 아직 기능이 별로 없다.
