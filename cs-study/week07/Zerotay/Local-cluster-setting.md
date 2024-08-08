# 개요
인턴 전형 과제~~를 빙자한 해보고 싶은 거 다 해보기~~
## 문제
- 쿠버네티스 인프라 환경 구성 및 구성도 작성
- 필수사항
	- 헬스체크 가능한 CRUD API 서버
	- 디비 자유
	- [[Terraform]]을 통한 인프라 구성
- 우대사항
	- [[Rust]]를 사용한 api 서버 
	- [[Helm]]을 통한 배포
## 제출
- 서버 레포지토리, 환경 구성 레포지토리 링크
## 목표
- 실현 가능성으로는 어차피 망함
	- 교육과 프로젝트도 병행 중이라 시간을 많이 투자할 수 없음
	- 테라폼은 영상으로만 본 정도
	- 쿠버네티스는 기본 개념만 익힌 정도
	- 러스트는 취업 이후로 미룸
- 그럼 그냥 과제 빌미로 하고 싶은 거 다 해보자.
- 여태 시간을 빌미로 시도하지 못했던 것들, 도전만 해보기
### 기술적 목표
- 테라폼을 통한 [[프로비저닝]]
- 기본적인 수준의 쿠버네티스 아키텍처 세우고 실제로 빌드하기
- 클러스터 모니터링
- 헬름 차트?
	- 아직 매니페스트 방식에 완벽히 숙달되지 않았으므로 이번에는 시도하지 않는다.
### 운영적 목표
- 자동화된 프로비저닝
	- [[Infrastructure As a Code]]
	- 기록이 아닌 정의를 통한 환경 관리
- 클러스터 내 고가용성
	- 자가 회복 능력 확보
		- 테스트 진행하면 좋을 듯
	- 오토스케일링 
	- 분산 파드 배치
- 이상 탐지 및 알림 연동
	- [[Grafana]] 활용
	- 위기 감지 및 장애 대응성 확보
- 배포 전략 
	- 카나리 배포
	- 무중단 배포를 통한 서비스 불가용 시간 단축
	- 롤백 기능 도입
- 디비 안전 배치
	- 분산 디비 동기화
	- stateful하고, 백업 보장
- 시크릿 관리

# 설계
## 환경
- 금액을 고려하여 노트북에서 진행
	- 사양 이슈로 하나의 마스터 플레인, 3개의 워커 노드
- [[Vagrant]]로 [[VirtualBox]] 클러스터 환경 구축
- 운영 환경을 고려하여 kubectl은 호스트 환경에서 명령할 수 있도록 세팅
## 아키텍처
![[api-server-kube.jpg]]
- 노드마다 api 서버 분산 배치
	- [[DaemonSet]]을 활용해 최소 하나를 보장하여 가용성 확보
	- [[ConfigMap]]을 통해 배포 환경의 환경변수 통일 관리
- 데이터베이스 -> 구현 실패
	- mysql operator 활용
	- 호스트 환경에 nfs 서버를 배치하고 동적 프로비저닝을 위한 스토리지 클래스로 활용
- 통신  -> 구현 실패
	- nginx ingress controller 활용
	- 외부에서 `api`엔드포인트를 넣어서 보내면 내부 api 서버로 rewrite되도록

