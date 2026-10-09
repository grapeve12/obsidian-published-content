---
title: Jev in Nutshell
date: 2026-10-09
tags:
  - Jev
  - LLM
description: TypeSafe AI가 개발한 의사결정 모델인 Jev의 개념과 특징을 살펴보고, LLM과의 차이점 및 효율적인 결합 활용 방안을 통해 신뢰성 있는 AI 자동화 시스템을 구축하는 방법을 설명합니다.
category: study
thumbnail: Docs/gh_71b355dfdb05_OAXYKYX.webp
draft: false
---

<br>

# 1. What is Jev?  

<br>

## 1.1 Jev란 무엇인가  

<br>

**Jev**  
> TypeSafe AI가 만든 인공지능 의사결정 모델  
> LLM이 만든 답을 얼마나 믿어도 되는가?에 답을 주는 모델  

<br>

Vercel AI Gateway[^1]에서 사용 가능하다.  

<br>

문장을 생성하는게 아니라 **구조가 정형화된 판단**을 돌려준다.  

<br>

만약에 내가 LLM으로 서비스를 만드는데  
1. **느린 응답**  
2. **높은 비용**  
3. **통제되지 않는 출력**  
에 진절머리가 났다면 Jev를 공부해보자.  

<br>

또한 AI Agent나 RAG를 운영하며 "어디까지 자동화할지"를 설계해야한다거나  
문서 분류, 문의 라우팅, 심사 같은 업무 자동화를 맡는다면  
**신뢰 가능한 AI 출력**을 원한다면 Jev를 사용해보자.  

<br>

## 1.2 Jev vs LLM  

<br>

Jev와 LLM의 응답 방식의 차이는 아래와 같다.  
<img src="https://raw.githubusercontent.com/EGU1832/obsidian-published-content/vault/Docs/gh_71b355dfdb05_OAXYKYX.webp" width="300">
  

<br>

그렇다면 Jev가 LLM을 대체하는거냐? 그런 건 아니다.  
<img src="https://raw.githubusercontent.com/EGU1832/obsidian-published-content/vault/Docs/gh_838d1a96850e_M32YHQ7.webp" width="300">
  

<br>

## 1.3 Jev + LLM vs LLM + LLM  

<br>

결론부터 말하자면 Jev + LLM를 사용했을 때 정말 좋은가 하면  
LLM + LLM을 사용하는 것보다는 **정확도가 살짝 낮다고** 한다.  
다만 응답 시간은 거의 10배로 **획기적으로 빨라진다**.  

<br>

> Jev의 가치는 답을 맞히는 능력이 아니라 **모르는 것을 아는 능력**에 있다.  

<br>

## 1.4 실습 환경  

<br>

실습은 Jev 따라하기 위키독스[^2]를 따랐다. 자세한 코드와 내용은 해당 문서를 참고하자.  

<br>

일단 해당 문서는 Jev가 공개되지 채 **10일도 안 된 시점**에 쓴거라 지금은 그냥 발급 받을 수 있는 API Key도 Early Access를 통해 신청해서 조금 기다려야 한다는 둥 최근 시점에서 수정되어야 하는 내용이 꽤 있으니 이 점을 유의하자.  

<br>

# 2. Why is Jev Different?  

<br>

## 2.1 생성형 AI와 의사결정형 AI  

<br>

LLM은 알다시피 다음에 올 토큰(token)을 하나씩 예측해서 이어붙이는 방식으로 답을 만든다.  

<br>

> **"자유로움"** 이 LLM의 양날의 검이다.  

<br>

사실 파싱에 있어서는 문제가 안 된다. 문제는 파싱된 문자열에 담긴 **내용의 일관성**이라는거다.  
**출력 토큰**도 모델에 따라 상이하게 달라진다.  
- gpt-6-astra는 17.0 토콘을 쓰는 반면  
- gpt-6-luna는 44.8 토큰이 소비된다.  

<br>

*아니 그러면 Jev가 아니라 Classifier로 대체하면 되지 않냐?*  
라고 묻는다면 또 그건 아닌게, 분류기는  
- 미리 라벨을 붙인 학습 데이터가 필요하고  
- 라벨 종류를 바꾸려면 다시 학습해야하기 때문이다.  
물론 재학습 비용이 적은 방법도 있지만 실무자들은 그냥 LLM에 프롬프트 한 줄 더 넣기만 하면 끝이다.  

<br>

Jev가 확률을 생성하는 방식은 10번 돌려서 9번 맞으면 0.90으로 평균을 내는게 아니라,  
**확률처럼 보이는 문자열을 생성**하는 것이다.  
```text  
billing   0.90
sales     0.10
technical 0.00
other     0.00
```  

<br>

Jev는 즉 **의사결정형 AI**인 것이다.  

<br>

**의사결정형 AI가 어울리는 문제**  
- 답의 후보가 미리 정해져 있음  
- 같은 판단을 아주 많이 반복  
- 결과로 코드가 분기  
- 틀렸을 때 비용이 있어서 확신도가 필요함  
- e.g. 문의 라우팅, 스팸 판정, 위험도 평가, 검색 결과 적합성 판정  

<br>

**의사결정형 AI가 어울리지 않는 문제**  
- 답을 새로 지어내야 하는 일  
- 긴 설명이 필요한 일  
- 코드를 작성하는 일  
- 여러 단계를 추론해야 하는 일  

<br>

## 2.2 System 1과 System 2  

<br>

