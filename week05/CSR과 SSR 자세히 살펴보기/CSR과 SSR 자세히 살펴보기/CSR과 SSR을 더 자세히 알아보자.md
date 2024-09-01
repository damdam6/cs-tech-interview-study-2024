# CSR과 SSR을 더 자세히 알아보자


### 웹페이지 로드 순서

- 고전적인 웹사이트(JSP/PHP) 로딩 방식
    
    ![Untitled](image/Untitled.png)
    
    1. client(브라우저)가 요청을 보냅니다.
    2. WAS(웹서버)는 내부 로직을 통해 동적으로 HTML 파일을 생성합니다.
        - DB로 데이터 요청을 보내고 HTML 파일에 데이터를 기록합니다.
        - 파일 생성을 위해 기본적인 HTML 템플릿을 쓰기도 합니다.
    3. 완성된 HTML 파일을 브라우저로 반환합니다. 브라우저는 HTML 파일을 파싱하여 DOM을 생성합니다. 
        
        또  브라우저는 HTML 내부에 `<script>` 태그를 통해 번들 된 javascript파일을 요청합니다. (<link>를 통해 CSS도 요청합니다.)
        
    4. 다운로드 받은 파일을 실행하여 사용자 인터렉션이 가능한 형태로 화면을 구성합니다.
    
- CSR을 통한 페이지 로드 방식
    
    ![Untitled](image/Untitled%201.png)
    
    1. client(브라우저)가 서버로 요청을 보냅니다.
    2. Nginx와 같은 웹 서버가 빌드된 정적 파일 HTML을 전송합니다.
        - HTML은 텅 비어있습니다.
        - 정적인 일부 요소만 포함합니다.
    3. 브라우저는 HTML 파일을 파싱하여 DOM을 생성합니다.
        
        또  브라우저는 HTML 내부에 `<script>` 태그를 통해 번들 된 javascript파일을 요청합니다. (<link> 등을 통해 CSS sheet도 요청합니다.)
        
    4. 웹 서버는 요청 받은 대로 build 된 정적 파일( CSS, JavaScript 등)을 반환합니다.
        - javascript 번들에는 React라이브러리와 애플리케이션 코드가 함께 들어가 있습니다.
    5. 브라우저는 다운로드 된 파일을 실행하여 DOM으로 구성된 화면을 수정/변경/추가합니다.
        - 사용자와 인턱렉션 가능한 요소 추가
        - 필요한 경우 백엔드 api endpoint로 요청을 보내고 응답 받은 데이터에 기반 해 화면을 그립니다.

