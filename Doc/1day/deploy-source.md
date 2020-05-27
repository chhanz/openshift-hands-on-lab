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
<div style="text-align:center;"><img src="/asset/o/src-8.png" style="max-width: 95%; height: auto;">original 배포 완료</br></p></div>
<img src="/asset/o/src-9.png" style="max-width: 95%; height: auto;">Scale Out 수행</br></p>
<div style="text-align:center;"> <img src="/asset/o/src-10.png" style="max-width: 95%; height: auto;">Load Balance 확인이 가능합니다.</br></p></div>
<img src="/asset/o/src-11.png" style="max-width: 95%; height: auto;">Source Version 변경</br></p>
<img src="/asset/o/src-12.png" style="max-width: 95%; height: auto;">Github 에 Source Push 완료</br></p>
<img src="/asset/o/src-13.png" style="max-width: 95%; height: auto;">Start Build</br></p>
<img src="/asset/o/src-14.png" style="max-width: 95%; height: auto;"></br></p>
<div style="text-align:center;"> <img src="/asset/o/src-15.png" style="max-width: 95%; height: auto;">Version v2 배포 완료</br></p></div>
   
# WAR File 을 이용하여 Jboss 서비스 배포
WAR(Web application ARchive) File 을 이용하여 App 을 배포하도록 하겠습니다.   
   
## 실습용 WAR File 
* [Ticket-Monster War file](/asset/war/ticket-monster.war)   
* [Tomcat Sample War file](/asset/war/sample.war)   
해당 파일을 OpenShift Client 에 Upload 합니다.   

## ROOT 배포
```bash
$ cp ticket-monster.war ROOT.war
$ oc new-project war-project
```
위와 같이 WAR File 을 `ROOT.war` 로 파일명을 변경합니다.   
이후 배포를 위한 신규 Project 를 생성합니다.   
   
```bash
$ oc get is -n openshift | grep jboss| grep tomcat
jboss-webserver30-tomcat7-openshift      image-registry.openshift-image-registry.svc:5000/openshift/jboss-webserver30-tomcat7-openshift      1.1,1.2,1.3                                           8 days ago
jboss-webserver30-tomcat8-openshift      image-registry.openshift-image-registry.svc:5000/openshift/jboss-webserver30-tomcat8-openshift      1.1,1.2,1.3                                           8 days ago
jboss-webserver31-tomcat7-openshift      image-registry.openshift-image-registry.svc:5000/openshift/jboss-webserver31-tomcat7-openshift      1.0,1.1,1.2,1.3,1.4                                   8 days ago
jboss-webserver31-tomcat8-openshift      image-registry.openshift-image-registry.svc:5000/openshift/jboss-webserver31-tomcat8-openshift      1.0,1.1,1.2,1.3,1.4                                   8 days ago
jboss-webserver50-tomcat9-openshift      image-registry.openshift-image-registry.svc:5000/openshift/jboss-webserver50-tomcat9-openshift      1.0,1.1,1.2,latest                                    8 days ago

```
사용할 Image stream 을 확인합니다.   
   
```bash
$ oc new-build jboss-webserver31-tomcat7-openshift:1.4 --name ticketmon --binary=true
--> Found image 4460fe4 (2 months old) in image stream "openshift/jboss-webserver31-tomcat7-openshift" under tag "1.4" for "jboss-webserver31-tomcat7-openshift:1.4"

    JBoss Web Server 3.1
    --------------------
    Platform for building and running web applications on JBoss Web Server 3.1 - Tomcat v7

    Tags: builder, java, tomcat7

    * A source build using binary input will be created
      * The resulting image will be pushed to image stream tag "ticketmon:latest"
      * A binary build was created, use 'oc start-build --from-dir' to trigger a new build

--> Creating resources with label build=ticketmon ...
    imagestream.image.openshift.io "ticketmon" created
    buildconfig.build.openshift.io "ticketmon" created
--> Success

```
`ticketmon` 이름으로 buildconfig 를 생성합니다.   
   
```bash
$ oc start-build ticketmon --from-file=ROOT.war
Uploading file "ROOT.war" as binary input for the build ...
.
Uploading finished
build.build.openshift.io/ticketmon-1 started
```
`ROOT.war` file 을 지정하고 build 를 진행합니다.   
   
