---  
noteType: published  
created: 2025-02-21 21:32  
tags:  
  - published  
  - study/homo-ssafyens  
  - group/aws/ec2  
dg-publish: true  
is-folder: false  
aliases:   
related: "[[Homo SSAFYens]]"  
webAddress:  
---  
# 개요  
이번 시간에는 간단하게 AWS의 기본이 되는 컴퓨팅 리소스 EC2와 그에 관련된 개념들을 간단하게 알아본다.  
ec2를 하나의 서버로서 활용하는데 다뤄야 하는 개념과 기술을 실습 위주로 따라가면서 살펴보도록 한다.  
최대한 가시적인 설정을 하기 위해 기본적으로 콘솔에서 실습을 진행하나, 궁극적으로는 테라폼으로 전부 코드화시킬 수도 있다.  
# 사전 세팅  
본격적으로 들어가기에 앞서, 두어 가지 정도의 설정을 미리 시작한다.  
보안을 위한 기본적인 설정이자, 앞으로 aws를 편하게 다루기 위한 몇가지 툴을 설치할 것이다.  
## IAM 설정  
### 개념  
 
IAM(Identity and Access Managment)은 aws의 리소스를 조작하거나, 활용하기 위해 설정하는 가상의 계정과 정책에 관한 리소스이다.  
위의 그림처럼 어떤 유저가 어떤 리소스에 어떤 조작을 할 수 있는지 지정할 수 있다.  
이런 식의 권한 관리를 RBAC(Role Based Access Control)라 부른다.  
어떤 역할(여기서는 정책)에는 어떤 리소스에 어떤 동작을 할 수 있는지를 지정한다.  
그 이후 주체에게 해당 역할을 지정하면, 해당 주체는 받은 역할 내에서의 동작을 수행할 수 있게 된다.  
참고로 여기에서 주체는 사람이 될 수도 있지만 aws의 다른 리소스가 될 수도 있다.  
  
즉, IAM은 기본적으로는 인가(Authorization)에 대한 조작을 도와주는 리소스이다(인증 관련 기능도 있긴 하다).  
### 설정  
사전 세팅으로서 이를 다루는 이유는 기본적으로 aws의 계정에 접속하면 해당 계정은 root로서, 모든 aws 리소스에 제약없는 권한을 발휘할 수 있기 때문이다.  
이때문에 이 계정이 탈취당하는 순간 매우 큰 보안 문제가 발생하기 때문에 가급적 root 유저를 대신할 수 있는 관리자 권한을 가진 유저를 만들어 운용하는 것이 그나마 적절한 선택이다.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/211ce7a8a3df351e602e8706cccffd93/raw/image.png)  
콘솔에서 iam으로 들어가 본격적으로 유저를 만들어보자.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/1fa969064ff830b05e42e1da1d0c62b8/raw/image.png)  
유저 타입은 iam으로 하는데, 위의 설정은 다중 계정 운영 간에 유용하게 활용하는 기능이다.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/db82b333397c5d5486fe771daad78032/raw/image.png)  
이후 해당 유저에게 어떤 권한을 줄 것인지, 붙일 정책을 지정하게 되는데, 이때 그룹을 만들고 이 그룹에 `AdministratorAccess`를 주도록 한다.  
기본적으로 관리자로서 웬만한 모든 리소스를 건드릴 수 있는 권한을 가지고 있는데, 그래도 root 유저가 가지는, iam 바깥의 권한을 가지고 있지는 않다.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/fd41e0b13a7d4f5dd5cc9d1160a03692/raw/image.png)  
이후 나온 유저로 로그인하면 끝.  
## aws cli 설치  
aws를 콘솔에서 조작할 수도 있지만, 자동화나 세밀한 조작을 위해서는 cli를 이용해서 조작하는 편이 더 좋을 때가 있다.  
```sh  
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"  
unzip awscliv2.zip >/dev/null 2>&1  
./aws/install  
echo "complete -C '/usr/local/bin/aws_completer' aws" >> ~/.bashrc  
source ~/.bashrc  
```  
aws cli와, 자동완성을 적용한다.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/f3cd1c367d605fb3fc660fec51dbc95e/raw/image.png)  
aws cli로 aws를 조작하기 위해서는 당연히 cli도 어떠한 신원을 가지는지 명시해야 하는데, 이를 위해 위에서 만든 유저를 활용한다.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/5820943180c42512dd109a290223ea8b/raw/image.png)  
성공적으로 만들어지고 난 이후에이 키를 cli 설정에 넣어줘야 한다.  
```  
aws configure  
```  
넣으라는 대로 넣으면 된다.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/692010c2590524576e35a88e4de74d3b/raw/image.png)  
이런 식으로 현재 사용자를 검증할 수 있고 입력한 값이 나오면 성공이다.  
# EC2 인스턴스 실행  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/e64d383470ac0863804bb24784d0e745/raw/image.png)  
EC2(Elastic Cloud Compute)는 aws의 가장 기본이 되는 컴퓨팅 리소스다.  
흔히 아는 cpu, 메모리, 디스크 구조로 이뤄지는 폰 노이만 아키텍쳐로 구성된 컴퓨터가 바로 ec2 인스턴스인 것이다.  
사용자는 사용할 물리적 자원의 스펙, 그 위에 실행될 OS, 과금 방식에 대한 다양한 요소들을 설정하여 인스턴스를 실행하고 활용할 수 있다.  
  
