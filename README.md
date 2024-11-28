# 기술면접 & CS 학습 스터디

<br>


<br>

**[🐣 시즌1 소개 바로 가기 🐣](https://github.com/damdam6/cs-tech-interview-study-2024/tree/main/season1_week01-17/Season1_README.md)**
<br>
**[🐣 시즌1 주제 리스트업 🐣](https://github.com/damdam6/cs-tech-interview-study-2024/wiki/%EC%8B%9C%EC%A6%8C1-%EC%A3%BC%EC%A0%9C-%EB%A6%AC%EC%8A%A4%ED%8A%B8%EC%97%85)**

<br>

### 스터디 소개

---

**스터디 구성원**

| [bbookng](https://github.com/bbookng) |                        [damdam6](https://github.com/damdam6)                        | [skydreamer21](https://github.com/skydreamer21) |
|:---:|:-----------------------------------------------------------------------------------:|:---:|
| <img alt="bbookng" src="https://github.com/bbookng.png" width="230" height="100%"/> | <img alt="damdam6" src="https://github.com/damdam6.png" width="230" height="100%"/> | <img alt="skydreamer21" src="https://avatars.githubusercontent.com/u/95271588?v=4" width="230" height="100%"/> |

| [thisiseunjin](https://github.com/thisiseunjin) | [wonjunJ](https://github.com/wonjunJ) | [Zerotay](https://github.com/Zerotay) |
|:---:|:---:|:---:|
| <img alt="thisiseunjin" src="https://github.com/thisiseunjin.png" width="230" height="100%"/> | <img alt="wonjunJ" src="https://github.com/wonjunJ.png" width="230" height="100%"/> | <img alt="김동건_T4026" src="https://avatars.githubusercontent.com/u/67823010?v=4" width="230" height="100%"/> |


<br>

**날짜** : 월 2회 --요일 강남 19:00 ~ 22:00

**목표**

- 다양한 주제에 대해 깊이 있는 이론 학습하기
- 우리만의 미니 테크톡 만들기!

<br>

### 스터디 진행 방식

---

**스터디 준비**

1. 매 모임마다 3명의 발표자를 선정합니다.
2. 발표자는 **모임 사흘 전 12:00**까지 자료 초안을 준비합니다.<br>
    - 발표자는 주제에 대해 10분~15분 분량의 발표를 준비합니다.
    - 발표자의 초안은 기본적인 발표내용을 포함하면 됩니다.
3. 레포지토리를 **fork**한 뒤, 자료 초안을 포함한 **PR을 요청**합니다.
4. **모임 이틀 전 00:00**까지 PR에 review를 다는 형식으로 피드백을 주고 받습니다.
    - 더 궁금한 점
    - 오류사항
    - 추가적으로 보충하면 좋을 주제 <br>
      위의 사항 중 하나를 각 자료별 1개 이상 작성해주세요.
5. 피드백을 바탕으로 초안을 수정하고 **0요일 19:00**에 스터디를 진행합니다.



<br>

### Commit & PR Convention

---

**폴더 및 파일 구조**

1. week{00}폴더 하위에 자신의 발표 주제명으로 폴더를 생성합니다.
2. 매주 제출할 파일들을 해당 폴더에 업로드 합니다.
3. 주제명 폴더 하위 구조는 자유롭게 구성해도 무방합니다. (이미지 추가 등 가능)
    - 전체적인 자료를 정리한 md 파일이 꼭 하나는 필요합니다. (초안 피드백용/자료체크용)

<br>


```angular2html
.
|   README.md
|
|
+---week01
|   +---http-https
|   |       http-https.md
|   |
|   +---mongoDB자세히알아보기
|   |   |   MongoDB 깊게 탐구하기.md
|   |   |
|   |   \---MongoDB 깊게 탐구하기 이미지
|   |           Untitled 1.png
|   |           Untitled.png
|   |
|   +---transaction isolation level
|   |       transaction isolation level.md
|   |
|   +---what_happens_if_you_enter_url
|   |       what_happens_if_you_enter_url.md
|   |
|   \---트랜잭션 스크립트 패턴 vs 도메인 모델 패턴
|       |   트랜잭션 스크립트 패턴 vs 도메인 모델 패턴.md
|       |
|       \---트랜잭션 스크립트 패턴 vs 도메인 모델 패턴
|               Untitled 1.png
|               Untitled.png
|
\---{week00}
    +---{topic}
    |   |   {topic}.md
    |   |
    |   \---{image folder}
    |           {image 1}.png
    |           {image 2}.png
    |           {image}.png
    |
    \---{topic 2}
            {topic 2-1}.md
            {topic 2-2}.md

```


<br>

**Commit Message Convention**

*임의의 커밋tag를 활용합니다.*

|             | 설명            |
| ----------- |---------------|
| feat     | 자료 제출 / 내용 추가 |
| fix      | 틀린 부분 수청      |
| chore    | 폴더 구조 수정      |
| merge    | merge         |

<br>

*커밋 구조*

```
feat(week00):{자신이 고른 주제}
- {구체적인 설명(옵션)}
```

<br>

*커밋 예시*

```
feat(week01):네트워크7계층-추가
- 계층 별 개념 설명 추가
- 계층 분리 방식 이론 단점 추가
```

<br>

**PR Convention**

1. PR 제목은 `week{00} - {github-id}`로 통일합니다.
2. PR 내용에는 자료에 대한 간략한 설명을 적어주세요.
3. 초안 PR 후 피드백을 받아 수정할 때는 PR 내용에 수정사항을 명시해주세요.
