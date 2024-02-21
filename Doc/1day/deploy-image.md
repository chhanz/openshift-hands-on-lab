# Deploying Applications From Images
이번 Lab 은 Container Image 로 생성된 App 을 배포 하도록 하겠습니다.   
   
## Django WebApp 배포
> Web Console 로 `developer` 계정으로 로그인합니다.

<img src="/asset/o/img-1.png" style="max-width: 95%; height: auto;"></p>
* 새로운 Project 를 생성합니다.   
   
<img src="/asset/o/img-2.png" style="max-width: 95%; height: auto;"></p>   
   
<img src="/asset/o/img-3.png" style="max-width: 95%; height: auto;"></p>
* [`openshiftkatacoda/blog-django-py`](https://github.com/openshift-katacoda/blog-django-py) Django WebApp 를 배포합니다.    
* Copy to Paste : `openshiftkatacoda/blog-django-py`   
<img src="/asset/o/img-4.png" style="max-width: 95%; height: auto;"></p>
   
<img src="/asset/o/img-5.png" style="max-width: 95%; height: auto;"></p>
   
<img src="/asset/o/img-6.png" style="max-width: 95%; height: auto;"></p>
   
<img src="/asset/o/img-7.png" style="max-width: 95%; height: auto;"></p>
배포 완료!   
      
## CLI 로 배포
```bash
$ oc login -u developer
$ oc project django-project
$ oc new-app openshiftkatacoda/blog-django-py --name blog-from-image

"expose service for http"
$ oc expose svc/blog-from-image
   
  ... or ... 
   
"expose service for https"
$ oc create route edge --service blog-from-image

$ oc get all
NAME                                  READY   STATUS      RESTARTS   AGE
pod/blog-django-py-6b787ccc9f-hl7tk   1/1     Running     0          16m
pod/blog-from-image-1-74snj           1/1     Running     0          41s
pod/blog-from-image-1-deploy          0/1     Completed   0          45s

NAME                                      DESIRED   CURRENT   READY   AGE
replicationcontroller/blog-from-image-1   1         1         1       45s

NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/blog-django-py    ClusterIP   172.30.139.80   <none>        8080/TCP   16m
service/blog-from-image   ClusterIP   172.30.210.95   <none>        8080/TCP   48s

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/blog-django-py   1/1     1            1           16m

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/blog-django-py-6b787ccc9f   1         1         1       16m
replicaset.apps/blog-django-py-6f84ff6b79   0         0         0       16m

NAME                                                 REVISION   DESIRED   CURRENT   TRIGGERED BY
deploymentconfig.apps.openshift.io/blog-from-image   1          1         1         config,image(blog-from-image:latest)

NAME                                             IMAGE REPOSITORY                                                                  TAGS     UPDATED
imagestream.image.openshift.io/blog-django-py    image-registry.openshift-image-registry.svc:5000/django-project/blog-django-py    latest   16 minutes ago
imagestream.image.openshift.io/blog-from-image   image-registry.openshift-image-registry.svc:5000/django-project/blog-from-image   latest   46 seconds ago

NAME                                       HOST/PORT                                           PATH   SERVICES          PORT       TERMINATION   WILDCARD
route.route.openshift.io/blog-django-py    blog-django-py-django-project.apps.ocp.chhan.com           blog-django-py    8080-tcp                 None
route.route.openshift.io/blog-from-image   blog-from-image-django-project.apps.ocp.chhan.com          blog-from-image   8080-tcp                 None
```
   
## Django WebApp 의 Dockerfile 분석
```docker
FROM centos/python-35-centos7:latest

USER root

COPY . /tmp/src

RUN mv /tmp/src/.s2i/bin /tmp/scripts

RUN rm -rf /tmp/src/.git* && \
    chown -R 1001 /tmp/src && \
    chgrp -R 0 /tmp/src && \
    chmod -R g+w /tmp/src

USER 1001

ENV S2I_SCRIPTS_PATH=/usr/libexec/s2i \
    S2I_BASH_ENV=/opt/app-root/etc/scl_enable \
    DISABLE_COLLECTSTATIC=1 \
    DISABLE_MIGRATE=1

RUN /tmp/scripts/assemble

CMD [ "/tmp/scripts/run" ]
```
   
## 간단한 Container Image 를 Build 해봅시다.
* 참고 자료 : [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)   
```docker
FROM httpd:2.4

COPY index.html /usr/local/apache2/htdocs/

EXPOSE 80
```   
해당 경로에는 [`Dockerfile`](https://raw.githubusercontent.com/chhanz/sample-httpd-example/master/Dockerfile) 및 [`index.html`](https://raw.githubusercontent.com/chhanz/sample-httpd-example/master/index.html) 파일이 필요합니다.   
   
### Build
```bash
$ docker build -t han0495/sample-httpd .
Sending build context to Docker daemon  3.072kB
Step 1/3 : FROM httpd:2.4
2.4: Pulling from library/httpd
afb6ec6fdc1c: Pull complete
5a6b409207a3: Pull complete
41e5e22239e2: Pull complete
9829f70a6a6b: Pull complete
3cd774fea202: Pull complete
Digest: sha256:db9c3bca36edb5d961d70f83b13e65e552641e00a7eb80bf435cbe9912afcb1f
Status: Downloaded newer image for httpd:2.4
 ---> d4e60c8eb27a
Step 2/3 : COPY index.html /usr/local/apache2/htdocs/
 ---> caf363ed04d9
Step 3/3 : EXPOSE 80
 ---> Running in d2052322cca7
Removing intermediate container d2052322cca7
 ---> bf4d56b96457
Successfully built bf4d56b96457
Successfully tagged han0495/sample-httpd:latest
```
### Push
```bash
$ docker login
$ docker push <DockerHub 계정>/sample-httpd
```
   
### Deploy
<img src="/asset/o/img-8.png" style="max-width: 95%; height: auto;"></p>
<img src="/asset/o/img-9.png" style="max-width: 95%; height: auto;"></p>

# 참고 자료
* [https://hub.docker.com/r/han0495/sample-httpd](https://hub.docker.com/r/han0495/sample-httpd)   
* [https://github.com/chhanz/sample-httpd-example](https://github.com/chhanz/sample-httpd-example)   
* [https://chhanz.github.io/openshift/2019/11/25/openshift4-dev-env/](https://chhanz.github.io/openshift/2019/11/25/openshift4-dev-env/)   