```bash
$ oc logs -f ticketmon-1-build
Caching blobs under "/var/cache/blobs".
Getting image source signatures
Copying blob sha256:74793cb6dd7cce7f562e57d1da904970587c384dd3f17154bc59adaa131e98b8
Copying blob sha256:c9ff3e9281bcbcadd57f37cc0e47a4081cc20a091749d7a33d56496a60a2c1be
Copying blob sha256:f897b9608c98d944929bd778316439ac000b43d974c70efb678187e436f095fa
Copying blob sha256:e02ef6f588fe9c0daa3e7425fab3d38a097610fb646cc2e33b0374d83e2bb09a
Copying config sha256:4460fe4f17e3acc41b5d0178c31086907e54c90da8252cd5c03fefb9eb233ecb
Writing manifest to image destination
Storing signatures
Generating dockerfile with builder image image-registry.openshift-image-registry.svc:5000/openshift/jboss-webserver31-tomcat7-openshift@sha256:dbf126eedbebb078ff6f2ba8449079c6922883ea1f039d5d4802405729f341a3
STEP 1: FROM image-registry.openshift-image-registry.svc:5000/openshift/jboss-webserver31-tomcat7-openshift@sha256:dbf126eedbebb078ff6f2ba8449079c6922883ea1f039d5d4802405729f341a3
STEP 2: LABEL "io.openshift.build.image"="image-registry.openshift-image-registry.svc:5000/openshift/jboss-webserver31-tomcat7-openshift@sha256:dbf126eedbebb078ff6f2ba8449079c6922883ea1f039d5d4802405729f341a3"       "io.openshift.build.source-location"="/tmp/build/inputs"
STEP 3: ENV OPENSHIFT_BUILD_NAME="ticketmon-1"     OPENSHIFT_BUILD_NAMESPACE="war-project"
STEP 4: USER root
STEP 5: COPY upload/src /tmp/src
STEP 6: RUN chown -R 185:0 /tmp/src
STEP 7: USER 185
STEP 8: RUN /usr/local/s2i/assemble
INFO S2I source build with plain binaries detected
INFO Copying deployments from . to /deployments...
'/tmp/src/./ROOT.war' -> '/deployments/ROOT.war'
STEP 9: CMD /usr/local/s2i/run
STEP 10: COMMIT temp.builder.openshift.io/war-project/ticketmon-1:3e81e555
time="2020-05-27T02:38:44Z" level=info msg="Image operating system mismatch: image uses \"\", expecting \"linux\""
time="2020-05-27T02:38:44Z" level=info msg="Image architecture mismatch: image uses \"\", expecting \"amd64\""
Getting image source signatures
Copying blob sha256:49577de6730168df9aef3494209d7618ebdbbc631977f4779fa3d4d079d91dd5
Copying blob sha256:d02565babdb971e3fb3066e40796b4b8d8f37a5294a27458895013c6c2513c0e
Copying blob sha256:8661055bc319237ba6dd4932a1b31ee294bce4b7d361f86fcca409b62b056efc
Copying blob sha256:355f694a72e4222f6fb0a0d9a57ffe9db8eb1574e78a9c168481722438585756
Copying blob sha256:4de09fddf83a94c13a0d9ff5cf712210b129ef0dae2adfd279f0dc11a61817e3
Copying config sha256:7478d68f4fcd024223da479bf3a10d76ea04da8beac42cb22e8d217319a461cc
Writing manifest to image destination
Storing signatures
--> 7478d68f4fc
7478d68f4fcd024223da479bf3a10d76ea04da8beac42cb22e8d217319a461cc

Pushing image image-registry.openshift-image-registry.svc:5000/war-project/ticketmon:latest ...
Getting image source signatures
Copying blob sha256:4de09fddf83a94c13a0d9ff5cf712210b129ef0dae2adfd279f0dc11a61817e3
Copying blob sha256:c9ff3e9281bcbcadd57f37cc0e47a4081cc20a091749d7a33d56496a60a2c1be
Copying blob sha256:74793cb6dd7cce7f562e57d1da904970587c384dd3f17154bc59adaa131e98b8
Copying blob sha256:e02ef6f588fe9c0daa3e7425fab3d38a097610fb646cc2e33b0374d83e2bb09a
Copying blob sha256:f897b9608c98d944929bd778316439ac000b43d974c70efb678187e436f095fa
Copying config sha256:7478d68f4fcd024223da479bf3a10d76ea04da8beac42cb22e8d217319a461cc
Writing manifest to image destination
Storing signatures
Successfully pushed image-registry.openshift-image-registry.svc:5000/war-project/ticketmon@sha256:85a63bfe28258b4ccdc5d0a76985fdd5df0a7eca330b7904722c8802b1f1de19
Push successful
```
위와 같이 WAR 파일을 이용하여 build 하고 OpenShift Registry 로 Push 합니다.   
   
```bash
$ oc new-app ticketmon
--> Found image 7478d68 (43 seconds old) in image stream "war-project/ticketmon" under tag "latest" for "ticketmon"

    JBoss Web Server 3.1
    --------------------
    Platform for building and running web applications on JBoss Web Server 3.1 - Tomcat v7

    Tags: builder, java, tomcat7

    * This image will be deployed in deployment config "ticketmon"
    * Ports 8080/tcp, 8443/tcp, 8778/tcp will be load balanced by service "ticketmon"
      * Other containers can access this service through the hostname "ticketmon"

--> Creating resources ...
    deploymentconfig.apps.openshift.io "ticketmon" created
    service "ticketmon" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose svc/ticketmon'
    Run 'oc status' to view your app.
```
신규 App 생성합니다.   
   
```bash
$ oc expose service/ticketmon
route.route.openshift.io/ticketmon exposed
``` 
route 를 생성합니다.   
   
