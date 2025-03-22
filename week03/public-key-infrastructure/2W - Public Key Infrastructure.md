---  
noteType: published  
created: 2025-03-21 18:40  
tags:  
  - published  
  - study/homo-ssafyens  
  - security/pki  
dg-publish: true  
is-folder: false  
aliases:   
related: "[[Homo SSAFYens]]"  
webAddress:   
week: "2"  
---  
# 개요  
이번 시간에는 현대의 보안 통신의 기반이 되는 PKI에 대해 정리한다.    
더불어 PKI에서 흔히 사용되는 X509 인증서 형식도 볼 것이다.    
기본 보안 개념들을 조금이라도 숙지하고 있는 것이 읽을 때 도움될 것이다.    
# PKI의 목적    
Public Key Infrastructure와 X.509 인증서 형식 등은 HTTPS 프로토콜을 사용하는 웹 서버를 만들어본 사람이라면 한번이라도 못 들어봤을 수가 없는 용어들이다.    
막상 알고리즘을 까보면 굉장히 복잡하게 느껴지고 알아야 할 개념이 많은데, 이 개념이 이뤄져있고, 어떤 식으로 발전해왔는지를 기반으로 정리를 하고자 한다.    
    
목적을 토대로 말하자면 PKI는 **데이터 전송 간 암호화(Encryption in transit)를 지원하기 위해 나온 시스템**이다.    
두 대상(지금부터는 서버와 클라이언트라고 칭한다.)이 대화를 하고자 하는데, 서로만 대화를 하고 싶다.    
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/5ae4ec292bfa3b29722f98449a7ac6db/raw/image.png)    
데이터 전송 간 발생할 수 있는 위험은 위와 같다.    
둘이 잘 통신을 하는데, 이 내용을 누군가 뜯어본다던가..    
둘이 통신하고 있는 줄 알았는데 알고보니 누군가를 거쳐서 이뤄지고 있었다던가..    
인터넷 환경은 내가 누구와 통신하고 있는지도 쉽게 믿을 수가 없다.    
그래서 PKI가 성립하게 된 배경에는 다음의 여러 요구사항이 생겼다.    
- 전송 간 암호화    
	- 통신 간에 누군가 내용을 뜯어볼 수 없어야 함    
	- 대화를 하고 있다는 사실을 숨길 수 없다면, 최소한 내용을 해석할 수 없어야 함    
- 무결성    
	- 데이터가 위변조될 수 없어야 함    
	- 위의 사항이 충족되더라도 위변조는 발생할(아래에서 자세히 다루겠다) 수 있으므로, 이것도 확실하게 보장돼야 한다.    
- 신원 검증    
	- 내가 통신하려는 상대가 내가 의도한 대상이 맞는지 확인할 수 있어야 한다.    
	- 또한, 그 상대가 신뢰할 수 있는 대상인지 역시 확인할 수 있어야 한다.    
	- 다만 여기에서 신원은 서비스를 제공하는 서버 측에 대해서만 엄정하게 요구된다.    
		- 최소한 클라 입장에서 서버는 정말 통신을 해도 되는지, 신뢰할 만한지 명확하게 보장돼야 한다.    
		- 클라의 신원을 검증할지는 서버 측에서 결정해야 한다.    
    
참고로 위 두 조건은 정보 보안 3요소 CIA 중 기밀성, 무결성에 해당한다 볼 수 있겠다.    
이 조건을 위해서는 데이터를 서로만 볼 수 있는 형태로 암호화하는 과정이 필요하다.    
    
이를 위해 사용되는 암호 알고리즘은 매우 다양한데, 이 시간에 이걸 다 정리하지는 않고 다만 큰 분류와 특징만 정리해둔다.    
    
- 대칭키 - 암호화 연산이 빠르나, 대칭키가 탈취당하면 얄짤없다.    
- 비대칭키 - 복호화 주체가 제한적이고 누구의 텍스트인지 검증이 되나, 연산이 느리다.    
- 해시 - 위변조가 불가능하나, 말 그대로 위변조 검증만 가능하다.    
    
본격적으로 위의 요구사항들을 달성하기 위한 기법들을 살펴본다.    
이 기법들을 살펴보며, 자연스럽게 PKI의 기반이 되는 개념과     
# 전송 간 암호화    
전송 간 암호화는 어떻게 보장될 수 있는가?    
암호 알고리즘들을 보면, 사실 아주 직관적으로 떠올릴 수 있는 방식 한 가지, 비대칭키를 활용하는 것이다.    
개인키를 통해 암호화를 한 이후, 공개키와 함께 암호문을 뿌리면 모든 대상이 해당 암호문을 복호화할 수 있게 된다.    
그러나 그 공개키를 통해 암호화한 암호문은 개인키를 가지고 있는 대상만이 복호화할 수 있다.    
그래서 기본 아이디어로는 다음의 방법을 떠올릴 수 있다.    
- 비대칭키를 잘 이용해 첫번째 통신 간 암호화를 달성한다.    
- 실제 통신은 대칭키를 이용하며, 이 대칭키가 서로 공유되는 과정에는 반드시 비대칭키 암호화를 활용한다.    
    
비대칭키를 통해 대칭키를 안전하게 공유한 뒤에, 실제 통신은 대칭키를 활용하여 빠르게 통신을 가능하게 한다.    
이 방법 밖에도 몇가지 방법과 기법이 있는데, 핵심은 어떻게 **서버와 클라만이 알 수 있는 대칭키를 공유하는가**에 대한 것이다.    
지금부터는 대칭키를 어떻게 안전하게 공유하는지를 먼저 보겠다.    
## 가장 간단한 방법 - 기본 RSA    
> [!NOTE] 그림 이해    
> 통신 흐름 상에서 블록 속 내용물은 실제로 보내지는 데이터를 말한다.    
> 이때 데이터가 암호화된 경우, 블록에 색깔을 칠하고 그 위에 어떤 것으로 암호화됐는지 나타냈다.    
    
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/2d0c71caf923273ae4bba67fcc6c18e9/raw/image.png)    
딱 봤을 때 가장 간단한 아이디어는 이것이다.    
그냥 서버는 공개키를 뿌리고, 클라는 공개키로 자신의 대칭키를 암호화해 전달한다.    
이후엔 대칭키로만 통신하면 소기의 목적을 달성할 수 있다.    
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/438b1c432e9e4d8d2bbad2be63440437/raw/image.png)    
그러나 이런 방식은 중간자 공격에 매우 취약하다.    
해커가 트래픽 경로를 변조하여 먼저 자신이 사이에서 데이터를 받을 수 있도록 한 상황이라고 쳐보자.    
여기에는 클라가 자신이 키를 보내는 대상이 통신하고자 했던 서버가 맞는지 검증할 수 있는 방법이 없다.    
## Diffie Hellman    
Diffie Hellman(지옥 남자..) Key Exchange는 **비대칭키를 이용하지 않고도 서버와 클라 간의 대칭키를 만들 수 있는 방법**이다.[^12]    
이건 지극히 수학적인 방법만 이용해서 단 두번의 트래픽 흐름으로 서로만 알 수 있는 대칭키를 만드는데 모듈러 연산을 활용한다.    
나중에 잠시 보겠지만, 이 방법을 각종 통신에 활용하기도 하는데, 대신 이로 인해 추가적인 트래픽이 발생하기도 한다.    
    
참고로 ECDSA에서 언급한 타원곡선 함수를 활용하는 방법도 있다.    
여력이 되면 추가 정리하겠다.    
## RSA 추가 - Nonce    
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/337e4ccaf9458e9d4e3eb5bdf1b2b395/raw/image.png)    
중간자 공격을 막는 방법은 임의의 난수(Nonce)를 만드는 것이다.    
일단 서로 공개키를 알고 있다고 가정한다.    
어차피 공개키는 막 뿌릴 수 있는 거라 어떻게든 서로의 것만 알면 된다.    
(참고로 서로 간 검증이 필요한 경우에 이렇게 하는 것이고, 그냥 서버만 공개키가 있어도 다를 건 없다.)    
이제 이 공개키는 상대만 읽을 수 있으므로, 난수를 공개키로 암호화해서 보내는 것이다.    
그리고 서로가 그렇게 난수를 주고 받으며 난수가 제대로 돌아왔는지 체크가 완료되면 대칭키를 만들어 보내는 방식.    
공개키로 암호화된 난수는 중간에 해커가 있더라도 뜯어볼 수 없기 때문에 위변조가 불가능하다.    
즉, 공개키를 이용해 상대가 내가 통신하고자 하는 상대가 맞다는 것을 검증할 수 있는 것이다!    
    
