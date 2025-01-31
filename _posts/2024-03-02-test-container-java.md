---
title: " 🚃[ko] Enhancing Test Reliability with TestContainers: Setting Up a Production-like Environment. "

categories: 
  - Test
  - SideProject  
layout: single
classes: wide
last_modified_at: now
---

<div style="text-align: center;">
    <img src="https://github.com/user-attachments/assets/c4b53e76-1f71-4c1f-b299-a719f26e0ff8" alt="image" width="600">
</div>

사이드 프로젝트, 뒤늦은 테스트 환경 구축기 (with. TestContainer)

> Using TestContainers allows developers to create reliable test environments that mimic production.

더 빠르게 달리기 위해 잠깐 멈춘다..

약간 모순이 되는 문장이기도 한데요,

이번 포스팅의 주제가 테스트 코드라는 것을 알고 다시 제목을 읽어본다면

어느정도는 다들 공감이 갈 만한 제목이라고 생각합니다. (😅 크흠)

작년 10월인가요? 

그때부터 시작한 사이드 프로젝트인 `대피로` 는 2 개월 안에 기획/개발/디자인이 완료되어

데모 어플을 시연할 수 있어야 했어요.

중간에 기획이 변경되기도 하고

시연 직전까지도 새로운 요구사항이 추가되거나 기존 기능 스펙이 수정되는 일이 매우 많았기 때문에

사실상 테스트 코드에 할애할 시간과 여력이 없었던 것 같아요…

해커톤 같은 느낌으로 하루 하루 개발했던 것 같습니다.

완벽한 DDD (데드라인 주도 개발) 이였죠 ㅋㅋㅋㅋㅋㅋㅋ

---

작년 하반기를 불태웠던 사이드 프로젝트 대피로는,

올 2월부터 다시 달려보기로 했습니다. 

와 🎉 ~

작년 하반기 2개월 간,

 쌓여버린 기술 부채를 리스트업하고 하나하나 해결해나가는 중인데요

이번에 이야기할 내용은 테스트 환경 구축입니다. (두둥)

---

테스트 환경..?

그거 그냥 하면 되는거 아니야..?

라고 생각할 수 있지만, 대피로 백엔드에서는 사실 많은 고뇌와 시행착오가 있었는데요..

차근차근 소개하도록 하겠습니다.. 😭

---

제목을 잠깐 다시 보면...

```
더 빠르게 달리기 위해 잠깐 멈추기.
```

이 문장은, 

테스트 코드를 왜 작성해야 하는 지 그 이유를 함축적으로 아주 잘 담고 있다고 생각합니다.

앞으로 기획이 고도화되면서

기존 기능을 확장시키거나

새로운 기능을 추가할 일이 분명 많아질 것 같은데요,

<img width="914" alt="image" src="https://github.com/versatile0010/versatile0010.github.io/assets/96612168/14f53ed0-d2a1-44e1-88d3-c1d7f2fb0bcb">

(자동화 된 테스트 코드는 없다고 가정)

A 영역을 열심히 수동으로 테스트해서 `보장된 기능`이 되었다고 해봅시다.

그리고 고도화 작업을 진행하며 A’ 기능을 확장하였어요.

물론 A와 A’ 이 겹치지 않는 부분은 새로 테스트를 진행해야겠지만,

A 와 A’ 이 겹치는 부분도 또 다시 수동으로 테스트를 해야겠죠.

이후에 B 라는 기능을 추가하면, 

완전히 새로운 기능인 B 뿐만이 아니라 A 와 B 가 겹치는 구간도 테스트해야 해요.

기존에 테스트를 했던 영역을 계속 반복적으로 테스트하고 있는 비효율적인 상황을 확인할 수 있어요.

위와 같은 상황에서는 견고한 시스템을 개발할 수 없을 것 같습니다.

새로운 서비스 피쳐를 개발하려면, 해당 개발 건이 현재 동작 중인 어떤 부분에 영향이 갈 지

수동으로 확인해야 하기 때문에 생산성이 현저히 떨어지겠죠.

만약..

잘 작성된, 자동화 된 테스트 코드가 있었더라면

새로운 기능을 구현하거나, 기존 기능을 아예 갈아엎더라도 전혀 두려움 없이 개발할 수 있겠죠.