먼저 ec2를 콘솔에서 만들어보자.  
## AMI  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/d9df78c1a92374ef9236c13f18b5aa8a/raw/image.png)  
콘솔에서 ec2를 생성할 때 첫번째로 마주하는 것은 AMI이다.  
이것은 물리적 자원 위에 프로비저닝될 OS와 각종 세팅이 된 이미지를 의미한다.  
직접 만드는 것도 가능하고 aws에서 제공하거나 혹은 유명한 os 이미지를 사용하는 것도 가능하다.  
참고로 어떤 ami를 쓰냐에 따라 사용 비용도 달라지는데, 아마존 리눅스를 쓰는 것이 가장 가격이 괜찮다.  
## Instance type  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/d15fcf1a68365473499effc36cf8aa2e/raw/image.png)  
올릴 이미지를 정하고, 실제 물리 자원은 어느 정도의 스펙으로 할 것인지 정할 수 있다.  
![[Pasted image 20240709090043.png]]  
표기법은 대충 이렇게 되는데, 싸게싸게 이용하고 싶다면 t타입, 그외의 범용적으로는 m타입 정도를 쓰는 것 같다.  
사진에도 살짝 나와있지만, 어떤 ami를 사용하는가에 따라 책정되는 비용이 달라진다.  
  
유형을 잘 고르는 것은 꽤나 중요한 작업이다.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/7a1217be30eb1a6e729bbc88588c1da2/raw/image.png)  
대체로 cpu와 메모리 개수만으로 인스턴스의 스펙이 결정되는 것 같지만, 스펙이 커질 때마다 보장되는 각종 성능이 더 높아진다.  
디스크 작업이 많이 일어나는 어플리케이션을 활용하는 상황이라면, diskIO가 매우 빠른 nvme가 부착된 환경을 이용하는 것이 적절할 것이다.  
## 키 타입  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/b98a1f45e4ffad78bbd153b003b2a493/raw/image.png)  
ssh 키페어를 설정하는 부분으로 우측 버튼을 눌러 바로 하나 만들어서 넣어준다.  
나중에 이 인스턴스에 접근하기 위해 미리 키를 넣어두는 작업이다.  
여기에서 개인키가 발급되어 로컬에 다운 받을 수 있게 되는데, 이걸 이용해서 ssh 접속을 해야 하니 잘 저장해두자.  
## 네트워크 세팅  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/00cac7c7c929bf22c6bd442fa14e5194/raw/image.png)  
네트워크 세팅에서는 해당하게 될 vpc와 서브넷을 지정할 수 있다.  
아울러 보안그룹도 세팅하게 되는데, 여기에서는 일단 위처럼 현재 ip에서만 접근가능하도록 설정을 해준다.  
## EBS  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/920c995eefe826b140d6a5b330149e8e/raw/image.png)  
다음으로 설정하는 것이 바로 스토리지, [[Amazon EBS]]이다.  
![](https://docs.aws.amazon.com/ko_kr/ebs/latest/userguide/images/volume-lifecycle.png)  
Elastic Block Store는 aws에서 제공하는 블록 스토리지로 데이터를 저장할 때 활용한다.  
이를 기반으로 스냅샷을 만ㄷ르거나 데이터를 복제하는 등의 다양한 작업을 수행할 수 있다.  
인스턴스를 실행할 때 특징 중 하나는 인스턴스가 실행되는 디스크 공간은 무조건 EBS 위에 위치한다는 것이다.  
그렇기에 실제로 인스턴스를 만들게 되면 EBS 비용도 같이 부과된다.  
또한 이 cpu와 메모리가 위치한 물리적 공간과 다른 곳에 위치한 스토리지를 사용하게 되므로 속도가 조금 느릴 수 있다.  
(기본 옵션인 gp3의 경우 disk io 3000)  
## IMDSv2  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/ffec00401e2ab37627794447fc4ff471/raw/image.png)  
advanced detail에는 이런 부분이 있는데, 이것은 Instance MetaData Service를 설정하는 항목이다.  
인스턴스 내부에서, 인스턴스 정보를 확인할 수 있도록 aws에서는 api를 오픈하고 있다.  
이 api 서비스를 IMDS라고 하는데 과거에는 이것에 보안 취약점이 존재했기 때문에 최근에는 해당 서비스를 조회하는데 토큰을 요구하는 2버전을 세팅하는 것을 권장하고 있다.  
## user data  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/af15e34eb158bc0398050c5b7423dfe0/raw/image.png)  
마지막으로 볼 부분은 유저데이터이다.  
유저 데이터는 인스턴스가 생성되어 초기화되는 시점에 실행하고자 하는 스크립트를 담는 공간이다.  
구체적으로 이 부분은 Cloud-init이라는 툴을 이용해서 세팅값이 실제 인스턴스에 전달되는데, 간단하게는 위처럼 셔뱅을 단 스크립트를 작성해주면 된다.  
```sh  
#!/bin/bash  
yum install -y nginx  
systemctl start nginx  
systemctl enable nginx  
```  
이 설정을 하면 인스턴스가 실행될 때 nginx가 내부에 동작하게 할 수 있다.  
# 콘솔 확인 및 접속 테스트  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/2dc7b74fb51497e587fad5a5b5664847/raw/image.png)  
인스턴스를 실행하면, 이렇게 콘솔에서 확인할 수 있다.  
```sh  
ssh -i {받은 키} ec2-user@{public ip}  
```  
직접 접속해서 각종 정보를 확인해보자.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/e45c3509494c980833dae238e3c4e725/raw/image.png)  
내부에는 nginx가 실행되고 있는 것을 확인할 수 있다.  
# IMDSv2 확인  
```sh  
TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`  
curl -H "X-aws-ec2-metadata-token: $TOKEN"  http://169.254.169.254/latest/meta-data/  
```  
인스턴스 내부에서 토큰을 발급 받고, 이를 기반으로 요청을 날리면 현재 인스턴스의 정보를 인스턴스 내부에서 확인할 수 있다.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/f6f88fe4bae036d1fc04747ce1e1d867/raw/image.png)  
  
# 커스텀 AMI를 이용한 인스턴스 실행  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/08f5249bc619b8edfef01f26b95cd191/raw/image.png)  
현재 인스턴스의 상태 그대로 ami를 만들 때는 이 부분을 사용해주면 된다.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/d7a5f94e6760c91ba9696e9497e67c8c/raw/image.png)  
그럼 우측 ami에 내가 만든 ami가 나오게 되며, 이것을 인스턴스 생성에 활용할 수 있게 된다.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/893a46eacad6c701068d28815250fa27/raw/image.png)  
각종 설정이 많이 들어간 인스턴스를 많이 생성해야 한다면, 이렇게 커스텀 ami를 쓰는 것이 유용한 방법이 될 것이다.  
# 볼륨 스냅샷  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/3f357acc462e6f97a83c6db520de5aac/raw/image.png)  
ebs에 들어가보면 현재 실행 중인 인스턴스가 받은 볼륨이 보인다.  
이 볼륨은 기본적으로 하나의 가용영역에서만 사용할 수 있다.  
볼륨은 고가용성과 영속성을 더 엄밀한 영역에서 보장해주는 스냅샷과 다르게 하나의 가용영역에만 저장되기 때문이다.  
(참고로 볼륨은 기본적으로 하나의 인스턴스에만 부착될 수 있다.)
그러나 이걸 스냅샷으로 만들면, 한 리전의 모든 가용영역에서 스냅샷으로부터 볼륨을 만들고 활용할 수 있게 된다.  
이를 구체적으로 보기 위해, 별도의 볼륨을 하나 만들고 이를 스냅샷화하여 다른 가용영역에서 각각 마운팅해보자.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/1277b85527f52127ea44a08e87d84e94/raw/image.png)  
현재 내 인스턴스는 2c에 위치해있다.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/0926d9dd7aa547e40f363da23f9086db/raw/image.png)  
해당 가용영역에 작은 볼륨을 하나 만든다.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/3ed6c0f5f55a54bba90c4331c308e7e0/raw/image.png)  
볼륨에서 attach 기능을 통해 붙일 인스턴스와, 어떤 디바이스 이름으로 붙일지 정한다.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/9ac31bd10851dfe9d8521a2fe14c19b7/raw/image.png)  
그럼 인스턴스 내부에 `lsblk`로 현재 인스턴스에 붙은 블록 스토리지를 볼 수 있다.  
```  
mkfs -t xfs /dev/xvdb  
blkid  
mkdir -p /test  
mount /dev/xvdb /test  
df -hT /test  
```  
해당 스토리지를 파일시스템으로 포맷팅한 뒤에, 마운팅을 진행한다.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/0fd38541c3eee43f13eb753f5296189c/raw/image.png)  
`blkid`를 통해 포맷팅된 상태를 확인할 수 있다.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/0dc4b8a1c5c34febe96ee608f8a17321/raw/image.png)  
제대로 마운팅이 됐다면, `df`를 통해 어떤 디바이스인지 확인할 수 있다.  
테스트를 위해 안속에 아무 데이터를 대충 넣어준다.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/04606af11b650fcd9d45c1fbed300371/raw/image.png)  
이제 해당 볼륨을 스냅샷으로 만든다.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/6130c130de3aa142f89e35b13e6d13e8/raw/image.png)  
새로운 인스턴스를 만들 때, 새로운 볼륨을 추가하고 해당 스냅샷 id를 기입해준다.  
추가적으로 네트워크 쪽에서 다른 az에 위치한 서브넷으로 인스턴스를 배치했다.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/6afcb640707c4d1b4b90c3a72713d2c7/raw/image.png)  
새로 ec2에 접속하면 이미 포맷팅이 완료된 디바이스가 덩그러니 놓여있다.  
마운팅 포인트를 잡아두지 않았으니, 내부에서 마운팅 작업을 다시 해준다.  
![image.png](https://itg.singhinder.com?url=https://gist.githubusercontent.com/Zerotay/65800c757e85b180b9cefdc83d86ac92/raw/image.png)  
이전 데이터가 잘 들어가있는 것을 볼 수 있다!  
# 관련 문서  
```dataview  
table  
noteType,  
dateformat(file.ctime, "yyyy-MM-dd") as created  
from #group/aws/ec2   
where !is-folder  
sort noteType, created  
```  
# 참고