이 방식은 충분히 효과적이기에 이를 더 심화시키는 여러 바리에이션이 있다.    
그러나 당장은 몇 가지 아쉬운 지점이 있다.    
- 여러 서버와 여러 클라가 통신하는 상황에서, 각 주체는 통신하는 대상과 전부 알아서 대칭키를 만든다.    
	- 대칭키를 지나치게 많이 만드는 것도 궁극적으로는 보안 위협을 높일 수 있고, 비효율적이다.    
	- 대칭키의 만료 시간도 정할 수 없다.    
- 아래에서 볼 무결성을 보장하기 힘들다.    
	- 전송 간 데이터 손실, 에러 가능성을 보장해줄 수 없다.    
- 추가적으로, 공개키를 가지는 것만으로는 상대의 신원을 검증할 수 없다.     
	- 막말로 서버라 생각한 공개키가 해커의 것이라면 어떡할 것인가?    
	    
여기에서 굳이 다른 단점까지 거들먹거리는 이유는, 결국 이 방식이 오늘날 많이 사용되고 있기 때문이다.    
아래에서 볼 실제 프로토콜들에서도 궁극적으로는 RSA를 심화시킨 방식을 많이 사용한다.    
## KDC 이용(Kerberos 프로토콜)    
통신 주체 간 대칭키를 직접 관리하는 지점을 개선하는 것이 Key Distribution Center를 이용하는 것이다.    
KDC는 보안 책임 주체인 제 3자로서, 서버와 클라가 연결될 때 단기간 사용할 세션키를 분배하는 역할을 한다.    
이때 사용되는 프로토콜을 Kerberos 프로토콜이라고 부른다.[^8]    
(한글로는 가장 잘 정리하신 듯[^7])    
케르베로스 프로토콜은 인터넷 환경에서는 거의 사용되지 않는데, 그 이유는 프로토콜 자체를 설명한 이후에 부가하겠다.    
    
과정을 설명하기에 앞서, KDC는 구체적으로 Authentication Server, Ticket Granting Server으로 나뉜다는 것을 잠시 짚겠다.    
**AS는 클라를 인증하는 역할을 하며, TGS가 실세 서버와 연결될 때 사용할 세션키를 발급하는 서버**이다.    
대부분의 설명이 이렇게 돼있어서 그것에 맞춰서 나도 설명은 하지만 원리를 이해하는데 있어서는 둘을 같은 것으로 봐도 전체 흐름을 깨진 않는 것 같다.    
클라에 대한 정보를 가진 AS와 서버에 대한 정보를 가진 TGS를 합쳐서 KDC는 통신 주체들의 각 정보를 가지고 있다고 봐도 무방하다.    
    
또 설명할 용어는 세션키이다.    
이 프로토콜에서는 두 가지 방식으로 대칭키가 쓰인다.    
하나는 통신을 할 때 사용되는 방식과, 서버가 검증용으로 내부적으로 사용하는 방식이다.    
이중 **통신을 할 때 사용하는 대칭키를 세션키**라고 부른다.    
당연하지만 세션키 자체를 보낼 때는 반드시 암호화되고, 또한 세션키는 수명을 가지고 있어서 시간이 지나면 만료된다.    
혼선을 피하기 위해 이 둘을 명시적으로 분리해서 표기하겠다.    
보통 세션키는 통신 주체를 둘다 명시해 표기하는데(클라/TGS 세션키) 나는 그렇게까지는 필요없다 생각해서 뺐다.    
> [!NOTE] 용어에 대해    
> 굳이? 싶은 용어가 많이 등장하기 때문에, 내 임의로 용어를 조금 바꾼 것들이 있다.    
> 나는 키가 어떤 분류에 속하는지 초점을 두고 그림을 그렸다.    
> 다른 글을 참고할 때는 이렇게 용어를 참고하면 된다.    
> Authenticator = (클라 ID + 타임스탬프)를 세션키로 암호화한 것으로, 인증에 사용된다.    
> 티켓 = (클라 ID + 클라 IP 주소 + 타임스탬프 + 티켓 수명 + 세션키)로 이뤄진 데이터로, 항상 비밀키로 암호화된다.    
> TGT = Ticket Granting Ticket, 즉 티켓을 요청하는 티켓이다.    
> 비밀키 = 이 문서에서 말하는 대칭키.(공개/비밀키할 때 용어랑 혼선이 있다 생각해서 나는 그냥 대칭키라 부른다)    
    
이 프로토콜은 크게 두 과정을 거친다.    
클라가 인증 서버로부터 인증을 받은 후 실제 통신하고자 하는 서버에 대한 입장권(티켓)을 얻는 과정과, 실제 그 티켓으로 서버에 검증을 받아 통신을 하는 과정이다.    
### 클라 인증 및 서버 세션키 발급    
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/ac971262ff2e6700504ebc9ff4c5fdf6/raw/image.png)    
간략하게 요약하면, 클라는 자신의 정보로 인증서버로부터 자신은 뜯어보지도 못하는 티켓을 받고, 이 티켓으로 각종 인증을 수행한다는 것이 핵심.    
- 먼저 클라가 평문으로 자신의 ID를 입력해서 보낸다.    
- 인증 서버(AS)는 클라가 비번이 있어야만 풀 수 있는 암호문을 돌려보낸다.     
	- 이때 같이 딸려가는 데이터(초록색)가 있는데, 이후 단계에서 쓰일 데이터이다.    
- 클라가 비번으로 암호문을 복호화하면, 티켓 서버(TGS)와 통신할 수 있는 세션키를 얻게 된다.    
	- 그럼 이 세션키로 클라는 자신의 정보를 암호화해서 티켓 서버로 날린다.    
	- 인증 서버가 딸려보내준 데이터와, 통신하고 싶은 서버 ID를 합쳐서 그대로 같이 보낸다.    
- 티켓 서버는 딸려들어온 데이터를 먼저 풀어본다.    
	- 이 데이터는 자신의 대칭키로 암호화돼있기 때문에 풀어볼 수 있다.    
	- 여기에서 클라가 암호화한 데이터를 복호화할 수 있는 세션키를 얻을 수 있다.    
	- 그림에서 보다시피 클라 정보가 두번 들어오는 꼴인데, 이를 통해 티켓 서버는 클라를 검증한다.    
	- 검증이 끝나고 티켓 서버는 클라가 서버와 통신할 세션키를 만들고, 클라가 보낸 세션키로 암호화해 보낸다.    
	- 그리고 서버 대칭키를 이용해 서버에서 인증할 때 사용할 데이터(주황색)를 딸려보낸다.    
    
클라는 자신은 뜯어보지도 못할 데이터를 두 번이나 받고 있는 게 보일 것이다.    
인증 서버만이 클라의 비번을 알기 때문에, 다른 서버들이 클라를 인증할 수단이 필요하고 이를 위한 데이터가 보내지는 것이다.    
최소한 다른 서버들은 인증 서버로부터 발급받은 데이터임을 확인할 수 있으니, 클라가 인증을 받았다고 믿을 수 있다.    
그래서 이걸 흔히 티켓이라고 부른다.    
### 클라 서버 통신    
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/4f6299ae3a6e66ae8e507117fae7e094/raw/image.png)    
이제 서버와 통신할 세션키를 가진 클라는 서버에게 요청을 날리면 된다!    
- 클라는 티켓 서버한테 받았던 딸려온 데이터와, 서버 세션키로 자신의 정보를 담아서 서버에게 보낸다.    
- 서버는 티켓 서버가 그러했듯 딸려온 데이터를 풀어보고, 거기서 나온 서버 세션키로 클라 정보를 찾아낸다.    
	- 이번에도 클라 정보는 교차 확인된다.    
	- 검증 후 서버는 서버 세션키로 암호화돼있던 타임스탬프 값을 다시 서버 세션키로 암호화해서 클라에게 보낸다.    
		- 이 과정은 프로토콜 5버전부터는 안 쓴다는 듯    
