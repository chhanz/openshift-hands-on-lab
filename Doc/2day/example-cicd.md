# CI/CD Pipeline 예제
이번 lab 은 CI/CD Pipeline 예제를 통해 실제로 Pipeline 이 어떤식으로 Build 를 하고 Deploy 되는지 확인 해보도록 하겠습니다.   
   
# Clone openshift-cd-demo 
CI/CD Demo 환경을 구성하기 위해 아래와 같이 `git` 명령을 통해 `clone` 을 수행합니다.   
* Demo : [https://github.com/siamaksade/openshift-cd-demo](https://github.com/siamaksade/openshift-cd-demo)   
```bash
$ git clone https://github.com/siamaksade/openshift-cd-demo.git
```
   
# Deploy Demo
아래와 같이 명령을 수행합니다.   
사용될 계정은 `developer` 계정입니다.   
```bash
$ cd openshift-cd-demo/scripts/
$ ./provision.sh deploy --ephemeral
```
   
# Demo
![](/asset/cicd/demo/d1.png)   
+ Gogs: `gogs`/`gogs`   
+ Nexus: `admin`/`admin123`   
+ SonarQube: `admin`/`admin`   
   
# 둘러보기
## Gogs
![](/asset/cicd/demo/gogs.png)   
Github 와 같은 형상 관리 도구 입니다.   
## Nexus
![](/asset/cicd/demo/nexus.png)   
Software Repository 인 Nexus 입니다.   
## SonarQube
![](/asset/cicd/demo/sona.png)   
SonarQube 는 아키텍쳐와 설계, 코드 중복, 단위 테스트, 복잡도, 발생 가능한 잠재적인 버그, 코딩 규칙, 주석의 기준으로 프로젝트의 품질을 관리할 수 있는 오픈소스 정적 분석 도구입니다.   


# Pipeline 
![](https://raw.githubusercontent.com/siamaksade/openshift-cd-demo/ocp-4.3/images/pipeline.svg?sanitize=true)   
   
# Task 1
현재 생성된 Pipeline 을 Build 진행합니다.   
아래와 같이 Pipeline 이 문제없이 Build 진행 될 것 입니다.   
   
![](/asset/cicd/demo/d2.png)   
![](/asset/cicd/demo/d3.png)   
![](/asset/cicd/demo/d4.png)   
![](/asset/cicd/demo/d5.png)   
![](/asset/cicd/demo/d6.png)   
![](/asset/cicd/demo/d7.png)   
![](/asset/cicd/demo/d8.png)   
![](/asset/cicd/demo/d9.png)   
![](/asset/cicd/demo/d10.png)   

# Task 2
Build 과정을 보면 Unit Test 부분이 4개 항목 중에 1개의 항목이 Skip 되는 것을 볼 수 있습니다.   

![](/asset/cicd/demo/task2/t1.png)   
   
아래 Source 를 `IDE` 혹은 기타 편집기를 이용하여 수정을 합니다.   
`src/test/java/org/jboss/as/quickstarts/tasksrs/service/UserResourceTest.java` 파일에서 `@Ignore` 구문을 제거합니다.   

```bash
$ git clone http://gogs-cicd-chhan.apps.ocp.chhan.com/gogs/openshift-tasks.git
$ cd openshift-tasks
$ git checkout eap-7
$ vi src/test/java/org/jboss/as/quickstarts/tasksrs/service/UserResourceTest.java
```   
![](/asset/cicd/demo/task2/t2.png)   
   
Source 변경이 완료된 이후,   
```bash
$ git config --local user.email "gogs@gogs.com"
$ git config --local user.name "gogs"
$ git add .
$ git commit -m "remove ignore"
[eap-7 660868a] remove ignore
 1 file changed, 1 insertion(+), 1 deletion(-)
 
$ git push origin eap-7 
Counting objects: 23, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (7/7), done.
Writing objects: 100% (12/12), 733 bytes | 0 bytes/s, done.
Total 12 (delta 5), reused 0 (delta 0)
Username for 'http://gogs-cicd-chhan.apps.ocp.chhan.com': gogs
Password for 'http://gogs@gogs-cicd-chhan.apps.ocp.chhan.com':
To http://gogs-cicd-chhan.apps.ocp.chhan.com/gogs/openshift-tasks.git
   bce83ce..660868a  eap-7 -> eap-7
```
다시 Build 를 진행합니다.  
   
아래와 같이 Unit Test 에서 Fail 이 발생하는 것을 볼 수 있습니다.   
이에 따라 Pipeline 도 Fail 됩니다.   
   
![](/asset/cicd/demo/task2/t3.png)   
![](/asset/cicd/demo/task2/t4.png)   


# Task 3
Unit Test 가 문제가 되는 부분은 `src/main/java/org/jboss/as/quickstarts/tasksrs/service/UserResource.java` 입니다.   
해당 Source 를 수정합니다. 주석을 제거합니다.   

![](/asset/cicd/demo/task2/t5.png)

이후 변경된 Source 를 gogs 에 `push` 합니다.   
`push` 가 완료가 되면 다시 Build 를 진행 합니다.   
   
# Task 4
APP 이 Build 될 때, 아래와 같이 slave pod 가 생성 되고 maven build 가 진행됩니다.   
```bash
$ oc get pod
NAME                        READY   STATUS      RESTARTS   AGE
cicd-demo-installer-hfkjh   0/1     Completed   0          58m
gogs-1-c5hfg                1/1     Running     0          58m
gogs-1-deploy               0/1     Completed   0          58m
gogs-postgresql-1-deploy    0/1     Completed   0          58m
gogs-postgresql-1-gkbrw     1/1     Running     0          58m
jenkins-2-7qffc             1/1     Running     0          58m
jenkins-2-deploy            0/1     Completed   0          58m
maven-3f2fx                 1/1     Running     0          13s     <<
nexus-1-deploy              0/1     Completed   0          58m
nexus-1-hook-post           0/1     Completed   0          55m
nexus-1-x9knd               1/1     Running     0          58m
sonardb-2-569cj             1/1     Running     0          58m
sonardb-2-deploy            0/1     Completed   0          58m
sonarqube-1-deploy          0/1     Completed   0          58m
sonarqube-1-wtt2l           1/1     Running     1          58m
```

![](/asset/cicd/demo/task2/t6.png)
위와 같이 모든 Unit Test 를 Pass 하고 DEV 배포 및 STAGE 배포가 완료 되었습니다.   
   
   
# 참고 자료
* [https://github.com/siamaksade/openshift-cd-demo](https://github.com/siamaksade/openshift-cd-demo)   