- 그럼 여기서 SSR View 페이지란?
    - 일반적으로 프로젝트에서 사용하진 않아 경험이 없을 수 있습니다.
    - 실제 서비스 사용 시  요구되는 기술입니다.
    
    위의 사진(SSR View를 표기한 사진)은 네이버 기술 블로그에서 따온 사진입니다. 
    
    일반적인 경우라면 아래와 같이 브라우저를 통해 화면을 그리는 데서 끝을 냅니다. 이 구조를 보통의 FE/BE가 구분된 프로젝트에서 선택하기 때문에 “나는 api만 만들었는데 SSR View를 만든다는게 무슨 소리지?”라고 의아할 수 있습니다.
    
    ![Untitled](image/Untitled%202.png)
    
    - CSR와 결합한 SSR View란?
        
        네이버 블로그 내에 SSR View가 무엇인지 이야기하는 정확한 문장은 없으나 앞뒤 최적화/기술 설명에 기반해 두 가지 방향을 유추해볼 수 있습니다.
        
        1. SEO 검색 엔진 최적화를  위해 HTML 내에 데이터를 삽입해야 하기 때문에 동적인 HTML 틀이 필요했다.
            
            → CSR 클라이언트는 텅 빈 HTML파일을 반환합니다. javascript를 통해 HTML을 파싱하여 구성한 DOM에 새로운 값을 부여한다고 이해할 수 있습니다. 
            
            - 즉, javascript를 실행시키지 못하는 검색엔진은 페이지 내부의 값을 알 수 없게 됩니다.
            - 이를 위해 가장 최초에 전달하는 텅 빈 HTML 파일에 동적으로 정보값을 기입할 필요가 있습니다.
                - ex) 00의 블로그 - “맛집 탐방 1차 기록”
            - 페이지 별로 바뀌는 동적인 형태의 정보값이라면 → DB의 정보를 받아와 HTML을 생성해야 하기 때문에 이를 Backend에서 실행하고 완성된 HTML을 client에 넘겨주는 형태로 작업했다고 보여집니다.
        2. 서비스 최적화를 위해 HTML 틀을 고정적으로 제공할 수 있어야 했다.
            
            → 네이버 블로그와 같은 서비스는 동일한 HTML 구조에 내용이 변경되는 형태로 삽입됩니다. 따라서 가장 처음 다운로드 되는 HTML에 이미 구조가 그려져 있다면 사용자는 빠른 페이지 인터렉션이 가능합니다. 
            
            → ‘나의 블로그’ 화면이라면 블로그 내부 게시글 파트만 변경하는 형태로 페이지 변경이 가능하지만, CSR는 모든 화면을 다 그리게 됩니다. CSR의 경우 텅 빈 HTML을 제공하기 때문에 반복적으로 그려지는 HTML 구조도 모두 동적으로 클라이언트 단에서 생성하게 됩니다.
            
        
        ⇒ 이러한 부분을 서포트 하기 위해 SSR View와 CSR View를 함께 사용했으리라 생각됩니다.
        
        (기존의 JSP의 SSR 구조에서 사용하던 JSP와 연결-유지하는 측면 있었을 거구요.-추측이긴 함-)
        
    
    - 즉, 단순한 페이지는 FE-BE가 완전히 분리된 형태로 구성할 수 있으나,
        
        ⇒ 가장 최초에 제공되는 HTML 파일에 동적 작업이 필요한 경우 Backend에서 기본 HTML을 생성해야 했습니다.
        
        (물론 지금의 React는 또 발전하여 SSR을 일부 포함하고 있는 경우도 있어…. 위의 자료-20년도-와는 다를 수 있습니다. 하지만 기본적인 CSR 개념에 기반하면 위와 같은 SSR View 작업을 해야 한다고 보시면 됩니다.)
        
    

### 드디어 SSR 랜더링 단계를 살펴보자

- SSR을 통한 페이지 로드 방식
    
    ![Untitled](image/Untitled%203.png)
    
    1. client가 요청을 보냅니다.
    2. nginx와 같은 웹서버를 통해 Next.js의 서버에 요청이 전달됩니다.
    3. Next.js의 서버는 요청을 파악하고 HTML 파일을 생성하여 반환합니다.
        - backend endpoint로 api 요청을 보내고 응답을 받아 HTML을 생성합니다.
        - SSR에 기반한 구조는 우선적으로 HTML로 구현합니다.
    4. 브라우저는 응답 받은 HTML 파일을 파싱하여 DOM을 생성합니다.
        
        또  브라우저는 HTML 내부에 `<script>` 태그를 통해 번들 된 javascript파일을 요청합니다. (<link> 등을 통해 CSS sheet도 요청합니다.)
        
        - 이 때 HTML 파일은 화면에 그려져야 하는 요소를 포함하고 있습니다. 따라서 텅 빈 화면이 아닌 SSR에서 구성한 화면이 보여집니다.
    5. 웹 서버는 요청 받은 대로 build 된 정적 파일( CSS, JavaScript 등)을 반환합니다.
        - javascript 번들에는 React라이브러리와 애플리케이션 코드가 함께 들어가 있습니다.
    6. 브라우저는 다운로드 된 파일을 실행하여 DOM으로 구성된 화면을 수정/변경/추가합니다.
        - 사용자와 인턱렉션 가능한 요소 추가
        - 필요한 경우 백엔드 api endpoint로 요청을 보내고 응답 받은 데이터에 기반 해 화면을 그립니다.

- SSR을 하는 방법은?
    - NextJS 프레임워크 사용
        - 가장 많이 선택하고 가장 흔한 방법
        - React도 SSR를 사용하려면 Nextjs를 쓰라고 추천했었음 (예전)
            
            *→ 지금은 React 자체에서 SSR를 지원함(…?) 그래도 편리함 때문에 NextJS를 많이들 사용합니다.*
            
        - SSR, CSR 혼합 사용이 가능함.
        - SSG(Static Side Generation)을 통해 렌더링 효율화를 도모 (하단부에서 설명)
    - Node.js, Gatsby, Nuxt.js ….
        - 여러가지 방법이 있음
        - 미들웨어 같은 걸 써서 최적화 하기도….