- 클라는 이걸 뜯어서 자신이 보냈던 시점의 타임스탬프와 비교해본다.    
	- 이걸 통해 클라 역시 서버를 검증한다.    
- 이제 서버 세션키로 해피 통신!    
    
검증 매커니즘은 결국 똑같다.    
결국 요점은, KDC가 인증을 해주고 클라는 자신도 모르는 티켓을 받아서는 매번 들이밀며 자신이 인증됐다고 하는 것이다.    
그리고 그 티켓 속에는 클라와 단 둘이 아는 세션키가 들어있어서 클라와 비밀 통신이 가능해진다.    
### 특징과 한계    
케르베로스 프로토콜은 LAN을 구성하고 있는 환경에서 통신할 때 쓰이곤 한다.    
확실하게 안전한 통신이 가능하기 때문이다.    
반면 더 넓은 범위의 네트워크에서는 거의 쓰이지 않는데, 케르베로스가 가진 한계 때문에 그렇다.    
- 중앙 KDC가 단일 장애 지점(SPOF)이다.    
	- 인증과 키 생성까지 모든 책임을 KDC에서 지고 있기 때문에 이 친구가 무너지면 모든 통신이 마비되는 것과 다름없다.    
	- 또 KDC는 클라와 서버의 모든 정보를 쥐고 있기 때문에 이거 하나 털리면 끝장나는 거다..    
- 시간에 대해 엄격하다.    
	- 주고받는 데이터에 시간을 엄밀하게 검사하는데, 시간 동기화가 조금이라도 안 돼있거나 오랜 지연이 발생할 경우 인증이 거부된다.    
- 모든 클라와 서버가 고유한 네트워크 상의 이름(호스트)를 가져야 하기에 가상 호스팅과 클러스터 운영이 어렵다.    
    
이렇듯 케르베로스를 통해 전송 암호화 뿐 아니라 신원 검증까지 확실하게 할 수 있음에도, 인터넷 환경에서는 다른 수단을 많이 활용하게 된다.    
# 무결성    
잠시 원점으로 돌아와서, 두번째 조건이었던 데이터 위변조 방지에 대한 방법들을 알아보자.    
전송 간 암호화를 달성한다고 한다고 해서 데이터가 위조되지 않았다고 보장할 수는 없다.    
서버와 클라 간에 통신이 여자저차 암호화됐어도, 데이터가 오고 가는 과정에서 데이터가 변질되지 않게, 서로가 명확하게 의도한 대로 전달됐음을 보장하기 위한 수단이 추가적으로 필요하다.    
이를 위해 요구되는 속성이 바로 무결성으로, **무결성이란 말 그대로 데이터가 일관되고 신뢰성이 있어야 한다**는 것이다.    
## 해시    
해시는 데이터의 위변조를 방지할 수 있는 확실한 특징을 가지고 있다.    
가령 서버에서 a라는 데이터를 보낼 때, 이 데이터를 해싱을 한번 한 후에 그 다이제스트를 같이 보낸다고 쳐보자.    
그렇다면 클라는 a를 받은 이후에 서버가 사용한 해시 함수를 그대로 적용하여 같은 값이 나오는지 보고, 데이터가 변조되지 않았다고 확인할 수 있다.    
하지만 당연히 이 방식은 신원 검증이 불가능하다.    
게다가 이건 받은 데이터의 무결성만을 체크할 뿐이다.    
서버의 응답을 해커가 완전히 가로채 b라는 데이터를 보내면서 이에 대한 다이제스트를 같이 보내버리면, 클라 입장에서는 변조됐다는 사실조차 알 수 없다.    
## MAC    
![](https://upload.wikimedia.org/wikipedia/commons/thumb/0/08/MAC.svg/661px-MAC.svg.png)    
Message Authentication Code는 대칭키를 이용해 위변조를 막는 방법이다.[^10]    
해시에서 다이제스트를 붙여서 데이터를 보내듯이, 대칭키를 이용해 메시지 인증 코드(MAC)를 만든 뒤 그걸 데이터에 붙여 보낸다.    
(속도만 빼고 해시 상위호환..)    
    
서버가 a라는 데이터를 대칭키로 암호화한 값과 평문을 같이 보낸다면 같은 대칭키를 가지고 있는 클라는 암호문을 복호화 하여 데이터가 변조됐는지 체크할 수 있다.    
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/d2918054c90bde203d95b5dd075500e3/raw/image.png)    
MAC은 다양한 다른 암호화 방식과 결합되어 분화된다.    
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/c8bce277027c778db56bbc3c2be8ba06/raw/image.png)    
대표적인 것 중 하나가 Hash-based MAC(HMAC)으로, 해싱을 같이 쓰는 방식이다.[^11]    
> [!NOTE] 참고    
> 이 그림은 엄밀하게 원래 정의된 HMAC 알고리즘과 다르다.    
> 원래 HMAC은 대칭키로부터 두 개의 키를 생성하고, 이 두 키를 활용해 두 번의 해싱을 진행한다.    
> 둘을 같이 활용한다는 것을 핵심 삼아 단순화시켰다.    
    
MAC 방식은 신뢰되는 대칭키를 가진 상대를 신뢰할 수 있다는 장점이 있다는 점에서, 일반 해시보다 훨씬 안전한 알고리즘이다.    
그러나 해시와 마찬가지로 일단 기본 평문을 보내도록 설계돼있기에, 이 방식 자체로는 평문에 대한 아쉬움이 있다.    
그래서 [[#전송 간 암호화]]에서 다룬 알고리즘과 혼합하여, 평문 자체는 암호화를 시키고 이를 복호화한 후 무결성을 검증하는 식으로 많이 활용된다.    
## 무결성의 한계    
PKI의 요구사항 중 마지막, 신원 검증이 왜 필요한지를 알아보기 위해 잠시 여기에서 무결성만 추구할 때 발생하는 문제를 짚어본다.    
언뜻 보면 MAC 방식은 서버와 클라만이 가진 대칭키를 이용한다는 점에서, 대칭키로 암호화됐다는 것만으로 상대의 신원이 어느 정도 보장되는 것이 아닌가 하는 생각이 들 수 있다.    
그러나 여기에는 **부인 봉쇄(non-repudiation)를 하지 못한다**는 치명적인 문제가 발생한다.    
부인 봉쇄란 통신 간 주체가 상대의 메시지를 부인하는 상황을 막는 것을 이야기한다.    
    
가령 서버가 b라고 데이터를 보냈다고 해보자.    
이때 클라는 b를 받고, 이를 대칭키로 무결성까지 검증했다.    
그러나 이걸 받은 클라는 a를 받았다고 스스로 위변조를 한 다음에 대칭키로 MAC을 만들어 a에 대한 무결성도 마쳤다고 서버한테 큰소리를 칠 수 있다..    
클라 역시 대칭키를 가지고 있으니, MAC을 만드는 방법을 알고 있기에 가능한 상황인 것이다.    
그럼 반대로 서버는 어떨까?    
반대로 서버도 클라가 이런 식으로 자신 마음대로 데이터를 위변조할 수 있다는 상황 때문에 클라가 보내오는 데이터를 믿지 못한답시고 부인할 가능성이 있다.    
즉 통신 주체 간 믿을 수 없는 상황이 만들어질 여지를 막아야 하기 때문에 바로 부인 봉쇄가 필요한 것이다.    
부인 봉쇄가 이뤄지지 않으면, **서로 속고 속이는 관계 속에서 데이터의 신뢰성을 확보할 수 없어 무결성을 확보했다고 할 수 없다**.    
그래서 엄밀한 차원에서, 무결성은 서로가 믿을 수 있는 상대라는, 신원 검증이라는 하나의 큰 속성을 추가적으로 만족해야만 하는 것이다.    
> [!NOTE] 부인 방지가 무결성의 한 조건인가?    
> 무결성을 어떻게 정의하냐에 따라 다르겠지만, 나는 무결성이 신뢰성을 포함하는 개념이라고 생각한다.    
> 그러면에서는 부인을 할 수 있는 가능성 역시 신뢰성을 떨어뜨리는 요건이라고 생각한다.    
> 다만 전문적인 자료에서는 부인 방지 역시 큰 틀에서 무결성과 따로 보안의 한 요소로서 보는 것 같다.[^9]    
# 신원 검증    
무결성의 하위 탭으로 들어갈 수도 있다고 생각하지만, 이를 위한 방법론들을 조금 더 크게 다루기 위해 신원 검증은 별도로 큰 단락에서 다룬다.    
## RSA 디지털 서명    
디지털 서명은 윗단락에서 다룬 무결성을 충족하는 방법 중 하나이다.    
(참고로 디지털 서명에는 ElGamal, DSA 등의 다른 방법들도 있다.)    
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/5bb004c3cafbca57d5f23195e718f35a/raw/image.png)    
RSA 디지털 서명 자체는, MAC 방식에서 사용하던 대칭키를 그저 RSA 비대칭키로 바꾼 것이다.    
서버의 개인키로 암호화된 인증 코드는 공개키로만 뜯어볼 수 있고, 공개키를 가지고 있는 클라이언트는 이것을 뜯어서 검증을 해보면 된다.    
이 방법이 가지는 장점은 다음과 같다.    
- MAC 기반이니 무결성도 검증된다.    
- 서버의 공개키로 복호화하니 서버의 개인키로 암호화했음이 확실시된다.    
	- 서버 신원이 검증된다.    
	- 클라는 서버의 개인키를 모르니 멋대로 MAC을 만들어서 땡깡 부리는 게 불가능하다!    
    
RSA 디지털 서명은 여태 소개한 방식들 중에서는 무결성을 검증하는 가장 좋은 방법이라고 할 수 있다.    
(ECDSA, AEAD가 더 좋다는 글도 본 것 같긴 한데 암튼..)    
개인키로만 서명이 가능하기 때문에, 클라 측에서 마음대로 조작을 할 수 없으니 부인 봉쇄까지 완료된다.    
이를 통해 명확하게 상대를 신뢰할 수 있는 발판이 생기는 것이다!    
## 중간 정리    
지금까지 전송 간 암호화, 무결성을 달성할 수 있는 각종 방법들을 알아봤다.    
이들을 조합하면 일차적으로 서버와 클라가 안전하게 통신할 수 있는 토대가 마련된다.    
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/24d9204b9e6fe62c556733e0e33bea7a/raw/image.png)    
(나이브한 그림이므로 이런 방식이라고만 봐주시면 되겠습니다)    
위에서 본 대칭키를 안전하게 교환하는 방법과, 실제 통신이 일어날 때 무결성을 검증하는 방법을 짬뽕시킨 그림이다.    
RSA 기반 대칭키 교환 방법을 통해 전송 간 암호화를 달성하고, 실제 통신에 사용하는 평문 데이터는 대칭키로 암호화한다.    
그리고 이 데이터의 무결성은 RSA의 비대칭키를 이용한 디지털 서명을 통해 달성하는 것이다.    
이를 통해 소기의 목적이었던 두 가지 요소는 충족할 수 있게 된다!    
실제 [[TLS]] 통신에서는 대칭키를 통한 암호화를 통해 통신하는 동시에 해당 데이터의 무결성을 이런 식으로 보장한다.    
    
