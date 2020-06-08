# High availabiltiy, Auto Scailing 테스트
이번 lab 은 High availabiltiy, Auto Scailing 관련하여 해보도록 하겠습니다.   
   
# High availabiltiy
OpenShift 는 고가용성 유지를 위해 어떻게 동작하는지 알아보도록 하겠습니다.    
OpenShift 는 Kubernetes 라는 Container Orchestration 를 이용하여 고가용성을 유지합니다.   
   
Kubernetes 의 주요 기능은 아래와 같습니다.   
> 1) 서비스 디스커버리와 로드 밸런싱 쿠버네티스는 DNS 이름을 사용하거나 자체 IP 주소를 사용하여 컨테이너를 노출할 수 있다.    
    컨테이너에 대한 트래픽이 많으면, 쿠버네티스는 네트워크 트래픽을 로드밸런싱하고 배포하여 배포가 안정적으로 이루어질 수 있다.   
> 2) 스토리지 오케스트레이션 쿠버네티스를 사용하면 로컬 저장소, 공용 클라우드 공급자 등과 같이 원하는 저장소 시스템을 자동으로 탑재 할 수 있다.
    자동화된 롤아웃과 롤백 쿠버네티스를 사용하여 배포된 컨테이너의 원하는 상태를 서술할 수 있으며 현재 상태를 원하는 상태로 설정한 속도에 따라 변경할 수 있다. 예를 들어 쿠버네티스를 자동화해서 배포용 새 컨테이너를 만들고, 기존 컨테이너를 제거하고, 모든 리소스를 새 컨테이너에 적용할 수 있다.   
> 3) 자동화된 빈 패킹(bin packing) 컨테이너화된 작업을 실행하는데 사용할 수 있는 쿠버네티스 클러스터 노드를 제공한다.    
    각 컨테이너가 필요로 하는 CPU와 메모리(RAM)를 쿠버네티스에게 지시한다. 쿠버네티스는 컨테이너를 노드에 맞추어서 리소스를 가장 잘 사용할 수 있도록 해준다.   
> 4) 자동화된 복구(self-healing) 쿠버네티스는 실패한 컨테이너를 다시 시작하고, 컨테이너를 교체하며,   
    ‘사용자 정의 상태 검사’에 응답하지 않는 컨테이너를 죽이고, 서비스 준비가 끝날 때까지 그러한 과정을 클라이언트에 보여주지 않는다.   
> 5) 시크릿과 구성 관리 쿠버네티스를 사용하면 암호, OAuth 토큰 및 SSH 키와 같은 중요한 정보를 저장하고 관리 할 수 있다. 컨테이너 이미지를 재구성하지 않고 스택 구성에 시크릿을 노출하지 않고도 시크릿 및 애플리케이션 구성을 배포 및 업데이트 할 수 있다.   
* 출처: [What is Kubernetes](https://kubernetes.io/ko/docs/concepts/overview/what-is-kubernetes/)   
   
위와 같이 제공되는 주요 기능중, Self-Healing 기능이 Container 환경에서 `Kubernetes`/`OpenShift` 를 사용하게 되는 주요 이유입니다.   
   
## Pod 단위 
`Deployment` 가 `4` 인 APP 에서 Pod 하나를 강제로 종료하여 복구가 되는 것을 확인합니다.   
   
```bash
$ oc delete pod/testapp-XXXXX
```
   
```bash
$ oc get all
```
   
## Node 단위
Worker node 장애 발생 시나리오

