---
title: " 🌲 Management of Hierarchical Data with Redis Caching "
layout: single
classes: wide
categories:
  - Cache
---

> Efficiently manage hierarchical data by leveraging Redis caching to improve performance and simplify maintenance.
Avoid repetitive executions of high-cost queries/logic.

계층형(카테고리 성) 데이터와 캐싱


오늘 포스팅 할 내용은 계층형 데이터를 레디스 캐시를 이용하여 효율적으로 개선했던 경험에 대해서 간단하게 후기를 남기려고 합니다.

먼저 여기서 말하는 계층형 데이터란 아래와 같이 하나의 테이블 내에서, 부모 - 자식 간의 관계가 정의된 형태를 의미합니다.

특정 자식이 주어졌을 때, 최상위 부모를 알아야 하는 경우가 발생한다면 어떻게 로직을 풀어내는 게 좋을까요?

Mysql 에서 제공하는 재귀적 CTE 를 사용하면 어떨까요?

최근에 Real Mysql 2 권 스터디를 진행하고 있는 김에 한번 재귀적 CTE 쿼리를 작성해볼까요!?

```sql
with recursive cte as (
    select id, name, parent_id, id as root_id
    from category
    where parent_id is null

    union all

    select child.id,
           child.name,
           child.parent_id,
           root.root_id
    from category child
         inner join cte root
                      on child.parent_id = root.id)
select cte.id, cte.name, root_id
from cte
```

위와 같은 쿼리로 쉽게 구현할 수 있겠군요!

하지만, QueryDsl, JPA 에서는 재귀적 CTE 를 공식적으로 제공하고 있지 않으므로

@Query 어노테이션 내에서 쿼리를 문자열로 잘 작성해서 네이티브 쿼리로 실행해도록 해야만 하겠죠.

하지만 이 방법은 문자열로 쿼리를 작성하다보니,

쿼리에 오류가 있었어도 컴파일 단계에서 잡지 못한다는 치명적인 단점이 있습니다.

실제로 해당 쿼리 메서드를 호출하는 시점에 오류가 발생할 수 밖에 없는데요,

이러한 구조는 추후 유지보수하는 관점에서 좋지 않을 게 분명하다고 생각합니다.

다른 방법은 없을까요?

만약 해당 테이블에 대해서 read 연산만 발생한다고 가정한다면 캐싱을 해두어 사용해도 좋을 것 같습니다.

<img width="871" alt="image" src="https://github.com/versatile0010/versatile0010.github.io/assets/96612168/9663a822-695f-4d14-9a6d-5392b120c4db">

캐시 메모리는 보통 레디스를 많이 사용하는데요,

인메모리 db 이기 때문에 빠른 I/O 가 가능하며, 저장되는 데이터마다의 ttl 을 별도로 지정해줄수도 있고

사용이 매우 간편하며 레퍼런스도 다양한 점이 장점인 것 같아요.

캐싱을 적절히 사용하면 API 성능을 높이는 데에 아주 좋은 효과를 볼 수 있습니다.

---

만약에 카테고리 테이블에 대한 update / delete 기능이 추가가 된다면

TTL 동안 데이터 정합성이 맞지 않는 문제가 발생할 것 같습니다.

이를 방지하기 위해서는 카테고리에 대한 update / delete 를 수행할 때 cache 에 저장된 데이터도 수정해서 싱크를 맞추어주는 방법을 사용하거나,

혹은 무효화해서 다시 캐싱하게끔 하는 것도 가능하겠죠.

상황에 따라 적절히 선택해주면 될 것 같아요!

또 하나 고려해야 할 점은 캐시 스탬피드(Cache Stampede) 에 의한 duplicated read/write 가 발생하여 성능이 저하될 수도 있다는 거에요.

![image](https://github.com/versatile0010/versatile0010.github.io/assets/96612168/99a66649-5d96-4235-80cc-33f6385deddc)

단순히 캐시 메모리에서 데이터를 get 하기보다는, 

API 호출 시점에, 

TTL 이 만료되기까지 얼마 남지 않았더라면, 새로 캐싱하도록 하면 해결할 수 있습니다. 

TTL 이 만료되기까지 얼마 남지 않았는지 판단하는 방법은 주로 PER ( Probabilistic Early Recomputation ) Algorithm 을 사용하는 것 같아요.

마지막으로 캐시 데이터의 종류마다 다른 TTL 을 적용해야만 한다면 어떻게 해야 할까요?

현재의 요구사항에서는 캐시 데이터마다 별도의 TTL 을 적용하지 않아도 되기 때문에,

RedisConfig 에서 일괄적으로 동일한 TTL 을 설정해두었지만 앞으로는 캐시 데이터마다 TTL 을 다르게 주어야 할 상황이 생길 수도 있을 것 같아요.

그러면 캐시 데이터 종류마다 redisCacheManager 을 별도로 Bean 으로 등록하여 사용하면 될까요?

물론 가능하지만, 좋은 구조는 아닌 것 같습니다.

하루하루가 시간이 부족하네요..!! 🥲

추후 업로드할 포스팅에서 깊게 다루어보도록 하겠습니다!