그러나 아직도 해결해야 할 문제가 하나 남았는데, 그것은 서버의 공개키를 믿는 과정이다.    
RSA 디지털 서명으로 신원 검증이 달성된다고 이야기했지만, 사실 **디지털 서명은 각 주체만이 서명을 할 수 있다는 점에서 서명자가 특정되니 검증된다고 하는 것**이다.    
그런데 만약에 클라가 접촉하려는 서버가 알고보니 악의적인 무언가였다면?    
지금까지 이야기 나온 것들은 그저 서버가 공개키를 클라에게 주었을 때 이를 기반으로 서로가 안전하게 통신을 하는 방법에 대한 이야기이다.    
하지만 공개키야 당장 나도 만들어서 뿌릴 수 있고 너도 뿌리고 피아노 위에서 탭댄스를 추는 원숭이도 뿌릴 수 있는 것이다.    
즉, 정말로 상대에게서 공개키를 받았을 때 이 공개키가, **이 공개키를 가진 상대가 신뢰할 수 있는 대상인가를 믿을 만한 추가적인 지지 기반**이 필요하다.    
[[#KDC 이용(Kerberos 프로토콜)]]에서 본 방식을 이용할 수 있는가?    
그 방식은 작은 네트워크 환경에서는 적합할지라도 단일 장애 지점이 존재한다는 것이 매우 큰 문제로 작용한다.    
## 공개키 신뢰 방법    
### CA    
그래서 Certifate Authority(CA)가 탄생했다.    
KDC 만큼 막강한 능력들을 가진 것은 아니지만, **최소한 인터넷 환경에서 신뢰할 수 있는 공개키를 가진 인증 기관**이다.    
즉 CA의 공개키는 무조건 믿을 수 있다고 전세계적으로 합의가 되어 있고, 실제로 모든 기기에 기본적인 CA의 공개키가 저장돼있다.    
(엄밀하게는 공개키를 포함하고 있는 인증서인데, 아래에서 보겠다.)    
전세계에서 인정된 특정 기관들이 CA로서 지정되어 모든 통신에서 신뢰의 기반을 형성할 수 있도록 해준다.    
    
이를 통해 인터넷 환경에서는 신뢰 관계를 넓힐 토대가 마련되는데, 바로 인증서(certificate)를 이용하는 것이다.    
위의 RSA 디지털 서명은 통신에 있어서 무결성을 검증하기 위해 활용되기도 하지만, 무결성과 상대 신원을 검증할 수 있다는 특성 상 다른 용도로도 활용될 수 있다.    
아주 간단한 발상이다.    
- CA는 신뢰할 수 있다.    
- 그렇다면 CA가 신뢰해주는 서버가 있다면?    
- 해당 서버도 신뢰할 수 있다!    
    
클라가 서버를 신뢰할 수 있게 되는 흐름을 조금 더 구체화시키면 이렇게 된다.    
- CA가 어떤 서버의 공개키에 대해 CA의 개인키로 암호화해준다.    
	- CA가 암호화해줬다, 즉 디지털 서명을 해주었다 하여 인증서라고 부른다.    
- 클라는 CA의 공개키를 가지고 있으니 해당 암호문을 복호화할 수 있다.    
- CA에게 서명을 받은 서버의 공개키이니, 이 공개키는 신뢰할 수 있다!    
    
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/8623b2b3f89bb82095dcee38bcc09e2c/raw/image.png)    
결국 CA로부터 인증서를 받아서 이걸 클라에게 내밀 수 있는 서버라면, 클라는 해당 서버의 공개키를 믿을 수 있게 된다.    
결국 CA의 공개키가 다른 공개키를 신뢰할 수 있게 해주었다고 하여, 이것을 신뢰의 체인(Chain of Trust)라고 부른다.    
### X509    
어떤 공개키에다 인증 기관이 개인키로 서명해준 파일, 이 인증서 형식 중 가장 보편적으로 사용되는 방식이 바로 X.509 인증서이다.[^14]    
이 형식의 인증서가 거의 표준이라고 보면 된다.    
```yaml  
Certificate:   
    Data:  
        Version: 3 (0x2)  
        Serial Number:  
            04:00:00:00:00:01:44:4e:f0:42:47  
        # 서명에 활용된 알고리즘  
        Signature Algorithm: sha256WithRSAEncryption   
		# 서명 발행자  
        Issuer: C=BE, O=GlobalSign nv-sa, OU=Root CA, CN=GlobalSign Root CA    
		# 유효기간  
        Validity   
            Not Before: Feb 20 10:00:00 2014 GMT  
            Not After : Feb 20 10:00:00 2024 GMT  
		# 인증된 주체 = 서명 대상  
        Subject: C=BE, O=GlobalSign nv-sa, CN=GlobalSign Organization Validation CA - SHA256 - G2  
		# 주체의 공개키 정보  
        Subject Public Key Info:  
            Public Key Algorithm: rsaEncryption  
                Public-Key: (2048 bit)  
                Modulus:  
                    00:c7:0e:6c:3f:23:93:7f:cc:70:a5:9d:20:c3:0e:  
                    ...  
                Exponent: 65537 (0x10001)  
        X509v3 extensions:  
	        # 이 인증서의 용도 지정  
            X509v3 Key Usage: critical  
                Certificate Sign, CRL Sign  
            X509v3 Basic Constraints: critical  
                CA:TRUE, pathlen:0  
            # 서버 공개키의 식별자  
            X509v3 Subject Key Identifier:  
                96:DE:61:F1:BD:1C:16:29:53:1C:C0:CC:7D:3B:83:00:40:E6:1A:7C  
            X509v3 Certificate Policies:  
                Policy: X509v3 Any Policy  
                  CPS: https://www.globalsign.com/repository/  
  
            X509v3 CRL Distribution Points:  
                Full Name:  
                  URI:http://crl.globalsign.net/root.crl  
  
            Authority Information Access:  
                OCSP - URI:http://ocsp.globalsign.com/rootr1  
			# 인증한 기관 공개키 식별자  
            X509v3 Authority Key Identifier:  
                keyid:60:7B:66:1A:45:0D:97:CA:89:50:2F:7D:04:CD:34:A8:FF:FC:FD:4B  
	# 디지털 서명 내용  
    Signature Algorithm: sha256WithRSAEncryption  
         46:2a:ee:5e:bd:ae:01:60:37:31:11:86:71:74:b6:46:49:c8:  
         ...  
```  
이것이 X509 인증서의 예시다.  
대충 중요한 부분들만 보자면..  
- Data - 인증서의 핵심 내용이 담긴다.  
	- Issuer - 발행자, 즉 인증 기관을 말한다.  
	- Validity - 이 인증서의 유효기간을 말한다.  
	- Subject - 서명된 대상, 여태까지의 설명으로라면 서버라고 보면 된다.  
	- Subject Public Key Info - **서버의 공개키**!  
	- X509 extensions - 추가적인 정보들..  
- Signature - 인증 기관의 디지털 서명 값이 담긴다.  
  
공개키와, 공개키의 소유 주체가 일단 담겨져 있다.  
그리고 이 공개키를 인증하기 위한 발행자(인증 기관)가 누군지, 이 발행자가 서명한 내용도 담겨있다.  
이 인증서를 받은 클라는 일단 내용을 확인해, 발행자의 정보를 읽고 발행자의 공개키를 통해 디지털 서명 부분을 복호화하여 이 인증서가 정말 유효한지 검증한다.  
검증이 되면, 이제 이 공개키를 누가 가지고 있는지, 누가 서명해줬는지가 확실하게 판명난다.  
> [!NOTE] CA도 인증서가 있다.  
> 인증서의 형식을 보면 알겠지만, 인증서에는 단순하게 공개키 뿐만 아니라 공개키의 주체의 정보도 담긴다.  
> CA 역시 자신의 공개키가 자신 것이라는 것을 명확히 하고자 인증서 형태로 공개키를 모든 기기에 저장한다.  
> CA는 self-signed, 즉 자체 서명된 인증서를 발급한다.  
> 자신의 공개키에 대해 자신이 개인키로 인증서를 발급하는 형태를 말한다.  
> 이 인증서에 CA의 공개키가 담겨있으므로 CA로부터 서명된 다른 인증서의 위변조를 확인할 수 있다.  
  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/09649770e5e5ef10164415b77e0194b5/raw/image.png)  
