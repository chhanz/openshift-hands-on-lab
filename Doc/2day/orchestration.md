# High availabiltiy, Auto Scailing 테스트
이번 lab 은 High availabiltiy, Auto Scailing 관련하여 해보도록 하겠습니다.   

# Auto Scailing 설정
OpenShfit 는 아래와 같이 내장되어 있는 Prometheus 를 이용하여 OpenShift 내의 Pod metric 을 수집하고 모니터링을 할수있도록 지원하고 있습니다.   
* 참고 자료: [https://access.redhat.com/solutions/3990701](https://access.redhat.com/solutions/3990701)   
   
또한 Horizontal Pod Autoscaler(이하 HPA) 를 이용하여 설정한 CPU 사용률을 기반으로 Replicaset, Deployment 의 Pod 수를 자동으로 Scaling 할 수 있습니다.
```bash
$ oc adm top node
NAME                    CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
master1.ocp.chhan.com   755m         21%    2259Mi          15%
master2.ocp.chhan.com   1104m        31%    3161Mi          21%
master3.ocp.chhan.com   1325m        37%    3823Mi          25%
worker1.ocp.chhan.com   218m         6%     948Mi           6%
worker2.ocp.chhan.com   985m         28%    4403Mi          29%
$ oc adm top pod
NAME             CPU(cores)   MEMORY(bytes)   
phpapp-1-6sj2z   3m           66Mi
phpapp-1-9s9lp   3m           59Mi
phpapp-1-lddgd   2m           57Mi
phpapp-1-s8z9z   3m           63Mi
```
## HPA 를 이용한 Auto-Scaling 구현
PHP Source 를 이용해서 세션 접속을 할 때, 부하가 발생되는 PHP APP 을 배포하도록 하겠습니다.   
```php
<?php
  $x = 0.0001;
  for ($i = 0; $i <= 1000000; $i++) {
    $x += sqrt($x);
  }
  echo "OK!";
?>
```
Dockerfile 도 생성합니다.      
```docker
FROM php:5-apache
ADD index.php /var/www/html/index.php
RUN chmod a+rx index.php
```
위 소스들을 이용하여 PHP APP 을 배포합니다.   
   
```bash
$ oc login -u developer
$ oc new-project hpa-project
```
신규 Project 생성합니다.   
   
```bash
$ oc login -u admin                                 << SCC 추가를 위해 
$ oc adm policy add-scc-to-user anyuid -z default   << SCC 추가
```
SCC 추가 작업 진행합니다.   
   
```bash
$ oc login -u developer
$ oc new-build --name=hpa-app --binary=true
$ oc start-build hpa-app --from-dir=.
$ oc new-app hpa-app
$ oc expose service hpa-app
$ oc set resources deployment hpa-app --limits=cpu=200m --requests=cpu=100m
```
신규 PHP APP 배포합니다.   
```bash
$ curl hpa-app-hpa-project.apps.ocp.chhan.com
```
APP 이 정상적으로 배포 되었습니다.   

## HPA 생성
```bash
$ oc autoscale deployment hpa-app --min=1 --max=10 --cpu-percent=20
```
위와 같이 HPA 를 생성하고 HPA 를 통해 `hpa-app` 이 metric 정보를 받아올때까지 약간의 시간이 필요합니다.   
모니터링이 진행되면 아래와 같이 HPA 를 확인 할 수 있습니다.   
```bash
$ oc get hpa -w
NAME      REFERENCE            TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
hpa-app   Deployment/hpa-app   <unknown>/20%   1         10        0          8s
hpa-app   Deployment/hpa-app   27%/20%         1         10        2          5m3s   
```
## load generator 생성
동일한 namespace 에 Load generator 를 생성합니다.   
```bash
$ oc run -it --rm load-generator --image=busybox /bin/sh
```
   
```bash
while true; do wget -q -O- http://hpa-app; done
```
위 명령을 입력하여 부하 발생을 시작합니다.   

## HPA 동작
```bash
$ oc get hpa
NAME      REFERENCE            TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
hpa-app   Deployment/hpa-app   31%/20%   1         10        6          7m25s
$ oc get pod
NAME                      READY   STATUS      RESTARTS   AGE
hpa-app-1-build           0/1     Completed   0          14m
hpa-app-1-deploy          0/1     Completed   0          13m
hpa-app-2-cp74l           1/1     Running     0          13m
hpa-app-2-deploy          0/1     Completed   0          13m
hpa-app-2-h6xdl           1/1     Running     0          2m45s
hpa-app-2-hvk4g           1/1     Running     0          74s
hpa-app-2-mbz8v           1/1     Running     0          104s
hpa-app-2-wwhj8           1/1     Running     0          2m15s
hpa-app-2-zgntw           1/1     Running     0          43s
load-generator-1-deploy   0/1     Completed   0          3m53s
load-generator-1-kknll    1/1     Running     0          3m50s
```
   
![HPA-1](/asset/hpa/1.png)   
![HPA-2](/asset/hpa/2.png)   
![HPA-3](/asset/hpa/3.png)   
![HPA-4](/asset/hpa/4.png)   

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
$ oc new-build php --name phpapp --binary=true
$ oc start-build phpapp --from-file=index.php
$ oc new-app phpapp
$ oc expose service phpapp
$ oc scale deploymentconfig --replicas=4 phpapp
```
   
```bash
$ oc get pod -o wide
$ oc delete pod phpapp-X-XXXXX
```
   
```bash
$ oc get pod -o wide
```
   
## Node 단위
Worker node 장애 발생 테스트를 해보도록 하겠습니다.   
   
```bash
$ oc get pod -o wide | grep -v Comp
NAME              READY   STATUS      RESTARTS   AGE     IP             NODE                    NOMINATED NODE   READINESS GATES
phpapp-1-675qm    1/1     Running     0          3h43m   10.131.0.124   worker1.ocp.chhan.com   <none>           <none>
phpapp-1-9s9lp    1/1     Running     0          3h43m   10.128.2.127   worker2.ocp.chhan.com   <none>           <none>
phpapp-1-ml9fp    1/1     Running     0          3h43m   10.131.0.123   worker1.ocp.chhan.com   <none>           <none>
phpapp-1-s8z9z    1/1     Running     0          3h44m   10.128.2.126   worker2.ocp.chhan.com   <none>           <none>
```
   
`worker1.ocp.chhan.com` 장애가 발생되는 경우,   
```bash
$ oc get nodes
NAME                    STATUS     ROLES    AGE   VERSION
master1.ocp.chhan.com   Ready      master   21d   v1.17.1
master2.ocp.chhan.com   Ready      master   21d   v1.17.1
master3.ocp.chhan.com   Ready      master   21d   v1.17.1
worker1.ocp.chhan.com   NotReady   worker   21d   v1.17.1
worker2.ocp.chhan.com   Ready      worker   21d   v1.17.1
```
Pod Self-healing 진행,   
```bash
oc get pod -o wide
NAME              READY   STATUS        RESTARTS   AGE     IP             NODE                    NOMINATED NODE   READINESS GATES
phpapp-1-675qm    1/1     Terminating   0          3h52m   10.131.0.124   worker1.ocp.chhan.com   <none>           <none>
phpapp-1-6sj2z    1/1     Running       0          42s     10.128.2.132   worker2.ocp.chhan.com   <none>           <none>
phpapp-1-9s9lp    1/1     Running       0          3h52m   10.128.2.127   worker2.ocp.chhan.com   <none>           <none>
phpapp-1-build    0/1     Completed     0          3h54m   10.128.2.124   worker2.ocp.chhan.com   <none>           <none>
phpapp-1-deploy   0/1     Completed     0          3h53m   10.128.2.125   worker2.ocp.chhan.com   <none>           <none>
phpapp-1-lddgd    1/1     Running       0          42s     10.128.2.140   worker2.ocp.chhan.com   <none>           <none>
phpapp-1-ml9fp    1/1     Terminating   0          3h52m   10.131.0.123   worker1.ocp.chhan.com   <none>           <none>
phpapp-1-s8z9z    1/1     Running       0          3h53m   10.128.2.126   worker2.ocp.chhan.com   <none>           <none>
```

장애 복구 절차,   
```bash
$ oc adm cordon <NODE NAME>
```
Worker node 장애 복구 진행,   
```bash
$ oc get nodes
NAME                    STATUS                     ROLES    AGE   VERSION
master1.ocp.chhan.com   Ready                      master   21d   v1.17.1
master2.ocp.chhan.com   Ready                      master   21d   v1.17.1
master3.ocp.chhan.com   Ready                      master   21d   v1.17.1
worker1.ocp.chhan.com   Ready,SchedulingDisabled   worker   21d   v1.17.1
worker2.ocp.chhan.com   Ready                      worker   21d   v1.17.1
```
장애의 문제점이 해결는 동안 `cordon` 옵션을 통해 Pod 이 Scheduling 이 안되도록 설정합니다.   
```bash
$ oc adm uncordon worker1.ocp.chhan.com
node/worker1.ocp.chhan.com uncordoned
```
장애 복구 완료 후, Pod 이 Scheduling 이 가능하도록 `uncordon` 옵션을 이용하여 Scheduling Enabled 을 합니다.   
   
# 참고 자료
* [https://chhanz.github.io/openshift/2019/07/15/okd-hpa/](https://chhanz.github.io/openshift/2019/07/15/okd-hpa/)   
* [https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)   