심리학자 대니얼 카너먼(Daniel Kahneman)은 사람의 사고를 두 체계로 나눠 설명했다.  
1. **System 1:** 빠르고 자동적, 노력이 들지 않고 의식하기도 전에 결론이 나와 있다.  
- e.g. 상대의 말투에서 화가 났음을 느끼는 것, 2x2에 답하는 것  
2. **System 2:** 느리고 의도적, 주의를 기울여야 하고 단계를 밟아야 하며, 하는 동안 다른 일을 하지 못한다.  
- e.g. 17x24를 암산하는 것, 낯선 도시에서 지도를 보며 길을 찾는 것, 계약서의 독소 조항을 찾는 것  

<br>

둘 중 한 작업이 우월하다는게 아니라 그냥 다르다. 어느것도 빠져서는 안되는 계층이다.  
중요한 점은 일상의 판단은 압도적으로 System 1이 처리하고, System 2는 꼭 필요할 때만 불려나온다는 것이다.  
똬한 **언제 System 2를 부를지 결정하는 것도 System 1의 몫이다.**  

<br>

System One Model이라는 표현도 딱 여기서 나온 말이라고 한다. 즉,  

<br>

> **"빠른 판단을 위한 모델을 따로 만들자!"**  

<br>

가 실현된 것이 Jev이다.  

<br>

<img src="https://raw.githubusercontent.com/EGU1832/obsidian-published-content/vault/Docs/gh_906fb83da1d4_j1fC5DO.webp" width="450">
  

<br>

## 2.3 Jev 개념 소개  

<br>

Jev를 한 문장으로 줄이면 이렇다.  
> **정리되지 않은 입력을 받아 타입이 정해진 확률적 판단을 돌려준다.**  

<br>

Jev의 호출은 두 부분으로 이루어진다.  
1. **State(상태):** 판단의 대상. 문장이 될수도 있고 JSON 객체일수도 있다.  
2. **Question(질문):** 그 State에 대해 내가 궁금한 내용. Choice, Noul, Score가 있다.  

<br>

```python  
result = client.system_one(
    state="결제가 두 번 됐습니다. 빨리 환불해주세요.",
    questions={
        "department": Choice(criteria={"billing": None, "technical": None,
                                       "sales": None, "other": None}),
        "urgency": Score(criteria=["급하지 않음", "보통", "급함", "매우 급함"]),
        "refund": Noul(instructions="고객이 환불을 요구하고 있는가?"),
    },
)
```  

<br>

LLM과 다른 점은 질문을 복합적으로 묻는다고 해서 서로 간섭을 받아 답이 달라지지는 않는다는 것이다.  

<br>

Jev가 받는 질문은 세 종류 뿐이다.  
1. **Choice:** 여러 후보 중 어느 것인가  
2. **Noul:** Yes/No  
3. **Score:** 어느 정도인가  

<br>

Choice는 내가 집어넣은 폴더 중에 어느 폴더인가 나오는 거고  
Score는 내가 집어넣은 점수 척도에 따르면 어느 정도인지 매기는 것이다. 좀 헷갈리는데 코드를 돌려보면 다음과 같다.  
```json  
// Choice
choice='billing'
confidence=1.0
probabilities={'billing': 1.0, 'technical': 0.0, 'sales': 0.0, 'other': 0.0}

// Score
score=2.77
confidence=0.78
legend={0: '급하지 않음', 1: '보통', 2: '급함', 3: '매우 급함'}
probabilities={0: 0.0, 1: 0.0, 2: 0.22, 3: 0.78}

// Noul
0.98
```  

<br>

참고로 저렇게 정형화된 답변을 생성하기 때문에 입력 요금에만 값이 청구가 된다.  
- 입력 요금: 100만 토큰 당 $`0.042  
- 출력 요금: 무료(`$0.00)

<br>

# 3. Let's Use Jev  

<br>

Jev 코드를 실제로 돌려보자.  
환경 설정과 기본 코드는 위키독스 2.1 항복에 기재되어있다.  

<br>

API 키는 TypeSate AI[^3]에 로그인하여 Billing에 먼저 $5 넣어놓고 `..` API Keys 탭에 들어가서 발급 받으면 된다.  
키 값은 한 번 보여주고 사라지니 꼭 안전한 곳에 복붙해두자.  

<br>

그리고 Jev API를 사용할 때 `models.list()`와 `print(result.model)`를 사용해서 현재 응답을 반환하고 있는 모델의 버전을 수시로 잘 확인하자.  
비단 이 실습 뿐만이 아니라 업데이트가 수시로 진행되는 AI API 생태계에서 디버깅 시 중요한 포인트이다.  

<br>

Jev 첫 호출 시 연결 시간이 발생하기 때문에 답변이 느려지는 현상이 발생하기 측정값을 원한다면 **첫번째는 빼고** 측정하자.  

<br>

참고로 Choice에서 후보를 나열할 때 이름만 나열해도 되지만  
아래와 같이 각 후보가 무엇을 뜻하는지 설명을 붙일 수 있다.  
```python  
Choice(criteria={
    "billing": "요금, 결제, 환불, 세금계산서 관련",
    # 후보마다 한 줄씩
})
```  

<br>

# Reference  

<br>

[Jev 따라하기 (LLM과 다른 의사결정 AI 시작하기) - WikiDocs](https://wikidocs.net/book/21376)  
[Jev - 나무위키](https://namu.wiki/w/Jev)  
[Home - TypeSafe AI](https://typesafe.ai/)  

<br>

# Footnote  

<br>

[^1]: 여러 AI 제공자의 모델을 하나의 통합API 엔드포인트와 계층으로 묶어주는 중계 서비스  
[^2]: [Jev 따라하기 (LLM과 다른 의사결정 AI 시작하기) - WikiDocs](https://wikidocs.net/book/21376)  
[^3]: [Home - TypeSafe AI](https://typesafe.ai/)  

<br>