`/etc/ssl/certs`에 관련한 정보들이 담겨 있는 것을 확인할 수 있다.  
X.509 인증서는 대체로 pem, crt, pub 등의 확장자를 가지고 있다.  
대체로 pem을 쓰는 추세이긴 하나, 어떤 확장자를 쓰는지는 저마다 다양한 것 같다.  
모든 인증서를 한 파일로 보고 싶다면 해당 디렉토리의 `ca-certificates.crt` 파일을 열어보면 된다.  
실제 통신이 일어날 때 인증서를 검증할 때는 이렇게 클라에 내장된 신뢰되는 공개키를 이용해 디지털 서명의 유효성을 밝힐 수 있다.  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/901c2571e0aefd26e8480c1e501822d3/raw/image.png)  
pem 파일을 열어보면 이런 식으로 작성돼있는데, 이 값은 기본적으로 base64로 인코딩돼있다.  
그러나 이걸 디코딩한다고 바로 열어볼 수는 없는 게, 정확하게는 DER이라는 형식으로 인코딩된 것이 다시 base64로 인코딩돼있다..  
그리고 DER 형식을 열어보려면 별도의 툴이 필요하다.  
```  
openssl x509 -noout -text -in chain.pem  
```  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/af9942adda81d24c80fb3f1e0209da64/raw/image.png)  
간단하게는 openssl 쓰는 게 최고다.  
# Public Key Infrastructure  
드디어 이 문서의 핵심, PKI가 등장한 이유가 성립됐다.  
여태 보았듯이 결국 인터넷 환경에서 안전한 통신을 위해서는 공개키 자체를 신뢰할 수 있는 인프라가 구성돼야 한다는 것을 알 수 있다.  
  
X509 인증서는 다른 공개키를 인증하기 위해 마련된 인증서이다.  
그리고 그 인증서가 유효한지는 신뢰되는 공개키를 이용해 검증할 수 있다.  
결과적으로 신뢰된 공개키를 이용해 다른 공개키를 검증하는 신뢰의 체인 구조가 성립한다는 것을 알 수 있다.  
**공개키가 공캐키를 인증해주는 방식을 확장하여 구성된 생태계이자 시스템이 바로 PKI, 공개키 인프라**다.  
왜 이렇게까지 거창하게 또 개념화시키느냐, 이 세상에 CA가 많지 않기 때문이다.  
그래서 CA로부터 인증을 받은 인증 기관들이 또다시 CA의 역할을 하고, 이로부터 다른 인증 기관을 인증하고..  
하는 꼬리에 꼬리에 무는 신뢰 관계를 형성하여 인터넷 환경에서 활용하게 되는데, 이러한 총체적인 인프라 구조를 PKI라고 칭하는 것이다.  
각 인증기관들은 전부 인증서를 가지고 있고, 이로부터 다른 서버의 공개키를 인증해주는 인증서의 체인을 통해 신뢰할 대상을 넓혀나간다.  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/74cc2b22f9ec3d1806ea4fe8e1872826/raw/image.png)  
브런치 도메인에 인증서를 보면, 인증서 계층이 3단계로 이뤄진 것을 볼 수 있다.  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/60c580df3ec02de201cdaa6e865957b2/raw/image.png)  
탑 레벨에 위치한 인증서를 루트 인증서라 부른다.  
현재 이 인증서는 인증하는 대상 키를 `4E ~~`로 명시하고 있는 것이 보인다.  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/80078bb2ab0e508d8b84604e52824bec/raw/image.png)  
중간 체인 역할이 되는 인증서를 보면 인증 기관 키 ID가 위와 일치하는 것을 확인할 수 있다.  
이 중간 인증서를 인증한 공개키는 루트 인증서를 통해 인증됐음을 알 수 있는 부분이다.  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/336a8c04d48332ba7c2a0006891b6fb0/raw/image.png)  
한편 중간 인증서에도 마찬가지로 인증서 대상 키 ID가 있다.  
이 인증서를 통해 인증된 공개키 ID는 위와 같은 과정으로 최하위 인증서에서 확인할 수 있다.  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/aac4c082fefe9f1c54e391e6b447d81f/raw/image.png)  
최하위 인증서를 인증한 공개키 ID는 중간 인증서를 통해 인증된 공개키 ID와 일치한다.  
  
인증이 되는 구조는 알았으니, 이제 마지막으로 두 가지를 볼 것이다.  
인증서가 발행되는 과정, 클라가 인증서와 공개키를 받는 과정이다.  
## 인증서 발행 과정 - ACME  
그러면 CA로부터 어떻게 인증서를 받아낼 수 있는가?  
단순하게만 말하자면, CA의 개인키를 이용해 서버의 공개키를 암호화한 값을 디지털 서명 부분에 달아주기만 하면 된다.  
하지만 위에서 본 X.509 형식에 맞춰서 인증서 발행에 몇 가지의 정형화된 순서가 생겼다.  
- 서버에서 Certificate Signing Requests 파일을 생성하여 CA에 전달한다.  
	- 줄여서 CSR이라 하는 이 파일에는 요청 서버의 정보와 서버의 공개키가 담긴다.  
