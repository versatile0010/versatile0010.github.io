---
title: " 📨 [ko] Coworking: Asynchronous Communication "
layout: single
classes: wide
categories:
  - Communication
  - Cowork
---

커뮤니케이션도 비동기로 최적화해보자.

> By providing detailed questions, we can minimize response time and reduce interruptions.

<div style="text-align: center;">
    <img src="https://github.com/user-attachments/assets/d5bfbe36-359f-41af-85e6-1effa4a28569" alt="image" width="600">
</div>

<br>

```
1. “아.. 이거 쿼리가 너무 느릴 것 같은데…”
```
```
2. “아 이거.. 코드 구조가 너무 비효율적인데…”
```
```
3. “비지니스 로직이 너무 복잡해서 구현을 못하겠는데…  ”
```
  아마 세 개 전부 다 아닐겁니다. 

개발자가 기술적인 고민만 할 수 있다면, 아마 행복하지 않을까요..?

보통은 기술적인 고민보다는 커뮤니케이션 관련 고민이 더 어렵고, 머리아픈 것 같더라구요.. 

(  사실 저만 그런 줄 알았는데, 우리 팀 개발 리드님께서도 커뮤니케이션하는게 가장 어렵다고 하시더라구요.. )

---

나름 실무를 반 년 정도 접한 병아리 개발자 입장에서,

협업, 커뮤니케이션이라는 문제를 어떻게 풀어내고 있는지 기록을 남겨보려고 합니다.

언젠가 시니어가 되었을 때 다시 읽어보면 재밌을 것 같네요.

---

### 1. 비동기 커뮤니케이션으로 리소스 최소화 하기

A 라는 개발자가 업무를 하고 있다가 막히는 게 생겼다고 가정해봅시다.

구글링을 해보지만, 좋은 솔루션이 없는 것 같아요.

1시간.. 

2시간..

고민 시간이 늘어납니다…

이러다가 스프린트에 지장이 생길 것 같다는 생각이 들어요.

큰일이 났다 싶어 A 개발자는 큰 마음 먹고 질문을 해보기로 다짐했습니다.

```
🙋‍♂️ A 개발자: B 님, 저 질문이 있습니다.
```
```
😳 B 개발자: ( 뭐지.. 뭐가 모르는 걸까.... 어느 정도 사이즈의 질문일까.. )
🤨 B 개발자: 어떤건데요...? (두려움)
```

물론 `A 개발자` 처럼 질문해도 되겠지만,

답변자인 `B 개발자` 의 입장에서 바라보면 조금 당황스러울 수 있을 것 같아요.

도대체 어떤 것을 질문할 지, 

어느 정도의 리소스를 투입해야 할 지..

급한 건인지..

등등

정보를 전혀 알 수 없기 때문이죠.

답변을 하려면,

다시 `A 개발자` 에게 무엇을 질문하려고 하는지 다시 물어봐야하고,

`A 개발자` 의 답변이 오기까지 `B 개발자` 는 기다려야해요. 

비효율적이지 않나요?

물론 `A 개발자` 입장에서는 분명,

주니어의 소심한 마음으로 질문 했을 가능성이 크지만

별로 좋진 않은 것 같습니다.

그러면 어떻게 하는 게 좋을까요?

답변자의 리소스를 최대한 줄여주는 게 핵심인 것 같아요!

이를 효과적으로 해주는 것이 바로  `비동기 커뮤니케이션` 입니다.

비동기 커뮤니케이션이란 무엇일까요?

잠깐… 동기/비동기부터 무엇인지 알고 넘어가볼까요?

<img width="735" alt="image" src="https://github.com/versatile0010/versatile0010.github.io/assets/96612168/cf14b102-7890-4692-ab98-4c4cb58c9345">

동기(Synchronous)라는 것은 Request 를 보내면,

그것에 대한 Response 를 받을 때 까지 Client 는 하던일을 멈춥니다.

반면 비동기(Asynchronous)는 Request 를 보내고,

Response 가 오던지 말던지 그냥 하던일을 계속 하는거에요.

