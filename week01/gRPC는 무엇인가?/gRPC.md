# RPC (Remote Procedure Call)

RPC는 네트워크 상 원격에 있는 서버의 서비스를 원격으로 호출하는데 사용되는 프로토콜로 IDL(Interface Definition Language)을 기반으로 이에 해당하는 Skeleton 과 Stub을 통해 프로그래밍 언어에서 호출해서 사용하는 방식

- **Stub(스텁)**
    - 서버와 클라이언트가 서로 다른 주소 공간을 사용하므로 함수 호출에 사용된 매개 변수를 반드시 변환해주어야 하는데, 이 변환을 담당하는 것이 스텁
    - 스텁(Stub)은 매개변수 객체를 메세지로 변환/역변환하는 레이어이며 **클라이언트의 스텁**과 **서버의 스텁**으로 나눠진다.
    - **클라이언트의 스텁**
        - 함수 호출에 사용된 파라미터의 변환 및 함수 실행 후 서버에서 전달된 결과의 변환을 담당
    - **서버의 스텁**
        - 클라이언트가 전달한 매개 변수의 역변환(Unmarshalling, 언마샬링) 및 함수 실행 결과 변환을 담당

![Untitled](https://velog.velcdn.com/images/qmflf556/post/f4a8f0f3-7761-4393-a2a4-ebe8936a5dfd/image.png)

### RPC의 작동 방식

1. IDL 을 사용하여 호출 규약 정의
    1. 함수명, 인자, 반환값에 대한 데이터형이 정의된 IDL 파일을 rpcgen 으로 컴파일시, stub code 가 자동으로 생성됨
2. Stub code 에 명시된 함수는 원시코드의 형태로 상세 기능은 server에서 구현된다.
3. Client stub은 RPC runtime 을 통해 함수를 호출
4. Server는 수신된 함수 호출에 대한 처리 후 결과 값을 반환한다.
5. 최종적으로 Client는 Server의 결과 값을 반환받고, 함수를 Local에 있는 것처럼 사용

### RPC의 한계

RPC는 매우 획기적인 방법론이었고 로컬에서 제공하는 빠른 속도와 분산 프로그래밍에서도 사용할 수 있어 좋았으나, 구현의 어려움과 지원 기능의 한계 등으로 인해 제대로 활용되지 못했다. 그러던 중, 데이터 통신을 익숙한 Web을 활용해보려는 시도로 이어졌고 이 자리를 REST 가 차지하게 되었다.

# gRPC (google Remote Procedure Calls)

gRPC는 Google에서 개발한 오픈소스 고성능 RPC 프레임워크이며, 몇 가지 최적화와 함께 기존 RPC를 구현하고 확장하는 시스템이다. <예) 또 다른 API를 사용할 때에 RPC의 경우, 개발자가 직접 RPC 개념을 HTTP 프로토콜에 매핑해야된다. 하지만 gRPC는 기본 HTTP 통신을 추상화한다. 이러한 부분들로 인해 gRPC는 다른 RPC 구현보다 더 빠르고 쉽게 구현할 수 있다.>

gRPC를 이용하면 원격에 있는 애플리케이션의 메서드를 로컬 메서드인 것처럼 직접 호출할 수 있기 때문에 분산 애플리케이션과 서비스를 보다 쉽게 만들 수 있다.

### gRPC의 특징

1. **크로스 플랫폼**: 다양한 언어와 플랫폼에서 사용 가능
2. **HTTP/2 사용**: 전송 프로토콜로 HTTP/2를 사용하여 성능을 향상
3. **Protocol Buffers**: IDL(Interface Definition Language)로 Protocol Buffers를 사용
4. **Netty**: 내부적으로 Netty(소켓통신)을 사용

### gRPC 서비스

```protobuf
/* gRPC 구현을 위한 proto 파일 작성 예시 */
/* hello.proto */
//
service HelloService {
  rpc SayHello (HelloRequest) returns (HelloResponse);
}

message HelloRequest {
  string greeting = 1;
}

message HelloResponse {
  string reply = 1;
}
```

- **Unary RPC (단일 호출)**
    - 클라이언트가 단일 함수를 서버에 보내고 단일 함수 호출과 같은 단일 응답을받는 단항 RPC
    - HTTP와 유사하게 stateless한 통신방식
- **Server streaming RPC (서버 스트리밍)**
    - 클라이언트가 서버에 요청을 보내고 스트림을 가져 와서 일련의 메시지를 다시 읽는 서버 스트리밍 RPC
    - 클라이언트는 더 이상 메시지가 없을 때까지 리턴 된 스트림에서 읽음
- **Client streaming RPC (클라이언트 스트리밍)**
    - 클라이언트가 일련의 메시지를 작성하고 제공된 스트림을 사용하여 다시 서버로 보내는 클라이언트 스트리밍 RPC
    - 클라이언트가 메시지 쓰기를 마치면 서버가 메시지를 읽고 응답을 반환 할 때까지 대기