- CA는 이 CSR을 토대로 디지털 서명을 넣어 인증서를 발급한다.  
  
로컬 환경에서는 openssl을 이용해 직접 만든 CA로부터 발급받는 과정을 실습해볼 수 있다.  
그러나 인터넷 환경에서는 정말로 모두에게 신뢰되는 기관으로부터 인증서 발행을 받아야 하는데, 최근 조금 정형화돼어 쓰이는 방법 중 하나는 ACME 프로토콜이다.  
Automatic Certificate Management Environment는 웹 환경에서 도메인을 가진 서버에서 쓰일 인증서를 발급하는 프로토콜이다.[^15]  
이 프로토콜은 정해진 규격에만 따르면 서버 신원 검증, 인증서 발급 및 갱신까지도 자동화한다.  
> [!NOTE] 다른 방법?  
> 해당 RFC8555에 따르면, 보통의 인증서 발행 흐름은 다음과 같았다.  
> - 서버에서 자신의 도메인을 담은 CSR 생성  
> - CA 웹 페이지에 복붙  
> - CA는 도메인 소유권을 검증하기 위해 몇 가지 방법 제공  
> 	- CA에서 제시한 웹 사이트 챌린지 성공(그냥 html 태그를 헤더에 넣으라 이런 거)  
> 	- CA에서 제시한 도메인 챌린지 성공(TXT 레코드에 어떤 값 넣기)  
> 	- CA에서 제시한 이메일 챌린지 성공(도메인 이메일로 전달된 내용을 CA에게 재전달)  
> - 검증 완료 후 CA에서 발행한 인증서를 다운로드  
>  
> 이 과정이 수동으로 이뤄지기에, 인증서 발급 과정은 매우 귀찮은 일이고 추가적으로 갱신 시에도 비슷한 짓을 반복해야 한다.  
> 그래서 나온 프로토콜이 ACME이다.  
  
ACME에서는 일단 서버는 CA에 계정을 등록해야 한다.  
(더 엄밀하게는 CA로부터 인증서를 받아주고 서버의 신원을 먼저 검증하는 앞단인 Registration Authority가 있을 수 있다.)  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/8ac93f2b9e59c598b035c6d8bf95c40c/raw/image.png)  
> [!NOTE] 그림 부언  
> 다른 트래픽 그림들과 다르게, 이 프로토콜은 완벽하게 HTTP, 즉 7계층에서 수행되는 프로토콜이다.  
> 그래서 요청과 응답을 조금 명확하게 하고자 화살표는 요청을 보내는 쪽을 나타내도록 표현했다.   
> 응답의 내용은 화살표 아래 배치했다.  
> 다만 마지막 부분 쯤에 반대로 화살표는 예외로, 시간이 조금 걸릴 수 있기 때문에 그 간극을 표현하고자 한 것이니 주의.  
  
긴단하게 말하자면 다음의 순서를 따른다고 보면 된다.  
- 서버가 인증서 주문  
	- 이 주문 때 CSR에서 채워야 하는 다양한 내용들을 바디로 전달한다.  
- CA에서 챌린지 제시  
	- CA마다 다양한 챌린지를 커스텀할 수 있는데, ACME를 만든 ISRG에서는 다음의 기본 챌린지를 제시한다.  
	- http-01  
		- 받은 토큰 값으로 `{도메인}/.well-known/acme-challenge/{토큰}` URL 200 반환하게 만들기  
		- 인증서 받기 전이니 무조건 80 포트로 http 통신  
	- dns-01  
		- 받은 토큰 값을 반환하는 `_acme-challenge.{도메인}` TXT 도메인 레코드 등록  
- 서버는 챌린지 설정 완료 후 CA에게 검사해달라 찡찡  
	- CA는 이 검사를 여러 번 수행할 수도 있다.  
- 주문이 준비됐다고 뜰 때까지, 주기적으로 서버는 폴링을 통해 주문 상태를 확인해야 한다.  
- 주문이 준비됐다면 정식으로 CSR을 건네서 인증서를 달라 찡찡댄다.  
	- CA는 초기 인증서 주문 당시 값과 CSR 값을 또 비교한다.  
	- 인증서 발행에 조금 시간이 걸릴 수 있어서 화살표를 반대로도 표시했다.  
- 인증서 url을 받으면 그 경로로 인증서를 받으면 된다!  
  
이 프로토콜에서 CA는 3개의 오브젝트와 그에 대한 상태값을 가진다.  
프로토콜 흐름 상으로 각 오브젝트를 정리하면 이렇게 된다.  
- Order - 주문  
	- 서버 요청에 해당하는 오브젝트로 pending 상태로 생성된다.  
	- 이게 ready 상태가 되면 비로소 서버는 인증서를 내놔라 요구할 수 있다.  
- Authorization - 권한  
	- 도메인에 대한 권한을 체크하는 오브젝트로 pending 상태로 생성된다.  
	- 한 Order에 대해 여러 개의 권한을 체크할 수 있다.  
	- 모든 권한이 체크돼야만 Order가 ready 상태가 된다.  
- Challenge - 챌린지  
	- 각 Authoritization 오브젝트마다 제시하는 챌린지 오브젝트로 pending 상태로 생성된다.  
	- 한 Authoritization에 대해 여러 개의 챌린지를 제시할 수 있다.  
	- 한 챌린지만 성공하더라도 권한은 체크된다.  
		- 이래서 Authrization과 Challenge가 별도의 오브젝트로 분리돼있는 것이다.  
  
