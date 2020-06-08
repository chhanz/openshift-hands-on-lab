# Deploying Applications From Template
이번 Lab 은 Template 로 생성된 App 을 배포 하도록 하겠습니다.   
   
## Django + pgsql 배포 (no pv)
<img src="/asset/o/temp-1.png" style="max-width: 95%; height: auto;">Project 생성</br></p>
<img src="/asset/o/temp-2.png" style="max-width: 95%; height: auto;"></br></p>
<img src="/asset/o/temp-3.png" style="max-width: 95%; height: auto;">Django + pgsql Ephemeral 선택합니다.</br></p>
<img src="/asset/o/temp-4.png" style="max-width: 95%; height: auto;"></br></p>
<img src="/asset/o/temp-5.png" style="max-width: 95%; height: auto;"></br></p>
<img src="/asset/o/temp-6.png" style="max-width: 95%; height: auto;"></br></p>
<img src="/asset/o/temp-7.png" style="max-width: 95%; height: auto;"></br></p>
<img src="/asset/o/temp-8.png" style="max-width: 95%; height: auto;"></br></p>
<img src="/asset/o/temp-9.png" style="max-width: 95%; height: auto;"></br></p>
   
## Django + pgsql 배포 (Use pv)
* [https://chhanz.github.io/kubernetes/2019/04/15/kubernetes-chapter-2-network-volume/](https://chhanz.github.io/kubernetes/2019/04/15/kubernetes-chapter-2-network-volume/)   
   
위와 같이 Template 를 이용하여 App 를 배포합니다.   
차이점은 PV 를 `cluster-admin` 이 PV 를 생성하고 제공해야됩니다.   
<img src="/asset/o/temp-10.png" style="max-width: 95%; height: auto;"></br></p>
<img src="/asset/o/temp-11.png" style="max-width: 95%; height: auto;"></br></p>
<img src="/asset/o/temp-12.png" style="max-width: 95%; height: auto;"></br></p>

```bash
$ oc get pvc -A
NAMESPACE                  NAME                     STATUS   VOLUME            CAPACITY   ACCESS MODES   STORAGECLASS   AGE
openshift-image-registry   image-registry-storage   Bound    registry-pv0001   100Gi      RWX                           32h
pv-test-project            postgresql               Bound    pv0003            10Gi       RWO                           55s

$ oc get pv
NAME              CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                                             STORAGECLASS   REASON   AGE
pv0001            10Gi       RWO            Recycle          Available                                                                             98m
pv0002            10Gi       RWO            Recycle          Available                                                                             98m
pv0003            10Gi       RWO            Recycle          Bound       pv-test-project/postgresql                                                98m
pv0004            10Gi       RWX            Retain           Available                                                                             98m
pv0005            10Gi       RWX            Retain           Available                                                                             98m
pv0006            10Gi       RWX            Retain           Available                                                                             98m
registry-pv0001   100Gi      RWX            Retain           Bound       openshift-image-registry/image-registry-storage                           32h
```
위와 같이 pvc 가 생성이 되고 pvc 조건에 맞는 pv 가 Bound 됩니다.   

## PV 할당 정보 확인
<img src="/asset/o/temp-13.png" style="max-width: 95%; height: auto;"></br></p>
<img src="/asset/o/temp-14.png" style="max-width: 95%; height: auto;"></br></p>