```bash
$ oc get all
NAME                     READY   STATUS      RESTARTS   AGE
pod/ticketmon-1-build    0/1     Completed   0          107s
pod/ticketmon-1-deploy   0/1     Completed   0          22s
pod/ticketmon-1-sd2d5    1/1     Running     0          19s

NAME                                DESIRED   CURRENT   READY   AGE
replicationcontroller/ticketmon-1   1         1         1       22s

NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/ticketmon   ClusterIP   172.30.29.246   <none>        8080/TCP,8443/TCP,8778/TCP   24s

NAME                                           REVISION   DESIRED   CURRENT   TRIGGERED BY
deploymentconfig.apps.openshift.io/ticketmon   1          1         1         config,image(ticketmon:latest)

NAME                                       TYPE     FROM     LATEST
buildconfig.build.openshift.io/ticketmon   Source   Binary   1

NAME                                   TYPE     FROM     STATUS     STARTED              DURATION
build.build.openshift.io/ticketmon-1   Source   Binary   Complete   About a minute ago   51s

NAME                                       IMAGE REPOSITORY                                                         TAGS     UPDATED
imagestream.image.openshift.io/ticketmon   image-registry.openshift-image-registry.svc:5000/war-project/ticketmon   latest   55 seconds ago

NAME                                 HOST/PORT                                  PATH   SERVICES    PORT       TERMINATION   WILDCARD
route.route.openshift.io/ticketmon   ticketmon-war-project.apps.ocp.chhan.com          ticketmon   8080-tcp                 None
```
위와 같이 서비스가 배포 완료된 것을 볼 수 있습니다.   
   
<img src="/asset/o/war-1.png" style="max-width: 95%; height: auto;"></br></p>

## 하위 경로로 배포
`http://example.com/sample` 과 같이 하위 경로로 App 을 배포 할 수 있습니다.    
   
```bash
$ oc new-build jboss-webserver31-tomcat7-openshift:1.4 --name mysample --binary=true
--> Found image 4460fe4 (2 months old) in image stream "openshift/jboss-webserver31-tomcat7-openshift" under tag "1.4" for "jboss-webserver31-tomcat7-openshift:1.4"

    JBoss Web Server 3.1
    --------------------
    Platform for building and running web applications on JBoss Web Server 3.1 - Tomcat v7

    Tags: builder, java, tomcat7

    * A source build using binary input will be created
      * The resulting image will be pushed to image stream tag "mysample:latest"
      * A binary build was created, use 'oc start-build --from-dir' to trigger a new build

--> Creating resources with label build=mysample ...
    imagestream.image.openshift.io "mysample" created
    buildconfig.build.openshift.io "mysample" created
--> Success

$ oc start-build mysample --from-file=sample.war
Uploading file "sample.war" as binary input for the build ...

Uploading finished
build.build.openshift.io/mysample-1 started

$ oc status
In project war-project on server https://api.ocp.chhan.com:6443

http://ticketmon-war-project.apps.ocp.chhan.com to pod port 8080-tcp (svc/ticketmon)
  dc/ticketmon deploys istag/ticketmon:latest <-
    bc/ticketmon source builds uploaded code on openshift/jboss-webserver31-tomcat7-openshift:1.4
    deployment #1 deployed 5 minutes ago - 1 pod

bc/mysample source builds uploaded code on openshift/jboss-webserver31-tomcat7-openshift:1.4
  -> istag/mysample:latest
  build #1 succeeded 1 second ago


2 infos identified, use 'oc status --suggest' to see details.

$ oc new-app mysample
--> Found image e6a3ac3 (41 seconds old) in image stream "war-project/mysample" under tag "latest" for "mysample"

    JBoss Web Server 3.1
    --------------------
    Platform for building and running web applications on JBoss Web Server 3.1 - Tomcat v7

    Tags: builder, java, tomcat7

    * This image will be deployed in deployment config "mysample"
    * Ports 8080/tcp, 8443/tcp, 8778/tcp will be load balanced by service "mysample"
      * Other containers can access this service through the hostname "mysample"

--> Creating resources ...
    deploymentconfig.apps.openshift.io "mysample" created
    service "mysample" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose svc/mysample'
    Run 'oc status' to view your app.

$ oc expose service/mysample
route.route.openshift.io/mysample exposed

$ oc get route
NAME        HOST/PORT                                  PATH   SERVICES    PORT       TERMINATION   WILDCARD
mysample    mysample-war-project.apps.ocp.chhan.com           mysample    8080-tcp                 None
ticketmon   ticketmon-war-project.apps.ocp.chhan.com          ticketmon   8080-tcp                 None
````

배포 과정은 `ROOT.war` 배포와 동일 하며, war 파일을 하위 경로 이름과 동일하게 작성하는 것이 포인트 입니다.   
   
<img src="/asset/o/war-2.png" style="max-width: 95%; height: auto;"></br></p>
   
## 실습 과제
```bash
http://chhan.example.com/                    # ticket-monster APP
                        /sample              # sample APP
                        /ticket-monster      # ticket-monster APP
```
위와 같이 WAR file 을 배포 하는 경우 어떻게 해야될까요?   