아마 이 프로토콜을 직접적으로 사용할 일은 없을 것이다.  
이 프로토콜은 [[Let's Encrypt]]를 관리하는 ISRG에서 만들었는데, let's encrypt는 이걸 활용해 인증서 자동 발급과 갱신을 할 수 있는 여러 에이전트를 만들고 있기 때문이다.  
도메인에 대한 완전한 소유권이 있다면 DNS 챌린지를 하는 게 상대적으로 쉬울 수 있다.  
HTTP 챌린지의 경우 무조건 80 포트를 사용하도록 제한되고 있기 때문에, 조금 더 제한적인 상황에 부닥칠 수 있다.  
## 서버 인증서 확인 과정 - TLS  
마지막으로, 서버의 신뢰된 인증서를 어떻게 클라가 받아들이는지 보자.  
이번에도 웹 프로토콜을 기준으로 설명하고자 한다.[^16]  
어차피 PKI를 기반으로 하는 다른 프로토콜들도 결국 방식은 거의 다 비슷할 것이다.  
Transport Layer Security 프로토콜은 PKI 시스템을 활용하는 가장 많이 알려진 보안 프로토콜 중 하나이다.  
실질적으로 이 문서에서 PKI를 설명하기 위해 활용한 전반적인 구조도 최종적으로는 TLS를 설명할 목적으로 작성됐다..  
  
엄밀하게 TLS(1.2)는 두 가지 계층으로 나뉘어져 있다.  
### Record Protocol  
하위 계층에 속하는 프로토콜로, 상위 계층(핸드쉐이킹)에서 설정된 방식에 의거해 데이터 암호화, 단편화, 압축 등의 작업을 수행하는 레코드 층에 대한 프로토콜이다.  
이 프로토콜이 책임지는 것은 **기밀성과 신뢰성**이다.  
서버와 클라 간의 연결을 대칭키로 암호화하는 과정이 여기에서 일어난다.  
대부분 암호화되지만, 설정에 따라서 암호화하지 않는 통신도 허용한다.  
여기에 MAC을 활용해 무결성을 보장하는 것도 이 프로토콜에서 수행된다.  
  
이 프로토콜이 동작하기 위해 서버, 클라의 난수, 마스터 시크릿, 해시 함수, 난수생성함수 등의 값이 정해진다.  
이들을 이용해 최종적으로 서버와 클라는 반드시 다음의 값들을 같은 상태로 동기화해야 한다.  
- **마스터 시크릿** - 이 놈으로부터 실제 암호화에 사용되는 모든 대칭키, 기타 필요한 값을 꺼낸다.  
- 데이터 압축 방식  
- 데이터 암호화 방식  
- MAC 키  
- 시퀀스 번호 - 연결할 때마다 올라가는 번호  
  
이것들이 같은 상태라면 서버와 클라는 통신 시 데이터를 복호화하고, 압축을 풀고, 무결성 검증을 할 수 있게 된다.  
이 프로토콜에서 안전성이 전부 확보되기 때문에, 이 프로토콜이 성립하기 위해 각종 필요한 값들을 서버와 클라가 공유하는 과정이 필요하다.  
### Handshaking Protocol  
그것이 바로 이 프로토콜이다!  
핸드쉐이킹 프로토콜은 위 레코드 프로토콜을 세팅하고 검증하기 위한 절차를 담은 프로토콜이다.  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/cfe9a85fa0cc4232da2eb27e32a35cb3/raw/image.png)  
핸드쉐이킹 과정은 대충 4번 오고 간다.  
여기에 에러가 발생하거나, 중간에 암호 방식을 교체한다던가 하는 각종 추가적인 과정이 있을 수 있긴 한데, 이것들은 생략했다.  
그런 트래픽은 항상 발생하는 것은 아니기도 하고, 핵심 플로우에 해당하진 않는다.  
빨간색 블록은 클라이언트의 신원까지 검증하는 mutual TLS로 핸드쉐이킹이 이뤄질 때만 발생하는 추가적인 트래픽이다.  
  
- 클라이언트 헬로  
	- 클라이언트에서 사용 가능한 각종 버전과 알고리즘 리스트를 전달한다.  
	- 이때 클라 난수를 같이 전달한다.  
- 서버 헬로  
	- 클라에게 받은 버전이나 알고리즘 등에서 서버 측도 사용 가능한 것을 선별하여 보낸다.  
		- 여기에는 최대한 최신 버전을 쓴다, 뭐 이런 몇 가지 규칙이 있다.  
	- 서명 받은 서버의 인증서를 전달한다.  
		- 이때 서명한 기관이 상위 기관을 두는 식으로 꼬리를 문다면 체인 인증서를 한꺼번에 전부 전달한다.(이거 알아보고 싶어서 RFC까지 뒤졌다)  
	- 이때 서버 난수를 같이 전달한다.  
	- 대칭키 교환 방법 중, 디피 헬만 알고리즘이 선택된 경우에는 대칭키 교환을 위한 추가 메시지를 보낸다.  
- 클라 피니쉬  
	- **클라이언트는 인증서 검증을 시작한다.**  
		- **최상단에 서버 인증서가 있고, 여기에서 발행자(Issuer) 값을 꺼낸다.**  
		- **체인 인증서가 이 다음에 있으면 발행자 값을 토대로 서버를 인증하는 인증 기관이 적힌 인증서를 찾는다.**  
		- **그 인증서에는 인증 기관의 공개키가 적혀있을 테니, 그것으로 서버 인증서를 검증한다.**  
		- **꼬꼬무한다..**  
		- **마지막 남은 인증서의 발행자 값은 로컬 환경에 저장된 루트 CA의 인증서로 검증한다.**  
	- 검증을 마치고 Pre Master Secret 값을 만들어 서버의 공개키로 암호화하여 보낸다.  
		- 디피 헬만 방식이라면 암호화를 거치지 않는다고 한다.  
- 서버 피니쉬  
	- 문제 없으면 그냥 끝 날린다!  
  
레코드 프로토콜에서 필요한 값 중 하나는 마스터 시크릿이란 값이었다.  
이 값은 클라 난수 + 서버 난수 + Pre Master Secret을 합쳐 만들어진다.  
각 난수는 암호화되지 않았지만, Pre Master Secret는 클라가 서버의 공개키로 암호화했기에 서버만이 복호화할 수 있다.  
이를 바탕으로 Master Secret을 만들면 둘만의 대칭키를 만드는데 활용할 수 있다!  
  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/522e8b796fbe7b3fe95598c0b27e180d/raw/image.png)  
TLS는 그 자체로도 복잡하고 중요한 포인트가 많다.  
그러나 PKI 구조를 다루고자 하는 문서이므로, 인증서 검증 구조가 어떻게 되는지 부분에 초점을 맞추겠다.   
그러기 위해서는 클라가 인증서를 어떻게 검증하는지에 관한 부분을 봐야 한다.  
서버는 자신을 증명할 수 있는 인증서와, 인증서에 사인해준 인증 기관의 인증서와, 그 인증서에 사인해준 인증 기관에 인증서와, 그 인ㅈ.. 를 모두 보낸다.  
그러면 클라는 그 인증서들을 전부 받아보고 순서대로 인증서들을 뜯어보며 검증한다.  
먼저 뜯은 인증서는, 이 다음의 인증서에 담긴 공개키로, 그 인증서는 또 다음의 담긴 공개키로..  
이런 식으로 체인을 이어나가고 마지막 인증서는 클라 로컬에 있는 루트 CA 인증서를 이용해 검증한다.  
이 과정을 통해 비로소 클라는 서버의 공개키를 믿을 수 있게 되고, 이 공개키로 핵심 비밀 변수인 Pre Master Secret을 암호화해서 보낼 수 있게 된다!  
#### TLS 번외  
몇 가지 재밌는 특징들을 정리해봤다.  
- 서버 키 교환 메시지  
	- 서버 헬로 과정에서 이 값은 디피헬만류의 대칭키 교환을 택할 때만 선택적으로 보내진다.  
- 클라 키 교환 메시지  
	- 이 트래픽은 PMS를 교환하는 과정으로서, 필수로 이뤄진다.  
- 클라 인증서 검증  
	- mTLS를 하는 경우, 클라의 인증서를 받은 서버는 인증서 검증 과정을 거친다.  
	- 그러나 다수를 대상으로 하는 서버인 만큼, 서버는 핸드쉐이킹을 하는 클라가 중간에 바꿔치기 되지 않았는지 꼼꼼하게 확인한다.  
	- 이를 위해 클라는 그간 핸드쉐이킹 과정 상에서 오고간 모든 메시지를 자신의 개인키로 암호화하여 서버에게 보낸다!  
	- 그럼 서버는 일단 정말 클라가 개인키를 가지고 있단 것을 확인하고, 또한 여태 통신한 클라가 맞다는 것까지 확인한다.  
  
이외에 서브 프로토콜로, 알람 프로토콜, 암호 스펙 변경 프로토콜, 어플리케이션 데이터 프로토콜이 있는데, 이것들은 핵심 동작이라 하기엔 부족한 것 같아서 뺀다.  
참고로 세션 유지 및 재개에 대한 절차도 잘 마련돼있다.  
세션 시간이 오래 되거나 데이터가 더 이상 오고가지 않을 때, 클라에게 서버가 헬로 좀 해달라 요청(HelloRequest)하기도 한다.  
이런 내용까지 다루지는 않겠다.  
# 간단한 실습  
이론적인 내용들을 봤으니, 마지막으로는 간단하게 직접 인증서를 만드는 실습을 하고자 한다.  
  
다음의 순서로 실습을 진행하며, 환경은 [리얼리눅스](https://reallinux.co.kr/)를 활용한다(속도 빨라서 조아용).  
- 루트 키 페어 생성 - rsa 2048  
- 루트 인증서 생성  
- 사용할 키 페어 생성 - ed20119  
- csr 파일 생성  
- 루트의 서명을 받은 인증서 생성  
- nginx에 curl 날려 통신 확인  
## 사전 준비  
https://reallinux.co.kr/  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/9a7bce63a8bf172d0738483e0c5160df/raw/image.png)  
로그인을 하고 나면 실습 환경에서 실습을 진행할 수 있다.  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/b28c009b8a4757960712d97d5b2f95ff/raw/image.png)  
우분투 22를 쓸 것이다.  
```sh  
sudo su  
# 웹사이트에 나온 비밀번호 입력  
```  
실습 편의를 위해 루트 계정으로 접속한다.  
## 루트 키페어 및 인증서 생성  
```sh  
# rsa 방식으로 2048 비트의 개인키 생성  
openssl genrsa -out root.key 2048  
```  
첫번째로, 루트가 될 키페어를 만들어본다.  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/23d25c6da14b66dff53f4f578bef6d0e/raw/image.png)  
매우 긴 개인키가 생성된다.  
```sh  
# 해당 키를 까보고 싶으면 이렇게 하면 정보를 볼 수 있다.  
openssl rsa -in root.key -text -noout  
```  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/1e64bd1d9966fb932bbbeaf67e3444e0/raw/image.png)  
그러나 사실 이건 개인키만 들어있는 것이 아니라, 키 페어를 만드는데 필요한 각종 값들이 함께 들어있다.  
실질적으로 공개키가 필요하진 않은데, 왜냐하면 실제 만들어졌다는 개인키에 공개키를 만들기 위한 각종 정보가 포함되어있기 때문이다.  
```sh  
# 굳이 공개키를 직접 꺼내보고 싶다면  
openssl rsa -in root.key -pubout -text  
```  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/61cef9b1465057d028cec923bbaa134c/raw/image.png)  
이렇게 인자를 주면 마지막에 실제 계산되는 공개키 정보도 확인할 수 있다.  
  