# vagrant 세팅
기본적으로 쿠버네티스의 노드 세팅과, 각 역할의 노드에 필요한 모듈들을 설치한다.
```sh
cat << "EOF" | sudo tee /etc/modules-load.d/kubernetes.conf
br_netfilter
EOF
sudo modprobe br_netfilter

```
넷필터를 통해 브릿지 네트워크 설정을 진행할 수 있도록 활성화하는 설정.
넷필터를 동적으로 로드한다.
참고로 넷필터 설정을 거는 이유는 브릿지 네트워크를 iptables에서 추적할 수 있도록 하기 위함이라고 한다.
원래는 모든 네트워크 통신을 iptables가 관리했으나 rhel 5버전? 이후부터 firewalld가 브릿지 통신을 가져갔다고 한다.
그런데 클러스터 환경은 이미 iptables를 기반으로 돌아가기에, 이러한 설정이 부득이하게 추가됐다고 한다.
브릿지라고 해서 무슨 노드끼리 정말 저수준에서 통신하나 싶지만, 실질적으로는 ip 주소를 가진 터널을 뚫는다고 한다.
이게 강사님 표현인데, 나는 네트워크에 자신이 없어서 완벽하게 이해했다고 말하기가 조금 어렵다.
ip를 가지지 않는 네트워크의 영역이 잘 상상이 안 가서 그렇다.
그런데 이 브릿지가 가지는 역할 중 하나가 그거라는 것은 알겠다.
브릿지를 통한 설정을 진행하면 모든 노드가 같은 대역의 ip로 통신이 가능해진다는 것이다.
그 좋은 예가 바로 클러스터 ip이다.
전부 하나의 프라이빗 망에 놓인 것처럼 구축하는 기술로서 브릿지가 사용된다는 것 같다.
```sh
swapoff -a && sed -i ‘/ swap / s/^/#/’ /etc/fstab
```
메모리 스왑을 막아둔다.
이건 kubelet 현재 노드의 상황을 제대로 파악하는 것에 방해가 되기 때문이라고 들었다.
```sh
cat <<-'EOF' | sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
```
커널에 네트워크 설정을 한다.
브릿지 네트워크를 탈 경우 전부 iptables의 설정을 받도록 설정한다.
패킷 포워딩도 활성화한다.
```sh
sudo apt update
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository -y "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y containerd.io
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```
기본 업데이트와 도커 저장소에서 [[containerd]] 설치.
tee 명령어는 디렉토리가 없으면 발동되지 않으니 중간 과정을 넣어준다.
[[cgroups]]가 systemd를 활용하도록 만든다.
https://www.slideshare.net/slideshow/systemd-cgroup/250042237
아주 흥미로운 이유가 있다.
```sh
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
마지막으로 실질적인 쿠버네티스 도구들을 설치한다.
가장 최근 버전인 1.30을 활용한다.
kubeadm를 통해 세팅을 진행한다.
![[Pasted image 20240718190524.png]]
https://dev.to/sherpaurgen/detected-that-the-sandbox-image-registryk8siopause38-of-the-container-runtime-is-inconsistent-with-that-used-by-kubeadm-1glc
설치 중 발생한 워닝.
컨테이너 런타임이 구닥다리라고 구박한다.
https://www.linuxtechi.com/install-kubernetes-on-ubuntu-22-04/
우분투 focal 버전을 사용하려는 게 문제일 가능성이 있다고 판단하여 jammy 버전으로 업그레이드
이것 때문인지 api 서버가 켜지지 않는데, 로그를 더 확인해보니 etcd가 제대로 깔리지 않았다.
![[Pasted image 20240718192909.png]]
해결했다.
하지만 버전 워닝은 사실 계속 뜨고 있고, 실제 문제는 마스터 노드의 ip 주소를 잘못 넣은 이슈였다.
etcd가 초기화되지 않은 이슈와의 연관성은 확실하지 않다.
![[Pasted image 20240718235402.png]]
성공
![[Pasted image 20240718235437.png]]
클러스터 초기화를 위한 세팅을 한 파일에 몰아넣고, 각 노드마다 특정한 세팅을 각 파일에 넣어주었다.
Vagrantfile에서 반복문을 쓰는 방법도 가능은 했으나, 그러려면 인라인으로 각종 세팅을 전달해주어야만 했기에 나누는 것이 조금 더 보기 좋다고 판단했다.
이렇게 하면 Vagrantfile를 이쁘게 만들 수 있고, 사용법이 달라지는 것도 없다.
```
cp -R ./.kube ~/
```
간단하게 내 로컬에서 접근할 수 있도록 세팅한다.
# 기본 세팅 with kubernetes
세팅에 필요한 각종 yaml 파일을 만든다.
먼저 api 서버의 사양과, 통신을 위한 서비스 사양이 필요하다.
![[Pasted image 20240719144829.png]]
계층에 따라 [[Namespace]]를 분리했다.
## api server
문서의 기본 템플릿을 사용한 후 `--dry-run` 옵션으로 커스텀할 템플릿을 구성한다.
![[Pasted image 20240719135552.png]]
간단하게 세팅하고 올려봤지만 원하는대로 반영되지 않는다.
![[Pasted image 20240719135640.png]]
도커로는 문제 없이 pull 가능했다.
https://waspro.tistory.com/570
![[Pasted image 20240719142051.png]]
이쪽과는 상관 없이 버전을 명시하지 않아 생긴 문제였다.
## 데이터베이스
- 구현 방법
	- [[PersistentVolume]]
		- 영구 볼륨과 영구 볼륨 클레임을 활용한다.
		- 이것 역시 분산 배치를 하도록 세팅한다.
		- 만들어진 mysql 프로비저너를 활용한다. 
	- [[Operator]]
		- 오퍼레이터 패턴은 더 높은 추상화 영역에서 고가용성, 백업, 자동 복구 등의 기능을 제공한다.
		- 마치 사람이 직접 했어야 할 일을 특정 태스크들에 한하여 추상화해준다는 느낌
		- 특히 디비는 장애가 발생해서 프로세스가 망가지면 블록 스토리지 단위로 문제가 나타나게 되는데 이를 해결하기 위해서 주기적인 관리는 필수적이다.
		- 기본적으로 클러스터 환경에서는 커스텀 리소스로서 관리된다.
		- mysql에서는 본인들이 따로 오퍼레이터를 제공하고 있기 때문에 이것을 이용해본다.
		- 이를 통해 이노디비 클러스터를 구축해본다.
### mysql 오퍼레이터
https://dev.mysql.com/doc/mysql-operator/en/mysql-operator-introduction.html
![[Pasted image 20240719155900.png]]
문서를 보면서 차근차근 진행해본다.
```sh
kubectl apply -f https://raw.githubusercontent.com/mysql/mysql-operator/trunk/deploy/deploy-crds.yaml
kubectl apply -f https://raw.githubusercontent.com/mysql/mysql-operator/trunk/deploy/deploy-operator.yaml
kubectl get deployment mysql-operator --namespace mysql-operator
```
필요한 디플로이먼트를 설치되게 만든다.
![[Pasted image 20240719170924.png]]
오퍼레이터가 제대로 동작하지 않고 있다.
도메인 이름을 제대로 획득하지 못하는 이슈가 있다.
![[Pasted image 20240719171623.png]]
그냥 직접 브라우저에서 접속해봤을 때는 이렇게 뜬다.
그렇다고 이게 해결책은 아닐 것 같은데, 조금만 더 파보자.
https://utest.co.kr/303
![[Pasted image 20240719172925.png]]
최소한 내부에 문제가 있는 것으로 해석해도 되려나?
https://github.com/kelseyhightower/kubernetes-the-hard-way/issues/254
https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/
https://d-life93.tistory.com/458
![[Pasted image 20240720145838.png]]
산으로 가는 것 같아 클러스터를 초기화하고 재진행.
돌아가서, 에러 로그에 집중해보도록 한다.
처음에는 가상머신 세팅 중의 이슈가 있지 않을까 했는데, 에러 메시지를 통한 대응이 먼저일 것이라 생각해 이 부분을 먼저 파보려고 한다.
제대로 적용이 되지 않아서 직접적으로 디플로이먼트 객체에 값을 표기했다.
![[Pasted image 20240720150956.png]]
이 부분, 실행되는 파드의 환경 변수에 내가 직접 값을 주입해 넣었다.
![[Pasted image 20240720151119.png]]
동작은 분명 잘하지만, 좋은 방법은 아니라고 생각한다.
https://bugs.mysql.com/bug.php?id=109971
이제야 여기에서 말하는 문제가 무엇인지 이해했는데, 결국 파드 실행 간에 사용되는 클러스터 도메인 이름이 오퍼레이터 개발자들의 하드 코딩 이슈인지 제대로 추적이 안 된 것이다.

혹시나..
https://discuss.kubernetes.io/t/dns-not-resolving-cluster-local/19144
애초에 cluster.local 자체를 못 읽고 있는 이슈일 가능성도 존재한다 생각했지만, 위의 방법으로 해결됐으니 넘어가자.

```sh
kubectl create secret generic mypwds \ 
> --from-literal=rootUser=root \ 
> --from-literal=rootHost=% \ 
> --from-literal=rootPassword="root"
```
다음으로 이노디비 내부 클러스터를 만든다.
이를 위해 루트 유저 세팅을 한다. 
![[Pasted image 20240720152637.png]]
64 인코딩해서 값을 저장해줄 것이다.
https://velog.io/@pinion7/Kubernetes-%EB%A6%AC%EC%86%8C%EC%8A%A4-Secret%EC%97%90-%EB%8C%80%ED%95%B4-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B3%A0-%EC%8B%A4%EC%8A%B5%ED%95%B4%EB%B3%B4%EA%B8%B0
![[Pasted image 20240720160156.png]]
라우터의 희망 상태가 0이다.
파드들의 펜딩도 풀리지 않는다.
![[Pasted image 20240720160325.png]]
기본 3개의 워커노드를 필요로 하는 듯..
![[Pasted image 20240720160645.png]]
뒤늦게 노드를 추가시킨다.
고가용성을 들먹이면서 워커 노드를 2개로 가정하는 것도 따지고 들면 비효율적인 길이긴 한 듯.
![[Pasted image 20240720160842.png]]
바로 노드를 하나 늘렸다.
https://bugs.mysql.com/bug.php?id=107322
그래도 해결은 되지 않았고, 비슷한 에러를 겪은 사람이 많다.
근데.. 오라클에서 에러 내는 사람들에 대해서 일말의 대응법도 알려주지 않았다..
단순히 버전만 올려보라 할 뿐, 가장 최신 버전인 나는 어쩌란 말이지
![[Pasted image 20240720191948.png]]
pv를 직접 만들라는 말인 걸까?
그러면 그건 공식 문서에서 밝혀야 하는 부분 아니냐;
헬름으로 하는 예시가 훨씬 많던데 처음부터 헬름을 썼으면 이런 일 없었으려나. 
![[Pasted image 20240720225815.png]]
기본 스토리지 클래스가 없는 게 문제인 것일까?
## nfs 서버 세팅
그렇다면 nfs를 사용해보도록 한다.
![[Pasted image 20240720204958.png]]
![[Pasted image 20240720205143.png]]
![[Pasted image 20240720211445.png]]
https://superuser.com/questions/310697/connect-to-the-host-machine-from-a-virtualbox-guest-os
게스트 os에서 호스트 os의 nfs에 접근하기 위한 ip를 찾기 위해 편하게 nginx를 host에 설치했다.
![[Pasted image 20240720211552.png]]
정확하게 내가 원하는 nfs가 확인된다.
![[Pasted image 20240720213326.png]]
방화벽이 막힌 듯?
https://community.hpe.com/t5/hp-ux/nfs%EC%97%90%EC%84%9C-%EC%82%AC%EC%9A%A9%EB%90%98%EB%8A%94-%ED%8F%AC%ED%8A%B8/td-p/1165747
찜찜해도 프라이빗에서만 접근가능하기에 열어둔다.
![[Pasted image 20240720213641.png]]
일단 메시지가 달라지는지 확인하기 위해 2049만 풀어본다.
https://jongsky.tistory.com/82
서비스가 죽어있는 것을 확인하고 재가동
![[Pasted image 20240720223936.png]]
이밖에도 호스트쪽 재가동, ip 범위 재설정 등의 다양한 시도를 해봤는데, 결국 성공한 것은 insecure 옵션을 추가하는 것이었다.
정말 에러 이유를 확인하기 편해서 참 참 좋다~
이건 vagrant로 자동 설정하기 쉽지는 않을 것 같다.
nfs 서버 설정을 내 호스트에서도 걸어줘야 가능하기 때문이다.
근데 개인적으로 이건 안 하고 싶은 게 공용 와이파이라도 쓰다가 이걸 끄는 거 깜빡해버리면..
아무리 자동화가 좋아도 위험한 길은 선택하지 않고 싶다.
```sh
apt install -y nfs-common rpcbind
mount -t nfs {my host ip}:{dir} {guess_dir}
```
결국 해야 하는 사항은 이것이다.
![[Pasted image 20240721124202.png]]
호스트 쪽 로그를 확인해보니 들어오는 요청은 A클래스 영역.
뭐..완벽히는 모르겠지만, 그렇다면 이 영역만 풀어주도록 한다.
ip 주소가 특정되지 않는데 어떻게 요청이 돌아갈 수 있는걸까?
브릿지 네트워크라는 이름 하나만으로, 3계층에서부터는 확인할 수 없는 통신의 길이 생기는 것인가?

## nfs 프로비저너 세팅
잠깐 정리하자면, 결국 내가 스토리지 클래스의 개념, 동적 프로비저닝의 개념이 약해서 생긴 문제다.
https://kubernetes.io/docs/concepts/storage/storage-classes/
아직 완벽히 시도하지는 디폴트 sc가 없는 상태이니까 이걸 만들어주는 작업이 필수다.
그리고 나는 디폴트로 nfs를 만들 수 있지 않을까 시도해보고 싶다.
그리고 이걸 자동화할 수 있다면, 나는 내 컴퓨터로 얼마든지 동적 프로비저닝을 수행할 수 있을 것이다.

> [!NOTE] Volume Attributes Classes
> https://kubernetes.io/docs/concepts/storage/volume-attributes-classes/
> 1.29버전부터 Attributes 클래스가 생겼다.
> 속성 클래스?
> 대충 이해했을 때는, 다양한 스토리지 클래스들을 가지고 관리자들은 운용할 텐데, 이것들을 pvc로 사용할 때 속성을 토대로 동적으로 선택할 수 있도록 해준다는 것 같다.
> 동적의 동적인 ㄷㄷ
> 알파 단계라 기본값은 비활성화되어 있다.

공식 문서에서도 다양한 외부 프로비저너를 소개하고 있는데, 그렇다면 한글 자료가 많은 subdir이 아무래도 편하지 않을까 한다.
그렇다면! 난 쿠버네티스 공식 문서에서 추천을 하고 있는 놈을 사용해보려고 한다.
가장 기본적이고 미니멀한 기능을 갖추고 있다는 뜻이 아닐까 해서 그렇다. 
~~힙스터 아님~~

https://github.com/kubernetes-csi/csi-driver-nfs/blob/master/docs/install-csi-driver-v4.8.0.md
![[Pasted image 20240721124544.png]]
원격 nfs 서버에 필요한 설치 진행이라고 생각하고 했는데, 결국 이건 클러스터 내부에서 실행하는 것이다.
api 서버를 조작할 수 있는 환경에서 실행만 하면 되는 듯하다.
결국 api 서버로 각종 설정을 전달하는 방식으로 생각된다.
![[Pasted image 20240721125424.png]]
깜빡하고 디폴트 옵션을 넣지 않았는데, 아무튼 이런 식으로 스토리지 클래스를 생성한다.
![[Pasted image 20240721125535.png]]
```
kubectl create -f https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/deploy/example/pvc-nfs-csi-dynamic.yaml
```
이걸로 간단하게 테스트해볼 수 있고, 위와 같은 결과가 나오면 성공.
![[Pasted image 20240721125602.png]]
공유한 nfs 내부에 성공적으로 디렉토리가 생성된다.
```
kubectl patch storageclass nfs-csi -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```
이렇게 하면 디폴트 스토리지 클래스를 바꿀 수 있다.
![[Pasted image 20240721130423.png]]
https://bugs.mysql.com/bug.php?id=108458
정말 아무런 답도 없는 거냐;;
사이드카가 설정되지 않은 이슈가 있다는 것인데, 이건 어떻게 하라는 걸까?
![[Pasted image 20240721131619.png]]
아직 속단할 수는 없는 게 하나 빼고는 전부 활성화가 됐다.
https://github.com/bitpoke/mysql-operator/issues/416
한 파드만 문제라면 그것만 강제 종료를 시도해보는 것도 답이 될 수 있다.
![[Pasted image 20240721133252.png]]
그러나 계속 문제는 해결되지 않고 있다.
![[Pasted image 20240721151402.png]]
그래서 전부 밀고 다시 세팅했다.
![[Pasted image 20240721154456.png]]
레디네스 프로브에서 진행되지 않는 이유가 dns 리졸빙이 안 돼서?
![[Pasted image 20240721154444.png]]
내부에 존재하는 네트워크 명령어 아는 게 curl밖에 없어서 일단 해보는데, 확실히 아예 해석을 못한다.
![[Pasted image 20240721160738.png]]
그냥 모든 dns가 실패한다.
혹시 coredns에 문제가 있는 것은 아닌가?
## 서비스
https://freedeveloper.tistory.com/431
디비와 api 서버 간 연결이 진행되지 않는 관계로 잉그레스 컨트롤러를 설치하려던 계획도 진행할 수 없다.
api 서버가 구동이 되지 않기 때문이다.
# 테라폼 마이그레이션
여태 한 작업을 간단하게 테라폼으로 재현할 수 있을까?
![[Pasted image 20240721205107.png]]
간단하게 옮겨본다.
처음에는 쿠버네티스 순정도 잘 못 다루는데 무슨 테라폼을 쓰는가 했는데, 생각보다 편리한 기능이 많아서 조금 마음에 들었다.
![[Pasted image 20240721205259.png]]
실시간으로 잘 반영이 된다.

한 가지 큰 이슈가 있다고 생각이 들었다.
바로 mysql 오퍼레이터는 crd, 커스텀 리소스라는 것.
https://github.com/jrhouston/tfk8s?tab=readme-ov-file
그런데 역시 누군가는 밟아온 길인 모양이다.
https://honglab.tistory.com/233
이런 방법도 있다고 한다.
둘 다 공식 문서에 나온 내용인데, 전자는 실행파일을 위해 고랭을 설치해야 하는 관계로 후자로 도전해본다.
![[Pasted image 20240721221442.png]]
.. 사실 별개의 yml 파일이 합쳐진 경우에 대해서 동작하지 않는다는 경고를 보기는 했다.
그럼에도 기왕이면 했던 것이, 파일 버전 관리를 용이하게 하고 싶었기 때문이다.
파일 변환기를 따로 사용하면 해당 버전이 남아버리게 되고, 이는 이후 업데이트에 추가 고려 요소가 된다. 
업데이트를 할 때마다 매번 변환 작업이 동반돼야 한다는 것이다.


# 정리
위에 성공사항들이 나열됐기에, 실패 사항을 짚는다.
## 실패 사항
- 디비 세팅
	- 마지막 관문.. 로그 사이드카 컨테이너의 dns가 지속적으로 리졸빙되지 않았고, 이것이 파드를 계속 레디 상태로 만들어 그 무엇도 동작하지 않게 됐다.
	- 조금 더 강해져서 재도전해보고 싶다.
- 네트워크 세팅
	- nginx ingress를 두고 싶었는데 못 뒀다.
- 테라폼
	- 지금까지 한 것만이라도 완벽하게 옮기고 싶었는데 너무 늦게 잡아서 그런가 조금 어려웠다.
	- crd를 쉽게 옮기는 방법에 대한 고민이 조금 부족하다.
	- 또 어떻게 파일 구조를 가져가는 게 좋을지에 대한 고민도 필요할 것 같다.
	- 한 시간 딸깍 도전으로 이 정도면 일단 만족하고 넘어가련다.
	- 어차피 순정 쿠버 완벽히 다루기 전까지 다른 것 안 쓰겠다는 생각에는 변함이 없다.
- 모니터링
	- 디비에서 막힌 이후로 실질적으로 아무것도 하지 못 했다.
# 짤막 소감
처음으로 쿠버네티스를 직접 써봤는데, 생각보다 할 수 있는 게 많았고, 그래서 어려웠지만 그래서 재밌었다.
결과가 발표된 이후에는 처음 목표로 잡은 클러스터 환경 구축을 계속 시도해보겠다.
# 참고 
- 온프렘 쿠버네티스 고려사항
	- https://blog.doctor-cha.com/on-premise-kubernetes-production-environment-10-month-operation-review
- ingress controller
	- https://velog.io/@dojun527/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EC%9D%B8%EA%B7%B8%EB%A0%88%EC%8A%A4Ingress#%EC%8B%A4%EC%8A%B5-%EA%B5%AC%EC%84%B1%EB%8F%84
- 오퍼레이터
	- https://kubernetes.io/docs/concepts/extend-kubernetes/operator/
	- https://nangman14.tistory.com/79
	- https://velog.io/@kubernetes/MySQL-Operator-for-Kubernetes
	- https://komodor.com/learn/kubernetes-operator/
- dns
	- https://stackoverflow.com/questions/54154112/kubernetes-granting-rbac-access-to-anonymous-users-in-kube-dns
	- https://d2.naver.com/helloworld/2905424
	- 