더 빨리 달리기 위해, 잠깐 멈춰야 한다는 의미는 이거에요.

우리 사이드 프로젝트인 `대피로` 는, 

작년 2개월 간 데모데이 출시라는 단거리 거리 질주는 해냈지만

테스트 환경이 전혀 구축되어있지 않았기 때문에

더 빠르게 오래 달릴 수 있는 여력이 남아있지 않았습니다.

그래서 지금은 잠깐 멈추고, 더 멀리 가기 위한 정비를 해야할 때라고 결정했죠.

---

서론이 길었던 것 같네요.

요약하자면, 사이드 프로젝트 `대피로` 의 기술 부채 중 하나인 테스트 환경 구축을 지금 해결해야만 한다.

이건데요..!

대피로 백엔드에서 테스트 환경을 쉽게 구현하지 못하게 했던 허들은 무엇이 있었을까요?

<img width="910" alt="image" src="https://github.com/versatile0010/versatile0010.github.io/assets/96612168/3399a0f5-2a1f-4754-a195-b95076897666">

대피로에서 제공하는 핵심 기능 중 하나는 현재 위치를 기반으로 가까운 대피소들을 조회하는 API 입니다.

각각의 위치는 위경도 정보를 기반으로 조회하며,

대피소와 사용자 간 거리 또한 실시간으로 계산이 가능해야 했습니다.

위와 같이 대피로는 공간 데이터를 적극적으로 활용해야 하는 경우가 많았습니다.

공간 데이터 관련 풍부한 기능을 제공해주는 mysql 을 메인 데이터베이스 엔진으로 선택한 이유도 이때문이었는데요..!

( 추후에 공간 인덱스 관련한 쿼리 튜닝 포스팅이 예정되어 있습니다 ~ )

보통 테스트 환경에서의 데이터베이스는 정말 간편하게 구축할 수 있는 h2 DB 를 많이 사용하죠.

다만 대피로에서는 지금도, 앞으로도 mysql 에서 제공하는 공간 데이터 타입 / 함수를 적극적으로 활용하여 성능 개선을 해나갈 계획이기 때문에

H2 기반의 테스트 환경으로는 제약이 많았습니다.

테스트 코드를 원활하게 하기 위해,

프로덕션 코드에서 mysql 에 의존적인 함수를 사용하지 않는 방법도 있었는데요

그러기에는 mysql 에서 제공해주는 위치 기반 기능이 강력하기 때문에 좋지 않다고 판단했습니다.

아니면 테스트 환경에서 해당 기능들을 모두 mocking 해도 될 텐데요,

외부 시스템이 아닌 대피로 서비스의 핵심 기능인데 이를 mocking 하는 게 과연 올바른 방향일까요?

당연히.. 아니겠죠.

그러면 남은 방법은 단 하나.

테스트 환경을 h2 가 아닌 mysql 을 사용하면 되는거죠! (당연)

자 그럼 문제 정의가 조금 바뀌었네요.

```
테스트 환경에서 mysql 을 어떻게 구축하면 좋을까요?
```

각자 로컬에 mysql 을 demon 으로 실행시킨 뒤, 프로덕션 db 를 dump 해서 사용하도록 하면 어떨까요?

흠…

너무 로컬 환경에 의존적이죠.

```
“쟤 컴에선 테스트가 다 성공하는데 왜 전 안되는거죠?” 
```

프로덕션 db 와 스키마를 항상 동기화해주어야 한다는 관리포인트도 하나 늘어납니다..

테스트 환경 구축이 매우 간편하고 편리해야,

더 많은 시간을 테스트 코드에 할애하도록 유도할 수 있다고 생각하기 때문에

이 방법은 기각했습니다.

음..

다른 좋은 방법이 없을까요?

로컬 환경에.. 종속적이지 않은 테스트 환경이라..

아 이거 완전 컨테이너 개념 아닌가요? (두둥)

테스트를 run 하면

테스트 용도의 도커 컨테이너가 띄워지고

테스트가 끝나면 컨테이너를 내려주면 되겠네요!

라고 생각하고 바로 구글링해보았는데,

정확히 일치하는 개념을 구현해놓은 라이브러리가 있더라구요.. ( 없는게없네요.. )
  
