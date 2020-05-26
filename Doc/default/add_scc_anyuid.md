# Security Context Constraints 설정
## 참고 자료
* [https://cloud.ibm.com/docs/openshift?topic=openshift-openshift_scc&locale=ko](https://cloud.ibm.com/docs/openshift?topic=openshift-openshift_scc&locale=ko)   
* [https://docs.openshift.com/container-platform/4.4/authentication/managing-security-context-constraints.html](https://docs.openshift.com/container-platform/4.4/authentication/managing-security-context-constraints.html)   
## Project 에 `anyuid` 부여   
`anyuid` 란, restricted SCC와 유사한 액세스를 정의하지만 사용자가 UID와 GID로 실행될 수 있게 합니다.   
   
0) Login `developer` 계정   
1) Project 생성   
    ```bash 
    $ oc new-project test-project
    ```   
2) Login `system:admin` 계정   
    ```bash
    $ oc login -u system:admin
    ```      
3) SCC 추가
    ```bash
    $ oc adm policy add-scc-to-user anyuid -z default
    scc "anyuid" added to: ["system:serviceaccount:test-project:default"]
    ```