```sh  
openssl req -new -x509 -key root.key -out root.pem -subj /CN=root  
```  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/67f3cc4ef0cf22aa6c571a48fafc26d3/raw/image.png)  
이제 자체서명 인증서를 만든다.  
원래는 CSR 파일을 만들어야 하나, `-x509` 인자를 주면 바로 인증서가 만들어지게도 할 수 있다.  
```sh  
openssl x509 -in root.pem -noout -text  
```  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/8a888006632ebeb5995be3785f0f25af/raw/image.png)  
인증서 내용을 확인해보면, Issuer와 Subject 정보가 같은, 자체 서명 인증서가 발급된 것을 확인할 수 있다.  
## 사용할 키페어 생성 및 인증서 발급  
```sh  
openssl genpkey -algorithm ed25519 -out server.key  
```  
다음으로는 이 루트 인증서에 서명을 받은 서버 인증서를 만들 것이다.  
이때 모바일 환경에서 많이 쓰이는, 차원곡선함수(ECC)를 활용한 공개키 알고리즘을 사용해본다.  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/27b29624914cc48416b12b381295024d/raw/image.png)  
진짜 어마무시하게 키 크기가 줄어든 것을 확인할 수 있다.  
```sh  
openssl req -new -key server.key -subj "/CN=nginx-test" -out server.csr  
```  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/80206c7717357dcecb744fde5ade4171/raw/image.png)  
이번에는 제대로 CSR, 즉 인증서 서명 요청 파일을 만들어본다.  
```sh  
openssl req -in server.csr -text -noout  
```  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/7cc67effbe94c8e6d93a3a3ad42ef54f/raw/image.png)  
내용을 까보는 것은 위에서 본 방식과 동일한데, 크기가 정말 작다.  
이제 이 인증서를 가지고 루트 인증서와 개인키를 이용해 서명을 진행하면 된다.  
```sh  
# 인증서 발급  
openssl x509 -req -in server.csr -CA root.pem -CAkey root.key -out server.pem   
```  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/52daf2f4d9dd596b73d23536b55ef6a5/raw/image.png)  
성공적으로 인증서를 발급 받았다.  
```sh  
openssl x509 -in server.pem -text -noout  
```  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/c2a255612c33cc38640d2624161e62b8/raw/image.png)  
이번에는 발행자는 root, 주체는 nginx-test인 인증서가 만들어진 것을 확인할 수 있다.  
```sh  
# 제대로 서명됐는지 검증  
openssl verify -CAfile root.pem server.pem  
```  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/6979eca7ccc61dd706f174c4d0fcc48b/raw/image.png)  
이런 식으로 루트의 인증서를 통해 서버의 인증서를 검증할 수 있다.  
실제 tls 통신이 발생할 때도 이런 식으로 상위 인증서를 가지고 서버의 인증서를 검증하는 방식으로 검증이 진행된다.  
## nginx 통신 테스트  
```sh  
apt update -y && apt install -y nginx  
curl http://localhost  
curl https://localhost  
```  
이제 만들어진 인증서를 가지고 nginx 서버를 만들어보자!  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/8f3a01bf3ecce90d504e0ee856a29412/raw/image.png)  
nginx를 설치하면 자동으로 systemctl에 등록되어 서비스가 동작한다.  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/a6682398f4fbfc4c462b8bdbd3265a8e/raw/image.png)  
아직 tls 설정이 되어있지 않기 때문에 그냥 https로 요청하면 소용이 없다.  
```sh  
vi /etc/nginx/sites-enabled/default  
```  
nginx의 기본 파일을 수정한다.  
```  
        listen 443 ssl default_server;  
        ssl_certificate /home/reallinux/server.pem;  
        ssl_certificate_key /home/reallinux/server.key;  
```  
이 값을 추가해준다.  
위에서 만든 인증서와 개인키를 사용하는 것이다.  
```  
systemctl restart nginx  
```  
그리고 설정 파일을 리로드할 수 있도록 프로세스를 재시작해준다.  
  
```sh  
curl -k https://localhost  
```  
k 옵션은 신뢰할 수 없는 서명을 받은 서버의 인증서라고 해도 이를 무시하고 통신이 가능하게 하는 설정이다.  
이렇게 하면 통신은 가능하지만, 우리는 이 상태로 만족할 수 없다.  
  
curl을 날릴 때 상대를 검증하는데 사용될 루트 인증서를 직접 지정해서 curl이 우리의 nginx를 신뢰할 수 있도록 해보자.  
```sh  
curl --cacert root.pem https://localhost  
```  
이미 루트 인증서를 만들었기 때문에, 이것을 그대로 활용해주면 된다.  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/4e56a8a4e9d99acc56521220ce5dbedb/raw/image.png)  
그러나, 위에서 CN(Common Name)을 nginx-test로 했기에 검증 단계에서는 실제 서버의 도메인이 nginx-test이길 기대한다.  
```sh  
echo "127.0.0.1 nginx-test" >> /etc/hosts  
curl --cacert root.pem https://nginx-test  
```  
그렇다면 dns 로컬 파일을 직접 수정해서 nginx-test라는 도메인에 대한 요청이 localhost로 가도록 설정하면 된다.  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/2d9d0ac9fad0b8bd12ac1df97e02c6c3/raw/image.png)  
석세스!  
  
# 결론  
이렇듯 PKI는 공개키를 신뢰할 수 있도록 하기 위해 고안된 시스템 구조이다.  
이 시스템 덕분에, 전송 간 암호화를 보장하기 위해 비대칭키를 안전하게 활용할 수 있게 된다.  
여기에 공개키를 이용한 디지털 서명의 검증과, 검증 체이닝을 통해 신원까지도 검증한다.  
![image.png](https://841lgfvhej.execute-api.ap-northeast-2.amazonaws.com/default/image?url=https://gist.githubusercontent.com/Zerotay/b9dd129b25243d1b6d06f48677d39669/raw/image.png)  
이를 기반으로 TLS가 성립하고, 여타 안전한 통신을 위한 다양한 프로토콜이 구현된다.[^13]  
그래서 먼저 이게 PKI가 뭔지 이해하고, 다른 프로토콜들을 차근차근 이해해나가는 것이 중요하다고 생각한다.  
# 관련 문서  
```dataview  
table  
noteType,  
dateformat(file.ctime, "yyyy-MM-dd") as created  
from #security   
where !is-folder  
sort noteType, created  
```  
# 참고  
  
[^5]: https://namu.wiki/w/RSA%20%EC%95%94%ED%98%B8%ED%99%94  
[^6]: http://cryptostudy.xyz/crypto/article/3-ECC-%ED%83%80%EC%9B%90%EA%B3%A1%EC%84%A0%EC%95%94%ED%98%B8  
[^7]: https://hyeo-noo.tistory.com/415  
[^8]: https://en.wikipedia.org/wiki/Kerberos_(protocol)  
[^9]: https://danyoujeong.tistory.com/222  
[^10]: https://en.wikipedia.org/wiki/Message_authentication_code#An_example_of_MAC_use  
[^11]: https://en.wikipedia.org/wiki/HMAC  
[^12]: https://blog.humminglab.io/posts/tls-cryptography-7-diffie-hellman/  
[^13]: https://en.wikipedia.org/wiki/Public_key_infrastructure#Uses  
[^14]: https://en.wikipedia.org/wiki/X.509  
[^15]: https://datatracker.ietf.org/doc/html/rfc8555  
[^16]: https://datatracker.ietf.org/doc/html/rfc5246#autoid-32  
  