---

#### 테스트 컨테이너(TestContainers)

<img width="893" alt="image" src="https://github.com/versatile0010/versatile0010.github.io/assets/96612168/fac12dd8-9300-45b7-a3a5-14a899c1e22e">

테스트 컨테이너를 이용하면, 자바 코드만으로 테스트 용 도커 이미지를 아주 아주 편리하게 제어할 수 있어요.

이번 포스트에서 다루는 mysql 뿐만이 아니라, mongoDB, Kafka 와 같은 관리 시스템들도 모두 다룰 수 있으므로 실제 서비스 환경과 매우 유사하지만 격리된 환경에서 테스트를 진행할 수 있죠.

단점은 로컬 환경에서 도커가 동작하고 있어야 한다는 점과,

테스트 시작할 때 컨테이너를 띄우는 정도의 소요 시간이 늘어난다는 점이 있어요.

하지만,

모든 개발자들의 로컬 환경에 전혀 종속적이지 않고, 매우 간단하고 편리하게 실제 서비스와 유사한 테스트 환경을 구축할 수 있다는 아주 큰 장점이 있기 때문에 위 단점들은 어느정도 감안이 가능하다고 생각했습니다.

동작하는 모습을 먼저 볼까요?

테스트 코드를 실행시키면,

<img width="663" alt="image" src="https://github.com/versatile0010/versatile0010.github.io/assets/96612168/7caadc6b-4064-4a85-aa5e-0ffb364b2b03">

미리 설정해둔 컨테이너가 실행되는 것을 확인할 수 있어요.

<img width="903" alt="image" src="https://github.com/versatile0010/versatile0010.github.io/assets/96612168/4633e22e-5af7-4ed6-9f44-5ea415db0521">

그리고 해당 컨테이너 환경에서 테스트를 진행할 수 있죠.

<img width="695" alt="image" src="https://github.com/versatile0010/versatile0010.github.io/assets/96612168/5a385766-61a7-498d-9576-39ea117e0ff5">

mysql 컨테이너를 띄웠기 때문에, mysql 에서 제공하는 위치 데이터 타입인 Point 와 위치 함수인 ST_DISTANCE_SPHERE 을 사용한 서비스 로직을 테스트 할 수 있게 되었네요!

( `ST_DISTANCE_SPHERE` 을 사용하면 쿼리가 공간 인덱스 `R-tree` 를 타지 못한다는 이슈가 있는데, 이는 추후 포스팅에서 공간 쿼리 개선 주제로 다시 다뤄볼 예정이에요!!  사실 전반적으로 비효율적인 쿼리가 대부분이
기 때문에, 전체적으로 쿼리를 갈아엎어보려고 합니다.. 대략 4~5월 안에 모두 교체해버리고 싶네요..ㅎㅎ)

테스트 컨테이너를 사용하기 위해서는 아래의 의존성을 받아주어야 합니다.

```gradle
testImplementation 'org.testcontainers:junit-jupiter:1.19.0'
testImplementation 'org.testcontainers:mysql'
```

그리고 아래와 같이 해주면
```java
@Container  
MySQLContainer mySQLContainer = new MySQLContainer("mysql:8");
```

mysql:8 이미지로 도커 컨테이너를 띄워주고, 테스트가 끝나면 종료시켜줍니다.

다만, 매번 모든 테스트에 대해서 컨테이너를 띄워주고 내려주는 작업을 반복하다 보니 전체적인 테스트 코드 실행시간이 많이 지체되겠죠.

이를 해결하기 위해 컨테이너를 static 필드로 선언하고 초기화해주도록 합시다.

각 컨테이너마다 필요한 환경 변수는 `@DynamicPropertySource` 을 통해 동적으로 주입받을 수 있는데요,

대피로에서는 아래와 같이 사용하고 있어요!