- *SSR과 CSR의 혼합 사용을 통한 동적 화면 구현(몰라도 괜찮음)*
    - *기본적으로 SSR로 로딩해도 Javascript를 통해 브라우저 상에서 화면 변경이 가능하도록 해야함.*
        - *React 등의 라이브러리를 사용하고 기본 HTML 구조를 React가 관리할 수 있도록 hydration해야 가능 → 브라우저 내의 DOM을 React가 관리함.*
    - Gatsby는 완전히 고정된 페이지(위의 작업을 안하므로)
    

### NextJS란 무엇인가?

- NextJS
    - 리액트 프레임 워크
        - 리액트는 원래 라이브러리다. 리액트 혼자는 실행이 안되고 원래 브라우저를 통해서 실행 되었다. (JS 스크립트 형식으로)
- NextJs는 무슨 역할을 하나?
    - 서버 구축을 통해 SSR 지원 (자체적인 Node.js 서버가 있음.)
    - SSR + CSR을 위한 기능 제공
    - SSG 등을 통한 화면 로드 최적화 제공

- SSG (Static Side Generation)
    - SSR과 유사하다.
    - 기본 HTML 생성 시점의 차이
        - SSR :  클라이언트가 요청할 때
        - SSG : FE서버를 빌드 할 때 미리 생성해둠.
        
        ⇒ 따라서 SSG를 사용한다면 클라이언트가 기본 HTML 다운로드를 대기하는 시간을 줄일 수 있다. SSR의 이점인 빠른 컨텐츠 접근(화면 그리기)이 극대화 된다.
        
        ⇒ api 통신을 포함한다면 → 요청 시점이 달라짐.
        
        - SSR :  클라이언트가 요청할 때 → api 요청을 보내서 HTML을 그림
        - SSG : FE 서버를 빌드할 때 → api 요청을 보내서 HTML을 그림
        
        *물론 꼭 이렇게만 되는 건 아니고 캐시 옵션 설정을 통해 api를 재호출하여 추가적으로  요청을 보내고 다시 그리는 등의 작업도 가능함*
        

- 동적 오픈 그래프 설정 (=/= SEO)
    - 오픈 그래프란?
        
        FaceBook에서 개발한 프로토콜로 SNS 처럼 웹 페이지의 정보를 미리보기로 구성해서 보여주는 형식임.
        
        → Header의 meta 값을 검색하기 때문에 동적으로 변경하려면 페이지마다 기본 HTML HEADER 구성이 달라져야함.
        
        ![Untitled](image/Untitled%204.png)
        
        ![Untitled](image/Untitled%205.png)
        
    - 오픈그래프 예시
        
        https://developers.kakao.com/tool/debugger/sharing
        
        - damdam6 깃허브 페이지 헤더
        
        ![Untitled](image/Untitled%206.png)
        
        ![Untitled](image/Untitled%207.png)
        
        ![Untitled](image/Untitled%208.png)
        
    - CSR에서는 동적 오픈그래프 생성이 안되나요?
        - A : 꾸역꾸역하면 되긴합니다. 단, 인프라의 힘을 빌려서요
            
            https://techblog.woowahan.com/15469/
            
        - A : 근데 저렇게까지 할 이유는 없음… 일반적인 프로젝트에서는 하기 어렵다고 보시면 됩니다.
        

---

### FE가 서버가 있다고?

- 네. NextJS는 서버가 있습니다. 그래서….
    - 자체적인 api 개발도 가능해집니다. (api 요청 x , 실제로 REST API 만들기 O)
        
        ```jsx
        import type { NextApiRequest, NextApiResponse } from 'next'
         
        type ResponseData = {
          message: string
        }
         
        export default function handler(
          req: NextApiRequest,
          res: NextApiResponse<ResponseData>
        ) {
          res.status(200).json({ message: 'Hello from Next.js!' })
        }
        ```
        
        - 사실상 Backend가 필요가 없어지는 기능
        - 개인 개발자들이 간단한 웹페이지를 만들기 위해 많이 사용한다.
        

- 일부 개발자 : 아니, 결국 PHP로의 회귀잖아!!!!
    
    ![Untitled](image/Untitled%209.png)
    
    ⇒ 그렇지만 편한데 어쩌겠어?
    

---

https://doit-fwd.tistory.com/10