- **Bidirectional streaming RPC (양방향 스트리밍)**
    - 양쪽에서 읽기 / 쓰기 스트림을 사용하여 메시지를 주고 받는 양방향 스트리밍 RPC
    - 두 스트림은 독립적으로 작동하므로 클라이언트와 서버는 원하는 순서대로 읽고/쓰기가 가능
    - 이러한 것들로 인해 gRPC 서비스는 폴링을 사용 하지 않고 실시간으로 메시지를 푸시할 수 있다.
    
    ```protobuf
    rpc BidiHello(stream HelloRequest) returns (stream HelloResponse){}
    ```
    

## Protocol Buffers

구글에서 만든 오픈소스로 XML문제점을 개선하기 위해 만들어진 IDL이며, 직렬화된 데이터 구조를 사용

### 특징

- 이진 데이터를 사용하기 때문에 파일의 크기가 작고, 속도가 빠르다.
    
    ```json
    {
        "userName": "Martin",
        "favouriteNumber": 1337,
        "interests": ["daydreaming", "hacking"]
    }
    ```
    
    ```protobuf
    message Person {
        required string user_name        = 1;
        optional int64  favourite_number = 2;
        repeated string interests        = 3;
    }
    ```
    
    ![Untitled](https://martin.kleppmann.com/2012/12/protobuf.png)
    
    <aside>
    💡
    
    위와 같이 같은 정보에 대해서 텍스트 기반인 JSON의 경우 82byte가 소요되는데 반해, 직렬화된 프로토콜 버퍼는 필드 번호, 유형 등을 1byte로 받아서 식별하고, 주어진 length 만큼만 읽도록 하여 33byte만 필요하게 된다.
    
    </aside>
    
- 다양한 언어를 지원한다.
- 이미 배포한 서비스를 중단할 필요 없이 데이터 구조를 바꿀 수도 있다.
- JSON 파일을 프로토콜 버퍼 파일 포맷으로 전환이 가능하고, 반대의 경우도 가능하다.
- XML과 비교해서 동일한 메세지를 인코딩하는데 시간이 단축되고, 코드가 직관적이다.

### gRPC의 작동 방식

![Untitled](https://velog.velcdn.com/images/qmflf556/post/32a644e0-c396-4b32-9dbf-ad880b7b9649/image.png)

1. .proto 파일에서 직렬화하려는 데이터 구조를 정의한다.
    1. 프로토콜 버퍼는 여러 언어를 지원하기 때문에, 특정 언어에 종속성이 없는 형태로 데이터 타입을 정의하게 되는데 이 파일을 `proto file`
    2. 이렇게 .proto suffix가 붙는 파일들은 서버와 클라이언트가 서로 공유하는 파일로, gRPC를 통해 서로 어떻게 통신할 것인지에 대해 정의한다.
    
    ```protobuf
    // 서비스 정의는 프로토콜 버퍼의 정의로 시작. 보통 proto3
    syntax = "proto3";
    
    option csharp_namespace = "GrpcGreeter"; //C#
    
    // option go_package = "maum-backend-protobuf/gen/go/protos/notification/v1"; //Golang
    
    // 패키지 이름은 프로토콜 메시지 타입 사이의 이름 충돌을 방지하고자 사용, 코드 생성에도 사용
    package greet;
    
    // gRPC 서비스의 서비스 인터페이스를 정의
    // The greeting service definition.
    service Greeter {
      // Sends a greeting. 인사를 건네는 원격 메서드
      rpc SayHello (HelloRequest) returns (HelloReply);
    }
    
    // The request message containing the user's name. 인사 요청 정보의 메시지 형식, 타입을 정의
    message HelloRequest {
      string name = 1;
    }
    
    // The response message containing the greetings.
    message HelloReply {
     repeated string message = 1;
    }
    ```
    
2. 이렇게 작성된 `proto file`을 `protoc 컴파일러`로 컴파일 하면 데이터에 접근할 수 있는 각 언어에 맞는 형태의 **데이터 클래스**를 생성해준다.

### 장점

- gRPC는 대부분의 아키텍쳐에 사용할 수 있지만 MSA에 가장 적합한 기술
- 많은 서비스 간의 API 호출로 인한 성능 저하를 개선하며 보안, API 게이트웨이, 로깅 등을 개발하기 쉽다.

### 단점

- gRPC 에서는 아직 브라우저 관련 API가 제공되지 않기 때문에 브라우저에서 직접 gRPC 서비스를 호출하는 것이 불가능(**gRPC-WEB** 사용)
- 기존 데이터 통신과 다르게 텍스트 기반 (XML, JSON)이 아니라 Encoding 된 Binary Sream 이기 때문에 사람이 읽기 어렵다.

## gRPC-WEB

gRPC를 브라우저에서 사용가능하도록 브라우저 지원을 추가한 스펙입니다. Google 팀과 Improbable 팀이 2016년부터 독립적으로 각각의 저장소에 다른 방식으로 구현하였으며, 스펙도 완전히 준수하지는 않았습니다.

![Untitled](https://velog.velcdn.com/images%2Fkyusung%2Fpost%2F0c9dedb3-f945-406c-ae78-92af8ef426c4%2F1_mAkZWyRD9gKyBEOaqEFm-A.png)

- Improbable 팀은 Typescript 기반으로 @improbable-eng/grpc-web을 구현
- Google팀은 Google의 클로저 라이브러리 베이스로 Javascript 기반의 grpc-web을 구현

![Untitled](https://velog.velcdn.com/images%2Fkyusung%2Fpost%2Fbb7da913-9569-4185-a55d-b5f14bd5fbbb%2FThe%20state%20of%20gRPC%20in%20the%20browser%202020-03-11%2018-07-02.png)

## gRPC와 RPC의 차이점

1. **성능**:
    - gRPC는 HTTP/2와 Protocol Buffers를 사용하여 더 효율적인 데이터 전송이 가능하다.
    - 전통적인 RPC는 텍스트 기반 형식을 사용하여 상대적으로 덜 효율적이다.
2. **데이터 형식**:
    - gRPC는 이진 데이터 형식을 사용하여 더 빠른 직렬화/역직렬화가 가능하다.
    - RPC는 일반적으로 JSON과 같은 텍스트 기반 형식을 사용한다.
3. **언어 지원**:
    - gRPC는 다양한 프로그래밍 언어를 지원
    - RPC는 구현에 따라 언어 지원이 제한적일 수 있다.
4. **확장성**:
    - gRPC는 로드 밸런싱, 추적, 인증 등의 기능을 플러그인 형태로 지원하여 확장성이 뛰어나다.
    - 전통적인 RPC는 이러한 기능들을 별도로 구현해야 한다.

## gRPC와 REST의 차이점

|  | gRPC | REST API |
| --- | --- | --- |
| 무엇인가 | RPC 클라이언트-서버 통신 모델을 기반으로 API를 만들고 사용하는 시스템 | 클라이언트와 서버 간의 정형 데이터 교환을 정의하는 일련의 규칙 |
| 설계 접근 방식 | 서비스 지향 설계. 클라이언트는 서버에 서버 리소스에 영향을 줄 수도 있고 아닐 수도 있는 서비스 또는 기능을 수행해 줄 것을 요청 | 엔터티 지향 설계. 클라이언트에서 서버에 리소스의 생성, 공유 또는 수정을 요청 |
| 통신 모델 | 단방향, 단일 서버-다중 클라이언트, 단일 클라이언트-다중 서버, 다중 클라이언트-다중 서버 등 다양한 옵션이 존재 | 단방향. 단일 클라이언트가 단일 서버와 통신 |
| 구현 | 작동하려면 클라이언트 측과 서버 측 모두에 gRPC 소프트웨어가 필요 | 공통 소프트웨어 없이 클라이언트 측과 서버 측에서 다양한 형식으로 구현이 가능 |
| 데이터 액세스 | 서비스(기능) 직접 호출 | 리소스를 정의하는 URL 형태의 여러 엔드포인트로 요청 |
| 반환된 데이터 | Protocol Buffer 파일에 정의된 서비스의 고정된 반환 유형으로 반환 | 서버에서 정의하는 고정 구조(일반적으로 JSON)로 반환 |
| 클라이언트-서버 커플링 | 긴밀하게 결합. 클라이언트와 서버 모두에 데이터 형식을 정의하는 동일한 Protocol Buffer 파일이 필요 | 느슨하게 결합. 클라이언트와 서버는 내부 세부 정보를 인식하지 못함 |
| 양방향 스트리밍 | 있음 | 없음 |
| 가장 적합한 용도 | 고성능 또는 데이터 사용량이 많은 마이크로서비스 아키텍처(MSA) | 리소스가 잘 정의되어 있는 단순한 데이터 소스 |

**참고**

https://aws.amazon.com/ko/compare/the-difference-between-grpc-and-rest/

[https://velog.io/@letsdev/이-얼마나-쉽고-빠른-gRPC-1-Concept](https://velog.io/@letsdev/%EC%9D%B4-%EC%96%BC%EB%A7%88%EB%82%98-%EC%89%BD%EA%B3%A0-%EB%B9%A0%EB%A5%B8-gRPC-1-Concept)

https://velog.io/@qmflf556/gRPC

https://velog.io/@kyusung/grpc-web-example

https://etloveguitar.tistory.com/m/107

https://yoongrammer.tistory.com/16

https://velog.io/@jo_love/TIL82.-protocol-buffer
