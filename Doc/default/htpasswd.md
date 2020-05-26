# OpenShift `htpasswd` Identity Provider 설정

## kubeadmin 계정 확인
```bash
[root@ocp-bastion www]# openshift-install --dir=baremetal wait-for install-complete
INFO Waiting up to 30m0s for the cluster at https://api.ocp.chhan.com:6443 to initialize…
INFO Waiting up to 10m0s for the openshift-console route to be created…
INFO Install complete!
INFO To access the cluster as the system:admin user when using 'oc', run 'export KUBECONFIG=/exports/www/auth/kubeconfig'
INFO Access the OpenShift web-console here: console-openshift-console.apps.ocp.chhan.com
INFO Login to the console with user: kubeadmin, password: LvPJP-svwLz-pfpzS-Rr3Bg
```   
위와 같이 설치가 종료된 이후에 console 접근을 위해 생성된 `kubeadmin` 계정을 이용하여 console 에 접근합니다.   
   
참고로 `openshift-install` 명령을 수행하던 경로 하위에 `auth/kubeadmin-passwd` 로도 확인이 가능합니다.   
```bash
[root@ocp-bastion www]# cat auth/kubeadmin-password
LvPJP-svwLz-pfpzS-Rr3Bg
```   
   
## htpasswd 생성
* [Document - Configuring an HTPasswd identity provider](https://docs.openshift.com/container-platform/4.4/authentication/identity_providers/configuring-htpasswd-identity-provider.html)   
   
* For RHEL/CentOS
    ```bash
    $  yum -y install httpd-tools    
    ```   
   
+ Procedure   
Create or update your flat file with a user name and hashed password:
    ```base
    $ htpasswd -c -B -b </path/to/users.htpasswd> <user_name> <password>
    ```
    The command generates a hashed version of the password.   
       
    For example:
    ```base
    $ htpasswd -c -B -b users.htpasswd admin P@ssw0rd
    Adding password for user admin
    ```
    Continue to add or update credentials to the file:
    ```base
    $ htpasswd -B -b </path/to/users.htpasswd> <user_name> <password>
    ```
   
위와 같은 방식으로 `admin` 계정과 `developer` 계정을 생성합니다.   
   
## htpasswd Secret 생성
* Procedure   
    Create an OpenShift Container Platform Secret that contains the HTPasswd users file.
    ```bash
    $ oc create secret generic htpass-secret --from-file=htpasswd=</path/to/users.htpasswd> -n openshift-config
    ```
## htpasswd CR 생성
```yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: my_htpasswd_provider 
    mappingMethod: claim 
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpass-secret 
```
위와 같이 `yaml` 파일을 작성 후, config 를 apply 합니다.  
```bash
$ $ oc apply -f </path/to/CR>
```

## Web Console 사용
> Navigate to **Administration** → **Cluster Settings**.    
> Under the **Global Configuration** tab, click **OAuth**.    
> Under the **Identity Providers** section, select your identity provider from the **Add** drop-down menu.    

<img src="/asset/d/1.png" style="max-width: 95%; height: auto;"></p>
   
<img src="/asset/d/2.png" style="max-width: 95%; height: auto;"></p>
   
<img src="/asset/d/3.png" style="max-width: 95%; height: auto;"></p>
위와 같이 생성된 `htpasswd` 파일을 선택하여 추가합니다.   

   
