# CI Pipeline Tool 설치 & 구성
이번 lab 은 CI/CD 를 위한 `Jenkins` 를 배포해보고 CI Pipeline 을 만들어 보도록 하겠습니다.   
   
# CI/CD 란?
![](/asset/cicd/devops.png)   
CI/CD는 애플리케이션 개발 단계를 자동화하여 애플리케이션을 보다 짧은 주기로 고객에게 제공하는 방법입니다.   
CI/CD의 기본 개념은 지속적인 통합, 지속적인 서비스 제공, 지속적인 배포입니다.   
CI/CD는 새로운 코드 통합으로 인해 개발 및 운영팀에 발생하는 문제(일명 "인테그레이션 헬(integration hell)")을 해결하기 위한 솔루션입니다.   
   
특히, CI/CD는 애플리케이션의 통합 및 테스트 단계에서부터 제공 및 배포에 이르는 애플리케이션의 라이프사이클 전체에 걸쳐 지속적인 자동화와 지속적인 모니터링을 제공합니다.   
이러한 구축 사례를 일반적으로 "CI/CD 파이프라인"이라 부르며 개발 및 운영팀의 애자일 방식 협력을 통해 지원됩니다.   
* 참고 자료: [https://www.redhat.com/ko/topics/devops/what-is-ci-cd](https://www.redhat.com/ko/topics/devops/what-is-ci-cd)   
   
# Jenkins 배포
![](/asset/cicd/pipeline.png)   
+ Example Pipeline Diagram   
   
![](/asset/cicd/ci-1.png)   
+ Project 생성   
![](/asset/cicd/ci-2.png)   
+ Template 를 이용한 `Jenkins` 배포   
![](/asset/cicd/ci-3.png)   
+ 배포 완료   
![](/asset/cicd/ci-4.png)   
+ `Jenkins` 의 SSO 가 OpenShift 와 연동되어 계정 통합 관리가 가능합니다.   
![](/asset/cicd/ci-5.png)   
+ `Jenkins` Dashboard   
   
# Buildconfig 생성
OpenShift 에서 `JenkinsPipeline` Type 의 `Buildconfig` 를 생성해야됩니다.   
그래야지 OpenShift 에서 Build 과정이 연동되어 Web Console 에서 확인이 가능합니다.   
***참고 : Jenkins 사용이 더 익숙하다면 Jenkins 의 Pipeline 을 이용하여 구성하면 됩니다.***   
   
```yaml
apiVersion: v1
kind: List
metadata: {}
items:
- apiVersion: v1
  kind: BuildConfig
  metadata:
    annotations:
      pipeline.alpha.openshift.io/uses: '[{"name": "jenkins", "namespace": "", "kind": "DeploymentConfig"}]'
    name: test-pipeline
  spec:
    strategy:
      type: JenkinsPipeline
      jenkinsPipelineStrategy:
        jenkinsfile: |-
          pipeline {
                    agent none
                              stages {
                                        stage ('Clone') {
                                                  steps {
                                                            echo 'clone source...';sleep 5
                                                  }
                                        }
                                        stage ('Build') {
                                                  steps {
                                                            echo 'shell scripts to build project...';sleep 5
                                                  }
                                        }
                                        stage ('Tests') {
                                                  steps {
                                                  parallel 'static': {
                                                            echo 'shell scripts to run static tests...';sleep 6
                                                  },
                                                  'unit': {
                                                            echo 'shell scripts to run unit tests...';sleep 10
                                                  },
                                                  'integration': {
                                                            echo 'shell scripts to run integration tests...';sleep 3
                                                  }
                                        }
                              }
          }
          }
```
위와 같이 `Buildconfig` yaml 을 생성하여 테스트를 진행합니다.   
   
```bash
$ oc create -f test-pipeline.yml
```
# CI 과정
![](/asset/cicd/bc-1.png)   
![](/asset/cicd/bc-2.png)   
![](/asset/cicd/bc-3.png)   
![](/asset/cicd/bc-4.png)   
![](/asset/cicd/bc-5.png)   
      
# Slack 연동
![](/asset/cicd/slack-1.png)   
![](/asset/cicd/slack-2.png)   
![](/asset/cicd/slack-3.png)   
![](/asset/cicd/slack-4.png)   
![](/asset/cicd/slack-5.png)   
![](/asset/cicd/slack-6.png)   
![](/asset/cicd/slack-7.png)   
![](/asset/cicd/slack-8.png)   
![](/asset/cicd/slack-9.png)   
![](/asset/cicd/slack-10.png)   

```yaml
post {
          success {
                    slackSend channel: 'ci-notification', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})", tokenCredentialId: 'slack-token'
                  }
          failure {
                    slackSend channel: 'ci-notification', message: 'Failed CI', tokenCredentialId: 'slack-token'
                  }
          }
```
![](/asset/cicd/slack-11.png)   