```java
@Slf4j
@SpringBootTest(classes = ServiceIntegrationTestConfiguration.class)
@Import({CoreTestConfiguration.class})
public abstract class ServiceIntegrationTestBase {

    private static final String REDIS_DOCKER_IMAGE = "redis:5.0.3-alpine";
    private static final String MYSQL_DOCKER_IMAGE = "mysql:8.0";
    private static final String MYSQL_ROOT = "root";
    private static final String MYSQL_PASSWORD = "1234";

    private static final int REDIS_PORT = 6379;

    @Container
    protected static MySQLContainer mySQLContainer;

    @Container
    protected static GenericContainer redisContainer;

    @DynamicPropertySource
    static void configureProperties(final DynamicPropertyRegistry registry) {
        // mysql env init
        log.info("""
                    \n
                    ✅ mysql property 를 주입합니다.
                """);
        registry.add("spring.datasource.url", mySQLContainer::getJdbcUrl);
        registry.add("spring.datasource.username", () -> MYSQL_ROOT);
        registry.add("spring.datasource.password", () -> MYSQL_PASSWORD);

        // redis env init
        log.info("""
                    \n
                    ✅ redis property 를 주입합니다.
                """);
        registry.add("spring.redis.host", redisContainer::getHost);
        registry.add("spring.redis.port", () -> "" + redisContainer.getMappedPort(REDIS_PORT));
    }

    static {
        // mysql test container init
        mySQLContainer = (MySQLContainer) new MySQLContainer(MYSQL_DOCKER_IMAGE)
                .withUsername(MYSQL_ROOT)
                .withPassword(MYSQL_PASSWORD)
                .withDatabaseName("test")
                .withEnv("MYSQL_ROOT_PASSWORD", MYSQL_PASSWORD);

        log.info("""
                \n
                🐳 MYSQL 테스트 컨테이너를 시작합니다.
                - base image: {}
                """, MYSQL_DOCKER_IMAGE);
        mySQLContainer.start();

        // redis test container init
        redisContainer = new GenericContainer<>(REDIS_DOCKER_IMAGE)
                .withExposedPorts(REDIS_PORT)
                .withReuse(true);
        log.info("""
                \n
                🐳 Redis 테스트 컨테이너를 시작합니다.
                - base image: {}
                """, REDIS_DOCKER_IMAGE);
        redisContainer.start();
    }

}

```

서비스 레이어 통합 테스트는 모두 위 클래스를 상속받도록 해서,

Spring Context 를 중복으로 띄워서 발생하는 시간 지연을 최소화하려고 합니다.

예시로 작성해둔 테스트 코드는 아래와 같아요!

```java
public class ShelterServiceTest extends ServiceIntegrationTestBase {

    @Autowired
    private ShelterService shelterService;

    @Autowired
    private ShelterRepository shelterRepository;

    @DisplayName("1000 m 이내에 위치한 주변 대피소 리스트를 조회할 수 있습니다.")
    @Test
    void getNearestShelterTest() {
        // given
        shelterRepository.save(Shelter.of(ShelterType.CIVIL_DEFENCE, "1000m 이내의 대피소 민방위대피소", 35.504, 35.5));
        shelterRepository.save(Shelter.of(ShelterType.CIVIL_DEFENCE, "1000m 보다 먼 대피소", 36.504, 36.5));

        // when
        NearbyShelterListResponse response = shelterService.getNearbyShelterList(NearbyShelterRequest.of(35.5, 35.5, "민방위"));

        // then
        Assertions.assertThat(response.getCount()).isEqualTo(1);
    }

}

```

사실 위에 작성된 간단한 테스트 코드만 보고도, 프로덕션 코드에서 개선해야 할 점을 하나 찾아낼 수 있어요.

무엇일까요?

바로, 1000m 이내라는 조건인데요..!!

위 테스트코드를 보면 짐작할 수 있겠지만 Query 레벨에서 where 절에 1000 m 이하인 레코드만 조회하도록 하고 있어요.

이보다는, 1000 이라는 값을 외부의 파라미터로부터 주입되도록 하면

조금 더 테스트가 쉬워지는 코드가 됨과 동시에 프로덕션 코드도 더 유연해지지 않을까요?

이와 같이 테스트 코드를 작성하다 보면,

미처 발견하지 못했던 프로덕션 코드 상의 문제점이 눈에 보이게 된다는 아주 좋은 장점이 있어요..!

테스트 코드를 작성할 수 있는 환경을 구성했으니,

당분간은 테스트 코드를 작성하며

프로덕션 코드의 문제점을 제대로 진단해보려고 합니다.

