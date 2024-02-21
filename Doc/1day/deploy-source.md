# Deploying Applications From Source
이번 Lab 은 Source 를 이용하여 App 을 배포 하도록 하겠습니다.   

# PHP WebApp 배포
> Web Console 로 `developer` 계정으로 로그인합니다.   

<img src="/asset/o/src-1.png" style="max-width: 95%; height: auto;">Project 생성</br></p>
<img src="/asset/o/src-2.png" style="max-width: 95%; height: auto;"></br></p>
<img src="/asset/o/src-3.png" style="max-width: 95%; height: auto;">소스 주소 : https://github.com/chhanz/docker-swarm-demo.git</br></p>
<img src="/asset/o/src-4.png" style="max-width: 95%; height: auto;">PHP Version 선택</br></p>
<img src="/asset/o/src-5.png" style="max-width: 95%; height: auto;"></br></p>
<img src="/asset/o/src-6.png" style="max-width: 95%; height: auto;"></br></p>
<img src="/asset/o/src-7.png" style="max-width: 95%; height: auto;"></br></p>
<p align="center"><img src="/asset/o/src-8.png"><br>original 배포 완료</br></p>
<img src="/asset/o/src-9.png" style="max-width: 95%; height: auto;">Scale Out 수행</br></p>
<p align="center"><img src="/asset/o/src-10.png"><br>Load Balance 확인이 가능합니다.</br></p>
<img src="/asset/o/src-11.png" style="max-width: 95%; height: auto;">Source Version 변경</br></p>
<img src="/asset/o/src-12.png" style="max-width: 95%; height: auto;">Github 에 Source Push 완료</br></p>
   
혹은 아래와 같이 `BuildConfig` 를 수정합니다.   
```
...
  source:
    type: Git
    git:
      uri: 'https://github.com/chhanz/docker-swarm-demo.git'
      ref: "patch-v2"
...
```
   
<img src="/asset/o/src-13.png" style="max-width: 95%; height: auto;">Start Build</br></p>
<img src="/asset/o/src-14.png" style="max-width: 95%; height: auto;"></br></p>
<p align="center"><img src="/asset/o/src-15.png"><br>Version v2 배포 완료</br></p>
   
# S2I 를 이용하여 WildFly 서비스 배포
EJB, JSF, JPA example application 을 이용하여 App 을 배포하도록 하겠습니다.   
   
## 실습용 Source 
* [https://github.com/wildfly/wildfly-s2i/tree/main/examples/jsf-ejb-jpa](https://github.com/wildfly/wildfly-s2i/tree/main/examples/jsf-ejb-jpa)   

## APP 빌드
```bash
$ oc new-project s2i-project
$ git clone https://github.com/wildfly/wildfly-s2i
$ cd wildfly-s2i/examples/jsf-ejb-jpa
   
$ oc new-build wildfly-s2i --name my-app --binary=true
```
`my-app` 이름으로 buildconfig 를 생성합니다.   
   
```bash
$ oc start-build my-app --from-dir=./
Uploading directory "." as binary input for the build ...

Uploading finished
build.build.openshift.io/my-app-1 started
```
`./wildfly-s2i/examples/jsf-ejb-jpa/` example source 경로를 지정하고 build 를 진행합니다.   
   
```bash
$ oc logs -f my-app-1-build
Caching blobs under "/var/cache/blobs".
Getting image source signatures
...
[INFO] Generating configurations
[INFO] Delayed generation, waiting...
[INFO] Copy deployment /tmp/src/target/jsf-ejb-jpa-demo-1.0.war to /tmp/src/target/server/standalone/deployments/ROOT.war
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  05:48 min
[INFO] Finished at: 2024-02-21T13:31:58Z
[INFO] ------------------------------------------------------------------------
[WARNING] The requested profile "openshift" could not be activated because it does not exist.
...
Push successful
```
위와 같이 source 를 build 하고 OpenShift Registry 로 Push 합니다.   
   
```bash
$ oc new-app my-app
```
신규 App 생성합니다.   
   
```bash
$ oc expose service/my-app
route.route.openshift.io/my-app exposed
``` 
route 를 생성합니다.   
   
```bash
$ oc get all

```
위와 같이 서비스가 배포 완료된 것을 볼 수 있습니다.   
