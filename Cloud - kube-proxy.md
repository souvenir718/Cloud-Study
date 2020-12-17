# kube-proxy

  쿠버네티스에서 서비스를 만들었을 때 나오는 ClusterIP나 NodePort로 접근하게 해주는건 **kube-proxy**다. **kube-proxy**는 쿠버네티스 클러스터의 각 노드마다 실행되고 있으면서 클러스터 내부 IP로 연결되기 바라는 요청을 적절한 곳으로 전달해주는 역할을 한다. **kube-proxy**가 네트워크를 관리하는 방법은 **userspace**, **iptables**, **ipvs** 3가지가 있다. userspace - iptables - ipvs 모드순으로 넘어가고 있다.



## userspace 모드

![](data:image/png;base64,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)

  **userspace **모드로 설정하면 위의 그림과 같은 구조를 가지게 된다. 클라이언트에서 서비스IP를 통해서 요청을 하게 되면 **iptables**를 거쳐서 큐브프록시가 요청을 받은 다음에 그 서비스IP가 연결되어야 하는 적절한 포드로 연결을 해준다. 이때 요청을 포드들에 나눠주는 방식은 라운드 로빈이다.

> 라운드로빈 : 시분할 시스템을 위해 설계된 선점형 스케줄링의 하나로서, 프로세스들 사이에 우선순위를 두지 않고 순서대로 시간단위로 CPU를 할당하는 방식의 CPU 스케줄링 알고리즘이다.





## iptables 모드

![](data:image/png;base64,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)

  **iptables** 모드는 위의 그림과 같은 구조를 갖는다. **userspace** 모드와 다른 점은 큐브 프록시는 **iptable**을 관리하는 역할만 하고 직접 클라이언트로부터 트래픽을 받지 않는다. 클라이언트로부터 오는 모든 요청은 **iptables**를 거쳐서 직접 포드로 전달된다. 그래서 **userspace** 모드보다 빠른 성능을 갖는다. **userspace** 모드에서는 포드 하나로의 요청이 실패하면 자동으로 다른 포드로 연결을 재시도하지만 **iptables** 모드에서는 포드 하나로 가는 요청이 실패하면 재시도를 하지 않고 그냥 요청이 실패한다. 이런걸 방지하기 위해서 **readiness Probe** 설정이 잘 되어있어야 한다.

> readiness Probe(준비성 프로브)) : 컨테이너가 요청을 처리할 준비가 되어있는지 여부를 나타낸다. 





## ipvs 모드

![](data:image/png;base64,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)

  **ipvs**(IP Virtual Server) 모드는 리눅스 커널에 있는 L4 로드밸런싱 기술로 Netfilter에 포함되어 있다. 그래서 IPVS 커널모듈이 설치되어 있어야 한다. **ipvs**는 커널스페이스에서 작동하고 데이터 구조를 해시테이블로 저장해서 가지고 있기 때문에 **iptables**보다 빠르고 좋은 성능을 낸다. 그리고 **ipvs**는 더 많은 로드밸런싱 알고리즘을 가지고 있어서 이걸 이용할 수 있다. rr(round-robin), lc(least connection), dh(destination hashing), sh(source hasing), sed(shortest expected delay), nq(never queue) 등의 다양한 로드밸런싱 알고리즘을 제공해준다.