그러다가 상대측에서 Response 를 보내면 하던 일을 멈추고 그것에 대한 처리를 하러 가는 거죠.

Response 가 오지 않으면,

일정 시간 이후에 적절히 재시도(timeout & retry) 정책을 마련해두는 게 일반적이죠.

협업 관점에서 비동기 커뮤니케이션(Asynchronous Communication) 도 똑같습니다. 

질문을 보낼 때, 상대방에게 응답이 오지 않을 것이라고 가정한 질문을 보내는 거에요. 

그러려면 필요한 정보를 잘 정리해서 보내야겠죠?

대략 아래처럼 할 수 있겠네요!

```
🙋‍♂️ A 개발자: 
B 개발자님. 제가 OOO 티켓을 진행하던 중에 질문이 있어 DM 보냅니다.
~~~ 한 것 까지는 시도했는데요, ooo 라는 현상이 발생합니다.
서치해본 결과, 유사 사례를 발견하지는 못했는데요 제 생각에는 ooo 때문일 것 같다는 생각이 들어요.
그래서 bbb 한 방법이나, 혹은 ccc, ddd 방법으로 우회하는 것이 좋을 것 같은데 어떤 게 더 좋은 방법일 지 고민이 되네요.. 코멘트 부탁드려도 될까요?   
```

어떤가요?

B 개발자의 입장이라고 생각했을 때, 이전의 질문에 비해서 어떠한 생각이 먼저 드나요?

```
😀 B 개발자: 
( A 개발자 님이 업무를 잘 진행하고 있었는지 걱정되었는데, 잘 하고 계셨구나. 대충 20 % 정도 진행된 것 같고... 고민하시던 방향을 보니 큰 문제는 없겠네.. 이거 비슷한 사례 소개해드리고, 가볍게 가이드 해드리자.)
A 개발자님 안녕하세요~ 이거 OOO 티켓이랑 유사한 사례네요~ 저번에는 ooo 해서 ooo 했던 것 같아요. 이번 사례에도 ooo 가 좋을 것 같은데요? 참고해서 보시다가 질문있으시면 다시 DM 주세요~ `
```

답변에 필요한 정보가 질문글에 모두 담겨있으니, 

질문자에게 다시 정보를 얻고자 하는 불필요한 리소스를 소모할 필요가 없습니다.

무엇을 하려다가 어디서 막혔는지,

어느 부분을 고민하고 있는지까지 모두 잘 정리되어있으니 가볍게 코멘트를 하기가 더더욱 쉬워지죠..!! 

게다가 주니어 개발자라면,

질문을 잘 하는 것 만으로도 신뢰도가 100 % 올라간다고 합니다!!

( 가장 무서운 것은 조용한 신입 개발자 라고 하네요.. ㅎㅎ)

### 1.1 general 한 질문은 DM 이 아닌 채널 단위로

질문하고자 하는 내용이 특정 사람만 알고 있는 지식이 아니라면,

DM 보다는 채널 단위로 질문을 하는 것이 효과적 입니다.

물론 MBTI 가 I 인 사람에게는

정말 망설여지는 일이라는 걸 알고 있지만… 

서로 서로의 커뮤니케이션 비용을 줄여야

더욱 효율적으로 일할 수 있겠죠..!

( 질문 할땐 읽씹을 두려워하지 맙시다. )

저는 개발하다가 고민되는 점이 있으면,

백엔드 개발자가 모여있는 채널에 질문을 투척하곤 합니다!

물론 충분히 고민하고, 나름의 솔루션을 찾은 뒤에 말이죠..!

해당 질문에 대해서 알고 있는 사람은 가볍게 코멘트를 남기고 갈 수 있도록...

`비동기 메세지` 를 투척합시다!

가끔… 질문 쓰레드에서 치열한 개발 토론(?)이 이루어지기도 하는데요..

좋은 질문을 많이 하는 것은, 개발 조직에 확실히 긍정적인 영향을 주는 것 같아요.
  
세상엔 나쁜 질문은 없다..

더 많이 질문하고, 답변할 수 있는 사람이 되어야겠네요.
