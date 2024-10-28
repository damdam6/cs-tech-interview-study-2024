
# NEXT.JS 렌더링 방식 알아보기

NEXT.JS의 렌더링 방식이 다양하다는 것을 한 번 정리해보고 싶어 해당 내용을 조사하였다.

## SSR과 CSR

기본적으로 SSR와 CSR의 개념이 무엇인지 가볍게 살펴보고 넘어가고자 한다.

### SSR과 CSR의 기본 개념

- CSR = Client Side Rendering
    - SPA의 기본적인 렌더링 방식이자 React에서 일반적으로 활용하는 렌더링 방식
    - 빈 HTML을 웹 서버가 client에 전달해주게 되고, 브라우저에서 javascript 번들을 통해 화면을 그리게 된다.
- SSR = Server Side Rendering
    - 서버에서 기본적인 화면을 그려주는 방식(HTML이 채워져 있다.)
    - 과거에는 SSR 기반 페이지는 사용자와 인터렉션에 한계가 있었으나 (javascript가 연결 되어 있지 않으므로) 현대에 와서는 렌더링 요소가 있는 HTML과 client 단에서  javascript를 다운로드 하고 “hydrate”하는 과정을 통해 동적 인터렉션이 가능한 형태로 변모하였다.

### SSR과 CSR의 특징

|  | SSR | CSR |
| --- | --- | --- |
| 초기 로딩 속도 | 빠름 | 느림 |
| SEO 최적화 | 최적화 | 별도 처리 필요 |
| 서버 부하 | 증가 | 적음 |
| 초기 로딩 후 상호작용 | 느림 | 빠름 |
- 초기 로딩 속도
    - 주로 FCP로 측정 되는 자료
        - FCP : First Contentful Paint
    - SSR은 미리 렌더링 된 HTML을 제공하므로 브라우저는 즉시 콘텐츠를 렌더링할 수 있다.
    - CSR은 브라우저가 javascript 파일을 다운로드 해서 콘텐츠를 생성하기 때문에 FCP가 느릴 수 밖에 없다.
- 서버 부하
    - 여기서의 서버는 HTML을 생성하는 FE 서버를 의미함.
    - SSR은 페이지 요청마다 HTML을 생성하기 때문에 서버 부하가 크다.
    - CSR은 HTML을 생성할 필요가 없기 때문에 부담이 없다. (주로 FE 서버 자체가 없기도 하다. 일반 웹서버로 정적 파일만 전달함.)
- 초기 로딩 후 상호작용
    - 주로 TTI 로 측정된다.
        - TTI : Time to Interactive
        - 페이지가 완전히 로드 되어 사용자와 상호 작용할 수 있게 되는데 걸리는 시간
    - SSR은 초기 로딩 후 상호작용이 느리다. hydration작업을 거치면서 연결에 시간이 걸릴 수 있기 때문이다.
    - CSR은 로딩이 완료되면 즉시 상호작용이 가능하다.

## NEXT.JS란?

### NEXT.JS에 관하여 알려진 일반적인 사실

- NEXT.JS는?
    - React를 기반으로 한 Framework
    - 자동 코드 스플리팅 & 이미지 사이즈 등 성능 최적화
    - (알려진 바에 따르면) SSR을 지원하여 SEO 최적화가 가능함.

- 이때 생기는 궁금증
    
    ![image.png](image.png)
    
    - 정답 : 꼭 그렇지만은 않습니다.
        
        ⇒ 이를 이해하기 위해서 NEXT.JS의 렌더링 방식을 알아볼 필요성이 있습니다.
        

## NEXT.JS의 렌더링 과정

### Pre-rendering

![image.png](image 1.png)

- Pre-rendering이란?
    - 모든 페이지를 미리 렌더링 한다.
        
        = HTML을 미리 생성한다.
        

### Pre-rendering의 방식 SSR vs SSG

- SSR with Pre-rendering
    
    ![image.png](image 2.png)
    
    - 매 요청마다 HTML 생성한다.
    - 여기서 `매 요청` 은 접속 혹은 새로 고침 시 발생하는 매 요청을 의미
- SSG with Pre-rendering
    
    ![image.png](image 3.png)
    
    - SSG = Static Site Generation
    - 빌드 타임에 HTML이 생성되고 이를 재사용한다.
        - 즉, 빌드 시점 이후에는 서버에게 생성 요청을 보내지 않는다.

### SSG와 ISR까지.

- SSG는
    - `빌드 타임에 HTML이 생성되고 이를 재사용한다.`
        - 매번 서버에 생성 요청하지 않기 때문에 속도가 빠르다
        - 생성 요청이 오지 않기 때문에 서버 부담도 줄어든다. (SSR 대비)
        - **빌드 타임 이후 변화한 데이터가 반영될 수 없다.**
    - SEO 최적화가 가능하나… 한계가 존재한다.
- ISR
    - Incremental Static Regeneration
    - SSG의 한 방식이라고 볼 수 있다.
    - 기존 SSG의 단점을 보완하기 위한 방식
    - `빌드 타임에 HTML이 생성되고 이를 재사용한다.`
        
        BUT **일정 시간 마다 HTML을 재생성한다.**
        
        ⇒ 이는 곧 새로운 데이터 반영이 가능함을 의미한다.
        
    - HTML 생성 주기가 SSR보다 길기 때문에 서버 부하가 적은 편이다!

- 공식 문서의 ISR 설명 뜯어보기
    
    > When a request is made to a page that was pre-rendered at build time, it will initially show the cached page.
    > 
    > - Any requests to the page after the initial request and before 10 seconds are also cached and instantaneous.
    > - After the 10-second window, the next request will still show the cached (stale) page
    > - Next.js triggers a regeneration of the page in the background.
    > - Once the page generates successfully, Next.js will invalidate the cache and show the updated page. If the background regeneration fails, the old page would still be unaltered.
    - 빌드 시점에 미리 렌더링 된 페이지 요청이 들어오면, 처음에는 캐시된 페이지가 표시된다.
    - 첫 요청 후 10초(`revalidate`) 이내에 페이지에 대한 모든 요청은 캐시된 페이지에 기반해 반환 된다.
    - 10초 후 NEXT.JS는 백그라운드에서 페이지를 재생성한다.
        - 재생성 되는 동안에는 캐시된 페이지에 기반해 요청이 응답된다.
    - 페이지 재생성에 성공하면 NEXT.JS는 기존 캐시를 무효화하고 업데이트 된 페이지를 표시한다.
    
    ⇒ 생성된 페이지를 캐싱하고 있다는 개념이 중요하다.
    

### 렌더링 방식 선택하기

### Hybrid

- NEXTJS의 Hybrid
    - NEXT.JS는 여러가지 렌더링 방식을 혼합해서 사용이 가능하다.
    - CSR + SSR + SSG(ISR) 모두 활용 가능하다.
    - Hybrid가 NEXT.JS의 강력한 이점
- 렌더링 방식 사용 예시
    - 서비스 소개 페이지
        - SSG
            - 페이지가 변경될 일이 거의 없다.
            - 모든 사용자가 동일한 페이지를 본다.
    - 메인 페이지
        - ISR
            - 대부분의 페이지가 고정되어 있으며 페이지의 일부만 주기적으로 수정된다.
                - ex. 주간 인기글 등
            - 모든 사용자가 동일한 페이지를 본다.
    - 사용자 프로필 페이지
        - SSR + CSR
            - 사용자 별로 페이지를 생성해야 한다. (SSR의 특징 활용)
            - 사용자가 등록한 이미지에 따라 프로필 이미지 등이 빠르게 변경 될 필요성이 있다. (CSR 적용)
